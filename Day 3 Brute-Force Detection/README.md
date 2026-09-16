# 🚨 Day 3: SSH Brute-Force Detection & Correlation with Wazuh

## 📋 Overview

This lab demonstrates the detection, event correlation, and SOC investigation of an **SSH brute-force attack** against an **Ubuntu Server** using **Wazuh SIEM**.

A controlled SSH brute-force attack was conducted from a Kali Linux endpoint using **Hydra**. The resulting authentication failure logs were ingested by the Wazuh Agent, correlated by the Wazuh Manager, and escalated into a high-severity security alert (`Rule ID 5763`, Level 10).

---

## 🎯 Objectives

- **Attack Simulation:** Execute controlled SSH authentication attack vectors using Hydra from Kali Linux.
- **Log Telemetry Collection:** Capture raw endpoint authentication events in `/var/log/auth.log`.
- **SIEM Event Detection:** Detect repeated authentication failures and monitor real-time event ingestion in Wazuh.
- **Event Correlation:** Correlate atomic login failures (`Rule 5760`) into a high-severity brute-force alert (`Rule 5763`).
- **Attacker Attribution:** Extract and verify source IP (`10.10.10.30`), destination IP (`10.10.10.40`), and targeted accounts.
- **MITRE ATT&CK Mapping:** Map observed adversary TTPs to the MITRE ATT&CK framework (T1110.001, T1021.004).
- **Incident Response:** Formulate a SOC analyst investigation workflow and mitigation playbook.

---

## 💻 Lab Environment

| Component | Details |
|---|---|
| **SIEM Platform** | Wazuh SIEM 4.x |
| **Attacker Endpoint** | Kali Linux (`10.10.10.30`) |
| **Target Endpoint** | Ubuntu Server (`10.10.10.40`) |
| **Wazuh Agent Name** | `Ubuntu-Server` (Agent `002`) |
| **Attack Tool** | Hydra (SSH Protocol Module) |
| **Monitored Service** | OpenSSH (`sshd`) |
| **Log Source** | `/var/log/auth.log` |
| **Detection Console** | Wazuh Threat Hunting Dashboard |

```text
+-----------------------+              +-----------------------+
|  Kali Linux Attacker  |              |     Ubuntu Server     |
|     10.10.10.30       |              |      10.10.10.40      |
+-----------+-----------+              +-----------+-----------+
            |                                      |
            | SSH Brute-Force (Hydra)              | Wazuh Agent (002)
            v                                      v
    +---------------+                      +---------------+
    | OpenSSH Daemon|                      | Wazuh Manager |
    | Port 22 / PAM |                      | SIEM Dashboard|
    +---------------+                      +---------------+
```

---

## ⚔️ Attack Simulation & Endpoint Evidence

### 1. Hydra Brute-Force Attack Execution
A controlled dictionary attack was initiated from Kali Linux targeting SSH on the Ubuntu Server:

```bash
hydra -l ubuntu -P ~/Desktop/lab-passwords.txt ssh://10.10.10.40
```

![Hydra Brute-Force Execution](kali%20brute%20force%20using%20hydra.png)
*Figure 1: Executing Hydra SSH brute-force attack from Kali Linux against target endpoint 10.10.10.40.*

---

### 2. Interactive SSH Verification
Following the automated brute-force simulation, manual SSH authentication tests were conducted from the Kali Linux system:

![Kali SSH Login Attempt](ssh%20login%20with%20kali.png)
*Figure 2: Verifying network connectivity and SSH authentication behavior from Kali Linux.*

---

### 3. Ubuntu Endpoint Log Monitoring
Real-time authentication logs were monitored on the Ubuntu target using `tail -f /var/log/auth.log`:

```bash
sudo tail -f /var/log/auth.log
```

![Ubuntu Auth Logs](ubuntu%20auth%20logs.png)
*Figure 3: Monitoring `/var/log/auth.log` on the Ubuntu endpoint showing rapid authentication failures.*

---

## 📸 Wazuh SIEM Detection & Telemetry Analysis

### 1. Real-Time Security Event Detection
The Wazuh Manager aggregated authentication failures and correlated them into high-severity brute-force alerts:

![Wazuh Events Overview](wazuh%20events.png)
*Figure 4: Real-time event ingestion and brute-force alert visualization in the Wazuh Threat Hunting dashboard.*

---

### 2. Detailed Alert JSON Telemetry & Metadata
Deep inspection of the Wazuh alert metadata confirmed event correlation across Rule 5760 and Rule 5763:

![Wazuh Detailed Info](wazuh%20detailed%20info.png)
*Figure 5: In-depth inspection of alert payload, rule metadata, and severity scoring in Wazuh.*

---

## 📊 Detection & MITRE ATT&CK Mapping

### 🚨 Escalated Brute-Force Alert (Primary Detection)

| Telemetry Field | Value |
|---|---|
| **Rule ID** | `5763` |
| **Rule Level / Severity** | `Level 10` (High Risk) |
| **Description** | `sshd: brute force trying to get access to the system. Authentication failed.` |
| **Supporting Rule** | `5760` (`sshd: authentication failed`, Level 5) |
| **Frequency Trigger** | `frequency: 8` attempts within `timeframe: 120` seconds |
| **Target User** | `ubuntu` |
| **Source IP** | `10.10.10.30` (Kali Linux) |
| **Destination Host** | `Ubuntu-Server` (Agent `002`) |
| **Decoder** | `sshd` |

#### 🛡️ MITRE ATT&CK Framework Mapping:
- **[T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)**: Adversaries attempt to log into targeted accounts by systematically guessing passwords.
- **[T1021.004 — Remote Services: SSH](https://attack.mitre.org/techniques/T1021/004/)**: Adversaries access remote systems via OpenSSH services.

---

## 🔍 SOC Analyst Investigation Workflow

```text
           [ Authentication Attempt Detected ]
                          |
                          v
         [ Raw Event: sshd Failed Login (Rule 5760) ]
                          |
                          v
   [ Event Correlation: >8 Failures in 120s Window ]
                          |
                          v
      [ Trigger High-Severity Alert: Rule 5763 (L10) ]
                          |
                          v
          [ Triage: Extract Source & Target IP ]
                          |
                          v
      [ Check for Subsequent Successful Authentication ]
                          |
    +---------------------+---------------------+
    |                                           |
[ Success Found ]                      [ Only Failures ]
    |                                           |
    v                                           v
(CRITICAL: Compromise)                  (Active Brute-Force)
 - Contain Host                          - Block Source IP
 - Reset Account                         - Rate Limit SSH
```

---

## 🛡️ Recommended Defense & Response Actions

1. **Immediate Containment:** Block the attacker IP address (`10.10.10.30`) at the host firewall (`ufw`) or perimeter firewall.
2. **Account Audit:** Verify whether any successful login attempt (`Rule 5715`) occurred following the brute-force activity.
3. **SSH Hardening:**
   - Disable password-based SSH authentication (`PasswordAuthentication no` in `/etc/ssh/sshd_config`).
   - Mandate SSH Public/Private key-based authentication.
   - Change default SSH port (Port 22) to a non-standard port.
   - Restrict root login (`PermitRootLogin no`).
4. **Automated Active Response:** Enable Wazuh Active Response or install `fail2ban` to automatically drop IP addresses exceeding 5 failed login attempts.

---

## 📜 Summary & Conclusion

This lab successfully demonstrated how Wazuh SIEM correlates individual login failures into an actionable, high-severity SSH brute-force alert (`Rule 5763`, Level 10). By evaluating event frequency, source IP attribution, and temporal correlation, SOC analysts can rapidly distinguish routine authentication errors from automated brute-force attacks.

---

## 🛠️ Tools & Technologies Used

- **SIEM:** Wazuh SIEM 4.x (Manager & Dashboard)
- **Agent:** Wazuh Agent for Linux
- **Attacker OS:** Kali Linux
- **Attack Tool:** Hydra
- **Target OS:** Ubuntu Server
- **Service:** OpenSSH (`sshd`), PAM
