# Phase 2: Kernel-Level Event Auditing (`auditd`)


## Overview
Phase 2 implements low-level kernel telemetry via `auditd` to track system call executions, identity adjustments, and unauthorized access to critical administrative files.


---


## 🛠️ Tools & Technologies
* **Operating System:** Debian 12 (Bookworm)
* **Kernel Auditing Framework:** Linux Audit Subsystem (`auditd`)
* **Plugin Architecture:** `audispd-plugins`
* **Log Query & Reporting Utilities:** `ausearch`, `aureport`
* **Monitored System Files:** `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`

## Technical Implementation

### 1. Watch Rules Configuration
Engineered persistent kernel rules in `/etc/audit/rules.d/audit.rules` to capture mutations on credential repositories and privilege structures.

```text
# Identity & Credential Monitoring
-w /etc/passwd -p wa -k identity_changes
-w /etc/shadow -p wa -k identity_changes

# Privilege Escalation Monitoring
-w /etc/sudoers -p wa -k privilege_changes

```

### 2. Local Telemetry Verification

Validated telemetry collection by triggering controlled identity modifications (useradd) and querying system logs with ausearch and aureport.

```bash
# Query audit telemetry by key
sudo ausearch -k identity_changes

# Generate summary audit reports
sudo aureport -summary

#Query account modifications report (IAM)
sudo aureport -m
```

## Next Steps
Proceed to Phase 3: Centralized SIEM Log Pipeline to configure the Splunk Universal Forwarder and stream host telemetry to Splunk.
