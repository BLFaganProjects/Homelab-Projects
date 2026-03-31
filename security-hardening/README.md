# 🛡️ Security Hardening

## Overview

Security hardening applied across both the homelab server and NUC after initial setup and penetration testing revealed the attack surface. All hardening was validated by re-running Nmap and Nikto scans post-implementation.

---

## Fail2ban

Installed on both the homelab server and NUC to automatically ban IPs that exhibit brute force behavior.

### Installation

```bash
sudo apt install -y fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### Configuration — /etc/fail2ban/jail.local

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
ignoreip = 127.0.0.1/8 192.168.xxx.0/24

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h

[nextcloud]
enabled = true
port = 8080
logpath = /var/lib/docker/volumes/nextcloud_nextcloud_data/_data/nextcloud.log
maxretry = 5
bantime = 1h
filter = nextcloud
```

### Nextcloud Filter — /etc/fail2ban/filter.d/nextcloud.conf

```ini
[Definition]
failregex = ^.*Login failed: '.*' \(Remote IP: '<HOST>'\).*$
ignoreregex =
```

### Useful Commands

```bash
# Check jail status
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo fail2ban-client status nextcloud

# Manually ban an IP
sudo fail2ban-client set sshd banip x.x.x.x

# Unban an IP
sudo fail2ban-client set sshd unbanip x.x.x.x
```

---

## UFW Firewall

Configured on the homelab server to whitelist only necessary ports and deny all other incoming traffic.

### Rules Applied

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 53          # Pi-hole DNS
sudo ufw allow 80/tcp      # Pi-hole admin
sudo ufw allow 4000/tcp    # Dashy dashboard
sudo ufw allow 8080/tcp    # Nextcloud
sudo ufw allow 9443/tcp    # Portainer
sudo ufw allow 51820/udp   # WireGuard VPN
sudo ufw enable
```

### Verify Rules

```bash
sudo ufw status verbose
```

---

## SSH Hardening

### Key-Based Authentication Setup

Generate SSH key pair on client machine:

```bash
ssh-keygen -t ed25519 -C "NUC8-homelab-key"
```

Copy public key to homelab server:

```bash
ssh-copy-id brandon@192.168.xxx.xx
```

### sshd_config Changes

Edit `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
PermitRootLogin no
MaxAuthTries 3
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

**Important:** Always verify key-based login works in a second terminal before closing existing session.

---

## Dashy Authentication

Password hash generated using SHA256:

```bash
echo -n 'yourpassword' | sha256sum | awk '{print $1}'
```

Added to `/var/lib/docker/volumes/dashy_dashy_config/_data/conf.yml`:

```yaml
appConfig:
  theme: colorful
  auth:
    users:
      - user: brandon
        hash: <SHA256-HASH>
        type: admin
```

Restart Dashy to apply:

```bash
sudo docker restart dashy
```

---

## NUC Sleep Disabled

Disabled all sleep/suspend targets to keep Splunk running 24/7:

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

---

## Splunk Port Scan Detection Alert

SPL query saved as a scheduled Splunk alert running every 15 minutes:

```spl
index=* sourcetype=zeek_json 
| stats dc(id.resp_p) as unique_ports by id.orig_h 
| where unique_ports > 100
```

**Alert settings:**
- Runs every 15 minutes over last 30 minutes
- Triggers when results > 0
- Adds to Triggered Alerts dashboard

---

## Splunk Universal Forwarder

Installed on homelab server to forward logs to Splunk Enterprise on NUC.

### Installation

```bash
wget -O splunkforwarder.deb "https://download.splunk.com/products/universalforwarder/releases/9.2.1/linux/splunkforwarder-9.2.1-78803f08aabb-linux-2.6-amd64.deb"
sudo dpkg -i splunkforwarder.deb
sudo /opt/splunkforwarder/bin/splunk start --accept-license
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

### Configuration

Point forwarder at Splunk Enterprise:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.xxx.xx:9997 -auth admin:password
```

Add log monitors:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -sourcetype syslog
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -sourcetype linux_secure
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/pihole/pihole.log -sourcetype pihole
```

Enable receiving on Splunk Enterprise (NUC):

```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:password
```
