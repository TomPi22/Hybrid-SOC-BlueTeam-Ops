# Hybrid-SOC-BlueTeam-Ops  ((under construction))
A production-grade Blue Team portfolio focused on Hybrid SOC operations, Threat Hunting (KQL), Incident Response, and MITRE ATT&amp;CK mapping.


# Hybrid SOC Blue Team Operations & Incident Response

A production-grade portfolio demonstrating active threat monitoring, log correlation, incident triage, and playbook automation across hybrid and Azure cloud environments. Designed to map directly to modern Security Operations Center (SOC) workflows and the MITRE ATT&CK framework.

## 🗺️ Strategic Roadmap & SOC Competency Index

📁 **Phase 1: Log Analysis & Event Correlation (Windows, Linux, Firewall)**
* [Lab 1: Forensic Log Analysis - Investigating Linux Hardening Failures & SSH Brute Force](#Lab1)
* [Lab 2: Windows Event ID Auditing - Tracking Privilege Escalation and Remote Executions](#Lab2)
* [Lab 3: Hybrid Cloud Onboarding & Identity Provisioning (Azure Arc)](#Lab3)
  
📁 **Phase 2: SIEM Operations & Advanced Threat Hunting (Microsoft Sentinel & KQL)**
* [Lab 4: Modern Threat Hunting - Detecting Cloud Phishing & Lateral Movement via KQL](#Lab4)
* [Lab 5: MITRE ATT&CK Mapping - Developing Custom Analytics Rules for Ransomware Detection](#Lab5)

📁 **Phase 3: Endpoint Defense & XDR Integration (Wazuh / Defender for Endpoint)**
* [Lab 6: Open-Source EDR Deployment - Deploying Wazuh to Monitor Host-Based Indicators of Compromise (IoCs)](#)

📁 **Phase 4: SOAR Automation & Incident Response Playbooks**
* [Lab 7: Automated Contained & Triage - Orchestrating Logic Apps for Real-Time Threat Mitigation](#)

📁 **Phase 5: Operational Reporting & Security Metrics (SOC Governance)**
* [Lab 8: Cyber Threat Intelligence (CTI) & Executive Incident Reporting for Critical Outages](#)

---

## 🛠️ Tech Stack & SOC Tools Blueprint

* **SIEM / Data Analytics:** Microsoft Sentinel, Log Analytics Workspaces, KQL (Kusto Query Language).
* **EDR / XDR Platform:** Wazuh (Open-Source EDR), Microsoft Defender for Cloud / Endpoint.
* **Network & Ingestion Controls:** Syslog, Azure Activity Logs, Host-Based Firewalls (UFW/Windows Firewall).
* **Frameworks & Framework Mapping:** MITRE ATT&CK, NIST Computer Security Incident Handling Guide (SP 800-61 Rev. 2), GDPR Art. 32 Compliance.

---

<br>

# Lab1
## Forensic Log Analysis - Linux SSH Brute Force Investigation

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

# Lab2
## 🛡️ Windows Event ID Auditing & Privilege Escalation Detection

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

<br>
<br>

# Lab3
## 🛡️ Lab 3: Hybrid Cloud Onboarding & Identity Provisioning (Azure Arc)

**Objective:** Project an on-premises Linux server into the Azure control plane using Azure Arc, establishing a secure, identity-driven management bridge for centralized SIEM ingestion and SOC visibility.

**Scenario:** To achieve full observability across the hybrid infrastructure, the SOC requires telemetry from local endpoints to be ingested into Microsoft Sentinel. Before log ingestion can occur, the physical/virtual on-premises Linux server must be authenticated and recognized as a native Azure resource.

### 1. The Engineer's Perspective (Agent Deployment & Authentication)
The Azure Connected Machine agent (`azcmagent`) was deployed on the local Ubuntu Operations Server. To securely bind the on-premises machine to the Azure tenant without hardcoding credentials, a device code authentication flow was utilized. The terminal successfully retrieved the Managed Service Identity (MSI) certificate, cryptographically proving the machine's identity to the cloud control plane.

<img width="1109" height="644" alt="image" src="https://github.com/user-attachments/assets/e934e58a-8574-4d63-bfd4-d5dffe8a7321" />
<br>
<img width="727" height="549" alt="image" src="https://github.com/user-attachments/assets/d566f1c9-0946-4f7a-83dc-f25da6e5bb97" />


### 2. The Analyst's Perspective (Cloud Validation)
Operating from the Azure Portal, the SOC can now verify that the hybrid bridge is active. The Linux server successfully reports its status as "Connected" within the designated Resource Group (`RG-Hybrid-SOC`). The machine is now fully manageable via Azure Resource Manager (ARM), paving the way for the deployment of the Azure Monitor Agent (AMA) and direct Microsoft Sentinel integration.
<br>
<img width="1915" height="842" alt="image" src="https://github.com/user-attachments/assets/481291dc-6bd6-4960-ab03-90fe348a2230" />


**Skills Applied:** Hybrid Cloud Architecture, Azure Arc Integration, Managed Service Identity (MSI), Device Code Authentication, Cloud Security Posture Management (CSPM) Preparation.

<br>
<br>

# Lab4
## 🛡️ Lab 4: Cloud SIEM Ingestion & Advanced Threat Hunting (KQL)

**Objective:** Ingest on-premises security telemetries into a cloud-native SIEM (Microsoft Sentinel) utilizing the Azure Monitor Agent (AMA) and execute advanced KQL (Kusto Query Language) threat hunting to detect credential compromise attempts.

**Scenario:** Following the successful Azure Arc hybrid onboarding, the SOC requires centralized, real-time visibility into the on-premises Linux infrastructure. The objective is to establish a telemetry pipeline and proactively hunt for the SSH brute-force attack simulated in Lab 1 directly from the cloud console.

### 1. The Engineer's Perspective (Data Collection Rule & AMA)
To securely route telemetry to the SIEM, a Data Collection Rule (DCR) was engineered within Azure. This policy explicitly targeted the hybrid Linux server (`Pielt`), instructing the silent deployment of the Azure Monitor Agent (AMA). To optimize SIEM ingestion costs and adhere to FinOps best practices, the DCR was strictly scoped to collect only security-relevant Syslog facilities (`auth` and `authpriv` at `LOG_INFO` level), dropping unnecessary system noise.

<br>

<img width="1070" height="932" alt="image" src="https://github.com/user-attachments/assets/b8f8610c-2b46-4b36-b3b0-a015e09651cc" />

<br>

### 2. The Analyst's Perspective (KQL Threat Hunting)
With the telemetry pipeline active, a simulated SSH brute-force attack was executed locally. Operating from the Microsoft Sentinel workspace, a custom KQL query was authored to hunt for Indicators of Compromise (IoCs) within the ingested raw logs in real-time.

```kusto
Syslog
| where ProcessName == "sshd"
| where SyslogMessage contains "Failed password"
| project TimeGenerated, Computer, ProcessName, SyslogMessage
| sort by TimeGenerated desc
```

<br>

### 3. Forensic Evidence (Cloud Telemetry Validation)
The KQL query successfully parsed the Syslog data, directly linking the on-premises attack to the cloud SIEM. The results explicitly identified the exact timestamps, the targeted machine (Pielt), the process (sshd), and the malicious source IP (10.0.2.2) attempting to use the hacker credential. This confirms the hybrid bridge is fully operational and the SOC has complete visibility over the local perimeter.

**Skills Applied:** SIEM Engineering, Microsoft Sentinel, Kusto Query Language (KQL), Azure Monitor Agent (AMA), Data Collection Rules (DCR), FinOps (Log Filtering), Proactive Threat Hunting.

<br>
<br>

# Lab5
## 🛡️ Lab 5: SecOps Automation & Incident Response (Analytics Rules)

**Objective:** Automate the detection of credential access attempts by translating proactive KQL threat hunting queries into active SIEM Analytics Rules, generating high-fidelity SOC incidents.

**Scenario:** Manual threat hunting is insufficient for 24/7 Operations. To minimize Mean Time to Detect (MTTD), the SOC requires an automated alerting mechanism that monitors hybrid Linux infrastructure for SSH brute-force attacks and generates actionable incidents within the unified SecOps platform.

### 1. The Engineer's Perspective (Detection Engineering)
A custom Analytics Rule was engineered within Microsoft Sentinel. The previously utilized KQL query was embedded as the core detection logic. To account for cloud ingestion latency inherent in hybrid environments, the scheduling parameters were strategically tuned (executing every 5 minutes with a 1-hour lookback). The rule was explicitly mapped to the MITRE ATT&CK framework (T1110.001: Password Guessing) and configured with Entity Mapping (HostName) to ensure proper graph correlation.

### 2. The Attacker's Perspective (Triggering the Trap)
A rapid succession of unauthorized SSH login attempts was executed locally against the Operations Server. The simulated threat actor attempted to brute-force the `hacker` credential, unknowingly triggering the deployed telemetry pipeline.

### 3. The Analyst's Perspective (Incident Triage)
The automated detection engine successfully intercepted the ingested telemetry and raised a High-Severity incident within the Microsoft Defender unified portal. The SOC analyst is immediately presented with a correlated alert ("SSH Brute Force Detection"), prioritized and enriched with the exact compromised endpoint (`Pielt`), allowing for immediate containment actions.

<br>

<img width="1916" height="1036" alt="image" src="https://github.com/user-attachments/assets/f1ff9a12-a417-48fc-8aae-1b4b13302918" />


**Skills Applied:** Detection Engineering, SecOps Automation, Microsoft Defender XDR, Incident Generation, MITRE ATT&CK Mapping (T1110.001), Ingestion Latency Mitigation.
