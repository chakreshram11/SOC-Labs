**# 👤 User Account Creation & Deletion Monitoring with Wazuh**

**## 📌 Overview**

This lab demonstrates how ********Linux user account creation and deletion activities****** can be monitored and investigated using ********Wazuh******.

A controlled test account named `testuser` was created on the Ubuntu Server and later deleted. The activity was investigated using Linux authentication logs, Auditd, and Wazuh alerts.

The purpose of this exercise is to understand how a SOC analyst can monitor ********account lifecycle events****** and determine who performed the action and what happened around the event.

---

****## 🎯 Objectives****

* Create a controlled Linux user account.

* Monitor user and group creation activity.

* Verify the newly created account.

* Analyze Linux authentication logs.

* Investigate Auditd events.

* Detect account creation in Wazuh.

* Delete the test account.

* Detect account deletion in Wazuh.

* Correlate user, process, and system activity.

* Practice a basic SOC investigation workflow.

---

****## 🧪 Lab Environment****

| Component     | Details            |

| ------------- | ------------------ |

| SIEM          | Wazuh              |

| Agent         | Ubuntu-Server      |

| Agent ID      | `002`              |

| Ubuntu IP     | `10.10.10.40`      |

| Wazuh Manager | `10.10.10.10`      |

| Test Account  | `testuser`         |

| Monitoring    | PAM, Auditd, Wazuh |

---

****## 🏗️ Lab Architecture****

```text

                    ┌──────────────────────┐

                    │    Wazuh Manager     │

                    │      10.10.10.10     │

                    └──────────┬───────────┘

                               │

                         Wazuh Agent

                               │

                    ┌──────────▼───────────┐

                    │    Ubuntu Server     │

                    │      10.10.10.40     │

                    │                      │

                    │  • useradd           │

                    │  • userdel           │

                    │  • PAM               │

                    │  • Auditd            │

                    └──────────────────────┘

```

---

****# 👤 Account Creation****

****## 1. Create the Test User****

A controlled test account was created using:

```bash

sudo useradd -m testuser

```

A password was then configured:

```bash

sudo passwd testuser

```

---

****## 2. Verify the Account****

The account was verified using:

```bash

id testuser

```

Example output:

```text

uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)

```

The account entry was also checked with:

```bash

getent passwd testuser

```

Example:

```text

testuser:x:1001:1001::/home/testuser:/bin/sh

```

---

****# 📜 Authentication Log Analysis****

The account creation activity was visible in the Ubuntu authentication logs.

Command:

```bash

sudo grep -i "testuser" /var/log/auth.log | tail -20

```

Relevant events included:

```text

useradd: new group: name=testuser, GID=1001

```

and:

```text

useradd: new user: name=testuser, UID=1001, GID=1001

```

The password change was also recorded:

```text

passwd: password changed for testuser

```

---

****# 🔎 Auditd Investigation****

Auditd was used to investigate the account creation event.

Command:

```bash

sudo ausearch -m ADD\\\_USER -ts recent

```

The audit event showed information such as:

```text

op=add-user

acct="testuser"

exe="/usr/sbin/useradd"

res=success

```

This provides useful investigation information including:

* Operation performed

* Account affected

* Executable responsible

* Result of the operation

* User/session context

---

****# 🚨 Wazuh Account Creation Detection****

Wazuh detected the account creation activity.

****### New User****

| Field       | Value                           |

| ----------- | ------------------------------- |

| Rule ID     | `5902`                          |

| Level       | `8`                             |

| Description | `New user added to the system.` |

****### New Group****

| Field       | Value                            |

| ----------- | -------------------------------- |

| Rule ID     | `5901`                           |

| Level       | `8`                              |

| Description | `New group added to the system.` |

****### Password Change****

| Field       | Value                         |

| ----------- | ----------------------------- |

| Rule ID     | `5555`                        |

| Level       | `3`                           |

| Description | `PAM: User changed password.` |

---

****# 🗑️ Account Deletion****

After completing the account creation investigation, the controlled test account was deleted.

Command:

```bash

sudo userdel -r testuser

```

The `-r` option removes the user's home directory and associated mail spool along with the account.

---

****## 1. Verify Account Deletion****

The account was verified using:

```bash

id testuser

```

Expected result:

```text

id: ‘testuser’: no such user

```

The password database was also checked:

```bash

getent passwd testuser

```

No output confirms that the account no longer exists in the local account database.

---

****# 📜 Account Deletion Log****

The deletion activity generated a `userdel` event.

Relevant log:

```text

userdel: delete user 'testuser'

```

The event was processed by the Wazuh `open-userdel` decoder.

---

****# 🚨 Wazuh Account Deletion Detection****

Wazuh detected the deletion activity.

| Field       | Value                                      |

| ----------- | ------------------------------------------ |

| Agent       | `Ubuntu-Server`                            |

| Agent ID    | `002`                                      |

| Agent IP    | `10.10.10.40`                              |

| User        | `testuser`                                 |

| Decoder     | `open-userdel`                             |

| Rule ID     | `5903`                                     |

| Level       | `3`                                        |

| Description | `Group (or user) deleted from the system.` |

| Process     | `userdel`                                  |

The Wazuh event contained:

```text

userdel: delete user 'testuser'

```

This confirms that Wazuh successfully received and analyzed the account deletion event.

---

****# 🔍 SOC Investigation****

Account creation and deletion events should be investigated in context.

A SOC analyst should determine:

****### 1. Who performed the action?****

Identify the source user responsible for creating or deleting the account.

****### 2. Which account was affected?****

In this lab:

```text

testuser

```

****### 3. What process performed the action?****

For deletion:

```text

userdel

```

For creation:

```text

useradd

```

****### 4. Why was the account created or deleted?****

The analyst should determine whether the activity was:

* Authorized administration

* Employee/user management

* System maintenance

* Suspicious account manipulation

****### 5. What happened before and after the event?****

Check for:

* Privilege changes

* Login activity

* Sudo activity

* SSH activity

* Suspicious processes

* File modifications

* Additional account changes

---

****# 🔄 SOC Investigation Workflow****

```text

        User Account Activity

                 │

                 ▼

       Linux System Logs

                 │

                 ▼

              Auditd

                 │

                 ▼

        Wazuh Agent

                 │

                 ▼

        Wazuh Detection

                 │

                 ▼

       Rule & Event Analysis

                 │

                 ▼

       User / Process Context

                 │

                 ▼

       SOC Investigation

```

---

****# 🛡️ Recommended SOC Response****

If an unexpected account creation or deletion is detected:

1\\. Identify the administrator or process responsible.

2\\. Verify whether the activity was authorized.

3\\. Review the affected account.

4\\. Check account privileges and group membership.

5\\. Review authentication and sudo activity.

6\\. Investigate activity before and after the event.

7\\. Check for other newly created accounts.

8\\. Preserve relevant logs and evidence.

9\\. Escalate the incident if unauthorized activity is confirmed.

---

****# 🎯 MITRE ATT&CK Mapping****

****### T1136.001 — Create Account: Local Account****

Account creation can be relevant to adversaries attempting to establish or maintain access to a system.

For this lab, the account creation was ********authorized and controlled******, so the MITRE mapping represents the security technique being monitored rather than claiming that the lab activity was malicious.

---

****# 📊 Detection Summary****

| Activity            | Wazuh Rule | Level | Detection                               |

| ------------------- | ---------: | ----: | --------------------------------------- |

| User creation       |     `5902` |     8 | New user added to the system            |

| Group creation      |     `5901` |     8 | New group added to the system           |

| Password change     |     `5555` |     3 | PAM: User changed password              |

| User/group deletion |     `5903` |     3 | Group (or user) deleted from the system |

---

****# 💡 Key Learning****

This exercise demonstrated how a SOC analyst can monitor the complete lifecycle of a Linux user account:

```text

Account Creation

      ↓

User Verification

      ↓

Password Configuration

      ↓

Log & Audit Analysis

      ↓

Wazuh Detection

      ↓

Account Deletion

      ↓

Wazuh Detection

      ↓

SOC Investigation

```

The important takeaway is that ********account creation or deletion is not automatically malicious******. The analyst needs to investigate the context, authorization, affected account, privileges, process, and surrounding activity.

---

****# 🧰 Tools Used****

* Wazuh

* Ubuntu Server

* Linux PAM

* Auditd

* `useradd`

* `userdel`

* `passwd`

* `ausearch`

* Linux authentication logs

* Wazuh Dashboard

---

**# 📸 Evidence & Screenshots

**## 1. New User and New Group Added**

![New User and New Group Added](new%20user%20and%20new%20group%20added.png)

This screenshot shows the Wazuh events generated when the `testuser` account and its associated group were created.

---

**## 2. New User Creation Details**

![New User Created Detail](new%20user%20created%20detail.png)

This screenshot shows the detailed Wazuh event for the newly created user.

****Detection:****

\- Rule ID: `5902`

\- Level: `8`

\- Description: `New user added to the system.`

---

**## 3. New Group Added**

![New Group Added Log](new%20group%20added%20log.png)

This screenshot shows the Wazuh detection for the group created along with the user account.

****Detection:****

\- Rule ID: `5901`

\- Level: `8`

\- Description: `New group added to the system.`

---

**## 4. User and Group Deleted**

![User and Group Deleted Log](user%20and%20group%20deleted%20log.png)

This screenshot shows the Wazuh event generated after deleting the test account and its associated group.

****Detection:****

\- Rule ID: `5903`

\- Level: `3`

\- Description: `Group (or user) deleted from the system.`

---

**## 5. Deleted User Log Details**

![Deleted User Log](deleted%20user%20log.png)

This screenshot shows the detailed Wazuh event generated by the `userdel` process.

****Event details:****

\- User: `testuser`

\- Decoder: `open-userdel`

\- Process: `userdel`

\- Log: `userdel: delete user 'testuser'`

---

**# 📁 Repository Structure**

```text

Day 6 User Account Creation and Deletion Monitoring/

│

├── README.md

├── deleted user log.png

├── new user and new group added.png

├── new user created detail.png

├── new group added log.png

└── user and group deleted log.png

```

**# ⚠️ Disclaimer****

This activity was performed in an ********isolated, controlled SOC lab environment****** for cybersecurity monitoring and defensive learning purposes.

The account creation and deletion activities were authorized test actions and were not performed against unauthorized systems.