
# Enterprise Linux Hardening, Kernel Auditing \& Splunk SIEM Integration

A comprehensive security project demonstrating enterprise Linux host hardening (UFW, SSH keys), low-level kernel auditing via auditd, and an agent-based Splunk Universal Forwarder pipeline for centralized SIEM detection, alerting, and real-time SOC dashboarding.


## Table of Contents

- Executive Summary

- System Architecture & Data Flow

- Tech Stack \& Security Controls

- Phase-by-Phase Breakdown

  - Phase 1: Host Hardening & Access Control

  - Phase 2: Kernel-Level Event Auditing

  - Phase 3: Centralized SIEM Log Pipeline

  - Phase 4: Threat Detection & SOC Dashboarding

- Key Threat Detections Queries (SPL)

- Key Visual Evidence

- Author & Contact


## Executive Summary

This project demonstrates an end-to-end security architecture designed to secure, monitor, and audit a Debian 12 (Bookworm) server environment. Starting from a zero-trust network baseline, the host's perimeter was hardened using Uncomplicated Firewall (UFW) to enforce default-deny incoming policies and restrict management access exclusively to port 22/tcp [SSH protocol]. Cryptographic authentication was established using ed25519 SSH key pairs, completely disabling password-based and root logins to prevent unauthorized remote entry.

To establish continuous system visibility, low-level kernel auditing was deployed using auditd. Custom watch rules were engineered to monitor identity and privilege files (/etc/passwd, /etc/shadow, and /etc/sudoers), tagging events with dedicated audit keys (identity\_changes and privilege\_changes). Authentication and session logs were restored using rsyslog to populate /var/log/auth.log.

An enterprise log pipeline was constructed by deploying the Splunk Universal Forwarder as a systemd service (SplunkForwarder.service). Target log files were configured in inputs.conf and securely forwarded over TCP port 9997 to a central Splunk Enterprise instance. Using Splunk Processing Language (SPL), threat detection queries were written to identify SSH brute-force attacks, file integrity violations, and privileged command executions (sudo). The project culminates in a 4-panel real-time Security Operations Center (SOC) dashboard that visualizes system health, authentication trends, and kernel security events.

## System Architecture & Data Flow

```text
  [ Attack Simulation / Client Access ]
      │
      ├─► Remote SSH Access (Port 2222/tcp)
      └─► Synthetic Attacks (Brute force, account creation, sudo edit)
              │
              ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                   TARGET: DEBIAN 12 HOST                    │
  │                                                             │
  │  [ Security Controls ]                                      │
  │   ├─ UFW Firewall (Default Deny Incoming | Port 2222)       │
  │   └─ Hardened OpenSSH (Ed25519 Keys Only | No Root Password)│
  │                                                             │
  │  [ Telemetry Generation ]                                   │
  │   ├─ rsyslog ───────────────► /var/log/auth.log             │
  │   └─ auditd (Kernel Engine) ─► /var/log/audit/audit.log     │
  │                                   │                         │
  │  [ Log Transport ]                │                         │
  │   └─ Splunk Universal Forwarder ◄─┘                         │
  └──────────────────────────┬──────────────────────────────────┘
                             │
                             │ Encrypted Transport (TCP / Port 9997)
                             ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                    SIEM: SPLUNK ENTERPRISE                  │
  │                                                             │
  │  [ Data Pipeline ]                                          │
  │   ├─ Receiver Port: 9997                                    │
  │   ├─ Index: main                                            │
  │   └─ Sourcetypes: syslog, auditd                            │
  │                                                             │
  │  [ SOC Analytics & Operations ]                             │
  │   ├─ Custom SPL Threat Detection Queries                    │
  │   ├─ Automated Security Alerts                              │
  │   └─ Real-Time Security Operations Dashboard                │
  └─────────────────────────────────────────────────────────────┘
```

## Tech Stack & Security Controls

**OS:** Debian 12 (Bookworm)

**Hardening:** UFW, OpenSSH (`ed25519` key-auth)

**Telemetry:** `auditd`, `rsyslog`

**SIEM:** Splunk Enterprise, Splunk Universal Forwarder

**Language:** Splunk Processing Language (SPL), Bash



## Phase-by-Phase Breakdown

### [Phase 1: Host Hardening & Zero-Trust Access Control]

The target Debian 12 server enforces strict perimeter filtering via UFW (allowing incoming traffic only on port 22/tcp for SSH') by implementing Zero Trust and limits authentication exclusively to asymmetric 'ed25519' SSH keys by disabling Password Authentication and enabling Pubkey Authentication in /etc/ssh/sshd\_config.

Refer to [01 Host Hardenining](./docs/01-host-hardening.md) for more details

### [Phase 2: Kernel-Level Event Auditing (`auditd`)]

Authentication attempts, 'sudo' execution history, and session management are handled by 'ryslog' and recorded in 'var/log/auth.log'. System call monitoring, file integrity changes ('/etc/passwd', '/etc/shadow', '/etc/sudoers'), and user modifications are hooked at the kernel level by 'auditd' and stored in '/var/log/audit/audit.log'.

Refer to [02 Kernel Auditing](./docs/02-kernel-auditing.md) for more details

### [Phase 3: Centralized SIEM Ingestion Pipeline]

An agent-based \*\*Splunk Universal Forwarder\*\* running as a 'systemd' service reads incoming logs, applies host metadata tags, and securely routes the stream over TCP port '9997'.

Refer to [03 Splunk Pipeline](./docs/03-splunk-pipeline.md) for more details

### [Phase 4: Threat Detection, Alerting & SOC Dashboarding]

The central Splunk Enterprise instance indexes the incoming events into 'sourcetype=syslog' and 'sourcetype=auditd'. Custom Splunk Processing Language (SPL) queries correlate these events in real time to populate automated alerts and drive a centralized SOC operations dashboard.

Refer to [04 SOC Dashboarding](./docs/04-soc-dashboarding.md) for more details

## Key Threat Detection Queries (SPL)

### SSH Brute-Force & Credential Stuffing Detection

Detects multiple failed password attempts originating from a single IP to identify brute-force activity

```spl

index=main host="debian-server" source="/var/log/auth.log" "Failed password"

| stats count by src\_ip, user

| where count >= 5
```

### Kernel-Level Identity & File Integrity Violation (FIM)

Detects unauthorized user creation or privilege escalation displayed as a table to identify vertical privilege escalation

```spl

index=main host="debian-server" sourcetype="auditd" (key="privilege\_changes" OR key="identity\_changes")

| table \_time, host, AUID, exe, res

```

See all of the queries in ./queries/splunk\_detections.spl

## Key Visual Evidence

### UFW Status

![](./screenshots/1_ufw_status.png)



### Splunk Forwarder Status

![](./screenshots/7_splunk_forwarder_status.png)



### SOC Dashboard

![](./screenshots/9_soc_dashboard.png)


## Author & Contact

**Author:** [Fakiha Tariq](https://github.com/FakihaTariq)

**Email:** fakihatariq1@outlook.com

**LinkedIn:** https://www.linkedin.com/in/fakiha-tariq-665aa8344/



