# 🛡️ SOC Labs: Security Operations Center & Threat Detection Engineering

Welcome to the **SOC Labs** repository. This project documents hands-on security operations center (SOC) setup, security information and event management (SIEM) detection engineering, log analysis, threat hunting, and attack simulation labs using **Wazuh SIEM**, **Ubuntu Server**, **Windows 10 Enterprise**, and **Kali Linux**.

---

## 📌 Repository Overview

This repository acts as a comprehensive lab portfolio showcasing real-world SOC workflows, SIEM infrastructure implementation, log telemetry correlation, and adversary technique mapping aligned with the **MITRE ATT&CK Framework**.

```text
                  +-----------------------------------+
                  |         Kali Linux Attacker       |
                  |     (SSH / Brute-Force Client)    |
                  +-----------------+-----------------+
                                    |
                                    | Attack Vectors (Port 22 / PAM)
                                    v
+-----------------+       +---------+---------+       +-------------------+
| Wazuh Dashboard | <==== |   Ubuntu Server   | <==== |    Windows 10     |
| (Alert Console) | Syslog|    (Agent 002)    | Agent | (Agent + Sysmon)  |
+-----------------+       +-------------------+       +-------------------+
```

---

## 📂 Lab Modules & Directory Index

| Module / Guide | Topic | Target / Scope | Primary Focus | Documentation Link |
|:---:|---|:---:|---|:---:|
| 🛠️ **Setup Guide** | **Wazuh SOC Home Lab Architecture & Implementation** | SIEM Infra + Endpoints | VMware dual-homed networking, Wazuh Manager, Agent onboarding, Sysmon, troubleshooting | [Read Guide ➔](./SOC-Home-Lab-Setup-Guide/) |
| 🔐 **Day 1** | **Authentication & Login Monitoring** | Ubuntu Server | PAM sessions, `unix_chkpasswd`, baseline authentication logins | [Read Module ➔](./Day%201%20Authentication%20and%20Login%20Monitoring/) |
| 🚨 **Day 2** | **SSH Attack Detection & Unauthorized Access** | Ubuntu Server | `sshd` failures, successful SSH entry, source IP attribution, MITRE T1110.001 | [Read Module ➔](./Day%202%20SSH%20Attacks%20and%20Unauthorized%20Access%20Attempts/) |
| ⚡ **Day 3** | **SSH Brute-Force Detection & Correlation** | Ubuntu Server | Hydra brute-force simulation, event correlation (`Rule 5763`), MITRE T1110.001 | [Read Module ➔](./Day%203%20Brute-Force%20Detection/) |
| 🔍 **Day 4** | **Invalid / Non-Existent User Detection** | Ubuntu Server | Account enumeration detection, PAM failure correlation (`Rule 5710`), MITRE T1087.001 | [Read Module ➔](./Day%204%20Invalid%20%20Non-Existent%20User%20Detection/) |

---

## 🔬 Key Highlights & Lab Summaries

### 🛠️ Wazuh SOC Home Lab Implementation & Configuration Guide
* **Overview:** Complete installation guide for building a multi-endpoint virtual SOC.
* **Key Components:** VMware Host-Only (`VMnet1`) isolation, Ubuntu Wazuh Manager 4.x All-in-One deployment, Windows 10 Agent + Sysmon telemetry integration, Linux Agent setup, PowerShell/CMD troubleshooting, and alert verification.

### 🔐 Day 1: Authentication & Login Monitoring with Wazuh
* **Overview:** Establish a baseline for Linux authentication activity and analyze PAM/syslog events.
* **Key Detections:**
  * `Rule ID 5503` (Level 5) – PAM login failure
  * `Rule ID 5557` (Level 5) – Password check failure (`unix_chkpasswd`)
  * `Rule ID 5501` (Level 3) – PAM session established
* **Outcome:** Developed an end-to-end timeline correlating failed authentication attempts to successful session initialization and termination.

### 🚨 Day 2: SSH Attack Detection & Unauthorized Access Monitoring
* **Overview:** Simulate controlled SSH authentication attacks from Kali Linux against an Ubuntu agent and perform SOC investigation.
* **Key Detections & TTPs:**
  * `Rule ID 5760` (Level 5) – `sshd` authentication failure ➔ **MITRE ATT&CK T1110.001 (Password Guessing)**
  * `Rule ID 5715` (Level 3) – `sshd` authentication success ➔ **MITRE ATT&CK T1078 (Valid Accounts)**
* **Outcome:** Attributed attack origin to the external Kali client IP, extracted targeted user credentials (`ubuntu`), and formulated active response & hardening controls.

---

## 🛠️ Lab Environment & Technologies

* **SIEM Platform:** Wazuh SIEM 4.x (Manager, Indexer, Dashboard)
* **Endpoints:** Windows 10 Enterprise (`WIN10-ENDPOINT`), Ubuntu Server (`Ubuntu-Server`)
* **Adversary Node:** Kali Linux 2024.x
* **Security Extensions:** Microsoft Sysmon, OpenSSH (`sshd`), PAM, `syslog`
* **Frameworks:** MITRE ATT&CK Framework

---

## 🚀 Quick Start / How to Navigate

1. Begin with the [**SOC Home Lab Setup Guide**](./SOC-Home-Lab-Setup-Guide/) to understand the network topology, hardware resources, and SIEM installation.
2. Explore individual investigation modules:
   - [Day 1: Authentication & Login Monitoring](./Day%201%20Authentication%20and%20Login%20Monitoring/)
   - [Day 2: SSH Attack Detection](./Day%202%20SSH%20Attacks%20and%20Unauthorized%20Access%20Attempts/)
3. Each folder contains a structured `README.md` complete with step-by-step notes, high-resolution figures, alert telemetry tables, and flowcharts.

---

## 📜 Author & License

* **Author:** Kudupudi Chakresh Ram ([@chakreshram11](https://github.com/chakreshram11))
* **License:** MIT License — Free for educational and security research use.
