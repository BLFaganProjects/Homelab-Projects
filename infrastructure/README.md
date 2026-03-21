# 🐳 Infrastructure & Containerization

## Overview

The homelab server is a repurposed HP laptop running Ubuntu Server 24.04 with a static IP address, managed headlessly via SSH. All services run as Docker containers orchestrated through Portainer.

## Hardware & OS

- **Device:** HP Laptop (repurposed)
- **OS:** Ubuntu Server 24.04 LTS
- **IP:** 192.168.254.47 (static, DHCP reservation bound to MAC)
- **Access:** SSH only, lid closed headless operation

## Static IP Configuration

Configured via netplan. See [netplan/50-cloud-init.yaml](netplan/50-cloud-init.yaml).

Key settings:
- Interface: `enp2s0`
- Static IP: `192.168.254.47/24`
- Gateway: `192.168.254.254`
- DNS: `8.8.8.8`, `8.8.4.4`

Lid switch disabled for headless operation:
```bash
# /etc/systemd/logind.conf
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

## Docker Installation

Docker Engine installed from the official Docker repository:

```bash
sudo apt update && sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Portainer

Web-based Docker management UI running on port 9443.

```bash
sudo docker volume create portainer_data
sudo docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Access at: `https://192.168.254.47:9443`

## Running Containers

| Container | Purpose | Port |
|-----------|---------|------|
| Portainer | Docker management UI | 9443 |
| Pi-hole | DNS filtering & ad blocking | 80, 53 |
| WireGuard | VPN server | 51820/UDP |
| DuckDNS | Dynamic DNS updater | — |

## Docker Compose Files

See the [docker-compose/](docker-compose/) directory for all stack definitions:

- `pihole.yaml` — Network-wide DNS sinkhole
- `wireguard.yaml` — Self-hosted VPN server
- `duckdns.yaml` — Public IP auto-updater

## Network-Wide DNS Filtering

Pi-hole deployed as network DNS server. Router configured with DHCP Option 6 to push `192.168.254.47` as the DNS server to all network clients automatically.

Router setting: **Frontier NVG468MQ → Advanced → LAN & DHCP → Custom Option: 6 = 192.168.254.47**
