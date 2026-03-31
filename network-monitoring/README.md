# 🔍 Network Security Monitoring

## Overview

Network traffic monitoring handled by Zeek and Suricata on the NUC8. Logs forwarded to Splunk for analysis and dashboarding.

## Zeek 8.1.1

### Installation

```bash
echo 'deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/ /' | sudo tee /etc/apt/sources.list.d/zeek.list
curl -fsSL https://download.opensuse.org/repositories/security:zeek/xUbuntu_22.04/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/zeek.gpg
sudo apt update && sudo apt install -y zeek
```

Installed at: `/opt/zeek/`

### Management

```bash
/opt/zeek/bin/zeekctl status
/opt/zeek/bin/zeekctl start
/opt/zeek/bin/zeekctl stop
/opt/zeek/bin/zeekctl deploy
```

### Auto-start & Self-healing Crontab

```cron
@reboot sleep 30 && /opt/zeek/bin/zeekctl start
*/5 * * * * /opt/zeek/bin/zeekctl cron
```

### Log Types

| Log | Description |
|-----|-------------|
| `conn.log` | All network connections |
| `dns.log` | DNS queries and responses |
| `ssl.log` | SSL/TLS handshakes |
| `weird.log` | Anomalous activity |
| `notice.log` | Policy notices |
| `x509.log` | Certificate info |
| `quic.log` | QUIC protocol traffic |

### Splunk Integration

```ini
[monitor:///opt/zeek/spool/zeek]
disabled = false
sourcetype = zeek_json
```

Fix permissions if needed:
```bash
sudo chmod -R 755 /opt/zeek/spool/zeek/
```

---

## Suricata IDS

### Installation

```bash
sudo apt install -y suricata
sudo systemctl enable suricata
sudo systemctl start suricata
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

# External connections
sourcetype=zeek_json source=*conn* local_resp=false
| top limit=20 id.resp_h

# Suricata high severity alerts
sourcetype=suricata alert.severity=1
| table _time alert.signature src_ip dest_ip

# Weird activity summary
sourcetype=zeek_json source=*weird*
| stats count by name
| sort -count
```
