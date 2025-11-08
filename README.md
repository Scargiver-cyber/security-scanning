# Security Scanning Automation

Automated network security scanning system that performs monthly assessments of your public-facing network and generates security reports in Obsidian.

## Overview

This project uses `nmap` to scan your public IP address for open ports and potential vulnerabilities. It generates detailed markdown reports with security ratings, analysis, and recommendations, automatically saved to your Obsidian vault for tracking over time.

## Features

- **Automated Port Scanning**: Scans top 100 ports on your public IP
- **Security Rating System**: 10-point scale with clear status indicators
- **Service Detection**: Identifies services running on detected ports
- **Risk Analysis**: Categorizes ports by security risk level
- **Historical Tracking**: Month-over-month comparison capability
- **Actionable Recommendations**: Specific steps to improve security posture
- **Obsidian Integration**: Markdown reports with task lists and wikilinks

## Security Rating Scale

- **EXCELLENT (10/10)**: No open ports, all filtered
- **GOOD (8/10)**: No open ports detected
- **FAIR (6/10)**: 1-2 open ports (needs review)
- **POOR (3/10)**: 3+ open ports (immediate action required)

## Project Structure

```
security-scanning/
├── scripts/
│   └── monthly-security-scan.sh   # Main scanning script
├── logs/                           # Execution logs (created at runtime)
└── README.md
```

## Prerequisites

- **nmap**: Network scanning utility
- **curl**: For retrieving public IP address
- macOS or Linux operating system

### Install nmap (macOS)

```bash
brew install nmap
```

### Install nmap (Linux)

```bash
# Debian/Ubuntu
sudo apt-get install nmap

# RHEL/CentOS
sudo yum install nmap
```

## Installation

1. Ensure nmap is installed
2. Make script executable:
```bash
chmod +x scripts/monthly-security-scan.sh
```

3. Configure Obsidian vault path in script:
```bash
OBSIDIAN_VAULT="/path/to/your/Cyber Vault/Network Scan"
```

## Usage

### Manual Execution

```bash
./scripts/monthly-security-scan.sh
```

Expected output:
```
Getting public IP address...
Scanning IP: XXX.XXX.XXX.XXX
Running nmap scan (this may take 1-2 minutes)...

✅ Scan complete!
📄 Report saved to: [vault]/Network Scan/2025-11-08 Security Scan Report.md

Summary:
  IP Address:      XXX.XXX.XXX.XXX
  Security Rating: EXCELLENT ✅
  Open Ports:      0
  Filtered Ports:  5
```

### Automated Scheduling (macOS launchd)

Create a launch agent for monthly execution:

**File**: `~/Library/LaunchAgents/com.security-scan.monthly.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.security-scan.monthly</string>

    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/[username]/ai-terminal/projects/security-scanning/scripts/monthly-security-scan.sh</string>
    </array>

    <key>StartCalendarInterval</key>
    <dict>
        <key>Day</key>
        <integer>1</integer>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>

    <key>StandardOutPath</key>
    <string>/Users/[username]/security_scan.log</string>

    <key>StandardErrorPath</key>
    <string>/Users/[username]/security_scan_error.log</string>
</dict>
</plist>
```

Load the agent:
```bash
launchctl load ~/Library/LaunchAgents/com.security-scan.monthly.plist
```

### Automated Scheduling (Linux cron)

Add to crontab:
```bash
# Run on the 1st of each month at 9:00 AM
0 9 1 * * /path/to/security-scanning/scripts/monthly-security-scan.sh
```

## Report Format

### Report Sections

1. **Header**: Date, IP, hostname, security rating
2. **Quick Summary**: Port counts at a glance
3. **Detailed Scan Results**: Raw nmap output
4. **Security Analysis**: Port-by-port breakdown with risk levels
5. **Recommendations**: Specific actions based on findings
6. **Historical Comparison**: How to track changes over time
7. **Related Notes**: Wikilinks to relevant security documentation
8. **Next Actions**: Checklist of follow-up tasks

### Example Report Output

```markdown
# Network Security Scan Report

**Date:** November 08, 2025 at 12:00 PM
**Target IP:** 203.0.113.42
**Security Rating:** EXCELLENT ✅ (10/10)

---

## 🔍 Quick Summary

Open Ports:     0
Filtered Ports: 5
Closed Ports:   95

---

## 🛡️ Security Analysis

**Port 80 (http):** filtered - ✅ Good
- *Action:* Firewall is protecting this port

**Port 443 (https):** filtered - ✅ Good
- *Action:* Firewall is protecting this port

---

## 💡 Recommendations

✅ **Excellent security posture!** No action required.

### Monthly Maintenance:
- Review this report for any changes from previous month
- Verify no new ports have been opened
- Keep firewall enabled
- Avoid unnecessary port forwarding
```

## Scan Parameters

The script uses the following nmap options:

- `-Pn`: Skip host discovery (assume host is online)
- `-sV`: Service version detection
- `-T4`: Aggressive timing (faster scan)
- `--top-ports 100`: Scan the 100 most common ports

### Ports Scanned

The top 100 ports include common services like:
- HTTP/HTTPS (80, 443, 8080, 8443)
- SSH (22)
- FTP (21)
- SMTP/Email (25, 587, 465)
- DNS (53)
- Database ports (3306, 5432, 27017)
- RDP (3389)
- And 83 more...

## Understanding Results

### Port States

- **OPEN** 🔴: A service is actively accepting connections
  - **Risk**: Potential attack vector
  - **Action**: Verify if needed, close if unnecessary

- **FILTERED** 🟡: Firewall is blocking or filtering packets
  - **Risk**: Low (protected by firewall)
  - **Action**: This is normal for home networks with NAT/firewall

- **CLOSED** 🟢: No service is listening on this port
  - **Risk**: None
  - **Action**: Ideal state, no action needed

### Common Open Ports to Investigate

If found open, these require immediate review:

- **Port 22 (SSH)**: Remote access - ensure strong passwords/keys
- **Port 80/443 (HTTP/HTTPS)**: Web server - verify if intentional
- **Port 3389 (RDP)**: Windows Remote Desktop - high-risk if exposed
- **Port 21 (FTP)**: Insecure file transfer - consider closing
- **Port 23 (Telnet)**: Extremely insecure - close immediately
- **Port 3306 (MySQL)**: Database - should never be public
- **Port 5432 (PostgreSQL)**: Database - should never be public

## Troubleshooting

### Error: Could not determine public IP

- Check internet connection
- Verify `curl` is installed
- Try alternative IP service: `curl -s ifconfig.me`

### Permission denied errors

Ensure script is executable:
```bash
chmod +x scripts/monthly-security-scan.sh
```

### nmap: command not found

Install nmap:
```bash
brew install nmap  # macOS
```

### Scan takes too long

The script uses `-T4` (aggressive timing). If it's still slow:
- Use fewer ports: Change `--top-ports 100` to `--top-ports 20`
- Check network connectivity
- Verify your ISP isn't rate-limiting

### Reports not appearing in Obsidian

- Verify `OBSIDIAN_VAULT` path is correct
- Ensure write permissions to vault directory
- Check if vault is syncing (iCloud, Dropbox)
- Manually create "Network Scan" folder in vault

## Best Practices

1. **Run Monthly**: Track changes in your security posture over time
2. **Compare Results**: Look for new open ports or changes in service versions
3. **Document Changes**: If you intentionally open ports, document why
4. **Act on Findings**: Don't ignore POOR or FAIR ratings
5. **Follow Up**: Complete the "Next Actions" checklist in each report

## Security Considerations

- This tool scans your **public IP** as seen from the internet
- It shows what external attackers could potentially discover
- Internal network scans require different tools and permissions
- Always have authorization before scanning any network
- Your ISP may flag aggressive scanning activity

## Advanced Usage

### Scan a specific IP

```bash
# Edit script to replace PUBLIC_IP line
PUBLIC_IP="192.168.1.1"  # Instead of curl command
```

### Full port scan (slower but comprehensive)

```bash
# Replace --top-ports 100 with -p-
nmap -Pn -sV -T4 -p- $PUBLIC_IP
```

### Include OS detection

```bash
# Add -O flag (requires sudo on macOS)
sudo nmap -Pn -sV -O -T4 --top-ports 100 $PUBLIC_IP
```

## Contributing

This is a personal project, but suggestions and improvements are welcome.

## License

Personal use project. Modify as needed for your security scanning needs.

## Disclaimer

- Only scan networks you own or have explicit permission to scan
- Unauthorized port scanning may be illegal in your jurisdiction
- This tool is for defensive security monitoring only
- The author assumes no liability for misuse

## Author

Jason Tilson - Cybersecurity professional focusing on defensive security and network hardening

## Related Projects

- [cyber-news-automation](../cyber-news-automation/) - Automated security news aggregation
- [password-security-toolkit](../password-security-toolkit/) - Password security analysis tools
