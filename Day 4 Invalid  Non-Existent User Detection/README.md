# 🚨 Day 4: Invalid / Non-Existent User SSH Detection with Wazuh

## 📋 Overview

This lab demonstrates the detection, telemetry analysis, and SOC investigation of SSH authentication attempts targeting **invalid or non-existent usernames** using **Wazuh SIEM**.

The activity was executed in a controlled lab environment where an attacker system (Kali Linux) initiated SSH login attempts using non-existent usernames against a target system (**Ubuntu Server**). The resulting events were captured in real time by the Wazuh Agent, forwarded to the Wazuh Manager, and correlated to detect **reconnaissance and username enumeration TTPs**.

---

## 🎯 Objectives

- **Reconnaissance Simulation:** Generate SSH login attempts using non-existent user accounts from Kali Linux.
- **Endpoint Log Telemetry:** Monitor raw authentication records in `/var/log/auth.log` on the target system.
- **SIEM Alert Detection:** Detect invalid user authentication attempts in Wazuh (`Rule ID 5710`, Level 5).
- **PAM Event Correlation:** Correlate user verification failures with supporting PAM alerts (`Rule ID 5503`, Level 5).
- **Threat Attribution:** Identify the source IP address, destination endpoint, attempted usernames, and timestamps.
- **MITRE ATT&CK Mapping:** Map the activity to MITRE ATT&CK TTPs (T1087.001 - Account Discovery: Local Account).
- **Analyst Playbook:** Formulate SOC triage workflows and defense strategies for username enumeration.

---

## 💻 Lab Environment

| Component | Details |
|---|---|
| **SIEM Platform** | Wazuh SIEM 4.x |
| **Source / Attacker** | Kali Linux (`10.10.10.30`) |
| **Target Endpoint** | Ubuntu Server (`10.10.10.40`) |
| **Wazuh Agent Name** | `Ubuntu-Server` (Agent `002`) |
| **Monitored Protocol** | SSH (Port 22) |
| **Log Source** | `/var/log/auth.log` |
| **Detection Console** | Wazuh Threat Hunting Dashboard |

```text
+-----------------------+              +-----------------------+
|  Kali Linux Attacker  |              |     Ubuntu Server     |
|     10.10.10.30       |              |      10.10.10.40      |
+-----------+-----------+              +-----------+-----------+
            |                                      |
            | Invalid Username Attempts            | Wazuh Agent (002)
            v                                      v
    +---------------+                      +---------------+
    | OpenSSH Daemon|                      | Wazuh Manager |
    |  Invalid User |                      | SIEM Dashboard|
    +---------------+                      +---------------+
```

---

## ⚔️ Simulation & Log Telemetry

### 1. Controlled Invalid User Simulation
Controlled SSH authentication attempts were conducted from Kali Linux using arbitrary, non-existent usernames:

```bash
ssh admin@10.10.10.40
ssh administrator@10.10.10.40
ssh testuser@10.10.10.40
```

### 2. Endpoint Log Monitoring
On the target Ubuntu host, `/var/log/auth.log` recorded explicit entries for authentication attempts targeting non-existent accounts:

```text
Invalid user admin from 10.10.10.30
Invalid user administrator from 10.10.10.30
Invalid user testuser from 10.10.10.30
```

---

## 📸 Wazuh Detection & Evidence

### 1. Invalid User Detection Telemetry
Wazuh ingested the OpenSSH authentication telemetry and triggered alerts specifically identifying non-existent user login attempts:

![Wazuh Invalid User Detection Log](wazuh%20invalid%20user%20log.png)
*Figure 1: Wazuh SIEM event alert confirming detection of SSH authentication attempt with non-existent user (Rule 5710).*

---

## 📊 Detection & MITRE ATT&CK Mapping

### 🚨 Primary Detection Details

| Telemetry Field | Value |
|---|---|
| **Rule ID** | `5710` |
| **Rule Severity** | `Level 5` |
| **Description** | `sshd: Attempt to login using a non-existent user` |
| **Supporting Rule** | `5503` (`PAM: User login failed`, Level 5) |
| **Target Usernames** | `admin`, `administrator`, `testuser`, `backupadmin` |
| **Source IP** | `10.10.10.30` (Kali Linux) |
| **Destination Host** | `Ubuntu-Server` (Agent `002`) |
| **Decoder** | `sshd` |

#### 🛡️ MITRE ATT&CK Framework Mapping:
- **[T1087.001 — Account Discovery: Local Account](https://attack.mitre.org/techniques/T1087/001/)**: Adversaries attempt to get a listing of local system accounts to refine brute-force targeting.
- **[T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)**: Adversaries test username/password pairs against exposed authentication portals.

---

## 🔍 SOC Analyst Investigation Workflow

```text
       [ Invalid User SSH Attempt Initiated ]
                         |
                         v
       [ OpenSSH Log: "Invalid user <name>" ]
                         |
                         v
    [ Wazuh Trigger: Rule 5710 (Level 5 Alert) ]
                         |
                         v
        [ Extract Source IP & Target Usernames ]
                         |
                         v
   [ Correlate Related Events from Same Source IP ]
                         |
       +-----------------+-----------------+
       |                                   |
[ High Frequency ]                 [ Low Frequency ]
       |                                   |
       v                                   v
(Possible Username                  (Isolated Misconfiguration
 Enumeration / Scan)                 / User Typo)
       |                                   |
       v                                   v
- Block / Rate-Limit Source        - Monitor for Escalation
- Review Valid User Attempts
```

---

## 🛡️ Defensive Hardening & Recommendations

1. **Disable SSH Password Authentication:** Transition to SSH public key authentication (`pubkey`) to mitigate password guessing and username enumeration.
2. **Restrict SSH Allowed Users:** Limit SSH access to specific accounts using the `AllowUsers` or `AllowGroups` directives in `/etc/ssh/sshd_config`.
3. **Deploy Active Response:** Configure Wazuh Active Response or `fail2ban` to automatically drop IP addresses that trigger repeated `Rule 5710` alerts.
4. **Implement Port Knocking or Bastion Host:** Remove direct SSH exposure from external networks by placing administrative interfaces behind a VPN or Bastion host.

---

## 📜 Summary & Conclusion

This lab demonstrated how Wazuh SIEM successfully detects SSH login attempts against non-existent accounts using `Rule ID 5710`. Identifying invalid user login spikes allows SOC analysts to detect early-stage reconnaissance and account enumeration efforts before adversaries discover valid credentials.

---

## 🛠️ Tools & Technologies Used

- **SIEM:** Wazuh SIEM 4.x (Manager & Dashboard)
- **Agent:** Wazuh Agent for Linux
- **Attacker OS:** Kali Linux
- **Target OS:** Ubuntu Server
- **Service:** OpenSSH (`sshd`), PAM
