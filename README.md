# Azure-VM-monitoring
This project demonstrates how to monitor a Linux VM on Azure and set up custom alerts for security-related events.

## Overview

- Connected a Linux VM to a Log Analytics Workspace.
- Collected Syslog (auth, syslog, kern) and performance metrics (CPU, RAM, Disk, Network).
- Created custom KQL queries to detect failed SSH login attempts (brute force) and track successful logins.
- Configured Azure Alerts to notify via email when thresholds are exceeded.

## Architecture
[VM] --> [Azure Monitor Agent] --> [Log Analytics Workspace] --> [Alerts/Action Group]



## Setup Steps

1. **Connect VM to Log Analytics Workspace**
   - Create a workspace in the same region as your VM.
   - Enable guest-level diagnostics and Syslog collection.

2. **Verify Logs**
   - SSH into your VM and generate test logs (failed/successful logins).
   - Use the Logs blade in Azure to run KQL queries.

3. **Custom Queries**
   - **Failed SSH by IP (alert)**:
     ```kusto
     Syslog
     | where TimeGenerated > ago(5m)
     | where SyslogMessage contains "Failed password"
     | extend src = extract(@"from (\d+\.\d+\.\d+\.\d+)", 1, SyslogMessage)
     | where isnotempty(src)
     | summarize Attempts = count() by Computer, src
     | where Attempts > 5
     | project ResourceId = strcat("/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/", Computer), Computer, src, Attempts
     ```
   - **Successful SSH logins** (optional):
     ```kusto
     Syslog
     | where TimeGenerated > ago(15m)
     | where SyslogMessage contains "Accepted"
     | project TimeGenerated, Computer, SyslogMessage
     | sort by TimeGenerated desc
     ```

4. **Create Alerts**
   - Scope: Log Analytics Workspace
   - Signal: Custom log search
   - Period: 5 minutes, Frequency: 1 minute
   - Logic: Trigger when query returns results > 0
   - Actions: Send email via Action Group

5. **Test Alerts**
   - Generate failed SSH attempts to trigger alerts.
   - Verify email notifications.

## Documentation
All screenshots are stored in the `doc/` folder.

## Skills & Technologies
- Azure Monitor & Log Analytics
- Kusto Query Language (KQL)
- Linux Syslog
- Security Monitoring & Alerting

