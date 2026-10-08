# Guardian Tools Configuration

This directory contains wrappers for various penetration testing tools.

## Available Tools

### Network Scanning
- **Nmap**: Comprehensive port scanning and service detection
  - Installation: `apt-get install nmap` or `choco install nmap`
  - Features: Port scanning, service version detection, OS fingerprinting

- **Masscan**: Ultra-fast TCP port scanner
  - Installation: `apt-get install masscan` or build from [source](https://github.com/robertdavidgraham/masscan)
  - Features: Fast large-scale port scanning, banner grabbing, rate limiting



### Web Reconnaissance  
- **httpx**: HTTP probing and technology detection
  - Installation: `go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest`
  - Features: HTTP headers, status codes, technology fingerprinting

- **WhatWeb**: Web technology fingerprinting
  - Installation: `apt-get install whatweb` or `gem install whatweb`
  - Features: CMS detection, framework identification, plugin detection

- **Wafw00f**: Web Application Firewall detection
  - Installation: `pip install wafw00f`
  - Features: Detect WAF products, identify vendors

### Subdomain Enumeration
- **Subfinder**: Passive subdomain discovery
  - Installation: `go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest`
  - Features: Multiple sources, DNS resolution

- **Amass**: Advanced network mapping and asset discovery
  - Installation: `go install -v github.com/owasp-amass/amass/v4/...@master`
  - Features: Active/passive enumeration, ASN/CIDR discovery, relationship mapping

### Vulnerability Scanning
- **Nuclei**: Template-based vulnerability scanner
  - Installation: `go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest`
  - Features: Community templates, CVE detection, misconfigurations

- **Nikto**: Web vulnerability scanner
  - Installation: `apt-get install nikto`
  - Features: 6700+ potentially dangerous files/CGIs, outdated versions

- **SQLMap**: Automated SQL injection and database takeover
  - Installation: `pip install sqlmap` or `apt-get install sqlmap`
  - Features: SQL injection detection, database enumeration, risk levels

- **WPScan**: WordPress vulnerability scanner
  - Installation: `gem install wpscan` or download from [wpscan.com](https://wpscan.com/)
  - Features: Plugin/theme enumeration, vulnerability database, user enumeration

### SSL/TLS Testing
- **TestSSL**: SSL/TLS security testing
  - Installation: Download from https://testssl.sh/
  - Features: Protocol support, cipher suites, certificate validation, vulnerabilities

- **SSLyze**: Advanced SSL/TLS configuration analyzer
  - Installation: `pip install sslyze`
  - Features: Certificate analysis, protocol support, vulnerability detection (Heartbleed, ROBOT)

### Content Discovery
- **Gobuster**: Directory/file brute forcing
  - Installation: `go install github.com/OJ/gobuster/v3@latest`
  - Features: Fast directory enumeration, status code filtering, extensions

- **FFuf**: Fast web fuzzer
  - Installation: `go install github.com/ffuf/ffuf/v2@latest`
  - Features: Advanced fuzzing, JSON output, recursion, filtering/matching

## Tool Wrapper Architecture

Each tool wrapper inherits from `BaseTool` and implements:
- `get_command()`: Build command with parameters
- `parse_output()`: Parse tool output into structured data
- `_check_installation()`: Verify tool is available

## Adding New Tools

To add a new tool:

1. Create a new file in `tools/` directory
2. Inherit from `BaseTool`
3. Implement required methods
4. Add to `tools/__init__.py`
5. Register in `ToolAgent.available_tools`
6. Update prompt templates

Example:
```python
from tools.base_tool import BaseTool

class MyToolTool(BaseTool):
    def __init__(self, config):
        super().__init__(config)
        self.tool_name = "mytool"
    
    def get_command(self, target, **kwargs):
        return ["mytool", target]
    
    def parse_output(self, output):
        return {"findings": []}
```

## Tool Configuration

Tools can be configured in `config/guardian.yaml`:

```yaml
tools:
  nmap:
    enabled: true
    default_args: "-sV -sC"
    timing: T4
  
  whatweb:
    enabled: true
    aggression: 1
  
  nikto:
    enabled: true
    tuning: "x"  # All tests except DoS
```

## Testing Tools

Check tool availability:
```python
from core.tool_agent import ToolAgent

tool_agent = ToolAgent(config, gemini, memory)
status = tool_agent.get_available_tools()
# Returns: {"nmap": True, "httpx": False, ...}
```
