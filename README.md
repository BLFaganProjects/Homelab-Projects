# 🏠 Cybersecurity Home Lab

A multi-node home lab environment built for hands-on cybersecurity practice, SOC analyst skill development, and self-hosted infrastructure management.

## Lab Overview

| Node | Hardware | OS | Role |
|------|----------|----|------|
| NUC8 | Intel NUC8 | Kali Linux Purple | Primary security workstation, Zeek, Suricata, Splunk SIEM |
| Homelab Server | HP Laptop | Ubuntu Server 24.04 | Docker host, Portainer, Pi-hole, WireGuard, DuckDNS, Nextcloud |
| Workstation | Netbook | antiX Linux | WireGuard client, secondary testing node |

## Network Architecture

```
Internet
    │
Router (192.168.xxx.xxx)
    │
    ├── NUC8 / Kali Linux Purple (192.168.xxx.xx) - WiFi
    │       ├── Zeek 8.1.1 (network monitor)
    │       ├── Suricata IDS
    │       ├── Splunk Enterprise (SIEM)
    │       └── Fail2ban
    │
    ├── HP Homelab Server (192.168.xxx.xx) - Ethernet / Static IP
    │       ├── Docker + Portainer
    │       ├── Pi-hole (DNS sinkhole)
    │       ├── WireGuard VPN server
    │       ├── DuckDNS updater
    │       ├── Nextcloud (self-hosted cloud)
    │       ├── Dashy (dashboard)
    │       ├── Splunk Universal Forwarder
    │       ├── Fail2ban
    │       └── UFW Firewall
    │
    └── antiX Netbook (192.168.xxx.xx) - WiFi
            └── WireGuard client
```

## Lab Components

### 🔍 Network Security Monitoring
- **Zeek 8.1.1** — Network traffic analysis, protocol logging, anomaly detection
- **Suricata** — Intrusion detection with EVE JSON log output
- **Splunk Enterprise** — SIEM log aggregation, dashboards, alerting
- **Splunk Universal Forwarder** — Forwarding homelab server logs to Splunk

See [network-monitoring/README.md](network-monitoring/README.md)

### 🐳 Infrastructure & Containers
- **Docker + Portainer** — Containerized service management via web UI
- **Pi-hole** — Network-wide DNS filtering and ad blocking
- **WireGuard** — Self-hosted VPN with DuckDNS dynamic DNS
- **Nextcloud** — Self-hosted cloud storage and photo backup with 2FA
- **Dashy** — Unified homelab dashboard with authentication

See [infrastructure/README.md](infrastructure/README.md)

### 📊 SIEM & Dashboards
- Splunk ingesting Zeek, Suricata, auth.log, syslog, Pi-hole, Nginx, ClamAV logs
- Custom dashboards for DNS analysis, connection monitoring, anomaly detection
- Port scan detection alert — fires when >100 unique ports contacted in 30 minutes

See [siem/README.md](siem/README.md)

### 🔒 VPN
- WireGuard server accessible via DuckDNS domain
- Peer configs for Android phone and Linux laptop
- Pi-hole DNS routed through VPN tunnel for remote ad blocking

See [vpn/README.md](vpn/README.md)

### 🛡️ Security Hardening
- Fail2ban — Brute force protection on SSH and Nextcloud
- UFW Firewall — Whitelist-only port rules
- SSH hardening — Key-only authentication, root login disabled
- Dashy authentication — SHA256 hashed credentials
- Port scan detection — Splunk alert for reconnaissance activity

See [security-hardening/README.md](security-hardening/README.md)

### 🔴 Penetration Testing
- Nmap — Host discovery, port scanning, service fingerprinting
- Nikto — Web vulnerability scanning
- Log analysis — Reviewing attack signatures in Splunk and Zeek

See [pentesting/README.md](pentesting/README.md)

## Skills Demonstrated

- Linux server administration (Ubuntu Server, Kali Linux Purple, antiX)
- Docker containerization and service orchestration
- Network traffic analysis and IDS/IPS deployment
- SIEM engineering — log ingestion, parsing, dashboard creation, alerting
- VPN server deployment and client configuration
- DNS filtering and network security hardening
- Firewall configuration (UFW)
- Intrusion prevention (Fail2ban)
- SSH hardening and key-based authentication
- Penetration testing and vulnerability scanning
- Log analysis and attack signature identification
- Cron-based automation and service persistence

## Certifications

- ISC2 Certified in Cybersecurity (CC)
- CompTIA Security+ *(In Progress)*
- SQL & Python — Programming Hub
