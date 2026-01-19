## 1. Incident Overview

In **September 2020**, the SOC detected suspicious activity originating from a user workstation within the organization. The alert was triggered due to **unusual SMB protocol activity**, which often indicates lateral movement, remote execution, or credential abuse in Windows environments.

Initial triage suggested a possible **compromise of a privileged account** and misuse of SMB-related services. To investigate further, the network team captured network traffic and provided **multiple PCAP files** for detailed analysis.

The goal of this investigation was to analyze the SMB traffic, identify Indicators of Compromise (IOCs), understand the attacker’s methods, and reconstruct a timeline of the intrusion.

---

## 2. Investigation Objectives

* Identify suspicious SMB connections and communicating hosts
* Determine how authentication was performed and whether privileged credentials were abused
* Analyze post-authentication activity such as file access and remote execution
* Identify persistence mechanisms and privilege escalation attempts
* Reconstruct the attack timeline using packet timestamps

---

## 3. Tools Used

* **Wireshark**


---

## 4. Investigation & Analysis

### Step 1: SMB Traffic Identification and Timeline Analysis

The investigation began by reviewing the **Statistics** section in Wireshark, focusing on:

* Conversations
* Protocol Hierarchy
* Capture File Properties

This helped identify:

* The timeframe during which the traffic was captured
* The dominant protocols in use
* IP addresses involved in SMB communication

**Finding:**

* SMB was the primary protocol used during the suspicious activity
* Multiple SMB sessions were established between the attacker and the victim system

---

### Step 2: SMB Authentication Analysis

SMB session setup packets were analyzed to understand how the attacker authenticated.

**Observations:**

* The attacker successfully authenticated using a **privileged (Administrator) account**
* NTLM-based authentication was observed during the session setup phase

This confirmed that valid credentials were used, indicating either credential compromise or reuse.

---

### Step 3: Post-Authentication Activity and Defense Evasion

After successful authentication, SMB **Tree Connect** and file access operations were examined.

**Observations:**

* The attacker accessed sensitive system files, including **Windows Event Logs**
* Evidence of log access and clearing activity was observed

This behavior strongly indicates **defense evasion**, as the attacker attempted to remove traces of the intrusion.

---

### Step 4: Named Pipes and Remote Execution

Further analysis focused on SMB **named pipe** traffic to detect signs of remote execution.

**Observations:**

* Named pipes were identified within the SMB traffic
* The **`atsvc` named pipe** was used

The `atsvc` pipe is commonly associated with **remote task scheduling and command execution**, confirming that the attacker executed commands remotely on the compromised system.

---

### Step 5: Privilege Escalation and Persistence

In the final phase of analysis, the PCAP revealed activity related to maintaining long-term access.

**Findings:**

* The attacker created a **backdoor account** on the system
* A **non-standard username** was used, likely to evade detection
* A specific executable was leveraged to execute processes remotely

These actions indicate deliberate **persistence and privilege escalation** tactics.

---

## 5. Attack Summary

* Suspicious SMB traffic identified from a compromised workstation
* Successful NTLM authentication using an Administrator account
* Unauthorized access to sensitive system logs
* Log clearing activity indicating defense evasion
* Remote command execution via SMB named pipes (`atsvc`)
* Backdoor account creation for persistence
* Use of a specific executable for remote process execution

---

## 6. Conclusion

The investigation confirmed that the system was compromised through **SMB abuse** using valid privileged credentials. The attacker leveraged SMB authentication, named pipes, and remote execution techniques to escalate privileges, evade detection, and establish persistence.

This incident highlights the importance of:

* Monitoring SMB authentication activity
* Detecting named pipe abuse
* Correlating timestamps to build attack timelines
* Continuous network traffic analysis for SOC operations

---

## Badge

<a href="https://cyberdefenders.org/blueteam-ctf-challenges/achievements/nitishapunmiya16406/packetdetective/">
  <img src="https://img.shields.io/badge/-PacketDetective_Network_Forensics-0A1AFF?&style=for-the-badge&logo=CyberDefenders&logoColor=white" />
</a>
