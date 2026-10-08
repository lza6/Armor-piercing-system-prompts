# Guardian Developer Guide: Creating Custom Workflows

This guide explains how to create custom penetration testing workflows for Guardian.

## Table of Contents
1. [Workflow Architecture](#workflow-architecture)
2. [Creating a Basic Workflow](#creating-a-basic-workflow)
3. [Workflow Steps](#workflow-steps)
4. [Advanced Workflows](#advanced-workflows)
5. [Using Workflows](#using-workflows)
6. [Best Practices](#best-practices)

---

## Workflow Architecture

Guardian workflows are YAML files that define:
- **Sequential steps** for pentesting
- **Tool configurations** and parameters
- **AI analysis points**
- **Report generation**

### Workflow Structure

```yaml
name: workflow_name
description: Brief description

steps:
  - name: step_identifier
    type: tool|analysis|report
    # ... step configuration

settings:
  # Workflow settings
```

---

## Creating a Basic Workflow

### Step 1: Create Workflow File

Create a YAML file in `workflows/` directory:

```bash
touch workflows/my_workflow.yaml
```

### Step 2: Define Metadata

```yaml
# Custom Penetration Test Workflow
# Description of what this workflow does

name: custom_pentest
description: Custom security assessment workflow
```

### Step 3: Add Steps

```yaml
steps:
  - name: discovery
    type: tool
    tool: nmap
    objective: "Discover open ports and services"
    parameters:
      ports: "1-1000"
      scan_type: "-sV"
  
  - name: http_check
    type: tool
    tool: httpx
    objective: "Probe for HTTP services"
    parameters:
      tech_detect: true
  
  - name: analyze
    type: analysis
    agent: analyst
    objective: "Analyze discovered services"
```

### Step 4: Configure Settings

```yaml
settings:
  max_parallel_tools: 2
  require_confirmation: true
  save_intermediate: true
```

---

## Workflow Steps

### Tool Step

Execute a pentesting tool:

```yaml
- name: port_scan
  type: tool
  tool: nmap
  objective: "Comprehensive port scan"
  parameters:
    ports: "1-65535"
    scan_type: "-sS"
    timing: "T4"
```

**Fields:**
- `name`: Unique step identifier
- `type`: Must be "tool"
- `tool`: Tool name (must exist in Tool Agent)
- `objective`: Why we're running this tool
- `parameters`: Tool-specific arguments (passed to tool's `execute()`)

**Available Tools:**
- `nmap` - Port scanning
- `httpx` - HTTP probing
- `subfinder` - Subdomain enumeration
- `nuclei` - Vulnerability scanning
- `whatweb` - Technology fingerprinting
- `wafw00f` - WAF detection
- `nikto` - Web vulnerability scanning
- `testssl` - SSL/TLS testing
- `gobuster` - Directory brute forcing

### Analysis Step

Run AI analysis on collected data:

```yaml
- name: correlate_findings
  type: analysis
  agent: analyst
  objective: "Correlate findings from multiple tools"
```

**Fields:**
- `name`: Unique step identifier
- `type`: Must be "analysis"
- `agent`: Which agent to use (analyst, planner)
- `objective`: What to analyze

### Report Step

Generate report:

```yaml
- name: generate_report
  type: report
  agent: reporter
  format: markdown
```

**Supported Formats:**
- `markdown` - Markdown report (.md)
- `html` - HTML report with styling
- `json` - Structured JSON output

---

## Advanced Workflows

### Conditional Steps

While not natively supported, you can create multiple workflows for different scenarios:

```yaml
# workflows/web_basic.yaml - Basic web scan
name: web_basic
steps:
  - name: http_probe
    type: tool
    tool: httpx
    # ...

# workflows/web_deep.yaml - Deep web scan
name: web_deep
steps:
  - name: http_probe
    type: tool
    tool: httpx
    # ...
  - name: directory_brute
    type: tool
    tool: gobuster
    # ...
  - name: vuln_scan
    type: tool
    tool: nikto
    # ...
```

### Multi-Stage Workflow

```yaml
name: comprehensive_assessment
description: Multi-stage penetration test

steps:
  # Stage 1: Reconnaissance
  - name: subdomain_enum
    type: tool
    tool: subfinder
    objective: "Enumerate all subdomains"
    parameters:
      all_sources: true
  
  - name: tech_fingerprint
    type: tool
    tool: whatweb
    objective: "Identify technologies"
    parameters:
      aggression: 1
  
  # Stage 2: Scanning
  - name: port_scan
    type: tool
    tool: nmap
    objective: "Scan for open ports"
    parameters:
      ports: "top-1000"
  
  - name: waf_detection
    type: tool
    tool: wafw00f
    objective: "Detect WAF"
  
  # Stage 3: Vulnerability Assessment
  - name: web_vulns
    type: tool
    tool: nikto
    objective: "Scan for web vulnerabilities"
  
  - name: ssl_test
    type: tool
    tool: testssl
    objective: "Test SSL/TLS security"
    parameters:
      severity: "HIGH"
  
  - name: nuclei_scan
    type: tool
    tool: nuclei
    objective: "Scan with Nuclei templates"
    parameters:
      severity: ["critical", "high"]
  
  # Stage 4: Analysis
  - name: correlate_results
    type: analysis
    agent: analyst
    objective: "Correlate all findings"
  
  # Stage 5: Reporting
  - name: final_report
    type: report
    agent: reporter
    format: html

settings:
  max_parallel_tools: 3
  require_confirmation: false
  save_intermediate: true
```

### API/Cloud Testing Workflow

```yaml
name: api_security_test
description: API security assessment

steps:
  - name: endpoint_discovery
    type: tool
    tool: httpx
    objective: "Discover API endpoints"
    parameters:
      tech_detect: true
      status_code: true
  
  - name: directory_enum
    type: tool
    tool: gobuster
    objective: "Find hidden API endpoints"
    parameters:
      wordlist: "/path/to/api-wordlist.txt"
      extensions: "json,xml"
  
  - name: api_vulns
    type: tool
    tool: nuclei
    objective: "Scan for API vulnerabilities"
    parameters:
      severity: ["critical", "high"]
      templates_path: "~/nuclei-templates/http/vulnerabilities/apis/"
  
  - name: analyze_api
    type: analysis
    agent: analyst
    objective: "Analyze API security posture"
  
  - name: api_report
    type: report
    agent: reporter
    format: json

settings:
  max_parallel_tools: 2
  require_confirmation: true
```

### SSL/TLS Assessment Workflow

```yaml
name: ssl_tls_audit
description: Comprehensive SSL/TLS security audit

steps:
  - name: ssl_scan
    type: tool
    tool: testssl
    objective: "Deep SSL/TLS analysis"
    parameters:
      severity: "MEDIUM"
      fast: false
  
  - name: cert_analysis
    type: analysis
    agent: analyst
    objective: "Analyze certificate and cipher configurations"
  
  - name: ssl_report
    type: report
    agent: reporter
    format: markdown

settings:
  max_parallel_tools: 1
  require_confirmation: false
```

---

## Using Workflows

### Run a Workflow

```bash
# Basic usage
python -m cli.main workflow run --name my_workflow --target example.com

# With config file
python -m cli.main workflow run \
  --name comprehensive_assessment \
  --target example.com \
  --config config/guardian.yaml
```

### List Available Workflows

```bash
python -m cli.main workflow list
```

### Dry Run

Test workflow without executing:

```bash
python -m cli.main recon --domain example.com --dry-run
```

---

## Workflow Settings

### Complete Settings Reference

```yaml
settings:
  # Maximum tools to run in parallel
  max_parallel_tools: 3
  
  # Require user confirmation before each step
  require_confirmation: true
  
  # Save intermediate results after each step
  save_intermediate: true
  
  # Maximum workflow duration (minutes)
  max_duration_minutes: 120
  
  # Maximum total steps
  max_steps: 50
  
  # Enable safe mode (prevents destructive actions)
  safe_mode: true
```

---

## Best Practices

### 1. Logical Progression

Order steps from passive to active:

```yaml
steps:
  # Passive reconnaissance first
  - name: passive_enum
    type: tool
    tool: subfinder
    # ...
  
  # Active scanning second
  - name: port_scan
    type: tool
    tool: nmap
    # ...
  
  # Aggressive testing last
  - name: exploit_scan
    type: tool
    tool: nuclei
    # ...
```

### 2. Analysis Between Stages

Insert analysis steps after major stages:

```yaml
steps:
  - name: recon_step1
    type: tool
    # ...
  
  - name: recon_step2
    type: tool
    # ...
  
  # Analyze before moving to next stage
  - name: analyze_recon
    type: analysis
    agent: analyst
    objective: "Review reconnaissance findings"
  
  - name: scanning_step1
    type: tool
    # ...
```

### 3. Clear Objectives

Always specify clear objectives:

```yaml
- name: web_tech_detection
  type: tool
  tool: whatweb
  objective: "Identify web technologies, frameworks, and CMS"
  # Clear, specific objective helps AI understand context
```

### 4. Appropriate Parallelism

```yaml
# Sequential for dependent steps
settings:
  max_parallel_tools: 1  # One at a time

# Parallel for independent steps
settings:
  max_parallel_tools: 5  # Multiple simultaneous
```

### 5. Documentation

```yaml
# Comprehensive Security Assessment Workflow
# 
# Purpose: Complete penetration test covering:
#   - Network scanning
#   - Web application testing
#   - SSL/TLS analysis
#   - Vulnerability assessment
#
# Duration: ~30-60 minutes
# Requirements: nmap, httpx, nuclei, testssl
#
# Author: Security Team
# Last Updated: 2025-12-22

name: comprehensive_assessment
# ...
```

---

## Example Workflows by Use Case

### Quick Web Scan

```yaml
name: quick_web_scan
description: Fast web application security check

steps:
  - name: http_probe
    type: tool
    tool: httpx
    objective: "Quick HTTP service check"
  
  - name: waf_check
    type: tool
    tool: wafw00f
    objective: "Detect WAF"
  
  - name: basic_vulns
    type: tool
    tool: nuclei
    objective: "Scan for common vulnerabilities"
    parameters:
      severity: ["critical", "high"]
  
  - name: quick_report
    type: report
    agent: reporter
    format: markdown

settings:
  max_parallel_tools: 3
  require_confirmation: false
```

### OSINT Workflow

```yaml
name: osint_recon
description: Open Source Intelligence gathering

steps:
  - name: subdomain_discovery
    type: tool
    tool: subfinder
    objective: "Passive subdomain enumeration"
    parameters:
      all_sources: true
  
  - name: technology_detection
    type: tool
    tool: whatweb
    objective: "Identify technologies"
  
  - name: http_services
    type: tool
    tool: httpx
    objective: "Probe HTTP services"
  
  - name: osint_analysis
    type: analysis
    agent: analyst
    objective: "Correlate OSINT data"
  
  - name: osint_report
    type: report
    agent: reporter
    format: json

settings:
  max_parallel_tools: 3
  require_confirmation: false
  save_intermediate: true
```

---

## Troubleshooting

### Workflow Not Found

Ensure filename matches workflow name:
```yaml
# File: workflows/my_pentest.yaml
name: my_pentest  # Must match for discovery
```

### Tool Not Available

Check tool installation:
```bash
# Verify tool is installed
which nmap
which httpx

# Check Guardian recognizes it
python -m cli.main workflow list
```

### Step Failures

Add error handling:
```yaml
settings:
  # Continue on tool failure
  stop_on_error: false
  
  # Save state for debugging
  save_intermediate: true
```

---

## Next Steps

1. Review example workflows in `workflows/` directory
2. Create your custom workflow
3. Test with `--dry-run` first
4. Run on safe test targets
5. Iterate based on results

For tool creation, see `TOOLS_DEVELOPMENT_GUIDE.md`.
