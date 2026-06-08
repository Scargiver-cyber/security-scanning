# Security Scanning Automation

Automated monthly network security scanning that checks your public IP for exposed ports and saves a markdown report you can review over time.

> **Authorization reminder:** Only scan networks and IP addresses you own or are explicitly authorized to test. Unauthorized port scanning may be illegal in your jurisdiction.

---

## Quick start

```bash
# 1. Clone the repo
git clone https://github.com/Scargiver-cyber/security-scanning.git
cd security-scanning

# 2. Install nmap (if not already installed)
brew install nmap          # macOS
# sudo apt-get install nmap  # Debian/Ubuntu
# sudo yum install nmap      # RHEL/CentOS

# 3. (Optional) Set a custom report destination — defaults to ~/security-reports
export OBSIDIAN_VAULT="$HOME/Documents/SecurityReports"

# 4. Make the script executable
chmod +x scripts/monthly-security-scan.sh

# 5. Run it
./scripts/monthly-security-scan.sh
```

Expected output:

```
Getting public IP address...
Scanning IP: XXX.XXX.XXX.XXX
Running nmap scan (this may take 1-2 minutes)...

✅ Scan complete!
📄 Report saved to: /home/you/security-reports/2025-11-08 Security Scan Report.md

Summary:
  IP Address:      XXX.XXX.XXX.XXX
  Security Rating: EXCELLENT ✅
  Open Ports:      0
  Filtered Ports:  5
```

---

## Make it yours

The only variable you need to change is `OBSIDIAN_VAULT`. You can set it in your shell before running:

```bash
# Before (default — saves to ~/security-reports)
./scripts/monthly-security-scan.sh

# After (save to your Obsidian vault, Dropbox, or any folder)
OBSIDIAN_VAULT="$HOME/Documents/Obsidian/Security" ./scripts/monthly-security-scan.sh
```

Or add it to your shell profile (`~/.bashrc`, `~/.zshrc`) so it's always set:

```bash
export OBSIDIAN_VAULT="$HOME/Documents/Obsidian/Security"
```

The folder is created automatically if it does not exist.

---

## Automated scheduling

### macOS (launchd)

Create a launch agent for monthly execution.

**File:** `~/Library/LaunchAgents/com.security-scan.monthly.plist`

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
        <string>/Users/YOUR_USERNAME/security-scanning/scripts/monthly-security-scan.sh</string>
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
    <string>/Users/YOUR_USERNAME/security_scan.log</string>

    <key>StandardErrorPath</key>
    <string>/Users/YOUR_USERNAME/security_scan_error.log</string>
</dict>
</plist>
```

Replace `YOUR_USERNAME` with your actual username (`whoami` to find it), then load the agent:

```bash
launchctl load ~/Library/LaunchAgents/com.security-scan.monthly.plist
```

### Linux (cron)

```bash
# Open crontab
crontab -e

# Run on the 1st of each month at 9:00 AM
0 9 1 * * /home/YOUR_USERNAME/security-scanning/scripts/monthly-security-scan.sh
```

---

## What the reports look like

Each scan produces a markdown file named `YYYY-MM-DD Security Scan Report.md` in your report folder. Sections include:

1. **Header** — date, IP, hostname, security rating
2. **Quick Summary** — open / filtered / closed port counts at a glance
3. **Detailed Scan Results** — raw nmap output
4. **Security Analysis** — port-by-port breakdown with risk levels
5. **Recommendations** — specific actions based on findings
6. **Historical Comparison** — how to track changes over time
7. **Next Actions** — checklist of follow-up tasks

### Security rating scale

| Rating | Score | Meaning |
|---|---|---|
| EXCELLENT ✅ | 10/10 | No open ports, all filtered |
| GOOD ✅ | 8/10 | No open ports detected |
| FAIR ⚠️ | 6/10 | 1–2 open ports (needs review) |
| POOR 🚨 | 3/10 | 3+ open ports (immediate action required) |

---

## Scan parameters

The script runs:

```
nmap -Pn -sV -T4 --top-ports 100 <your-public-IP>
```

| Flag | Purpose |
|---|---|
| `-Pn` | Skip host discovery (assume host is online) |
| `-sV` | Service version detection |
| `-T4` | Aggressive timing (faster scan) |
| `--top-ports 100` | Scan the 100 most common ports |

Common ports scanned include HTTP/HTTPS (80, 443), SSH (22), FTP (21), email (25, 587), databases (3306, 5432), RDP (3389), and more.

---

## Understanding the results

### Port states

- **OPEN** 🔴 — A service is actively accepting connections. This is a potential attack surface. Verify it is intentional and close it if not.
- **FILTERED** 🟡 — A firewall is blocking or filtering packets. This is normal and expected on most home networks.
- **CLOSED** 🟢 — No service is listening. The ideal state.

### Ports that need attention if open

| Port | Service | Why it matters |
|---|---|---|
| 22 | SSH | Remote access — use key-based auth, not passwords |
| 80 / 443 | HTTP/HTTPS | Web server — verify it is intentional |
| 3389 | RDP | Windows Remote Desktop — high risk if exposed |
| 21 | FTP | Insecure protocol — consider closing |
| 23 | Telnet | No encryption — close immediately |
| 3306 | MySQL | Database — should never be public |
| 5432 | PostgreSQL | Database — should never be public |

---

## Advanced usage

### Scan a specific IP instead of your public one

Edit line ~32 of the script or set the IP directly:

```bash
# Inside the script, replace the curl line with:
PUBLIC_IP="192.0.2.1"   # Use your own authorized IP
```

### Full port scan (slower but more thorough)

```bash
# Replace --top-ports 100 with -p-
nmap -Pn -sV -T4 -p- YOUR_IP
```

### Include OS detection (requires sudo on macOS)

```bash
sudo nmap -Pn -sV -O -T4 --top-ports 100 YOUR_IP
```

---

## Troubleshooting

### "Could not determine public IP"

- Check your internet connection.
- Verify `curl` is installed: `curl --version`
- Try an alternative: `curl -s ifconfig.me`

### "Permission denied"

Make the script executable:

```bash
chmod +x scripts/monthly-security-scan.sh
```

### "nmap: command not found"

Install nmap:

```bash
brew install nmap          # macOS
sudo apt-get install nmap  # Debian/Ubuntu
```

### Reports not appearing in the expected folder

- Check that `OBSIDIAN_VAULT` points to the right place: `echo $OBSIDIAN_VAULT`
- Confirm the folder exists and is writable: `ls -la "$OBSIDIAN_VAULT"`
- If using iCloud or Dropbox sync, allow a moment for the file to appear.

### Scan takes longer than expected

The `-T4` flag is already aggressive. If it still takes too long:

```bash
# Scan fewer ports
nmap -Pn -sV -T4 --top-ports 20 YOUR_IP
```

---

## Security and ethics

- This script scans your **public IP** — what an outside attacker would see from the internet.
- It does not scan your internal (LAN) network. That requires different tools and permissions.
- **Never run this against an IP you do not own or have written authorization to test.**
- Your ISP may flag aggressive scanning activity. `-T4` is moderate; `-T5` is more likely to trigger alerts.

---

## Project structure

```
security-scanning/
├── scripts/
│   └── monthly-security-scan.sh   # Main scanning script
├── logs/                           # Execution logs (created at runtime, gitignored)
├── LICENSE
└── README.md
```

---

## License

[MIT](LICENSE) — Copyright (c) 2026 Jason Tilson

---

## Author

Jason Tilson — [github.com/Scargiver-cyber](https://github.com/Scargiver-cyber)

---

## Related projects

- [cyber-news-automation](https://github.com/Scargiver-cyber/cyber-news-automation) — automated security news aggregation
- [password-security-toolkit](https://github.com/Scargiver-cyber/password-security-toolkit) — password security analysis tools
