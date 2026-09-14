# 🚨 Day 2: SSH Attack Detection & Unauthorized Access Monitoring with Wazuh

## 📋 Overview

This lab focuses on detecting and investigating controlled **SSH authentication attacks** and unauthorized access attempts against an **Ubuntu Server** using **Wazuh SIEM**.

The primary goal of the investigation was to trace remote SSH authentication attempts initiated from an external source system (Kali Linux), identify the targeted account, evaluate authentication results, analyze Wazuh detection rules and severity scores, and map the activity to the **MITRE ATT&CK Framework**.

---

## 🎯 Objectives

- **Attack Generation:** Perform controlled SSH authentication attempts from a simulated adversary node.
- **Failed Access Detection:** Detect and investigate failed SSH login attempts.
- **Successful Access Detection:** Detect and verify legitimate/unauthorized successful SSH logins.
- **Attacker Attribution:** Extract and identify the source IP address of the attack system.
- **Target Identification:** Determine the targeted endpoint (`Ubuntu-Server`) and user account (`ubuntu`).
- **Telemetry Analysis:** Examine event timestamps, decoders (`sshd`), and Wazuh Rule IDs.
- **MITRE ATT&CK Mapping:** Map detections to relevant TTPs (Password Guessing, Valid Accounts, SSH).
- **Incident Response:** Formulate a SOC analyst triage workflow and defensive response strategy.

---

## 💻 Lab Environment

| Component | Details |
|---|---|
| **SIEM Platform** | Wazuh SIEM 4.x |
| **Attack Simulation Host** | Kali Linux (Lab SSH Client) |
| **Target Endpoint** | Ubuntu Server |
| **Wazuh Agent Name** | `Ubuntu-Server` |
| **Wazuh Agent ID** | `002` |
| **Monitored Protocol** | SSH (Port 22) |
| **Decoder** | `sshd` |
| **Environment Type** | Isolated Cyber Range Lab |

---

## ⚔️ Attack Simulation

Controlled SSH authentication attempts were conducted from the Kali Linux attack platform directed at the Ubuntu Server agent.

The test scenarios simulated brute-force / credential testing patterns followed by a successful login:
1. ❌ **Multiple Failed SSH Login Attempts:** Testing invalid passwords for targeted user `ubuntu`.
2. ✅ **Successful SSH Login Attempt:** Establishing an interactive SSH terminal session with valid credentials.

> ⚠️ **Notice:** All activities were executed within a sandboxed, permissioned laboratory environment for educational and security research purposes.

---

## 📸 Wazuh Detection & Investigation Telemetry

### 1. SSH Authentication Log Analysis
Wazuh captured both failed and successful SSH authentication alerts under the `sshd` decoder:

![Wazuh SSH Logs](wazuh%20logs.png)
*Figure 1: Real-time SSH authentication alerts displayed in the Wazuh SIEM alert console.*

---

### 2. Identifying Attacker Source IP Address
During triage, the analyst drilled into the alert JSON telemetry to extract the source IP of the attacking machine:

![Attacker Source IP Identification](wazuh%20find%20source%20ip%20of%20attacker.png)
*Figure 2: Pinpointing the source IP address and connection origin of the SSH attack.*

---

## 📊 Detection & MITRE ATT&CK Mapping

### ❌ SSH Authentication Failure

| Parameter | Telemetry Value |
|---|---|
| **Rule ID** | `5760` |
| **Severity Level** | `Level 5` |
| **Description** | `sshd: authentication failed` |
| **Target User** | `ubuntu` |
| **Source System** | Kali Linux (Lab SSH Client) |
| **Destination Host** | `Ubuntu-Server` (Agent 002) |
| **Decoder** | `sshd` |
| **Authentication Result** | **Failed** |

#### 🛡️ MITRE ATT&CK Mapping:
- **[T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)**: Adversaries attempt to log into account credentials by guessing passwords.
- **[T1021.004 — Remote Services: SSH](https://attack.mitre.org/techniques/T1021/004/)**: Adversaries communicate over SSH to gain remote access.

---

### ✅ Successful SSH Authentication

| Parameter | Telemetry Value |
|---|---|
| **Rule ID** | `5715` |
| **Severity Level** | `Level 3` |
| **Description** | `sshd: authentication success` |
| **Target User** | `ubuntu` |
| **Source System** | Kali Linux (Lab SSH Client) |
| **Destination Host** | `Ubuntu-Server` (Agent 002) |
| **Decoder** | `sshd` |
| **Authentication Result** | **Successful** |

#### 🛡️ MITRE ATT&CK Mapping:
- **[T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/)**: Adversaries obtain and abuse existing account credentials.
- **[T1021 — Remote Services](https://attack.mitre.org/techniques/T1021/)**: Exploiting valid remote access pathways.

---

## 🔍 SOC Analyst Investigation Workflow & Correlation

When triaging SSH authentication telemetry, SOC analysts follow a structured event correlation pattern:

```text
       ┌──────────────────────────────────────────┐
       │   🌐 Inbound SSH Connection Request     │
       └────────────────────┬─────────────────────┘
                            │
                            ▼
       ┌──────────────────────────────────────────┐
       │ ❌ SSH Authentication Failure (Rule 5760) │
       └────────────────────┬─────────────────────┘
                            │
                            ▼
       ┌──────────────────────────────────────────┐
       │ 🔍 Repeated Failures / Password Guessing │
       └────────────────────┬─────────────────────┘
                            │
                            ▼
       ┌──────────────────────────────────────────┐
       │ ✅ Successful SSH Authentication (5715)  │
       └────────────────────┬─────────────────────┘
                            │
                            ▼
       ┌──────────────────────────────────────────┐
       │  🔓 Interactive Shell Session Established │
       └────────────────────┬─────────────────────┘
                            │
                            ▼
       ┌──────────────────────────────────────────┐
       │  🚨 Investigate Scope & Authorization    │
       └──────────────────────────────────────────┘
```

### 💡 Remediation & Defense Recommendations

1. **Implement Fail2Ban / Active Response:** Automatically block source IP addresses after multiple failed SSH authentication attempts (e.g., 5 failures in 2 minutes).
2. **Disable Password Authentication:** Enforce SSH key-based authentication (`PubkeyAuthentication yes`) and disable plain passwords (`PasswordAuthentication no`) in `/etc/ssh/sshd_config`.
3. **Disable Direct Root & Default User Logins:** Restrict administrative access and require multi-factor authentication (MFA) for SSH access.
4. **Network Segmentation & Firewalls:** Limit SSH exposure to trusted subnet ranges or require a VPN/Bastion Host.
