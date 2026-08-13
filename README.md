\# Enterprise Linux Hardening, Kernel Auditing \& Splunk SIEM Integration



A comprehensive security project demonstrating enterprise Linux host hardening (UFW, SSH keys), low-level kernel auditing via auditd, and an agent-based Splunk Universal Forwarder pipeline for centralized SIEM detection, alerting, and real-time SOC dashboarding.



\## Table of Contents

\- \[Executive Summary](#-executive-summary)

\- \[System Architecture \& Data Flow](#-system-architecture--data-flow)

\- \[Tech Stack \& Security Controls](#-tech-stack--security-controls)

\- \[Phase-by-Phase Breakdown](#-phase-by-phase-breakdown)

&#x20; - \[Phase 1: Host Hardening \& Access Control](./docs/01-host-hardening.md)

&#x20; - \[Phase 2: Kernel-Level Event Auditing](./docs/02-kernel-auditing.md)

&#x20; - \[Phase 3: Centralized SIEM Log Pipeline](./docs/03-splunk-pipeline.md)

&#x20; - \[Phase 4: Threat Detection \& SOC Dashboarding](./docs/04-soc-dashboarding.md)

\- \[Key Threat Detections Queries (SPL)](#-featured-threat-detections-spl)

\- \[Key Visual Evidence](#-visual-evidence)

\- \[Author \& Contact](#-author--contact)



\## Executive Summary

This project demonstrates an end-to-end security architecture designed to secure, monitor, and audit a Debian 12 (Bookworm) server environment. Starting from a zero-trust network baseline, the host's perimeter was hardened using Uncomplicated Firewall (UFW) to enforce default-deny incoming policies and restrict management access exclusively to port 22/tcp [SSH protocol]. Cryptographic authentication was established using ed25519 SSH key pairs, completely disabling password-based and root logins to prevent unauthorized remote entry.



To establish continuous system visibility, low-level kernel auditing was deployed using auditd. Custom watch rules were engineered to monitor identity and privilege files (/etc/passwd, /etc/shadow, and /etc/sudoers), tagging events with dedicated audit keys (identity\_changes and privilege\_changes). Authentication and session logs were restored using rsyslog to populate /var/log/auth.log.



An enterprise log pipeline was constructed by deploying the Splunk Universal Forwarder as a systemd service (SplunkForwarder.service). Target log files were configured in inputs.conf and securely forwarded over TCP port 9997 to a central Splunk Enterprise instance. Using Splunk Processing Language (SPL), threat detection queries were written to identify SSH brute-force attacks, file integrity violations, and privileged command executions (sudo). The project culminates in a 4-panel real-time Security Operations Center (SOC) dashboard that visualizes system health, authentication trends, and kernel security events.



\## System Architecture \& Data Flow

\[ Attack Simulation / Client Access ]

&#x20;     │

&#x20;     ├─► Remote SSH Access 

&#x20;     └─► Synthetic Attacks (Brute force, account creation, sudo edit)

&#x20;             │

&#x20;             ▼

&#x20; ┌──────────────────────────────────────────────────────

&#x20;                    TARGET: DEBIAN 12 HOST                                        

&#x20;                                                              

&#x20;   \[ Security Controls ]                                      

&#x20;    ├─ UFW Firewall (Default Deny Incoming | Port 22)       

&#x20;    └─ Hardened OpenSSH (ed25519 Keys Only | No Password Authentication)

&#x20;                                                              

&#x20;   \[ Telemetry Generation ]                                   

&#x20;    ├─ rsyslog ───────────────► /var/log/auth.log       

&#x20;    └─ auditd (Kernel Engine) ─► /var/log/audit/audit.log 	 

&#x20;                                                            

&#x20;   \[ Log Transport ]                                         

&#x20;    └─ Splunk Universal Forwarder ◄─┘                       

&#x20; └──────────────────────────┬────────────────────────────

&#x20;                            │

&#x20;                            │ Encrypted Transport (TCP / Port 9997)

&#x20;                            ▼

&#x20; ┌───────────────────────────────────────────────────────

&#x20;                     SIEM: SPLUNK ENTERPRISE                  

&#x20;                                                              

&#x20;   \[ Data Pipeline ]                                          

&#x20;    ├─ Receiver Port: 9997                                   

&#x20;    ├─ Index: main                                           

&#x20;    └─ Sourcetypes: syslog, auditd                           

&#x20;                                                              

&#x20;   \[ SOC Analytics \& Operations ]                             

&#x20;    ├─ Custom SPL Threat Detection Queries                    

&#x20;    ├─ Automated Security Alerts                              

&#x20;    └─ Real-Time Security Operations Dashboard                

&#x20; └───────────────────────────────────────────────────────



\## Tech Stack \& Security Controls

\*\*OS:\*\* Debian 12 (Bookworm)

\*\*Hardening:\*\* UFW, OpenSSH (`ed25519` key-auth)

\*\*Telemetry:\*\* `auditd`, `rsyslog`

\*\*SIEM:\*\* Splunk Enterprise, Splunk Universal Forwarder

\*\*Language:\*\* Splunk Processing Language (SPL), Bash



\## Phase-by-Phase Breakdown

\### \[Phase 1: Host Hardening \& Zero-Trust Access Control]

The target Debian 12 server enforces strict perimeter filtering via UFW (allowing incoming traffic only on port 22/tcp for SSH') by implementing Zero Trust and limits authentication exclusively to asymmetric 'ed25519' SSH keys by disabling Password Authentication and enabling Pubkey Authentication in /etc/ssh/sshd\_config.



Refer to ./docs/01-host-hardening.md for more details



\### \[Phase 2: Kernel-Level Event Auditing (`auditd`)]

Authentication attempts, 'sudo' execution history, and session management are handled by 'ryslog' and recorded in 'var/log/auth.log'. System call monitoring, file integrity changes ('/etc/passwd', '/etc/shadow', '/etc/sudoers'), and user modifications are hooked at the kernel level by 'auditd' and stored in '/var/log/audit/audit.log'.



Refer to ./docs/02-kernel-auditing.md for more details



\### \[Phase 3: Centralized SIEM Ingestion Pipeline]

An agent-based \*\*Splunk Universal Forwarder\*\* running as a 'systemd' service reads incoming logs, applies host metadata tags, and securely routes the stream over TCP port '9997'.



Refer to ./docs/03-splunk-pipeline.md for more details



\### \[Phase 4: Threat Detection, Alerting \& SOC Dashboarding]

The central Splunk Enterprise instance indexes the incoming events into 'sourcetype=syslog' and 'sourcetype=auditd'. Custom Splunk Processing Language (SPL) queries correlate these events in real time to populate automated alerts and drive a centralized SOC operations dashboard.



Refer to ./docs/04-soc-dashboarding.md for more details



\## Key Threat Detection Queries (SPL)

\### SSH Brute-Force \& Credential Stuffing Detection

Detects multiple failed password attempts originating from a single IP to identify brute-force activity

'''spl

index=main host="debian-server" source="/var/log/auth.log" "Failed password"

| stats count by src\_ip, user

| where count >= 5
'''


\### Kernel-Level Identity \& File Integrity Violation (FIM)

Detects unauthorized user creation or privilege escalation displayed as a table to identify vertical privilege escalation

'''spl

index=main host="debian-server" sourcetype="auditd" (key="privilege\_changes" OR key="identity\_changes")

| table \_time, host, AUID, exe, res

'''

See all of the queries in ./queries/splunk\_detections.spl



\## Key Visual Evidence

\###UFW Status

!./screenshots/1\_ufw\_status.png



\###Splunk Forwarder Status

!./screenshots/7\_splunk\_forwarder\_status.png



\###SOC Dashboard

!./screenshots/8\_soc\_dashboard.png


\## Author \& Contact

\*\*Author:\*\* \[Fakiha Tariq](https://github.com/FakihaTariq)

\*\*Email:\*\* fakihatariq1@outlook.com

\*\*LinkedIn:\*\* https://www.linkedin.com/in/fakiha-tariq-665aa8344/



