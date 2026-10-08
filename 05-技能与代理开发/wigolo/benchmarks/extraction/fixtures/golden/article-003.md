# Container Networking Deep Dive: How Docker Bridges Work

Understanding how Docker creates isolated network namespaces and bridges them together is essential for debugging connectivity issues in containerized applications.

## Network Namespaces

Every Docker container runs in its own Linux network namespace. A network namespace provides an isolated network stack with its own:

- Network interfaces
- Routing tables
- iptables rules
- `/proc/net` entries

```bash
# List network namespaces
ip netns list

# Create a namespace manually
ip netns add my-ns

# Run a command in the namespace
ip netns exec my-ns ip addr show
# Only shows loopback (lo), no external connectivity
```

## The Docker Bridge

When Docker starts, it creates a virtual bridge called `docker0`:

```bash
$ ip addr show docker0
4: docker0: <BROADCAST,MULTICAST,UP> mtu 1500
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

Each container gets a **veth pair** (virtual Ethernet):
- One end is placed inside the container (becomes `eth0`)
- The other end is attached to the `docker0` bridge

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Container A  │     │ Container B  │     │ Container C  │
│  eth0        │     │  eth0        │     │  eth0        │
│  172.17.0.2  │     │  172.17.0.3  │     │  172.17.0.4  │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │ vethA              │ vethB              │ vethC
       │                    │                    │
═══════╪════════════════════╪════════════════════╪═══════════
                      docker0 bridge
                      172.17.0.1/16
═══════════════════════════╪═════════════════════════════════
                           │
                      Host eth0
                      192.168.1.100
                           │
                       Internet
```

## Creating Custom Networks

### Bridge Network

```bash
docker network create --driver bridge \
  --subnet 10.0.1.0/24 \
  --gateway 10.0.1.1 \
  my-network

docker run --network my-network --name web nginx
docker run --network my-network --name api node:20
```

Containers on the same custom bridge can resolve each other by name:

```bash
# From the 'api' container:
curl http://web:80  # DNS resolves 'web' to its container IP
```

### Host Network

```bash
docker run --network host nginx
# Container shares the host's network stack
# No port mapping needed, no network isolation
```

### None Network

```bash
docker run --network none alpine
# Container has only loopback, completely isolated
```

## DNS Resolution

Docker's embedded DNS server (127.0.0.11) handles name resolution for custom networks:

| Query | Resolution |
|-------|-----------|
| Container name | Container IP on shared network |
| Service name (Compose) | Round-robin across replicas |
| External domain | Forwarded to host DNS |

```bash
# Inside a container on a custom network
$ cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0

$ nslookup web
Server:    127.0.0.11
Address:   127.0.0.11#53

Name:      web
Address:   10.0.1.2
```

## Port Mapping Internals

When you run `docker run -p 8080:80 nginx`, Docker creates iptables rules:

```bash
# NAT table - DNAT rule for incoming traffic
-A DOCKER -p tcp --dport 8080 -j DNAT --to-destination 172.17.0.2:80

# Filter table - allow forwarded traffic
-A DOCKER -d 172.17.0.2/32 -p tcp --dport 80 -j ACCEPT

# NAT table - masquerade outgoing traffic
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```

### Port Mapping Options

| Syntax | Meaning |
|--------|---------|
| `-p 8080:80` | Map host 8080 to container 80 (all interfaces) |
| `-p 127.0.0.1:8080:80` | Map only on localhost |
| `-p 8080:80/udp` | Map UDP port |
| `-p 8080-8090:80-90` | Map port range |
| `-P` | Map all exposed ports to random host ports |

## Debugging Network Issues

### Common Problems and Solutions

```bash
# Check container's network config
docker inspect --format='{{json .NetworkSettings}}' container_name | jq .

# Check connectivity between containers
docker exec container_a ping container_b

# Check DNS resolution
docker exec container_a nslookup service_name

# Check iptables rules
sudo iptables -t nat -L DOCKER -n -v

# Watch traffic on the bridge
sudo tcpdump -i docker0 -n port 80

# Check which network a container is on
docker network inspect my-network
```

### Troubleshooting Checklist

1. Are both containers on the same network?
2. Is DNS resolving correctly? (`nslookup`)
3. Is the target port open? (`nc -zv host port`)
4. Are iptables rules correct? (`iptables -L`)
5. Is the application listening on 0.0.0.0, not 127.0.0.1?
6. Are there firewall rules blocking traffic?

## Performance Characteristics

| Network Mode | Throughput | Latency | Isolation |
|-------------|-----------|---------|-----------|
| Host | ~Native | ~Native | None |
| Bridge (default) | ~95% native | +50us | Full |
| Bridge (custom) | ~95% native | +50us | Full + DNS |
| Overlay (Swarm) | ~80% native | +200us | Cross-host |
| Macvlan | ~98% native | +10us | L2 isolation |

## Best Practices

- Use custom bridge networks instead of the default `docker0`
- Never use `--network host` in production unless absolutely necessary
- Use Docker Compose networks for multi-container applications
- Limit published ports to specific interfaces (`127.0.0.1:port:port`)
- Use network aliases for service discovery
- Monitor bridge MTU settings when running in cloud environments

## References

- [Docker Networking Overview](https://docs.docker.com/network/)
- [Linux Network Namespaces](https://man7.org/linux/man-pages/man7/network_namespaces.7.html)
- [Container Networking From Scratch (talk)](https://www.youtube.com/watch?v=6v_BDHIgOY8)
