# Windows Security Monitoring with Splunk
## Detecting Brute Force Attack, Persistence & Privilege Escalation

---

## Project Overview

This project demonstrates how Splunk Enterprise can be used to investigate a Windows security incident by correlating Windows Security Event Logs.

During the investigation, an external attacker successfully performed a brute-force attack against the Administrator account, gained access to the system, created a hidden backdoor account for persistence, and elevated privileges.

The objective of this investigation was to reconstruct the attack timeline using SPL queries and Windows Event IDs.

---

## Skills Demonstrated

- Windows Security Monitoring
- Splunk Enterprise
- SPL (Search Processing Language)
- Windows Event Log Analysis
- Threat Hunting
- Incident Investigation
- IOC Identification
- Attack Timeline Reconstruction

---

# Environment

SIEM Platform
- Splunk Enterprise

Log Source
- Windows Security Logs

Index
```
dc01_windows_security_logs
```

---

# Windows Event IDs Used

| Event ID | Description |
|----------|-------------|
|4624|Successful Logon|
|4625|Failed Logon|
|4720|User Account Created|
|4732|User Added to Local Administrators Group|
|4647|User Logoff|
|4634|Logoff Completed|

---

# Investigation Process

## Step 1 – Identify Available Security Events

First, all Windows security logs were loaded into Splunk.

SPL

```spl
index=dc01_windows_security_logs
```

This returned approximately 400 Windows Security Events.

---

## Step 2 – Identify Failed Logon Attempts

To investigate authentication failures, Event ID **4625** was filtered.

SPL

```spl
index=dc01_windows_security_logs EventCode=4625
| table TimeCreated IPAddress EventCode LogonType ComputerName TargetUserName
```

### Findings

- Multiple failed logins
- Username targeted:
    Administrator
- Source IP

```
203.0.113.50
```

This strongly indicated a brute-force attack.

---

## Step 3 – Verify Successful Authentication

After identifying repeated failed logins, successful logins were investigated.

SPL

```spl
index=dc01_windows_security_logs EventCode=4624 "Administrator"
| table TimeCreated IPAddress EventCode LogonType ComputerName TargetUserName
```

### Findings

A successful login occurred immediately after multiple failures.

Source IP remained:

```
203.0.113.50
```

This confirmed the brute-force attack eventually succeeded.

---

## Step 4 – Track Attacker Activity

The attacker IP was searched across all security logs.

SPL

```spl
index=dc01_windows_security_logs 203.0.113.50
| table TimeCreated IPAddress EventCode LogonType ComputerName TargetUserName
```

This exposed the complete attacker timeline.

Observed Event IDs:

- 4625
- 4624
- 4720
- 4732

---

## Step 5 – Detect Persistence

Windows Event ID

```
4720
```

was observed.

Meaning:

New User Account Created

New account:

```
backup_admin
```

This indicates the attacker established persistence by creating a backdoor user.

---

## Step 6 – Detect Privilege Escalation

Windows Event ID

```
4732
```

was identified.

Meaning:

User Added to Local Administrators Group

This indicates the attacker elevated privileges after gaining access.

---

# Attack Timeline

| Time | Activity |
|-------|----------|
|09:55|Multiple failed Administrator logins|
|09:58|Successful Administrator login|
|09:58|Backdoor account (backup_admin) created|
|10:00|Privileges elevated|

---

# Indicators of Compromise (IOCs)

Attacker IP

```
203.0.113.50
```

Compromised Account

```
Administrator
```

Backdoor Account

```
backup_admin
```

Windows Event IDs

```
4625
4624
4720
4732
```

---

# MITRE ATT&CK Mapping

| Technique | MITRE ID |
|-----------|----------|
|Brute Force|T1110|
|Valid Accounts|T1078|
|Create Account|T1136|
|Account Manipulation|T1098|

---

# Investigation Summary

The investigation revealed a complete attack chain:

1. External brute-force attack
2. Successful Administrator login
3. Creation of a hidden backdoor account
4. Privilege escalation
5. Persistence established

By correlating multiple Windows Event IDs using Splunk SPL, the full attack timeline was successfully reconstructed.

---

# Key Learnings

- Event correlation is essential during investigations.
- Failed logons should always be correlated with successful logons.
- User creation events often indicate persistence.
- Administrator group modifications are high-risk indicators.
- Splunk SPL enables efficient incident reconstruction.

---

## Credits

Special thanks to **Rajneesh Gupta (Haxcamp)** for designing and mentoring this security monitoring lab, which provided valuable hands-on experience in Windows Security Monitoring and Splunk-based threat hunting.
