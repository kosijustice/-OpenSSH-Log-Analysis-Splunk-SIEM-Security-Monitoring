# -OpenSSH-Log-Analysis-Splunk-SIEM-Security-Monitoring
A real-world SOC analyst project covering log ingestion, threat detection, dashboard creation, automated alerting, and field extraction using Splunk Enterprise.

<div align="center">

![Splunk](https://img.shields.io/badge/Splunk-9.1.6-black?style=for-the-badge&logo=splunk&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-2025.1c-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![SIEM](https://img.shields.io/badge/SIEM-Threat_Detection-red?style=for-the-badge&logo=security&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

**A real-world SOC analyst project covering log ingestion, threat detection, dashboard creation, automated alerting, and field extraction using Splunk Enterprise.**

[📋 View Report](#-executive-summary) • [🔍 Findings](#-security-findings) • [📊 Dashboard](#-dashboard--alert-configuration) • [🛠️ Setup](#%EF%B8%8F-environment-setup) • [📁 Queries](#-spl-query-reference)

</div>

---

## 📌 Table of Contents

- [Executive Summary](#-executive-summary)
- [Project Objectives](#-project-objectives)
- [Environment Setup](#️-environment-setup)
- [Task 1 — User Management](#task-1--user-management)
- [Task 2 — Log Upload & Analysis](#task-2--log-upload--analysis)
- [Task 3 — Dashboard & Alert Configuration](#task-3--dashboard--alert-configuration)
- [Task 4 — Field Extraction](#task-4--field-extraction)
- [Security Findings](#-security-findings)
- [SPL Query Reference](#-spl-query-reference)
- [Recommendations](#-recommendations)
- [Key Takeaways](#-key-takeaways)
- [Tools & Technologies](#️-tools--technologies)
- [Author](#-author)

---

## 📋 Executive Summary

As a newly joined Security Operations analyst, I was tasked with analyzing OpenSSH log files to identify potential threats following a series of suspicious incidents at the organization. This project demonstrates a complete, end-to-end SOC workflow:

| Component | Detail |
|-----------|--------|
| **Platform** | Splunk Enterprise 9.1.6 on Kali Linux 2025.1c |
| **Target Log** | OpenSSH syslog — `openssh_logs.log` |
| **Host Analyzed** | `10.0.2.15` (Linux Kernel 6.16.8+kali-amd64) |
| **Total Events Indexed** | 18 events |
| **Threats Identified** | Brute force, credential stuffing, confirmed root compromise |
| **Dashboards Created** | 1 live monitoring dashboard |
| **Alerts Configured** | 1 scheduled daily alert (8AM cron) |
| **Field Extractions** | `src_ip` — regex-based inline extraction |
| **Users Managed** | 4 accounts with WAT timezone + role-based access |

> ⚠️ **Critical Finding**: A successful `Accepted password for root` from external IP `112.95.230.3` was confirmed — indicating a potential system compromise requiring immediate escalation.

---

## 🎯 Project Objectives

- [x] **User Management** — Create user accounts with correct roles, timezone (WAT), and forced password change policy
- [x] **Log Ingestion** — Upload and index OpenSSH log file into Splunk `main` index
- [x] **Threat Analysis** — Identify anomalies, suspicious IPs, and attack patterns
- [x] **Dashboard Creation** — Build a live monitoring panel for disconnect events
- [x] **Alert Configuration** — Schedule automated email alerts with threshold triggers
- [x] **Field Extraction** — Configure `src_ip` regex extraction for IP-based analytics
- [x] **Reporting** — Produce executive-ready documentation with evidence

---

## 🛠️ Environment Setup

### System Specifications

```
OS:          Kali Linux 2025.1c (VirtualBox amd64)
SIEM:        Splunk Enterprise 9.1.6
Host IP:     10.0.2.15
MAC:         08:00:27:B4:A1:05
Kernel:      Linux 6.16.8+kali-amd64
Splunk URL:  http://localhost:8000
Index:       main
Sourcetype:  syslog
```

### Splunk Installation

```bash
# Download Splunk Enterprise 9.1.6
wget -O splunk.deb "https://download.splunk.com/products/splunk/releases/9.1.6/linux/splunk-9.1.6-a28f08fac354-linux-2.6-amd64.deb"

# Install the package
sudo dpkg -i splunk.deb

# Fix libcrypto compatibility (required on newer kernels)
sudo apt install execstack -y
sudo find /opt/splunk -name "*.so*" -exec execstack -c {} \; 2>/dev/null

# Start Splunk and accept license
sudo /opt/splunk/bin/splunk start --accept-license

# Enable on boot
sudo /opt/splunk/bin/splunk enable boot-start

# Verify running
sudo /opt/splunk/bin/splunk status
```

> 💡 Access Splunk at `http://localhost:8000` after startup.

---

## Task 1 — User Management

### User Creation Steps

Each user was created via **Settings → Users → New User** with the following configuration:

| Field | Value Applied |
|-------|---------------|
| **Time Zone** | `Africa/Lagos` (WAT — West Africa Time, UTC+1) |
| **Default App** | `launcher (home)` |
| **Password Policy** | Force change on first login ✓ |
| **Roles** | Assigned by responsibility (see below) |

### Role Assignment

| Username | Role | Permissions |
|----------|------|-------------|
| `justice.alucho` | `admin` | Full system access — user management, index creation, configuration |
| `member2` | `power` | Search, dashboards, alerts — no system settings |
| `member3` | `power` | Search, dashboards, alerts — no system settings |
| `member4` | `user` | Read-only — view dashboards and basic search |

> **Why role-based access?** Principle of least privilege — each user only has the permissions needed for their responsibilities, reducing insider threat risk.

---

## Task 2 — Log Upload & Analysis

### Uploading the Log File

```
Settings → Add Data → Upload → Select File
Source Type: syslog
Index: main
```

### Verify Data Ingested

```splunk
index=main sourcetype=syslog
```

Expected result: **18 events** returned across all time.

---

### 📸 Evidence Screenshots

#### Figure 1 — All 18 Indexed Events

> *18 OpenSSH events successfully indexed in the `main` index with `sourcetype=syslog`. The timeline shows two event clusters corresponding to the two attack waves.*

---

#### Figure 2 — Failed Password & Invalid User Events

> *Search results showing failed password attempts and invalid user attacks from `183.62.140.253`, spanning multiple ports and targeted at the `root` account. Results span 2 pages.*

---

#### Figure 3 — ⚠️ CRITICAL: Accepted Password (Root Compromise)

> *3 events confirmed showing `Accepted password for root from 112.95.230.3 port 12345 ssh2` — a successful external root login indicating a confirmed compromise event.*

---

### Attack Pattern Analysis

#### Log Event Breakdown

```
Dec 10 06:55:48  Invalid user test from 183.62.140.253
Dec 10 06:55:50  Failed password for invalid user test from 183.62.140.253 port 4269
Dec 10 06:55:52  Received disconnect from 183.62.140.253: 11: Bye Bye [preauth]
Dec 10 07:02:47  Invalid user admin from 112.95.230.3
Dec 10 07:02:49  Failed password for invalid user admin from 112.95.230.3 port 4270
Dec 10 07:02:51  Received disconnect from 112.95.230.3: 11: Bye Bye [preauth]
Dec 10 07:10:00  Accepted password for root from 112.95.230.3 port 12345 ssh2  ⚠️ CRITICAL
Dec 10 07:15:00  Failed password for root from 183.62.140.253 port 54321
Dec 10 07:20:00  Failed password for root from 183.62.140.253 port 54322
Dec 10 07:25:00  Failed password for root from 183.62.140.253 port 54323
```

#### Threat Attribution

| IP Address | Role | Events | Attack Type |
|------------|------|--------|-------------|
| `183.62.140.253` | Primary Attacker | 5+ | Brute force — invalid users + failed root passwords |
| `112.95.230.3` | Secondary Attacker | 6+ | Credential stuffing → **successful root login** |

---

## Task 3 — Dashboard & Alert Configuration

### 3.1 Dashboard Creation

**Dashboard Name:** `OpenSSH Security Monitoring`
**Description:** Monitors suspicious SSH login attempts and disconnects

#### Search Query Used

```splunk
index=main "Received disconnect from 112.95.230.3: 11: Bye Bye [preauth]"
```

**Result:** 6 disconnect events confirmed from `112.95.230.3`.

#### Dashboard Panel Configuration

| Setting | Value |
|---------|-------|
| Panel Title | `Disconnect Events from 112.95.230.3` |
| Visualization | Events Table (Time + Event) |
| Permissions | Shared in App |
| App | Search & Reporting |

---

### 📸 Figure 4 — Received Disconnect Search Results

> *6 confirmed `Received disconnect from 112.95.230.3: 11: Bye Bye [preauth]` events — each representing a failed pre-authentication attempt. This is the search that powers the dashboard panel.*

---

### 📸 Figure 5 — Live Dashboard

> *The OpenSSH Security Monitoring dashboard displaying live disconnect events with timestamps, host info (`host=LabSZ`), and source file (`source=openssh_logs.log`). Panel confirms real-time data binding.*

---

### 3.2 Alert Configuration

#### Alert Settings

```
Alert Name:     SSH Disconnect Alert — 112.95.230.3
Search Query:   index=main "Received disconnect from 112.95.230.3: 11: Bye Bye [preauth]"
Schedule:       Cron — 0 8 * * *  (Daily at 8:00 AM)
Time Window:    Last 24 hours
Expires:        24 hours after trigger
```

#### Trigger Condition

```
Trigger when:  Number of results
Condition:     is greater than
Value:         2
```

#### Email Notification

```
Format:       Plain Text
Recipients:   justice.alucho@domain.com, member2@domain.com,
              member3@domain.com, member4@domain.com
Subject:      ALERT: Suspicious SSH Activity Detected — 112.95.230.3
Include:      Search results attached
```

#### Email Domain Restriction

```bash
# File: /opt/splunk/etc/system/local/alert_actions.conf
[email]
mailserver = localhost
use_ssl = false
use_tls = false
allowedDomainList = yourdomain.com
```

```bash
# Apply changes
sudo /opt/splunk/bin/splunk restart
```

---

## Task 4 — Field Extraction

### Creating the `src_ip` Field

**Navigation:** `Settings → Fields → Field Extractions → Add New`

| Setting | Value |
|---------|-------|
| Name | `src_ip` |
| Destination App | `search` |
| Apply To | `sourcetype = syslog` |
| Type | `Inline (Regular Expression)` |
| Regex Pattern | `from (?P<src_ip>\d+\.\d+\.\d+\.\d+)` |

### How the Regex Works

```
Raw log line:
Dec 10 06:55:52 LabSZ sshd[24200]: Received disconnect from 183.62.140.253: 11: Bye Bye [preauth]
                                                             ↑
                                        regex captures this IP address
Extracted field:
src_ip = 183.62.140.253
```

### Verification Query

```splunk
index=main | rex "from (?P<src_ip>\d+\.\d+\.\d+\.\d+)" | stats count by src_ip | sort -count
```

**Expected Output:**

| src_ip | count |
|--------|-------|
| 183.62.140.253 | 5 |
| 112.95.230.3 | 2 |

---

## 🚨 Security Findings

### Threat Summary Table

| # | Severity | Threat Type | Source IP | Event Count | Status |
|---|----------|-------------|-----------|-------------|--------|
| 1 | 🔴 **CRITICAL** | Confirmed Root Login | `112.95.230.3` | 1 | **Escalate Immediately** |
| 2 | 🟠 **HIGH** | Brute Force — Failed Passwords | `183.62.140.253` | 5+ | Monitoring Active |
| 3 | 🟠 **HIGH** | Credential Stuffing — Invalid Users | `183.62.140.253` | 3+ | Monitoring Active |
| 4 | 🟡 **MEDIUM** | Repeated Preauth Disconnects | `112.95.230.3` | 6 | Dashboard Panel Live |

---

### 📸 Figure 6 — Attack Timeline (Timechart)

> *Timechart visualization showing two distinct attack waves from `183.62.140.253`. The chart maps failed password and invalid user events over time, confirming sustained automated brute-force activity across different port numbers.*

---

### Attack Flow Diagram

```
ATTACKER 1: 183.62.140.253
─────────────────────────────────────────────────────────
06:55 AM  → Invalid user "test"          FAILED ✗
06:55 AM  → Failed password              FAILED ✗
06:55 AM  → Disconnect (preauth)         SESSION DROPPED
07:15 AM  → Failed password for root     FAILED ✗
07:20 AM  → Failed password for root     FAILED ✗
07:25 AM  → Failed password for root     FAILED ✗
──────────────────────────────────────
VERDICT: Brute force attack — blocked

ATTACKER 2: 112.95.230.3
─────────────────────────────────────────────────────────
07:02 AM  → Invalid user "admin"         FAILED ✗
07:02 AM  → Failed password              FAILED ✗
07:02 AM  → Disconnect x3 (preauth)      SESSION DROPPED
07:10 AM  → Accepted password for ROOT   SUCCESS ✅ ⚠️ BREACH
──────────────────────────────────────
VERDICT: CONFIRMED COMPROMISE — root access granted
```

---

## 📁 SPL Query Reference

### Core Analysis Queries

```splunk
# 1. View all indexed events
index=main sourcetype=syslog

# 2. Failed password attempts by source IP
index=main "Failed password"
| rex "from (?P<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort -count

# 3. Invalid user attempts by source IP
index=main "Invalid user"
| rex "from (?P<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort -count

# 4. Successful logins — CRITICAL QUERY
index=main "Accepted password"
| table _time, host, src_ip, user

# 5. Dashboard panel query
index=main "Received disconnect from 112.95.230.3: 11: Bye Bye [preauth]"

# 6. Attack timeline by IP
index=main ("Failed password" OR "Invalid user")
| timechart count by src_ip

# 7. Verify field extraction
index=main
| rex "from (?P<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip

# 8. All suspicious events combined
index=main ("Failed password" OR "Invalid user" OR "Received disconnect" OR "Accepted password")
| table _time, host, src_ip, sourcetype
| sort _time
```

---

## 🛡️ Recommendations

### Immediate Actions (0–30 Days)

```bash
# 1. Block attacking IPs at firewall
sudo ufw deny from 183.62.140.253
sudo ufw deny from 112.95.230.3

# 2. Disable root SSH login
sudo nano /etc/ssh/sshd_config
# Change: PermitRootLogin yes → PermitRootLogin no
sudo systemctl restart sshd

# 3. Enforce SSH key-only authentication
# In /etc/ssh/sshd_config:
# PasswordAuthentication no
# PubkeyAuthentication yes

# 4. Install Fail2Ban for automated IP blocking
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Short-Term (30–90 Days)

| Action | Tool | Priority |
|--------|------|----------|
| Deploy Content Security Policy | Nginx config | High |
| Enable HSTS | `add_header Strict-Transport-Security` | High |
| Enforce TLS 1.2/1.3 only | `ssl_protocols TLSv1.2 TLSv1.3` | High |
| Add X-Frame-Options header | Nginx config | Medium |
| Harden SSH cipher suite | `sshd_config` | Medium |
| Monthly automated patching | Ansible playbook | Medium |

### Long-Term (90+ Days)

- Establish formal **Vulnerability Management Program** with SLAs:
  - Critical → 24 hours
  - High → 7 days
  - Medium → 30 days
  - Low → 90 days
- Integrate Splunk with **Wazuh or Elastic SIEM** for extended telemetry
- Schedule **weekly Nessus credentialed scans** for continuous coverage
- Conduct external **penetration test** to validate control effectiveness
- Implement **MFA on all SSH access** (hardware key + TOTP)

---

## 🔑 Key Takeaways

```
✅  Splunk successfully ingested 18 OpenSSH log events and enabled full SPL analysis
✅  Identified two distinct attacking IPs with different TTPs
✅  Confirmed a root-level compromise from 112.95.230.3 — highest severity finding
✅  Built a live dashboard monitoring disconnect events in real-time
✅  Configured daily automated alerts with email delivery and threshold triggers
✅  Implemented src_ip field extraction enabling IP-based analytics across all searches
✅  Delivered complete executive report with supporting screenshots for stakeholders
⚠️  Root login from 112.95.230.3 requires immediate forensic investigation
```

---

## 🗂️ Repository Structure

```
openssh-splunk-siem-analysis/
│
├── README.md                          ← This file (full project documentation)
│
├── logs/
│   └── openssh_logs.log               ← Sample OpenSSH log file used for analysis
│
├── screenshots/
│   ├── 01_all_events_indexed.png      ← Figure 1: 18 events in Splunk
│   ├── 02_failed_password_search.png  ← Figure 2: Failed password & invalid user results
│   ├── 03_accepted_password.png       ← Figure 3: Critical root login confirmed
│   ├── 04_disconnect_search.png       ← Figure 4: 6 disconnect events from 112.95.230.3
│   ├── 05_dashboard.png               ← Figure 5: Live OpenSSH monitoring dashboard
│   └── 06_timechart.png               ← Figure 6: Attack timeline visualization
│
├── queries/
│   └── spl_queries.txt                ← All SPL queries used in this project
│
├── config/
│   └── alert_actions.conf             ← Splunk email domain restriction config
│
└── report/
    └── Splunk_Security_Report_Final.docx  ← Full executive submission report
```

---

## 🧰 Tools & Technologies

| Tool | Version | Purpose |
|------|---------|---------|
| ![Splunk](https://img.shields.io/badge/-Splunk-black?logo=splunk) **Splunk Enterprise** | 9.1.6 | SIEM platform — log ingestion, search, alerting, dashboards |
| ![Kali](https://img.shields.io/badge/-Kali_Linux-557C94?logo=kalilinux) **Kali Linux** | 2025.1c | Host operating system |
| **OpenSSH** | System default | Log source — authentication events |
| **Nessus** | Essentials | Vulnerability scanning (referenced in analyst profile) |
| **Ansible** | 2.15+ | Patch management automation |
| **Suricata** | 8.0.4 | Network IDS — threat detection |
| **Nginx** | 1.24.x | Web server — target of web app assessment |
| **VirtualBox** | Latest | Virtualization environment |

---

## 📊 Frameworks & Compliance Alignment

| Framework | Controls Addressed |
|-----------|-------------------|
| **NIST CSF 2.0** | ID.RA (Risk Assessment), PR.AC (Access Control), DE.CM (Continuous Monitoring) |
| **CIS Controls v8** | CIS 2 (Asset Inventory), CIS 8 (Audit Logs), CIS 12 (Network), CIS 18 (App Security) |
| **OWASP Top 10 (2021)** | A02 Cryptographic Failures, A05 Security Misconfiguration |
| **ISO 27001** | A.12.4 (Logging), A.16.1 (Incident Management) |
| **MITRE ATT&CK** | T1110 (Brute Force), T1078 (Valid Accounts), T1021.004 (Remote Services: SSH) |

---

## 👤 Author

<div align="center">

**Justice C. Alucho**
*Cybersecurity SOC Analyst | Threat Detection & Incident Response*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-justice--alucho-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/justice-alucho-30aba6145)
[![GitHub](https://img.shields.io/badge/GitHub-kosijustice-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/kosijustice)
[![Email](https://img.shields.io/badge/Email-kosijustice7alucho@gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:kosijustice7alucho@gmail.com)

📍 Clemson, SC 29631 | 📞 864-765-8789

</div>

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

*⭐ If this project was helpful or demonstrates skills you're looking for, feel free to star the repository.*

**Built with hands-on SOC analyst experience | Powered by Splunk Enterprise**

</div>

See below the full Report of my prject
[Splunk-SIEM-Security-Monitoring-Report](https://github.com/kosijustice/-OpenSSH-Log-Analysis-Splunk-SIEM-Security-Monitoring/blob/main/Splunk_Security_Report_Final_main%20-%20Google%20Docs.pdf)
