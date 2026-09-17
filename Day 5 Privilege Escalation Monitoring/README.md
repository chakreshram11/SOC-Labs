# 🔐 Day 5: Privilege Escalation Monitoring with Wazuh

## 📋 Overview

This lab demonstrates the detection, telemetry analysis, and SOC investigation of **privileged command execution** on an Ubuntu Server using **Wazuh SIEM**.

A controlled `sudo` activity was performed on the Ubuntu endpoint to generate security events. The resulting events were recorded by the Linux authentication subsystem, collected by the Wazuh Agent, forwarded to the Wazuh Manager, and investigated through the Wazuh Dashboard.

The exercise focuses on understanding how a SOC analyst can identify privileged activity, investigate the execution context, and determine whether the activity is authorized or suspicious.

> **Note:** This activity was performed in a controlled and isolated lab environment for security monitoring and detection practice.

---

## 🎯 Objectives

- **Privilege Activity Simulation:** Perform controlled privileged operations using `sudo`.
- **Endpoint Telemetry Collection:** Capture privileged command execution through Linux authentication logs.
- **SIEM Detection:** Detect successful `sudo` activity using Wazuh.
- **Event Investigation:** Analyze the source user, target account, command, session, and timestamp.
- **PAM Correlation:** Correlate sudo activity with PAM session events.
- **SOC Analysis:** Determine whether the observed privileged activity was authorized.
- **Incident Response:** Document appropriate investigation and response procedures for suspicious privileged activity.

---

## 💻 Lab Environment

| Component | Details |
|---|---|
| **SIEM Platform** | Wazuh SIEM 4.x |
| **Target Endpoint** | Ubuntu Server |
| **Wazuh Agent** | Ubuntu-Server |
| **Monitored Activity** | `sudo` privileged command execution |
| **Log Source** | `/var/log/auth.log` / journald |
| **Authentication Framework** | PAM |
| **Detection Console** | Wazuh Dashboard |
| **Environment** | Isolated SOC Lab |

> Internal usernames, IP addresses, and other environment-specific information have been intentionally omitted from this public GitHub documentation.

---

## 🏗️ Lab Architecture

```text
+-----------------------+
|    Ubuntu Server      |
|                       |
|  Normal User Session  |
|          |            |
|          v            |
|      sudo Command     |
|          |            |
|          v            |
|       root Context    |
|          |            |
|          v            |
| Linux Authentication |
|       Logs / PAM      |
+-----------+-----------+
            |
            | Wazuh Agent
            v
+-----------------------+
|    Wazuh Manager      |
|                       |
|   Detection Rules     |
|          |            |
|          v            |
|   Wazuh Dashboard     |
+-----------------------+
```
---


# ⚙️ Controlled Privileged Activity

## 1. Verify Current User

The current user context was first verified before performing any privileged operation.

```bash
whoami
```

The following command was then used to inspect the user's UID, GID, and group memberships:

```bash
id
```

This confirmed the normal user context and verified that the account had permission to use `sudo`.

---

## 2. Verify Privileged Context

A controlled `sudo` command was executed:

```bash
sudo whoami
```

Expected output:

```text
root
```

This confirmed that the command was executed with root privileges.

The privilege context was further verified using:

```bash
sudo id
```

This provided the UID, GID, and group information associated with the privileged execution.

---

## 3. Controlled Administrative File Operation

A temporary test file was created using elevated privileges:

```bash
sudo touch /tmp/soc_privilege_test
```

This operation was intentionally performed to generate a controlled administrative event that could be monitored through the SOC infrastructure.

---

# 📜 Linux Authentication Log Analysis

After executing the privileged commands, the Linux authentication logs were reviewed.

```bash
sudo grep -i "sudo" /var/log/auth.log | tail -20
```

The authentication logs recorded information related to the `sudo` activity.

Relevant fields included:

* Source user
* Target user
* Terminal / TTY
* Working directory
* Executed command
* Timestamp
* Sudo session information

Example log structure:

```text
sudo: user : TTY=... ; PWD=... ; USER=root ; COMMAND=/usr/bin/whoami
```

This confirmed that the Linux endpoint was successfully recording privileged command execution.

---

# 📸 Evidence & Screenshots

## 1. Ubuntu Server Commands

![Ubuntu Server Commands](ubuntu%20server%20commands.png)

**Figure 1:** Ubuntu Server commands used to verify the current user and perform controlled privileged operations.

---

## 2. Sudo Who Am I

![Sudo Who Am I](sudo%20who%20am%20i.png)

**Figure 2:** Verification of root-level execution using `sudo whoami`.

---

## 3. Sudo ID

![Sudo ID](sudo%20id.png)

**Figure 3:** Verification of the privileged UID, GID, and group context using `sudo id`.

---

## 4. Creating a Test File

![Creating a File Log](creating%20a%20file%20log.png)

**Figure 4:** Controlled creation of a temporary file using elevated privileges.

---

## 5. Wazuh Logs

![Wazuh Logs](wazuh%20logs.png)

**Figure 5:** Wazuh events showing detection of the privileged `sudo` activity.

---

# 🚨 Wazuh Detection

The Wazuh Agent collected the authentication events from the Ubuntu endpoint and forwarded them to the Wazuh Manager.

Wazuh generated multiple events associated with the privileged activity.

## Primary Detection

| Telemetry Field | Value                                   |
| --------------- | --------------------------------------- |
| **Rule ID**     | `5402`                                  |
| **Rule Level**  | `3`                                     |
| **Description** | `Successful sudo to ROOT executed`      |
| **Decoder**     | `sudo`                                  |
| **Activity**    | Successful privileged command execution |

The primary alert observed during the exercise was:

```text
Successful sudo to ROOT executed
```

This event indicates that a user successfully executed a command through `sudo` with `root` as the target account.

---

# 🔎 Related Wazuh Events

Additional events were observed during the investigation.

| Rule ID | Level | Event                            |
| ------- | ----: | -------------------------------- |
| `5402`  |     3 | Successful sudo to ROOT executed |
| `5501`  |     3 | PAM: Login session opened        |
| `5502`  |     3 | PAM: Login session closed        |

These events helped establish the sequence of the privileged activity.

---

# 📸 Evidence & Screenshots

## 1. Ubuntu Server Commands

![Ubuntu Server Commands](ubuntu%20server%20commands.png)

**Figure 1:** Ubuntu Server commands used to verify the current user and perform controlled privileged operations.

---

## 2. Sudo Who Am I

![Sudo Who Am I](sudo%20who%20am%20i.png)

**Figure 2:** Verification of root-level execution using `sudo whoami`.

---

## 3. Sudo ID

![Sudo ID](sudo%20id.png)

**Figure 3:** Verification of the privileged UID, GID, and group context using `sudo id`.

---

## 4. Creating a Test File

![Creating a File Log](creating%20a%20file%20log.png)

**Figure 4:** Controlled creation of a temporary file using elevated privileges.

---

## 5. Wazuh Logs

![Wazuh Logs](wazuh%20logs.png)

**Figure 5:** Wazuh events showing detection of the privileged `sudo` activity.
---

# 📊 Wazuh Event Investigation

The detailed Wazuh event was inspected to identify the context of the privileged command.

Important fields included:

| Field                 | Description                                      |
| --------------------- | ------------------------------------------------ |
| **Source User**       | User who initiated the `sudo` command            |
| **Destination User**  | Privileged account receiving elevated privileges |
| **Command**           | Command executed using `sudo`                    |
| **TTY**               | Terminal associated with the command             |
| **Working Directory** | Directory from which the command was executed    |
| **Agent**             | Endpoint generating the event                    |
| **Rule ID**           | Wazuh detection rule                             |
| **Rule Level**        | Alert severity                                   |
| **Timestamp**         | Time of the activity                             |

Commands observed during the controlled exercise included:

```text
/usr/bin/whoami
/usr/bin/id
/usr/bin/touch /tmp/soc_privilege_test
```

---

# 🔍 SOC Analyst Investigation Workflow

The investigation followed the workflow below:

```text
       [ Privileged Command Executed ]
                    |
                    v
       [ Linux Authentication Logs ]
                    |
                    v
             [ Wazuh Agent ]
                    |
                    v
            [ Wazuh Manager ]
                    |
                    v
           [ Detection Rule ]
                    |
                    v
           [ Alert Investigation ]
                    |
                    v
          [ Context Analysis ]
                    |
          +---------+---------+
          |                   |
          v                   v
     Authorized          Unauthorized
          |                   |
          v                   v
      Document           Investigate
      & Close            & Respond
```

---

# 🧪 Investigation Questions

During a real SOC investigation, the analyst should determine:

1. Which user performed the action?
2. Which account received elevated privileges?
3. What command was executed?
4. When did the activity occur?
5. Was the activity authorized?
6. Was the command expected for the system?
7. Was the user authorized to perform the administrative action?
8. Were there any suspicious events before or after the privileged activity?
9. Did the activity modify sensitive files, services, accounts, or configurations?
10. Are there related alerts from the same endpoint?

---

# 🛡️ Detection Analysis

The primary detection observed was:

```text
Rule ID: 5402
Level: 3
Successful sudo to ROOT executed
```

The alert itself does not indicate that the activity is malicious.

A successful `sudo` operation can represent:

* Normal system administration
* Authorized maintenance
* Security testing
* Software installation
* Configuration changes
* Potential unauthorized privilege use

Therefore, the alert must be investigated using the surrounding context.

---

# 🧠 SOC Analyst Takeaway

A privileged activity alert should be treated as an **investigation starting point**, rather than automatically being classified as malicious.

The analyst should correlate:

```text
User
  +
Command
  +
Target Account
  +
Timestamp
  +
Endpoint
  +
Related Events
```

This provides the context required to determine whether the activity is legitimate or requires further investigation.

---

# 🛡️ Recommended Defensive Actions

If unauthorized privileged activity is detected in a production environment, the SOC analyst should consider the following actions according to the organization's incident-response procedures.

### 1. Verify User Authorization

Confirm whether the user was authorized to perform the administrative action.

### 2. Review Executed Commands

Determine whether the command could modify:

* System configuration
* User accounts
* Services
* Security controls
* Sensitive files

### 3. Review Authentication Activity

Check for:

* Failed login attempts
* Successful logins
* Unusual login times
* New sessions
* Other privileged operations

### 4. Correlate Security Events

Review related Wazuh alerts from the same endpoint and user.

### 5. Investigate Suspicious Activity

If the activity is unauthorized, investigate related processes, files, accounts, and persistence mechanisms.

### 6. Containment

If compromise is suspected, follow the organization's incident-response procedure to contain the affected endpoint or account.

---

# 📋 Evidence Summary

| Evidence               | Description                                  |
| ---------------------- | -------------------------------------------- |
| User Verification      | Confirmed normal user context                |
| Privilege Verification | Confirmed successful root-level execution    |
| Sudo Activity          | Controlled privileged commands executed      |
| Authentication Logs    | Recorded sudo activity                       |
| Wazuh Alert            | Rule `5402` detected successful sudo to root |
| PAM Events             | Session opening and closing events observed  |
| Event Investigation    | Command and privilege context analyzed       |

---

# 🎯 Outcome

The exercise successfully demonstrated how privileged Linux activity can be monitored from endpoint logs through centralized SIEM detection.

The investigation covered:

* User privilege verification
* Controlled `sudo` execution
* Linux authentication logging
* Wazuh event collection
* Wazuh rule detection
* Event investigation
* Privileged activity analysis
* SOC response planning

---

# 📚 Key Learning

The exercise demonstrated the following SOC monitoring chain:

```text
Execute
   ↓
Log
   ↓
Collect
   ↓
Detect
   ↓
Investigate
   ↓
Correlate
   ↓
Determine Context
   ↓
Respond
```

The key lesson from this exercise is:

> **Privileged activity is not automatically malicious. Context and authorization are essential when investigating sudo events.**

---

# 🛠️ Tools & Technologies Used

* **Wazuh SIEM 4.x**
* **Wazuh Agent**
* **Wazuh Manager**
* **Wazuh Dashboard**
* **Ubuntu Server**
* **Linux Authentication Logs**
* **PAM**
* **sudo**

---

# 📁 Repository Structure

```text
Day 5 Privilege Escalation Monitoring/
│
├── README.md
│
├── creating a file log.png
├── sudo id.png
├── sudo who am i.png
├── ubuntu server commands.png
└── wazuh logs.png
```

---

# ⚠️ Disclaimer

This project was performed in a controlled and isolated laboratory environment for educational and SOC monitoring purposes.

The privileged operations were intentionally executed for security monitoring and detection testing.

No unauthorized systems or third-party infrastructure were targeted.

```
```
