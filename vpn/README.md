# 🔒 WireGuard VPN

## Overview

Self-hosted WireGuard VPN running as a Docker container, accessible remotely via DuckDNS. All VPN traffic routed through Pi-hole for ad blocking away from home.

## Architecture

```
Remote Device
    │
    │ Encrypted WireGuard tunnel
    │ your-domain.duckdns.org:51820
    │
Router (port forward 51820/UDP)
    │
Homelab Server
    │
WireGuard Container → Pi-hole DNS
```

## DuckDNS Setup

1. Create account at duckdns.org
2. Register subdomain
3. Deploy DuckDNS updater container
4. Container checks IP every 5 minutes automatically

## Deployment

See [../infrastructure/docker-compose/wireguard.yaml](../infrastructure/docker-compose/wireguard.yaml)

## Port Forwarding Required

- Protocol: UDP
- External port: 51820
- Internal port: 51820
- Internal IP: homelab server IP

## Client Setup

### View QR Codes

```bash
sudo docker exec wireguard /app/show-peer phone
sudo docker exec wireguard /app/show-peer laptop
```

### Android/iOS

1. Install WireGuard app
2. Tap + → Scan QR Code
3. Toggle tunnel on

### Linux

```bash
sudo apt install -y wireguard wireguard-tools resolvconf
sudo mkdir -p /etc/wireguard

# Get config from server
sudo docker exec wireguard cat /config/peer_laptop/peer_laptop.conf

# Paste into /etc/wireguard/wg0.conf then:
sudo wg-quick up wg0    # Connect
sudo wg-quick down wg0  # Disconnect
```

## Important Notes

- Disconnect WireGuard when on home network to avoid hairpin NAT issues
- VPN intended for untrusted networks (mobile data, public WiFi)
- All DNS over VPN routed through Pi-hole for remote ad blocking
- `--restart=always` ensures container starts after server reboots
