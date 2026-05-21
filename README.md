# Hybrid-SOC-BlueTeam-Ops  ((under construction))
A production-grade Blue Team portfolio focused on Hybrid SOC operations, Threat Hunting (KQL), Incident Response, and MITRE ATT&amp;CK mapping.


# Hybrid SOC Blue Team Operations & Incident Response

A production-grade portfolio demonstrating active threat monitoring, log correlation, incident triage, and playbook automation across hybrid and Azure cloud environments. Designed to map directly to modern Security Operations Center (SOC) workflows and the MITRE ATT&CK framework.

## 🗺️ Strategic Roadmap & SOC Competency Index

📁 **Phase 1: Log Analysis & Event Correlation (Windows, Linux, Firewall)**
* [Lab 1: Forensic Log Analysis - Investigating Linux Hardening Failures & SSH Brute Force](#Lab1)
* [Lab 2: Windows Event ID Auditing - Tracking Privilege Escalation and Remote Executions](#)

📁 **Phase 2: SIEM Operations & Advanced Threat Hunting (Microsoft Sentinel & KQL)**
* [Lab 3: Modern Threat Hunting - Detecting Cloud Phishing & Lateral Movement via KQL](#)
* [Lab 4: MITRE ATT&CK Mapping - Developing Custom Analytics Rules for Ransomware Detection](#)

📁 **Phase 3: Endpoint Defense & XDR Integration (Wazuh / Defender for Endpoint)**
* [Lab 5: Open-Source EDR Deployment - Deploying Wazuh to Monitor Host-Based Indicators of Compromise (IoCs)](#)

📁 **Phase 4: SOAR Automation & Incident Response Playbooks**
* [Lab 6: Automated Contained & Triage - Orchestrating Logic Apps for Real-Time Threat Mitigation](#)

📁 **Phase 5: Operational Reporting & Security Metrics (SOC Governance)**
* [Lab 7: Cyber Threat Intelligence (CTI) & Executive Incident Reporting for Critical Outages](#)

---

## 🛠️ Tech Stack & SOC Tools Blueprint

* **SIEM / Data Analytics:** Microsoft Sentinel, Log Analytics Workspaces, KQL (Kusto Query Language).
* **EDR / XDR Platform:** Wazuh (Open-Source EDR), Microsoft Defender for Cloud / Endpoint.
* **Network & Ingestion Controls:** Syslog, Azure Activity Logs, Host-Based Firewalls (UFW/Windows Firewall).
* **Frameworks & Framework Mapping:** MITRE ATT&CK, NIST Computer Security Incident Handling Guide (SP 800-61 Rev. 2), GDPR Art. 32 Compliance.

---

## Lab1: Forensic Log Analysis - Linux SSH Brute Force Investigation

**Objective:** Simulate an adversary attempting to compromise a Linux server via SSH brute force, capture the resulting telemetries, and perform forensic log parsing to extract Indicators of Compromise (IoCs) for incident containment.

**SOC Analyst Level:** L1 / L2
**Framework:** MITRE ATT&CK
* Tactic: Credential Access (TA0006)
* Technique: Brute Force: Password Guessing (T1110.001)

### 1. Incident Simulation (Red Team Operation)
To generate actionable telemetries, a targeted brute-force attack was simulated against the Linux Operations Server (Ubuntu). Multiple authentication attempts were executed using common default administrative usernames (`admin`, `root`, `hacker`) over TCP Port 22 (SSH).

### 2. Threat Triage & Log Parsing (Blue Team Operation)
As a SOC Analyst, the primary objective is to identify the scope of the attack by analyzing system security logs. 
In Debian-based environments, authentication events are written to `/var/log/auth.log`.

**Command Line Forensics:**
To filter the noise and extract the exact threat actor's IP address and the frequency of the attack, the following parsing pipeline was executed:
`sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr`

> *Below is the output of the forensic investigation, clearly identifying the attacker's origin IP and the volume of unauthorized access attempts.*
<img width="1422" height="627" alt="image" src="https://github.com/user-attachments/assets/711ca26b-f5f1-4ccf-826b-24c9eff09c04" />

<br>
<br>

> *Upon extracting the IoC (Threat Actor IP), the immediate containment action involves blacklisting the IP at the host-based firewall level to disrupt the cyber kill chain.*
<img width="542" height="62" alt="image" src="https://github.com/user-attachments/assets/cc85adb6-352b-45b7-b593-c729a2085de5" />

<br>
<br>

## 🛡️ Lab 2: Windows Event ID Auditing & Privilege Escalation Detection

**Objective:** Simulate an adversary creating a persistent backdoor account and escalating privileges on a Windows host, followed by forensic auditing of Windows Security Event Logs to detect the malicious activity.

**Scenario:** A sophisticated threat actor gained initial access to a corporate Windows endpoint and attempted to establish persistence by creating a hidden local administrator account. The SOC must identify this privilege escalation and track the attacker's footprint using native Windows Event telemetry.

### 1. The Attacker's Perspective (Persistence Execution)
To generate actionable telemetry, a simulated attack was executed via PowerShell. The adversary created a hidden local user (`sysupdate`), escalated its privileges by adding it to the `Administrators` group for full system control, and subsequently deleted the account to wipe visible traces.
<br>
<img width="826" height="297" alt="image" src="https://github.com/user-attachments/assets/953651ef-9c5d-44a1-ae6d-3771679c19e2" />


### 2. The Analyst's Perspective (Telemetry & Tracking)
Operating within the SOC, the analyst must identify unauthorized account modifications by querying the Windows Security Event Log. The investigation focused on filtering critical security events, specifically tracking unauthorized additions to highly privileged groups.
<br>
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0adf203d-c840-449a-845e-df1c4a1944ea" />


### 3. Forensic Evidence (Event ID 4720 & 4732)
Deep diving into the filtered logs reveals the exact forensic timeline. The presence of **Event ID 4720** (A user account was created) explicitly confirms the creation of the `sysupdate` account. Furthermore, the logs captured the exact timestamp and identified that the compromised primary account (`ELTON-DELL\Elton`) was utilized to execute the unauthorized actions, providing a clear path for incident containment.

**Skills Applied:** Windows Security Event Auditing, PowerShell Forensics, Threat Hunting, Incident Response, Local IAM (Identity and Access Management).
