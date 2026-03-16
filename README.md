# Azure Cloud SOC Lab – Part 2

## Threat Remediation & Hardening with Microsoft Sentinel

This project is **Part 2 of my Azure SOC Honeypot Lab**.
In the first project, I deployed a vulnerable VM to attract and monitor real-world attacks using **Microsoft Sentinel** and **Log Analytics**.

In this lab, I move beyond detection and focus on **remediation and security hardening** by analyzing attacker activity and implementing defensive controls.

The goal of this project is to demonstrate the **complete SOC lifecycle**:

Detection → Investigation → Remediation → Hardening

---

# Lab Overview

This lab builds on the previously deployed honeypot environment and focuses on reducing attack exposure by implementing security controls within Azure.

## Key Objectives

* Analyze attack telemetry collected by Microsoft Sentinel
* Identify the most common threats targeting the honeypot
* Create detection rules for malicious activity
* Implement remediation steps to block attacker activity
* Harden the environment to prevent future attacks
* Measure the reduction of malicious activity after remediation

---

# Technologies Used

* Microsoft Azure
* Microsoft Sentinel (SIEM)
* Log Analytics Workspace
* Azure Network Security Groups (NSG)
* Microsoft Defender for Cloud
* Kusto Query Language (KQL)

---

# Lab Architecture

Internet Attackers
↓
Azure Honeypot VM (RDP exposed)
↓
Windows Security Logs
↓
Log Analytics Workspace
↓
Microsoft Sentinel (Detection & Investigation)
↓
Remediation & Hardening Controls

---

# Step 4 – Analyze Attacker Activity in Sentinel

Using Microsoft Sentinel logs, I analyzed authentication failures to identify brute-force attacks targeting the honeypot.

Navigate to:

Microsoft Sentinel → Logs

Example KQL query used to identify failed login attempts:

```kql
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by IpAddress
| sort by FailedAttempts desc
```

This query reveals:

* Top attacking IP addresses
* Frequency of login attempts
* Potential brute-force campaigns

### Findings

Common attack patterns observed included:

* RDP brute-force attempts
* Password spraying
* Reconnaissance scans from multiple geographic locations

Screenshots were captured showing attacker IPs and activity trends.

---

# Step 5 – Create Sentinel Detection Rule

To automatically detect suspicious activity, a **Sentinel Analytics Rule** was created.

Navigate to:

Microsoft Sentinel → Analytics → Create Rule

Detection query:

```kql
SecurityEvent
| where EventID == 4625
| summarize Attempts = count() by IpAddress
| where Attempts > 10
```

Rule configuration:

Severity: Medium
MITRE Tactic: Credential Access
Technique: Brute Force (T1110)

When triggered, this rule automatically generates **incidents in Microsoft Sentinel** for investigation.

---

# Step 6 – Incident Investigation

When alerts were triggered, incidents were investigated through the Sentinel portal.

Navigate to:

Microsoft Sentinel → Incidents

Each incident provided valuable investigation data including:

* Attacker IP address
* Targeted user accounts
* Login attempt frequency
* Geographic source of the attack

Using Sentinel investigation graphs allowed correlation between:

* Devices
* IP addresses
* Accounts
* Log activity

---

# Step 7 – Threat Containment (Blocking Attacker IPs)

After identifying malicious IP addresses, remediation actions were taken by blocking the attackers at the network layer.

Navigate to:

Azure VM → Networking → Network Security Group

A new inbound rule was created:

Priority: 100
Source: Attacker IP Address
Destination Port: 3389 (RDP)
Action: Deny

Example blocked IP:

185.220.101.4

This prevented further RDP access attempts from known malicious sources.

---

# Step 8 – Harden Network Security Rules

Initially, the honeypot VM allowed RDP access from any IP address to intentionally attract attacks.

To secure the environment, inbound access was restricted.

Previous rule:

Any → Port 3389 → Allow

Updated rule:

Trusted IP → Port 3389 → Allow

This significantly reduced exposure to automated scanning and brute-force attacks.

---

# Step 9 – Enable Microsoft Defender for Cloud Security Recommendations

Microsoft Defender for Cloud was used to identify additional security improvements.

Navigate to:

Microsoft Defender for Cloud → Recommendations

Recommended actions included:

* Enable Just-In-Time VM access
