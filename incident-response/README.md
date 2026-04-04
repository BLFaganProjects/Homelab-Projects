# Incident Response — NUC Filesystem Recovery

## Overview

A power blip caused an unclean shutdown on the Intel NUC8 running Kali Purple. On next boot the filesystem mounted read-only, SSH was dead, and the XFCE4 desktop panel was broken. This documents the full recovery process.

---

## Environment

| Host | IP | OS |
|------|----|----|
| NUC (Kali Purple) | x.x.254.39 | Kali Linux Purple |
| Homelab Server | x.x.254.47 | Ubuntu 24.04 LTS |
| Gateway | x.x.254.254 | — |

---

## Symptoms

- Filesystem mounted read-only on boot
- SSH service inactive and disabled
- All `systemctl` commands failing: `update-rc.d error: Read-only file system`
- XFCE4 panel missing on desktop login
- `.zsh_history` errors on every login

---

## Step 1 — Filesystem Repair (fsck)

Attempted live remount failed:

```bash
sudo mount -o remount,rw /
# Result: Read-only file system error
```

Rebooted into initramfs recovery. At the initramfs prompt ran manual fsck:

```bash
fsck -y /dev/sda2
```

Errors found and repaired:
- Multiple deleted inodes with data
- Free inode count wrong for multiple block groups
- Directory count wrong
- Padding at end of inode bitmap not set
- Orphan file with unclean block

Output confirmed repair:

```
/dev/sda2: FILE SYSTEM WAS MODIFIED
/dev/sda2: 603865/69173344 files, 19087068/241876736 blocks
```

Exited initramfs with `exit` to resume normal boot.

---

## Step 2 — Restore SSH

With filesystem now writable:

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

Verified from homelab server:

```bash
ssh brandon@x.x.254.39
# Connection successful
```

---

## Step 3 — Disable Sleep Permanently

Sleep was masked to prevent future unclean shutdowns:

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

---

## Step 4 — XFCE4 Desktop Recovery

Panel failed to load with critical errors:

```
Failed to execute child process "/usr/lib/x86_64-linux-gnu/xfce4/panel/wrapper-2.0"
(No such file or directory)
```

The `wrapper-2.0` binary was corrupted/missing due to filesystem damage.

**Fix — reinstall panel and profiles:**

```bash
sudo apt install --reinstall xfce4-panel xfce4-panel-profiles
```

Saved Kali panel profiles (dated 03/05/2026) were also corrupted. Full desktop reinstall was required to resolve remaining dependency conflicts:

```bash
sudo apt --fix-broken install -y
sudo apt update && sudo apt upgrade -y
```

Panel was rebuilt manually and saved as a new backup profile via Panel Preferences → Backup and Restore.

---

## Step 5 — Filesystem Check Interval (tune2fs)

Set automatic fsck trigger every 10 mounts on all machines:

```bash
# NUC
sudo tune2fs -c 10 /dev/sda2

# Homelab Server
sudo tune2fs -c 10 /dev/sda2

# antiX Laptop (eMMC drive)
sudo tune2fs -c 10 /dev/mmcblk0p2
```

---

## Step 6 — Backup Strategy

### Timeshift Snapshots

Installed and configured on all three machines with RSYNC mode.

**Schedule:** Daily (keep 5), Weekly (keep 2), Monthly (keep 1)

**NUC:**
```bash
sudo apt install timeshift
sudo timeshift-gtk
# First snapshot taken: 2026-04-04
```

**Homelab Server (CLI over SSH):**
```bash
sudo apt install timeshift
sudo timeshift --create --comments "Clean server snapshot April 2026" --rsync
# Result: RSYNC Snapshot saved successfully (405s)
```

### Splunk Forwarder Config Backup

```bash
mkdir -p ~/backups
sudo tar -czf ~/backups/splunk-forwarder-backup-$(date +%Y%m%d).tar.gz /opt/splunkforwarder/etc
```

---

## Root Cause

Power blip caused unclean shutdown while files were being written, corrupting the ext4 filesystem. The NUC has no UPS protection. The homelab server (HP laptop) was unaffected due to its internal battery.

**Recommended fix:** APC Back-UPS 600VA (~$50) for the NUC.

---

## Lessons Learned

- Always use `sudo shutdown -h now` — never hard power off a running system
- Install Timeshift at build time, not after a corruption event
- Save a panel profile backup after any desktop customization
- A UPS is essential for any always-on machine without a built-in battery
- Remote access via WireGuard + SSH allowed diagnosis without physical access to the NUC
