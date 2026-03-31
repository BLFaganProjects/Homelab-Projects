# 📊 SIEM — Splunk Enterprise

## Overview

Splunk Enterprise runs on the NUC8 (Kali Linux Purple) and serves as the central SIEM, ingesting logs from multiple sources across the lab network. Sleep is disabled on the NUC to keep Splunk running 24/7.

## Installation

Splunk Enterprise installed at `/opt/splunk/`. Start/stop with:

```bash
sudo /opt/splunk/bin/splunk start
sudo /opt/splunk/bin/splunk stop
sudo /opt/splunk/bin/splunk status
```

Access at: `https://192.168.xxx.xx:8000`

## Log Sources

### NUC (Direct Monitoring)

| Source | Sourcetype | Description |
|--------|-----------|-------------|
| `/opt/zeek/spool/zeek` | `zeek_json` | Zeek network monitor logs |
| `/var/log/suricata/eve.json` | `suricata` | Suricata IDS alerts |
| `/var/log/suricata/fast.log` | `suricata_fast` | Suricata fast log |
| `/var/log/auth.log` | `linux_secure` | SSH and authentication events |
| `/var/log/syslog` | `syslog` | System log |
| `/var/log/nginx` | `nginx_access` | Nginx web server logs |

### Homelab Server (Via Universal Forwarder)

| Source | Sourcetype | Description |
|--------|-----------|-------------|
| `/var/log/syslog` | `syslog` | Server system log |
| `/var/log/auth.log` | `linux_secure` | Server auth events |
| `/var/log/pihole/pihole.log` | `pihole` | Pi-hole DNS queries |
| `/var/log/dpkg.log` | `linux_dpkg` | Package manager logs |

## Splunk Universal Forwarder

Installed on homelab server, forwarding to Splunk Enterprise on port 9997.

Enable receiving on Splunk Enterprise:
```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:password
```

## Dashboards

### Zeek Network Monitor

| Panel | SPL Query |
|-------|-----------|
| Top 10 Source IPs | `sourcetype=zeek_json source=*conn* \| top limit=10 id.orig_h` |
| Top 10 DNS Queries | `sourcetype=zeek_json source=*dns* \| top limit=10 query` |
| Connection Protocols Over Time | `sourcetype=zeek_json source=*conn* \| timechart count by proto` |
| Weird Activity | `sourcetype=zeek_json source=*weird* \| table ts name peer msg` |

### Homelab Server Dashboard

| Panel | SPL Query |
|-------|-----------|
| Server Events | `host=homelab \| table _time sourcetype _raw` |
| Auth Events | `host=homelab sourcetype=linux_secure \| table _time _raw` |
| Pi-hole DNS | `host=homelab sourcetype=pihole \| table _time _raw` |

## Alerts

### Port Scan Detection

Scheduled alert running every 15 minutes:

```spl
index=* sourcetype=zeek_json
| stats dc(id.resp_p) as unique_ports by id.orig_h
| where unique_ports > 100
```

Triggers when any host contacts more than 100 unique ports in a 30 minute window.

## Useful SPL Queries

```spl
# All Zeek events last 15 minutes
index=* sourcetype=zeek_json earliest=-15m

# DNS queries to a specific domain
sourcetype=zeek_json source=*dns* query="*google*"
| table ts id.orig_h query answers

# Failed SSH logins
sourcetype=linux_secure "Failed password"
| stats count by src_ip
| sort -count

# Suricata alerts by severity
sourcetype=suricata
| stats count by alert.severity alert.signature
| sort -count

# Top talkers by bytes
sourcetype=zeek_json source=*conn*
| stats sum(resp_ip_bytes) as total_bytes by id.orig_h
| sort -total_bytes

# Pi-hole blocked queries
sourcetype=pihole "gravity blocked"
| rex "gravity blocked (?<domain>\S+)"
| top limit=20 domain
```
