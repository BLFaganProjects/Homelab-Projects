# 📊 SIEM — Splunk Enterprise

## Overview

Splunk Enterprise runs on the NUC8 (Kali Linux Purple) and serves as the central SIEM, ingesting logs from multiple sources across the lab network.

## Installation

Splunk Enterprise installed at `/opt/splunk/`. Start/stop with:

```bash
sudo /opt/splunk/bin/splunk start
sudo /opt/splunk/bin/splunk stop
sudo /opt/splunk/bin/splunk status
```

Access at: `http://192.168.254.39:8000`

## Log Sources

All sources configured in `/opt/splunk/etc/apps/search/local/inputs.conf`:

| Source | Sourcetype | Description |
|--------|-----------|-------------|
| `/opt/zeek/spool/zeek` | `zeek_json` | Zeek network monitor logs |
| `/var/log/suricata/eve.json` | `suricata` | Suricata IDS alerts |
| `/var/log/suricata/fast.log` | `suricata_fast` | Suricata fast log |
| `/var/log/auth.log` | `linux_secure` | SSH and authentication events |
| `/var/log/syslog` | `syslog` | System log |
| `/var/log/nginx` | `nginx_access` | Nginx web server logs |
| `/var/log/apt` | `linux_apt` | APT package manager logs |
| `/var/log/dpkg.log` | `linux_dpkg` | DPKG package logs |
| `/var/log/clamav` | `clamav` | ClamAV antivirus logs |
| `/var/log/openvpn` | `openvpn` | OpenVPN logs |
| `/var/log/boot.log` | `linux_boot` | Boot logs |

## inputs.conf

```ini
[monitor:///opt/zeek/spool/zeek]
disabled = false
sourcetype = zeek_json

[monitor:///var/log/suricata/eve.json]
disabled = false
sourcetype = suricata

[monitor:///var/log/suricata/fast.log]
disabled = false
sourcetype = suricata_fast

[monitor:///var/log/auth.log]
disabled = false
sourcetype = linux_secure

[monitor:///var/log/syslog]
disabled = false
sourcetype = syslog

[monitor:///var/log/nginx]
disabled = false
sourcetype = nginx_access

[monitor:///var/log/apt]
disabled = false
sourcetype = linux_apt

[monitor:///var/log/dpkg.log]
disabled = false
sourcetype = linux_dpkg

[monitor:///var/log/clamav]
disabled = false
sourcetype = clamav

[monitor:///var/log/openvpn]
disabled = false
sourcetype = openvpn

[monitor:///var/log/boot.log]
disabled = false
sourcetype = linux_boot
```

## Dashboards

### Zeek Network Monitor

Custom dashboard with the following panels:

| Panel | SPL Query |
|-------|-----------|
| Top 10 Source IPs | `sourcetype=zeek_json source=*conn* \| top limit=10 id.orig_h` |
| Top 10 DNS Queries | `sourcetype=zeek_json source=*dns* \| top limit=10 query` |
| Connection Protocols Over Time | `sourcetype=zeek_json source=*conn* \| timechart count by proto` |
| Weird Activity | `sourcetype=zeek_json source=*weird* \| table ts name peer msg` |
| 404 Errors | `sourcetype=nginx_access status=404 \| table _time method uri status` |
| 200 vs 404 | `sourcetype=nginx_access \| timechart count by status` |

Dashboard XML exported to [dashboards/zeek_network_monitor.xml](dashboards/zeek_network_monitor.xml).

## Useful SPL Queries

```spl
# All Zeek events last 15 minutes
index=* sourcetype=zeek_json earliest=-15m

# DNS queries to a specific domain
sourcetype=zeek_json source=*dns* query="*google*" | table ts id.orig_h query answers

# Failed SSH logins
sourcetype=linux_secure "Failed password" | stats count by src_ip | sort -count

# Suricata alerts by severity
sourcetype=suricata | stats count by alert.severity alert.signature | sort -count

# Top talkers by bytes
sourcetype=zeek_json source=*conn* | stats sum(resp_ip_bytes) as total_bytes by id.orig_h | sort -total_bytes
```
