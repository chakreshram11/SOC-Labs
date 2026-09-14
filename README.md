# 🛡️ SOC Labs: Security Operations Center & Threat Detection Engineering

Welcome to the **SOC Labs** repository. This project documents hands-on security operations center (SOC) investigations, security information and event management (SIEM) detection engineering, log analysis, threat hunting, and attack simulation labs using **Wazuh SIEM**, **Ubuntu Server**, and **Kali Linux**.

---

## 📌 Repository Overview

This repository acts as a comprehensive lab portfolio showcasing real-world SOC workflows, rule evaluation, log telemetry correlation, and adversary technique mapping aligned with the **MITRE ATT&CK Framework**.

```text
                  +-----------------------------------+
                  |         Kali Linux Attacker       |
                  |     (SSH / Brute-Force Client)    |
                  +-----------------+-----------------+
                                    |
                                    | Attack Vectors (Port 22 / PAM)
                                    v
+-----------------+       +---------+---------+
| Wazuh Dashboard | <==== |   Ubuntu Server   |
| (Alert Console) | Syslog|    (Agent 002)    |
+-----------------+       +-------------------+
```

---

## 📂 Lab Modules & Directory Structure

| Module | Topic | Target OS | Primary Detection / Focus | Link |
|:---:|---|:---:|---|:---:|
| **Day 1** | **Authentication & Login Monitoring** | Ubuntu Server | PAM sessions, `unix_chkpasswd`, baseline authentication logins | [Read Module ➔](./Day%201%20Authentication%20and%20Login%20Monitoring/) |
| **Day 2** | **SSH Attack Detection & Unauthorized Access** | Ubuntu Server | `sshd` failures, successful SSH entry, source IP attribution, MITRE T1110.001 | [Read Module ➔](./Day%202%20SSH%20Attacks%20and%20Unauthorized%20Access%20Attempts/) |

---

## 🔬 Lab Summaries

### 🔐 Day 1: Authentication & Login Monitoring with Wazuh
* **Objective:** Establish a baseline for Linux authentication activity and analyze PAM/syslog events.
* **Key Detections:**
  * `Rule ID 5503` (Level 5) – PAM login failure
  * `Rule ID 5557` (Level 5) – Password check failure (`unix_chkpasswd`)
  * `Rule ID 5501` (Level 3) – PAM session established
* **Outcome:** Developed an end-to-end timeline correlating failed authentication attempts to successful session initialization and termination.

### 🚨 Day 2: SSH Attack Detection & Unauthorized Access Monitoring
* **Objective:** Simulate controlled SSH authentication attacks from Kali Linux against an Ubuntu agent and perform SOC investigation.
* **Key Detections & TTPs:**
  * `Rule ID 5760` (Level 5) – `sshd` authentication failure ➔ **MITRE ATT&CK T1110.001 (Password Guessing)**
  * `Rule ID 5715` (Level 3) – `sshd` authentication success ➔ **MITRE ATT&CK T1078 (Valid Accounts)**
* **Outcome:** Attributed attack origin to the external Kali client IP, extracted targeted user credentials (`ubuntu`), and formulated active response & hardening controls.

---

## 🛠️ Lab Environment & Technologies

* **SIEM Platform:** Wazuh SIEM 4.x
* **Target Endpoint:** Ubuntu Server (`Agent ID: 002`)
* **Adversary Node:** Kali Linux
* **Monitored Services:** OpenSSH (`sshd`), Pluggable Authentication Modules (`PAM`), `syslog`
* **Frameworks:** MITRE ATT&CK Framework

---

## 🚀 Quick Start / How to Navigate

1. Navigate to any lab folder (e.g., `Day 1 Authentication and Login Monitoring/` or `Day 2 SSH Attacks and Unauthorized Access Attempts/`).
2. Each module contains a dedicated `README.md` with:
   * Technical objectives & component specs
   * Step-by-step simulation details
   * High-resolution telemetry screenshots & alert JSON analysis
   * Detection mapping tables & investigation flowcharts
   * Hardening and SOC analyst takeaways

---

## 📜 Author & License

* **Author:** Kudupudi Chakresh Ram ([@chakreshram11](https://github.com/chakreshram11))
* **License:** MIT License — Free for educational and research use.
