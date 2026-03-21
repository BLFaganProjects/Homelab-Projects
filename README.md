# 🏠 Cybersecurity Home Lab

A multi-node home lab environment built for hands-on cybersecurity practice, SOC analyst skill development, and self-hosted infrastructure management.

## Lab Overview

| Node | Hardware | OS | Role |
|------|----------|----|------|
| NUC8 | Intel NUC8 | Kali Linux Purple | Primary security workstation, Zeek, Suricata, Splunk SIEM |
| Homelab Server | HP Laptop | Ubuntu Server 24.04 | Docker host, Portainer, Pi-hole, WireGuard, DuckDNS |
| Workstation | Netbook | antiX Linux | WireGuard client, secondary testing node |

## Network Architecture

```
Internet
    │
Frontier NVG468MQ Router (192.168.254.254)
    │
    ├── NUC8 / Kali Linux Purple (192.168.254.39) - WiFi
    │       ├── Zeek 8.1.1 (network monitor)
    │       ├── Suricata IDS
    │       └── Splunk Enterprise (SIEM)
    │
    ├── HP Homelab Server (192.168.254.47) - Ethernet / Static IP
    │       ├── Docker + Portainer
    │       ├── Pi-hole (DNS sinkhole)
    │       ├── WireGuard VPN server
    │       └── DuckDNS updater
    │
    └── antiX Netbook (192.168.254.27) - WiFi
            └── WireGuard client
```

## Lab Components

### 🔍 Network Security Monitoring
- **Zeek 8.1.1** — Network traffic analysis, protocol logging, anomaly detection
- **Suricata** — Intrusion detection with EVE JSON log output
- **Splunk Enterprise** — SIEM log aggregation, dashboards, alerting

See [network-monitoring/README.md](network-monitoring/README.md)

### 🐳 Infrastructure & Containers
- **Docker + Portainer** — Containerized service management via web UI
- **Pi-hole** — Network-wide DNS filtering and ad blocking
- **WireGuard** — Self-hosted VPN with DuckDNS dynamic DNS
- **DuckDNS** — Automatic public IP updates for remote VPN access

See [infrastructure/README.md](infrastructure/README.md)

### 📊 SIEM & Dashboards
- **Splunk** ingesting Zeek, Suricata, auth.log, syslog, Nginx, ClamAV, APT/DPKG, OpenVPN logs
- Custom dashboards for DNS analysis, connection monitoring, and anomaly detection

See [siem/README.md](siem/README.md)

### 🔒 VPN
- **WireGuard** server accessible via `barkingturtlelabs.duckdns.org:51820`
- Peer configs for Android phone and Linux laptop
- Pi-hole DNS routed through VPN tunnel for ad blocking on all devices remotely

See [vpn/README.md](vpn/README.md)

## Skills Demonstrated

- Linux server administration (Ubuntu Server, Kali Linux Purple, antiX)
- Docker containerization and service orchestration
- Network traffic analysis and IDS/IPS deployment
- SIEM engineering — log ingestion, parsing, dashboard creation
- VPN server deployment and client configuration
- DNS filtering and network security hardening
- Static IP and DHCP configuration
- SSH-based remote administration
- Cron-based automation and service persistence

## Certifications

- ISC2 Certified in Cybersecurity (CC)
- CompTIA Security+ *(In Progress)*
- SQL & Python — Programming Hub
