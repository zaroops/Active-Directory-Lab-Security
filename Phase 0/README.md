# Phase 0 – Active Directory Baseline & Intentional Misconfigurations

## Objective

The goal of Phase 0 in this project is to build a **realistic Active Directory (AD) environment** with proper logging and **intentional, explainable misconfigurations**. This phase establishes a clean baseline before any attacks are performed and mirrors common weaknesses found in small-to-mid sized enterprise environments. This includes My Active Directory Splunk + Sysmon SIEM SOC Simulation lab (see active directory lab setup repository)

No offensive tooling is used during this phase.

---

## Lab Environment

| System | Purpose |
|------|--------|
| Windows Server 2022 | Domain Controller (AD DS + DNS) |
| Windows 10 | Domain-joined workstation |
| Kali Linux | Attacker machine (idle in Phase 0) |

**Domain Name:** `homelb.local`

---

## Active Directory Structure

### Organizational Units (OUs)

homelb.local
├── Users
├── Groups
├── Workstations
└── IT


This OU structure follows basic Active Directory hygiene and supports delegated administration. It supports the structure necessary for my lab setup as well as possible attack vectors within offensive tooling phases of this lab. 

---

## User Accounts

| Username | Purpose |
|--------|--------|
| zach | Standard domain user |
| itadmin | IT support staff |
| hruser | HR user |
| sales1 | Sales user |
| svc-backup | Service account (intentional misconfiguration) |

---

## Security Groups

| Group | Purpose |
|-----|--------|
| IT-Support | Help desk / IT permissions |
| HR | HR users |
| Sales | Sales users |
| Workstation-Admins | Local admin access on workstations |

Group-based permissions are used instead of assigning rights directly to users, reflecting real-world best practices. Individual user permission assignment is defined as unhealthy and an attack-prone business practice.

---

## Logging & Visibility

## Logging & Monitoring Setup

To provide centralized visibility and detection capability, a SIEM had to be set up for accurate log ingestion. Sysmon and Splunk were deployed across the environment prior to any attack activity.

### Splunk
Splunk Enterprise (Free) is utilized as the central log aggregation platform as my choice of SIEM for this enterprise lab setup. In shorter terms, Splunk turns raw data into digestable intelligence logs. Splunk does the following:
- Collects logs from multiple machines
- Indexes and stores them
- Makes them searchable
- Allows detection engineering
- Enables dashboards, alerts, and investigations

Splunk Universal Forwarders were installed on both Windows systems and configured to forward:

- Windows Security logs
- Sysmon logs
- PowerShell operational logs

The purpose of Splunk Universal Forwarder is that of an enterprise level log aggregator. It collects local logs (Event Viewer, Sysmon, PowerShell, etc.) and forwards them to Splunk Enterprise. Log ingestion was validated prior to attacks to establish a clean baseline.

![Splunk Sysmon Logs](screenshots/splunk-sysmon-logs.png)

Splunk Universal Forwarder was installed on:
- Domain Controller
- Windows 10 workstation

The following logs are forwarded:
- Windows Security logs
- Sysmon logs
- PowerShell activity

This ensures visibility **before** any attacks occur.

---

### Sysmon
Sysmon was installed on both the Domain Controller and the Windows 10 workstation using a hardened configuration (SwiftOnSecurity). The purpose of Sysmon is to give me EDR level detection via logs on windows processes. Windows logs are generally very limited in their information, so his enabled detailed telemetry including:

- Process creation
- Network connections
- LSASS access attempts
- Privilege escalation activity
- Process creation (Event ID 1)
- Network connections (Event ID 3)
- File creation (Event ID 11)
- Registry modifications
- Parent/child process relationships
- CMD arguments

Sysmon events were verified via the `Microsoft-Windows-Sysmon/Operational` event log.

![Sysmon Operational Log](screenshots/sysmon-operational.png)

---
## Intentional Misconfigurations

The following misconfigurations were intentionally introduced to support later attack simulations.

---

### 1. Kerberoastable Service Account

**Account:** `svc-backup`  
**Password:** `Backuser123!`  
**Password Expiration:** Disabled  

A Service Principal Name (SPN) was assigned to this user account:
setspn -A MSSQLSvc/backup.homelb.local:1433 homelb\svc-backup

This configuration makes the account vulnerable to Kerberoasting, allowing attackers to request Kerberos service tickets and crack the password offline.

### 2. Excessive Local Administrator Rights

User zach was added to the Workstation-Admins security group, which is a member of the local Administrators group on the Windows 10 workstation.

This reflects a common real-world misconfiguration where standard users are granted elevated rights for convenience.

### 3. Cached Domain Credentials

Multiple domain users were logged into the Windows 10 workstation, leaving cached credentials in memory. This enables credential access techniques later in the lab.

Validation

SPN assignment verified using:

setspn -L homelb\svc-backup


Sysmon service confirmed running

Splunk confirmed receiving Security and Sysmon logs

User and group memberships validated

Phase 0 Outcome

At the completion of Phase 0:
- The Active Directory domain is fully functional
- Logging and monitoring are in place
- Misconfigurations are present but controlled
- The environment is prepared for enumeration and attack simulation in subsequent phases


