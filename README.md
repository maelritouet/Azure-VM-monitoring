# Azure-VM-monitoring

## Overview
-This project demonstrates how to monitor a hardened Linux VM on Azure and create custom alerts for suspicious activity. The main focus was on detecting repeated failed SSH login attempts and triggering email notifications via Azure Monitor.

<img width="1536" height="1024" alt="ChatGPT Image Sep 10, 2025, 05_14_30 PM" src="https://github.com/user-attachments/assets/f0e8b087-f9dc-428d-ac05-d91649d525ab" />

## Step-by-Step Implementation
### 1. Connect VM to Log Analytics Workspace

Created a Log Analytics Workspace in the same region as the VM (uksouth).

Linked the VM to the workspace through the Insights blade in the Azure Portal.

### 2. Enable Syslog Collection

Enabled guest-level diagnostics on the VM.

Configured collection of auth and syslog logs so that login events are sent to the workspace.

### 3. Verify Logs in the Workspace

Opened the workspace → Logs.

Confirmed ingestion of syslog data with a simple query:

Syslog
| where TimeGenerated > ago(1h)
| sort by TimeGenerated desc
| take 20

### 4. Create a Custom KQL Query for Failed SSH Attempts

Built a query to detect multiple failed SSH logins from the same IP within 5 minutes:

```
Syslog
| where TimeGenerated > ago(5m)
| where SyslogMessage contains "Failed password"
| extend src = extract(@"(\d{1,3}(\.\d{1,3}){3})", 1, SyslogMessage)
| where isnotempty(src)
| summarize Attempts = count() by Computer, src
| where Attempts > 5
| project ResourceId = strcat("/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/", Computer), Computer, src, Attempts
```

Added a ResourceId projection to allow the alert to tie results directly to the VM.

### 5. Configure an Alert Rule

Scope: Log Analytics Workspace.

Condition: The failed SSH query above.

Evaluation: 5-minute window, evaluated every 1 minute.

Action: Created an Action Group to send email notifications.

### 6. Test the Alert

Generated syslog events simulating repeated failed SSH logins.

Verified that:

The query detected the events.

The alert fired as expected.

Email notifications were received.

## Results

VM activity is now continuously monitored through Azure Monitor.

Suspicious SSH login attempts trigger real-time email alerts, improving incident response.

Demonstrates practical use of Azure Monitor + Log Analytics + KQL for security monitoring.

## Skills & Technologies
- Azure Monitor & Log Analytics
- Kusto Query Language (KQL)
- Linux Syslog
- Security Monitoring & Alerting

