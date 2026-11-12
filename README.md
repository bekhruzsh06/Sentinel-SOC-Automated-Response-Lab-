# Sentinel SOC & Automated Incident Response Lab

## Objective

This project demonstrates the end-to-end lifecycle of a Security Operations Center (SOC) workflow in a cloud environment. The objective was to design and deploy a public-facing Azure honeypot to attract live threat actors, ingest telemetry into a cloud-native SIEM (Microsoft Sentinel), engineer custom detection analytics, and orchestrate an automated incident response pipeline that pushes actionable alerts to a SOC communication channel.

## Skills & Technologies Demonstrated

* **Cloud Infrastructure:** Microsoft Azure (Virtual Machines, NSG, Log Analytics Workspace)

* **SIEM & SOAR:** Microsoft Sentinel, Azure Logic Apps

* **Telemetry & Agents:** Azure Monitor Agent (AMA), Data Collection Rules (DCR), Windows Security Event Logs

* **Detection Engineering:** Kusto Query Language (KQL), MITRE ATT&CK Mapping

* **Offensive Security:** Hydra (Brute-force attack simulation)

* **Automation:** Slack API webhook integration

## Logical Architecture

1. **Attacker/Scanner:** Threat actors (or simulated attacks via Hydra) target the exposed RDP port (3389) on the Windows Virtual Machine.

2. **Telemetry Generation:** Windows Event Viewer logs all Failed Logon attempts (Event ID 4625).

3. **Log Ingestion:** The Azure Monitor Agent (AMA) forwards these security events to the Log Analytics Workspace (`SOC-Lab-LAW`).

4. **Threat Detection:** Microsoft Sentinel evaluates the logs against a custom KQL Scheduled Query every 5 minutes.

5. **SOAR Automation:** Upon breaching the detection threshold (10+ failed logins), Sentinel triggers an Azure Logic App playbook.

6. **Incident Notification:** The playbook parses the incident data and pushes a formatted alert to the SOC Tier 1 Slack channel.

## Project Execution & Phases

### Phase 1: Infrastructure & Telemetry Configuration

* Provisioned an intentionally vulnerable Windows 10 VM in Azure.

* Configured the Network Security Group (NSG) to allow inbound traffic from `Any` source on Port 3389 (RDP) to maximize visibility of internet-wide scanners.

* Deployed a Log Analytics Workspace and Microsoft Sentinel.

* Created a Data Collection Rule (DCR) instructing the Azure Monitor Agent to ingest only relevant Windows Security Events to optimize SIEM ingestion costs.

### Phase 2: Detection Engineering (KQL)

Engineered a custom KQL query to detect high-volume RDP brute-force attempts. The query filters for Event ID 4625, aggregates failures by IP address and targeted account, and sets a threshold to flag instances where a single IP fails to authenticate 10 or more times.

**MITRE ATT&CK Mapping:** Credential Access (TA0006) > Brute Force (T1110)

```
SecurityEvent
| where EventID == 4625
| where LogonType == 3
| summarize FailedLogonCount = count() by IpAddress, Computer, Account
| where FailedLogonCount >= 10
| project TimeGenerated = now(), IpAddress, Computer, Account, FailedLogonCount
| sort by FailedLogonCount desc
```

### Phase 3: SOAR Automation & Response

Rather than relying on manual dashboard monitoring, a Security Orchestration, Automation, and Response (SOAR) workflow was implemented to improve Mean Time to Respond (MTTR).

* Created an Azure Logic App with a Sentinel Incident trigger.

* Configured Sentinel Automation Rules to execute the playbook whenever the custom Brute Force alert fires.

* The Logic App connects to the Slack API and dynamically formats a message containing the Incident Title, Severity, Description, and a direct URL to the Azure portal investigation page.


### Phase 4: Attack Simulation

To validate the detection pipeline, an offensive simulation was executed from a Kali Linux attack machine. The network logon cracker `Hydra` was utilized to blast the Azure VM with a custom wordlist of fake passwords against a target user account.

```
# Generating the custom wordlist
for i in {1..15}; do echo "Password123!_$i" >> passwords.txt; done

# Executing the brute force attack
hydra -l hacker -P passwords.txt rdp://<ip> -V
```

---

The successful deployment of this environment proves the ability to engineer custom cloud infrastructure, write SIEM detection logic from scratch, simulate adversary behavior, and automate SOC notification workflows. Future iterations of this lab could include automated remediation, such as updating the Azure NSG to dynamically block the attacker's IP address.
