# Phase 3: Centralized SIEM Ingestion & Log Pipeline


## Overview
Phase 3 establishes an agent-based log forwarder pipeline using the Splunk Universal Forwarder to transmit host system and audit logs over TCP port `9997` to the central Splunk indexer.


---


## Technical Implementation

### 1. Syslog Service Restoration (`rsyslog`)
Debian 12 defaults strictly to `systemd-journald`. Installed `rsyslog` to populate continuous plain-text authentication logs at `/var/log/auth.log`.

```bash
sudo apt update && sudo apt install rsyslog -y
sudo systemctl enable --now rsyslog
```

### 2. Universal Forwarder & Inputs Configuration
Installed the Splunk Universal Forwarder package and configured /opt/splunkforwarder/etc/system/local/inputs.conf to target key log sources.

```text

[default]
host = debian-server

[monitor:///var/log/auth.log]
disabled = false
sourcetype = syslog
index = main

[monitor:///var/log/audit/audit.log]
disabled = false
sourcetype = auditd
index = main
```

### 3. Log Forwarding & Permissions
Configured world-read permissions on targeted log files and established the central receiver connection.

```bash
sudo chmod 644 /var/log/auth.log
sudo chmod o+r /var/log/audit/audit.log
sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_IP>:9997
```

## Engineering Notes & Troubleshooting
Systemd Service Naming: Splunk forwarder generates systemd integration under the service name SplunkForwarder.service (rather than splunkd). Service status verified via systemctl status SplunkForwarder.

