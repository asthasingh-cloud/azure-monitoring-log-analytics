# Centralised Azure Infrastructure Monitoring & Security Log Hub

## 📌 Project Overview
In this project, I built a centralized monitoring solution inside Microsoft Azure. Instead of checking system logs and performance on every virtual machine separately, I set up a pipeline that sends VM performance data and security event logs into a single Log Analytics workspace, where they can be queried in one place. I also created two alert rules from KQL queries.

---

## 🛠️ Infrastructure Components Deployed
*   **Resource Group:** `RG-MONITORING-PROJECT`.
*   **Log Analytics Workspace:** `SOC-APP-Insights` (a Log Analytics workspace, not Application Insights).
*   **Windows Server VM:** `Web-Prod-VM`, running the Internet Information Services (IIS) web server.
*   **Linux Server VM:** `App-Data-VM`, running the Nginx web server.
*   **Data Collection Rule (DCR):** One rule that targets both VMs, collects their performance and log data, and sends it to the workspace.
*   **Action Group:** Configured to send email notifications for the alerts.

---

## 📊 Environment Specs

| Resource Name | Type | Operating System | Web Server | Network |
| :--- | :--- | :--- | :--- | :--- |
| `SOC-APP-Insights` | Log Analytics Workspace | N/A (managed service) | N/A | N/A |
| `Web-Prod-VM` | Virtual Machine | Windows Server 2022 | IIS | Private VNet subnet (no public IP) |
| `App-Data-VM` | Virtual Machine | Ubuntu Linux 22.04 LTS | Nginx | Private VNet subnet (no public IP) |

---

## 🚨 KQL Queries Used for Alerts

### 1️⃣ Failed Logon Detection (Windows VM)
Looks for Event ID 4625 (failed logon). If an account has more than 5 failed attempts within 5 minutes, it can indicate a brute-force attempt, and the alert rule raises an alert for investigation.
```kusto
SecurityEvent
| where EventID == 4625
| summarize FailureCount = count() by TargetAccount, Computer, bin(TimeGenerated, 5m)
| where FailureCount > 5
```
*Note: Event ID 4625 applies to Windows only. Failed SSH logins on the Linux VM would use the `Syslog` table.*

### 2️⃣ High CPU Usage
Calculates the average CPU usage per VM in 5-minute intervals and returns only the VMs above 80%.
```kusto
Perf
| where CounterName == "% Processor Time" and InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| where AvgCPU > 80
```

---

## 🔔 Alert Rules
I created two log search alert rules in Azure Monitor from the queries above.

| Alert | Based on query | Severity |
| :--- | :--- | :--- |
| Failed Logons (Windows) | Query 1 | Sev 0 - Critical |
| High CPU | Query 2 | Sev 1 - Error |

Notification: an Azure Monitor action group sends an email to the alert owner.

---

## ℹ️ Status
Built on an Azure free trial using the Azure portal. The resources have been deleted after completion.
