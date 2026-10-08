# Guardian Developer Guide: Creating Custom Tools

This guide walks you through creating custom pentesting tools for Guardian.

## Table of Contents
1. [Tool Architecture](#tool-architecture)
2. [Creating a Basic Tool](#creating-a-basic-tool)
3. [Advanced Tool Features](#advanced-tool-features)
4. [Integrating Your Tool](#integrating-your-tool)
5. [Testing Your Tool](#testing-your-tool)
6. [Best Practices](#best-practices)

---

## Tool Architecture

Guardian tools follow a consistent architecture:

```
BaseTool (Abstract)
    ├── get_command()      # Build command-line arguments
    ├── parse_output()     # Extract structured data
    ├── execute()          # Run tool (inherited, don't override)
    └── _check_installation() # Verify tool exists
```

All tools inherit from `BaseTool` which provides:
- Async execution
- Timeout handling
- Error management
- Installation checking

---

## Creating a Basic Tool

### Step 1: Create Tool File

Create a new file in `tools/` directory:

```bash
touch tools/mytool.py
```

### Step 2: Implement Tool Class

```python
"""
MyTool wrapper for [description]
"""

from typing import Dict, Any, List
from tools.base_tool import BaseTool


class MyToolTool(BaseTool):
    """MyTool [purpose] wrapper"""
    
    def __init__(self, config):
        super().__init__(config)
        self.tool_name = "mytool"  # Actual command name
    
    def get_command(self, target: str, **kwargs) -> List[str]:
        """
        Build command-line arguments
        
        Args:
            target: Target to scan (IP, domain, or URL)
            **kwargs: Additional parameters
        
        Returns:
            List of command arguments
        """
        command = [self.tool_name]
        
        # Add flags
        if kwargs.get("verbose", False):
            command.append("-v")
        
        # Add options with values
        threads = kwargs.get("threads", 10)
        command.extend(["-t", str(threads)])
        
        # Add target
        command.append(target)
        
        return command
    
    def parse_output(self, output: str) -> Dict[str, Any]:
        """
        Parse tool output into structured data
        
        Args:
            output: Raw stdout from tool
        
        Returns:
            Dictionary with structured results
        """
        results = {
            "findings": [],
            "target": "",
            "summary": {}
        }
        
        # Parse output line by line
        for line in output.split('\n'):
            line = line.strip()
            
            if not line:
                continue
            
            # Extract findings
            if line.startswith("FOUND:"):
                finding = {
                    "type": "vulnerability",
                    "description": line.split("FOUND:")[-1].strip(),
                    "severity": "medium"
                }
                results["findings"].append(finding)
        
        results["summary"]["total"] = len(results["findings"])
        
        return results
```

### Step 3: Handle Different Output Formats

#### JSON Output
```python
def parse_output(self, output: str) -> Dict[str, Any]:
    """Parse JSON output"""
    import json
    
    results = {"data": []}
    
    for line in output.strip().split('\n'):
        try:
            data = json.loads(line)
            results["data"].append(data)
        except json.JSONDecodeError:
            continue
    
    return results
```

#### XML Output
```python
def parse_output(self, output: str) -> Dict[str, Any]:
    """Parse XML output"""
    import xml.etree.ElementTree as ET
    
    results = {"items": []}
    
    try:
        root = ET.fromstring(output)
        for item in root.findall('.//item'):
            results["items"].append({
                "name": item.get("name"),
                "value": item.text
            })
    except ET.ParseError:
        pass
    
    return results
```

#### Regex Parsing
```python
def parse_output(self, output: str) -> Dict[str, Any]:
    """Parse with regex"""
    import re
    
    results = {"matches": []}
    
    # Pattern: [SEVERITY] Description
    pattern = r'\[(\w+)\]\s+(.+)'
    
    for match in re.finditer(pattern, output):
        severity = match.group(1)
        description = match.group(2)
        
        results["matches"].append({
            "severity": severity.lower(),
            "description": description
        })
    
    return results
```

---

## Advanced Tool Features

### Authentication

```python
def get_command(self, target: str, **kwargs) -> List[str]:
    command = ["mytool"]
    
    # API key authentication
    api_key = kwargs.get("api_key") or self.config.get("tools", {}).get("mytool", {}).get("api_key")
    if api_key:
        command.extend(["--api-key", api_key])
    
    # Basic auth
    username = kwargs.get("username")
    password = kwargs.get("password")
    if username and password:
        command.extend(["-u", f"{username}:{password}"])
    
    command.append(target)
    return command
```

### Rate Limiting

```python
def get_command(self, target: str, **kwargs) -> List[str]:
    command = ["mytool"]
    
    # Rate limiting
    rate_limit = kwargs.get("rate_limit", 100)
    command.extend(["--rate-limit", str(rate_limit)])
    
    # Delay between requests
    delay = kwargs.get("delay", 0)
    if delay > 0:
        command.extend(["--delay", f"{delay}ms"])
    
    command.append(target)
    return command
```

### Custom Configuration

```python
def __init__(self, config):
    super().__init__(config)
    self.tool_name = "mytool"
    
    # Load tool-specific config
    self.tool_config = config.get("tools", {}).get("mytool", {})
    self.default_threads = self.tool_config.get("threads", 10)
    self.timeout = self.tool_config.get("timeout", 300)
```

### Severity Mapping

```python
def _map_severity(self, raw_severity: str) -> str:
    """Map tool's severity to Guardian's standard levels"""
    severity_map = {
        "critical": "critical",
        "high": "high",
        "medium": "medium",
        "low": "low",
        "info": "info",
        "informational": "info",
        "warning": "medium",
        "error": "high"
    }
    return severity_map.get(raw_severity.lower(), "info")
```

---

## Integrating Your Tool

### Step 1: Export from Package

Edit `tools/__init__.py`:

```python
from .mytool import MyToolTool

__all__ = [
    # ... existing tools ...
    "MyToolTool",
]
```

### Step 2: Register in Tool Agent

Edit `core/tool_agent.py`:

```python
def __init__(self, config, gemini_client, memory):
    super().__init__("ToolSelector", config, gemini_client, memory)
    
    from tools import MyToolTool
    
    self.available_tools = {
        # ... existing tools ...
        "mytool": MyToolTool(config),
    }
```

### Step 3: Update AI Prompts

Edit `ai/prompt_templates/tool_selector.py`:

```python
Available Tools:
- mytool: [Brief description of what it does]
- nmap: Port scanning...
# ... rest of tools
```

### Step 4: Add Configuration

Edit `config/guardian.yaml`:

```yaml
tools:
  mytool:
    enabled: true
    threads: 10
    timeout: 300
    api_key: "optional_api_key_here"
```

---

## Testing Your Tool

### Unit Test Example

Create `tests/test_mytool.py`:

```python
import pytest
from tools.mytool import MyToolTool

@pytest.fixture
def config():
    return {
        "tools": {
            "mytool": {
                "enabled": True,
                "threads": 5
            }
        }
    }

def test_command_generation(config):
    tool = MyToolTool(config)
    command = tool.get_command("example.com", verbose=True)
    
    assert "mytool" in command
    assert "example.com" in command
    assert "-v" in command

def test_output_parsing(config):
    tool = MyToolTool(config)
    
    sample_output = """
    FOUND: SQL Injection vulnerability
    FOUND: XSS vulnerability
    """
    
    results = tool.parse_output(sample_output)
    
    assert len(results["findings"]) == 2
    assert results["findings"][0]["type"] == "vulnerability"

@pytest.mark.asyncio
async def test_execution(config):
    """Integration test - requires tool installed"""
    tool = MyToolTool(config)
    
    if not tool.is_available:
        pytest.skip("MyTool not installed")
    
    result = await tool.execute("scanme.nmap.org")
    
    assert result["exit_code"] == 0
    assert "parsed" in result
```

### Manual Testing

```python
# test_mytool_manual.py
import asyncio
from utils.helpers import load_config
from tools.mytool import MyToolTool

async def main():
    config = load_config("config/guardian.yaml")
    tool = MyToolTool(config)
    
    print(f"Tool available: {tool.is_available}")
    
    if tool.is_available:
        result = await tool.execute("example.com", verbose=True)
        print(f"Exit code: {result['exit_code']}")
        print(f"Findings: {result['parsed']}")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Best Practices

### 1. Error Handling

```python
def parse_output(self, output: str) -> Dict[str, Any]:
    results = {"findings": [], "errors": []}
    
    try:
        # Parsing logic
        data = json.loads(output)
        results["findings"] = data.get("results", [])
    except json.JSONDecodeError as e:
        results["errors"].append(f"JSON parse error: {e}")
        # Fallback to text parsing
        results["findings"] = self._parse_text_fallback(output)
    except Exception as e:
        results["errors"].append(f"Unexpected error: {e}")
    
    return results
```

### 2. Output Truncation

```python
def parse_output(self, output: str) -> Dict[str, Any]:
    # Truncate very long outputs
    MAX_OUTPUT = 10000
    if len(output) > MAX_OUTPUT:
        output = output[:MAX_OUTPUT] + "\n... (truncated)"
    
    return self._parse(output)
```

### 3. Progress Indicators

```python
def get_command(self, target: str, **kwargs) -> List[str]:
    command = ["mytool"]
    
    # Disable progress bars for cleaner output
    command.append("--no-progress")
    command.append("--quiet")
    
    command.append(target)
    return command
```

### 4. Platform Compatibility

```python
def __init__(self, config):
    super().__init__(config)
    
    import platform
    
    # Handle different tool names on different platforms
    if platform.system() == "Windows":
        self.tool_name = "mytool.exe"
    else:
        self.tool_name = "mytool"
```

### 5. Documentation

```python
class MyToolTool(BaseTool):
    """
    MyTool wrapper for vulnerability scanning
    
    Installation:
        Linux: apt-get install mytool
        macOS: brew install mytool
        Windows: choco install mytool
    
    Features:
        - Vulnerability detection
        - Configuration scanning
        - CVE correlation
    
    Example:
        >>> tool = MyToolTool(config)
        >>> result = await tool.execute("example.com")
        >>> print(result["parsed"]["findings"])
    """
```

---

## Complete Example: Custom SQLMap Wrapper

```python
"""
SQLMap tool wrapper for SQL injection testing
"""

from typing import Dict, Any, List
from tools.base_tool import BaseTool


class SQLMapTool(BaseTool):
    """SQLMap SQL injection testing wrapper"""
    
    def __init__(self, config):
        super().__init__(config)
        self.tool_name = "sqlmap"
    
    def get_command(self, target: str, **kwargs) -> List[str]:
        """Build sqlmap command"""
        command = ["sqlmap"]
        
        # Target URL
        command.extend(["-u", target])
        
        # Batch mode (no user interaction)
        command.append("--batch")
        
        # Risk and level
        risk = kwargs.get("risk", 1)
        level = kwargs.get("level", 1)
        command.extend(["--risk", str(risk)])
        command.extend(["--level", str(level)])
        
        # Technique
        technique = kwargs.get("technique", "BEUSTQ")
        command.extend(["--technique", technique])
        
        # Skip static parameters
        command.append("--skip-static")
        
        # Output to file for parsing
        command.extend(["--output-dir=/tmp/sqlmap"])
        
        return command
    
    def parse_output(self, output: str) -> Dict[str, Any]:
        """Parse sqlmap output"""
        results = {
            "vulnerable": False,
            "injection_points": [],
            "databases": [],
            "findings": []
        }
        
        # Check for vulnerabilities
        if "heuristic (basic)" in output.lower():
            results["vulnerable"] = True
        
        # Extract injection types
        import re
        injection_pattern = r'Type:\s+(.+)'
        for match in re.finditer(injection_pattern, output):
            injection_type = match.group(1).strip()
            results["injection_points"].append({
                "type": injection_type,
                "severity": "critical"
            })
        
        # Extract databases
        db_pattern = r'\[\*\]\s+(\w+)'
        for match in re.finditer(db_pattern, output):
            db_name = match.group(1)
            if db_name not in results["databases"]:
                results["databases"].append(db_name)
        
        # Create findings
        if results["vulnerable"]:
            for injection in results["injection_points"]:
                results["findings"].append({
                    "title": f"SQL Injection - {injection['type']}",
                    "severity": "critical",
                    "description": f"SQL injection vulnerability found using {injection['type']}",
                    "remediation": "Use prepared statements and parameterized queries"
                })
        
        return results
```

---

## Next Steps

1. Review existing tool implementations in `tools/` directory
2. Create your tool following this guide
3. Test thoroughly with various inputs
4. Submit a pull request or integrate locally
5. Update workflows to use your new tool

For workflow integration, see `WORKFLOW_GUIDE.md`.
