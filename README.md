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
* [Lab 5: SecOps Automation - Developing Custom Analytics Rules for SSH Brute Force](#Lab5)

📁 **Phase 3: Endpoint Defense, Vulnerability Management & XDR (Wazuh / Elastic)**
* [Lab 6: Open-Source EDR Deployment & CIS Hardening - Proactive Vulnerability Management](#Lab6)
* [Lab 7: Malware & Ransomware Defense - Configuring File Integrity Monitoring (FIM) & Threat Hunting](#Lab7)

📁 **Phase 4: Network Security & Web Exploitation Defense**
* [Lab 8: Detecting Web Exploitation & SQL Injection via Wazuh and SIEM Correlation](#Lab8)

📁 **Phase 5: SOAR Automation & Incident Response Playbooks**
* [Lab 9: Active Response & Containment - Automating Endpoint Isolation via Custom Scripts](#Lab9)
* [Lab 10: SOC Governance - Tuning Dashboards, KPIs, and Executive Incident Reporting](#Lab10)

---

## 🛠️ Tech Stack & SOC Tools Blueprint

* **SIEM / Data Analytics:** Microsoft Sentinel, Log Analytics Workspaces, KQL, Elastic Stack.
* **EDR / XDR Platform:** Wazuh (Open-Source EDR), Microsoft Defender for Endpoint.
* **Network & Ingestion Controls:** Syslog, Azure Activity Logs, Host-Based Firewalls.
* **Frameworks & Framework Mapping:** MITRE ATT&CK, NIST Incident Handling (SP 800-61), CIS Benchmarks.
* **Security Domains:** Vulnerability Analysis, Hardening, Web Exploitation, SOAR Automation, Phishing/Malware Detection.

<br>
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

<img width="1161" height="774" alt="image" src="https://github.com/user-attachments/assets/0048a69b-c86b-4f5f-be5a-fc611fa596dd" />
<img width="1103" height="644" alt="Captura de tela 2026-05-27 150359" src="https://github.com/user-attachments/assets/fcec3b9d-eecb-4d62-b4e9-a23c6a85fb32" />
<img width="1106" height="644" alt="Captura de tela 2026-05-27 150429" src="https://github.com/user-attachments/assets/caebe97d-2cab-48e9-b585-1d851424211b" />
<img width="1099" height="636" alt="Captura de tela 2026-05-27 150502" src="https://github.com/user-attachments/assets/1ec1df42-dbae-408f-b26c-b1100ae55a53" />
<img width="1109" height="644" alt="image" src="https://github.com/user-attachments/assets/e934e58a-8574-4d63-bfd4-d5dffe8a7321" />
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
## 🛡️ Lab 5: SecOps Automation - Developing Custom Analytics Rules for SSH Brute Force

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

<br>
<br>

# Lab6
## 🛡️ Open-Source EDR Deployment & CIS Hardening - Proactive Vulnerability Management

**Objective:** Architect and deploy an open-source XDR/EDR solution (Wazuh) across a hybrid environment (Linux/Windows), establish secure telemetry tunnels, and conduct a proactive vulnerability assessment aligned with CIS (Center for Internet Security) Benchmarks.

**Scenario:** A critical requirement for modern SOC operations is continuous endpoint visibility. The organization requires a centralized EDR platform to monitor internal servers and workstations. Once deployed, the SOC analyst must identify misconfigurations and prioritize hardening efforts to reduce the attack surface before an incident occurs.

### 1. The Engineer's Perspective (EDR Infrastructure Deployment)
An "All-in-One" Wazuh architecture (Manager, Indexer, and Dashboard) was deployed on an Ubuntu server. To establish a secure communication channel, NAT port forwarding rules (Ports 1514, 1515, 443) were configured to allow external endpoints to reach the isolated SOC environment. Agents were successfully compiled, authenticated, and deployed across both Linux (`Ubuntu-SOC-Server`) and Windows (`Windows-Pielt`) endpoints.

<br>

<img width="1158" height="769" alt="image" src="https://github.com/user-attachments/assets/78a61f94-2e18-405b-a190-735a6e29c967" />
<img width="1103" height="644" alt="Captura de tela 2026-05-27 150359" src="https://github.com/user-attachments/assets/26202711-3fbe-40be-b9b3-81d82f182ba6" />
<img width="1106" height="644" alt="Captura de tela 2026-05-27 150429" src="https://github.com/user-attachments/assets/99fd9ac3-e7af-4dd5-89ba-5b9081954474" />
<img width="1099" height="636" alt="Captura de tela 2026-05-27 150502" src="https://github.com/user-attachments/assets/766c4bfc-1f33-43f0-9b2e-d3fb6fa660c6" />
<img width="1915" height="787" alt="image" src="https://github.com/user-attachments/assets/8aab087d-2040-41f4-81f5-3bf4df7f14d8" />

<br>

<img width="1916" height="1027" alt="image" src="https://github.com/user-attachments/assets/b786f7d4-035c-439e-ac2c-7fd56e264093" />

*Caption: Wazuh Endpoints Summary dashboard confirming the successful cryptographic enrollment and active connection of the Windows endpoint via the established NAT tunnels.*

### 2. The Analyst's Perspective (Vulnerability Management & Compliance)
Operating from the Wazuh Dashboard, the SOC analyst initiated a Configuration Assessment scan against the newly enrolled Windows endpoint. The system evaluated the host against the strict `CIS Microsoft Windows 11 Enterprise Benchmark v1.0.0` to identify deviations from corporate security policies.

<br>

<img width="1913" height="1026" alt="image" src="https://github.com/user-attachments/assets/8bbb8231-9f5b-4aea-9431-6582eb49a68c" />

*Caption: The Security Configuration Assessment (SCA) results, displaying the initial compliance score and the macroscopic view of passed versus failed security checks on the target endpoint.*

### 3. Forensic Evidence (Hardening Gaps Identified)
The initial scan resulted in a 29% compliance score, immediately highlighting critical security gaps. The telemetry pinpointed specific vulnerabilities, such as `Ensure password must meet complexity requirements is set to Enabled`, which was marked as 'Failed'. This provides the SOC with a prioritized, actionable remediation list to enforce system hardening and mitigate potential brute-force or lateral movement vectors.

<br>

<img width="1383" height="870" alt="image" src="https://github.com/user-attachments/assets/d7e5975c-4697-4cf5-a32c-781ff06902f6" />

*Caption: Granular log analysis revealing the specific registry key and policy misconfigurations (e.g., Password Complexity) that require immediate incident response and hardening.*

**Skills Applied:** XDR/EDR Architecture (Wazuh), Endpoint Security, Vulnerability Analysis, CIS Benchmarks, System Hardening, Agent-Manager Cryptographic Enrollment.

<br>
<br>

# Lab7
## 🛡️ Malware & Ransomware Defense - Configuring File Integrity Monitoring (FIM) & Threat Hunting

**Objective:** Engineer a highly sensitive host-based intrusion detection mechanism using Wazuh's File Integrity Monitoring (FIM) to detect, log, and alert on ransomware behavior (creation, encryption/modification, and deletion of critical files) in real-time.

**Scenario:** The SOC requires a proactive defense mechanism against ransomware and data exfiltration tactics. To minimize Mean Time to Detect (MTTD), a "digital tripwire" must be established within a highly sensitive Windows directory. Any unauthorized modification to files within this directory must immediately trigger a high-severity alert within the unified SecOps dashboard.

### 1. The Engineer's Perspective (FIM Configuration & XML Injection)
To transform the Windows endpoint into an active sensor, the Wazuh agent configuration file (`ossec.conf`) required surgical modification. Administrative privileges were utilized to inject a custom `<directories>` rule directly into the XML structure of the `syscheck` module. The rule was strictly configured with `check_all="yes"` to monitor metadata, checksums, and permissions, and `realtime="yes"` to bypass the default polling interval and trigger instantaneous alerts upon file manipulation within the targeted directory (`C:\Users\Public\Laboratorio_SOC`).

<br>

<img width="1090" height="573" alt="image" src="https://github.com/user-attachments/assets/0d0bcab2-7895-4a1b-93b0-2b831d10bd5e" />

<img width="1913" height="1034" alt="image" src="https://github.com/user-attachments/assets/091e04be-83ed-4f83-bd2a-adb00260628d" />

<br>

### 2. The Attacker's Perspective (Ransomware Kill Chain Simulation)
To validate the SOC's detection capabilities, a simulated ransomware attack was executed locally against the monitored directory. The simulation followed a strict, cadenced kill chain:
* **Payload Drop:** A sensitive file (`senhas_banco.txt`) was created within the tripwire directory.
* **Encryption Phase:** The file's contents were heavily modified to simulate the rapid encryption behavior characteristic of ransomware payloads.
* **Covering Tracks:** The original, unencrypted file was forcibly deleted from the system.

### 3. The Analyst's Perspective (Incident Triage & IoC Extraction)
Operating from the central Wazuh Dashboard on the Ubuntu server, the SOC analyst monitored the incoming telemetry. The attack triggered a cascade of high-fidelity alerts perfectly mapping the adversary's actions. 

<br>

<img width="1915" height="1027" alt="image" src="https://github.com/user-attachments/assets/78f3ea77-f0d9-434b-82a7-20bae11f5971" />

<br>

### 4. Forensic Evidence (Checksum Validation)
The critical success of this detection relies on the `syscheck` engine calculating cryptographic hashes (MD5, SHA1, SHA256) of the files in real-time. The `modified` alert (Rule 550) mathematically proved that the file's internal integrity was altered, which is the definitive Indicator of Compromise (IoC) for an active ransomware infection. This allows the SOC to isolate the endpoint from the network before lateral movement can occur.

**Skills Applied:** Endpoint Detection and Response (EDR), File Integrity Monitoring (FIM), Ransomware Kill Chain Analysis, XML Configuration Management, Real-time Threat Triage, Indicators of Compromise (IoC) Extraction.

<br>
<br>

# Lab 8
## 🛡️ Detecting Web Exploitation & SQL Injection via Wazuh and SIEM Correlation

**Objective:** Expose a local Apache web server and utilize Wazuh to ingest, parse, and correlate access logs in real-time to detect common web application vulnerabilities (OWASP Top 10), specifically SQL Injection (SQLi), Cross-Site Scripting (XSS), and Path Traversal (LFI).

**Scenario:** The organization hosts several public-facing web applications. The SOC requires continuous monitoring of web server access logs to identify active reconnaissance and exploitation attempts. The objective is to validate that the EDR/SIEM platform can accurately identify malicious payloads embedded in HTTP requests and escalate them as high-priority incidents.

### 1. The Attacker's Perspective (OWASP Top 10 Exploitation)
To validate the SOC's detection engine, a series of targeted web attacks were simulated against the local Apache server (`http://localhost`) using command-line HTTP clients. The attack vectors included:
* **SQL Injection (SQLi):** Attempting to manipulate backend database queries via malicious URL parameters (`/?id=1'+OR+'1'='1`).
* **Path Traversal (LFI):** Attempting to escape the web root directory and read sensitive system files (`/?file=../../../../etc/passwd`).
* **Cross-Site Scripting (XSS):** Injecting malicious JavaScript payloads into search parameters (`/?search=<script>alert('Hackeado')</script>`).

### 2. The Analyst's Perspective (Web Threat Triage)
By default, the Wazuh Manager was configured to ingest the Apache `access.log`. Operating from the Threat Intelligence dashboard, the SOC analyst monitored the real-time log ingestion. The Wazuh decoding engine successfully parsed the raw HTTP requests, correlated the malicious character strings against its threat signature database, and triggered multiple high-severity alerts.

<img width="1915" height="890" alt="image" src="https://github.com/user-attachments/assets/c69b2563-6d0b-4332-9ed1-51080a007a41" />

> *Caption: The Wazuh Security Events dashboard displaying the intercepted attack vectors, explicitly categorizing the threats as "SQL injection attempt", "Multiple URI path traversal", and "Cross Site Scripting (XSS)".*

### 3. Forensic Evidence & Mitigation
Granular log analysis revealed the attacker's methodology, capturing the exact payload, the targeted URI path, and the HTTP response codes. This visibility allows the SOC to extract the offending IP address and implement immediate containment measures, such as updating Web Application Firewall (WAF) rules or isolating the web server.

**Skills Applied:** Web Exploitation Detection, Apache Log Analysis, OWASP Top 10 (SQLi, XSS, LFI), Wazuh Rule Decoding, Threat Intelligence Correlation, SOC Incident Triage.
