\# Phase 4: Threat Detection, Alerting \& SOC Dashboarding



\## Overview

Phase 4 translates ingested host logs into actionable SOC analytics through custom Splunk Processing Language (SPL) queries and a real-time monitoring dashboard.



\---



\## Security Analytics \& SPL Queries



\### 1. SSH Brute Force Detection



'''spl

index=main host="debian12" source="/var/log/auth.log" "Failed password"

| stats count by src\_ip, user

| where count >= 5

'''



\### 2. File Integrity \& Identity Escalation



'''spl

index=main host="debian-server" sourcetype="auditd" (key="privilege\_changes" OR key="identity\_changes")

| table \_time, host, AUID, exe, res

'''



\### 3. Unauthorized Account Creation



'''spl

index=main host="debian-server" sourcetype="auditd" key="identity\_changes" (type=ADD\_USER OR "useradd") 

| table \_time, host, AUID, exe, success

'''



\## SOC Dashboard Architecture

Built a real-time dashboard featuring key panels:



1. Total Ingested Events (24h): Single-value metrics panel.

'''spl

index=main host="debian-server" 

| stats count

'''

Trigger these events by attempting to SSH into host server through fake usernames multiple times



2\. Authentication Activity Timeline: Line chart tracking authentication events over time.

'''spl

index=main host="debian12" source="/var/log/auth.log" 

| timechart count by sourcetype

'''

Trigger these events by performing successful SSH login using ed25519 key.



3\. Kernel Audit Events by Key: Categorical distribution of identity\_changes vs privilege\_changes.

'''spl

index=main host="debian-server" sourcetype="auditd" 

| stats count by key

'''

Trigger these events by switching between root and test session followed by a auditd restart.



Trigger these events by 

4\. Recent Privileged Commands: Interactive tabular view of sudo execution history.

'''spl

index=main host="debian12" source="/var/log/auth.log" "COMMAND=" 

| table \_time, user, COMMAND

'''

Trigger these events by creating or modifying a watched configuration file using sudo



