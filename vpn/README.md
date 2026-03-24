# 🔒 WireGuard VPN

## Overview

A self-hosted WireGuard VPN server running as a Docker container on the homelab server, accessible remotely via DuckDNS dynamic DNS. All VPN traffic is routed through Pi-hole for ad blocking even when away from home.

## Architecture

```
Remote Device (phone/laptop)
        │
        │ Encrypted WireGuard tunnel
        │ barkingturtlelabs.duckdns.org:51820
        │
Frontier Router (port forward 51820/UDP)
        │
HP Homelab Server (192.xxx.xxx.47)
        │
WireGuard Container
        │
Pi-hole DNS (192.xxx.xxx.47)
```

## DuckDNS Setup

Dynamic DNS provided by DuckDNS to handle non-static public IP:

1. Create account at duckdns.org
2. Register subdomain (e.g. `barkingturtlelabs`)
3. Deploy DuckDNS updater container (see [../infrastructure/docker-compose/duckdns.yaml](../infrastructure/docker-compose/duckdns.yaml))
4. Container checks and updates IP every 5 minutes automatically

## WireGuard Server Deployment

See [../infrastructure/docker-compose/wireguard.yaml](../infrastructure/docker-compose/wireguard.yaml)

Key environment variables:
- `SERVERURL` — Your DuckDNS domain
- `SERVERPORT` — 51820 (UDP)
- `PEERS` — Comma-separated list of peer names (e.g. `phone,laptop`)
- `PEERDNS` — Pi-hole IP for DNS filtering over VPN

## Port Forwarding

Required on router: Forward external UDP port 51820 to internal `192.xxx.xxx.47:51820`.

On Frontier NVG468MQ:
- **Firewall → Port Forwarding**
- Protocol: UDP
- Global Port Range: 51820 - 51820
- Local Base Port: 51820
- Device: 192.xxx.xxx.47

## Client Configuration

### Viewing QR Codes

After deployment, generate QR codes for each peer:

```bash
sudo docker exec wireguard /app/show-peer phone
sudo docker exec wireguard /app/show-peer laptop
```

### Android / iOS

1. Install WireGuard app from app store
2. Tap **+** → **Scan QR Code**
3. Scan the phone peer QR code
4. Toggle tunnel on

### Linux Client

```bash
# Install WireGuard
sudo apt install -y wireguard wireguard-tools resolvconf

# Create config directory
sudo mkdir -p /etc/wireguard

# Get config from server (run on server)
sudo docker exec wireguard cat /config/peer_laptop/peer_laptop.conf

# Paste output into /etc/wireguard/wg0.conf on client
sudo nano /etc/wireguard/wg0.conf

# Start VPN
sudo wg-quick up wg0

# Stop VPN
sudo wg-quick down wg0
```

## Notes

- WireGuard should be **disconnected when on the home network** to avoid hairpin NAT routing issues
- The VPN is intended for use on untrusted networks (mobile data, public WiFi)
- All DNS queries over the VPN tunnel are resolved by Pi-hole, providing ad blocking remotely
- The `--restart=always` flag ensures the WireGuard container starts automatically after server reboots
