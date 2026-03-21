# 🔍 Network Security Monitoring

## Overview

Network traffic monitoring is handled by Zeek and Suricata running on the NUC8 (Kali Linux Purple). Logs are forwarded to Splunk for analysis and dashboarding.

## Zeek 8.1.1

### Installation

Zeek 8.1.1 installed from the OpenSUSE repository to resolve libc6 dependency issues on Kali Linux Purple:

```bash
# Add OpenSUSE repo
echo 'deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/ /' | sudo tee /etc/apt/sources.list.d/zeek.list
curl -fsSL https://download.opensuse.org/repositories/security:zeek/xUbuntu_22.04/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/zeek.gpg
sudo apt update && sudo apt install -y zeek
```

Installed at: `/opt/zeek/`

### Configuration

Key paths:
- Config: `/opt/zeek/etc/`
- Logs (current): `/opt/zeek/spool/zeek/`
- Logs (archived): `/opt/zeek/logs/`

### Management

```bash
# Check status
/opt/zeek/bin/zeekctl status

# Start
/opt/zeek/bin/zeekctl start

# Stop
/opt/zeek/bin/zeekctl stop

# Deploy config changes
/opt/zeek/bin/zeekctl deploy
```

### Auto-start & Self-healing

Crontab configured for root to auto-start on boot and self-heal on crash:

```cron
@reboot sleep 30 && /opt/zeek/bin/zeekctl start
*/5 * * * * /opt/zeek/bin/zeekctl cron
```

The 30-second delay on reboot ensures the network interface is up before Zeek starts.

### Log Types

| Log File | Description |
|----------|-------------|
| `conn.log` | All network connections |
| `dns.log` | DNS queries and responses |
| `ssl.log` | SSL/TLS handshakes |
| `http.log` | HTTP requests |
| `weird.log` | Anomalous activity |
| `notice.log` | Policy notices and alerts |
| `x509.log` | Certificate information |
| `quic.log` | QUIC protocol traffic |

### Splunk Integration

Zeek logs ingested into Splunk via monitored input:

```ini
[monitor:///opt/zeek/spool/zeek]
disabled = false
sourcetype = zeek_json
```

Permissions fix required for Splunk to read Zeek logs:
```bash
sudo chmod -R 755 /opt/zeek/spool/zeek/
sudo chmod -R 755 /opt/zeek/logs/
```

---

## Suricata IDS

### Installation

```bash
sudo apt install -y suricata
```

### Configuration

Main config: `/etc/suricata/suricata.yaml`

Key settings:
- Output: EVE JSON to `/var/log/suricata/eve.json`
- Fast log: `/var/log/suricata/fast.log`

### Management

```bash
sudo systemctl status suricata
sudo systemctl start suricata
sudo systemctl enable suricata
```

### Splunk Integration

```ini
[monitor:///var/log/suricata/eve.json]
disabled = false
sourcetype = suricata

[monitor:///var/log/suricata/fast.log]
disabled = false
sourcetype = suricata_fast
```

---

## Useful Analysis Queries

```spl
# Top DNS queries
sourcetype=zeek_json source=*dns* | top limit=20 query

# Connections to external IPs
sourcetype=zeek_json source=*conn* local_resp=false | top limit=20 id.resp_h

# SSL certificates seen
sourcetype=zeek_json source=*ssl* | table ts id.orig_h id.resp_h version cipher

# Suricata high severity alerts
sourcetype=suricata alert.severity=1 | table _time alert.signature src_ip dest_ip

# Weird activity
sourcetype=zeek_json source=*weird* | stats count by name | sort -count
```
