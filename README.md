# NebulaScan Pentest Automation 🚀
![NebulaScan Logo](NebulaScan.png)

NebulaScan is a powerful pentest automation tool that allows you to run multiple security tools in parallel across different environments. It's designed to make your pentesting workflow more efficient and organized.

## Features

- **Parallel Execution**: Run multiple tools simultaneously
- **Environment Management**: Define multiple targets with specific configurations
- **Tool Filtering**: Include/exclude specific tools per environment
- **Automated Output**: Save results to organized files
- **Flexible Configuration**: Easy-to-use YAML configuration

## Installation 

1. Clone the repository:
   ```bash
   git clone https://github.com/PiCaroDD/NebulaScan.git
   cd NebulaScan
   ```

2. Install Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Ensure all required tools are installed and available in your PATH.

## Usage

```bash
python NebulaScan.py -c config.yml
```

## Configuration 

## The {target} Placeholder 

The `{target}` placeholder is a special syntax used in the `flags` section of tool configurations. It tells the automation where to insert the target IP/hostname in the command.

### Usage

```yaml
flags: -sV -sC -oN output.txt {target}
```

### How It Works

1. The automation replaces `{target}` with the actual target value from the environment
2. You can place `{target}` anywhere in the flags string
3. Each tool must include `{target}` in its flags

### Examples

#### Basic Usage
```yaml
flags: -sV -sC -oN output.txt {target}
```
Becomes: `nmap -sV -sC -oN output.txt 127.0.0.1`

#### Multiple Placements
```yaml
flags: -u http://{target}/ -o output.txt
```
Becomes: `httpx -u http://127.0.0.1/ -o output.txt`

#### Complex Commands
```yaml
flags: --url=http://{target}/api/v1 --output=results.json
```
Becomes: `nuclei --url=http://127.0.0.1/api/v1 --output=results.json`

### Best Practices

1. Always include `{target}` in your flags
2. Place it where the tool expects the target specification
3. Use it with protocol prefixes when needed (e.g., `http://{target}`)
4. Test your commands manually before adding them to the config

### Error Handling

If you forget to include `{target}`:
The automation will stop and show an error message to help you fix the configuration.

### YAML Structure Overview

```yaml
env:
  - name: prod networks
    type: io
    value: 127.0.0.1
    include: [PortProber, WebSleuth]
    exclude: [VulnHunter]

tools:
  - name: PortProber
    type: tool
    map: 1
    value: nmap
    flags: -sV -sC -oN nmap.txt {target}
    output: nmap.txt
```

### Environment Section (`env`)

| Field    | Required | Description                                                                 |
|----------|----------|-----------------------------------------------------------------------------|
| name     | Yes      | Descriptive name for the environment (e.g., "prod networks")               |
| type     | Yes      | Type of environment (currently only 'io' for IP/network targets supported) |
| value    | Yes      | Target specification (IP, CIDR, or hostname)                               |
| include  | No       | List of tools to run exclusively on this target                            |
| exclude  | No       | List of tools to exclude from running on this target                       |

### Tools Section (`tools`)

| Field    | Required | Description                                                                 |
|----------|----------|-----------------------------------------------------------------------------|
| name     | Yes      | Unique name for the tool (use our fun names or create your own)            |
| type     | Yes      | Always 'tool' (reserved for future expansion)                              |
| map      | Yes      | Execution group number (tools with same map value run in parallel)         |
| value    | Yes      | Actual command to execute (must be in PATH)                                |
| flags    | Yes      | Command-line flags (use {target} placeholder for target specification)     |
| output   | Yes      | Output file name (will be prefixed with target IP/hostname)                |

### Execution Order

1. Tools are executed in order of their `map` value (lowest to highest)
2. Tools with the same `map` value run in parallel
3. Each target is processed sequentially

## Example Scenarios

### Basic Scan
```yaml
env:
  - name: Basic Scan
    type: io
    value: 192.168.1.1

tools:
  - name: PortProber
    type: tool
    map: 1
    value: nmap
    flags: -sV -sC -oN nmap.txt {target}
    output: nmap.txt
```

### Comprehensive Web App Test
```yaml
env:
  - name: Web App Test
    type: io
    value: 10.0.0.1
    include: [WebSleuth, VulnHunter]

tools:
  - name: WebSleuth
    type: tool
    map: 1
    value: httpx
    flags: -silent -timeout 20 -threads 100 -output httpx.txt -u {target}
    output: httpx.txt

  - name: VulnHunter
    type: tool
    map: 2
    value: nuclei
    flags: -severity low,medium,high,critical -timeout 20 -u {target}
    output: nuclei.txt
```

## License 📄

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
