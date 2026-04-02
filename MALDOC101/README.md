# MalDoc101 Lab – Static Malware Analysis Documentation

## 🧠 Overview

The **MalDoc101 Lab** focuses on analyzing a malicious Microsoft Office document without executing it. The objective is to perform **static analysis** to extract meaningful Indicators of Compromise (IOCs), understand macro behavior, detect obfuscation, and identify the malware family involved.

This lab simulates a real-world phishing attack where a malicious Word document is used as the initial infection vector.

---

# 📌 What is a Macro?

A **macro** in Microsoft Office is a small program written in **VBA (Visual Basic for Applications)** that automates tasks.

Macros can:

* Automate formatting
* Perform calculations
* Execute commands
* Launch external programs

⚠️ Attackers abuse macros to execute malicious code when a document is opened.

---

# 🛠 Tools Used in This Analysis

## 1️⃣ oledump

Used to:

* Inspect internal streams of Office documents
* Identify macro-containing streams
* Extract compressed VBA code

Command example:

```bash
python3 oledump.py sample.bin
```

Indicators:

* **M (uppercase)** → Stream contains attributes + compressed VBA macro code
* **m (lowercase)** → Stream contains attributes only (no executable code)

---

## 2️⃣ olevba (from oletools)

Used to:

* Extract VBA macro code
* Detect AutoExec triggers
* Identify suspicious keywords
* Detect encoded strings

Command example:

```bash
olevba sample.bin
```

---

# 🔍 Question-Based Analysis

---

## Q1: Multiple streams contain macros. Provide the highest one.

Using `oledump`, multiple streams were identified with macro indicators.

The stream with the highest macro number was:

**Stream 16**

This stream contained compressed executable VBA macro code.

---

## Q2: What event is used to begin macro execution?

Using `olevba`, the AutoExec trigger was identified as:

**Document_Open**

This event automatically executes when the document is opened, requiring no user interaction.

---

## Q3: What malware family was this document attempting to drop?

Steps performed:

1. Generated SHA256 hash using:

   ```bash
   sha256sum sample.bin
   ```
2. Submitted hash to VirusTotal

Result:

The document was associated with the **Emotet** malware family.

Emotet is a banking trojan turned malware loader used to deploy ransomware and additional payloads.

---

## Q4: What stream stored the base64 encoded string?

Using `oledump`, stream:

**Stream 34**

contained a large obfuscated base64 string.

---

## Q5: What is the name of the user-form?

A user-form was identified in:

`Macros/VBA/roubhaol`

User-form name:

**roubhaol.frm**

---

## Q6: What value was used to pad/obfuscate the base64 string?

The macro used the following delimiter to split the encoded string:

```
2342772g3&*gs7712ffvs626fq
```

This string was repeatedly inserted to break the base64 sequence and evade detection.

---

## Q7: What program was executed by the base64 string?

After:

* Removing the obfuscation string
* Decoding the cleaned base64 data

The decoded command revealed:

```
powershell -e
```

This indicates execution of an encoded PowerShell command.

---

## Q8: What WMI class was used to launch the trojan?

The decoded PowerShell script referenced:

```
Win32_Process
```

Specifically:

```
Win32_Process::Create
```

This WMI class was used to create a new process and launch the payload stealthily.

---

## Q9: First FQDN contacted to download the trojan?

The decoded script contacted multiple domains.

First FQDN identified:

```
haoqunkong.com
```

---

# 🔐 Obfuscation Technique Observed

The attacker:

* Inserted junk delimiter inside base64
* Used VBA `Split()` function to remove it
* Decoded string dynamically
* Executed PowerShell

This is a common macro-based malware delivery technique.

---

# ⚔️ Full Attack Chain (Top-Level View)

```
User opens malicious document
        ↓
Document_Open macro auto-executes
        ↓
Macro removes obfuscation delimiter
        ↓
Base64 string decoded
        ↓
PowerShell executed
        ↓
WMI Win32_Process creates process
        ↓
Trojan downloaded from external domain
        ↓
Emotet infection
```

---

# 🎯 Key Learning Outcomes

This lab demonstrates:

* Understanding CFB (Compound File Binary) format
* Stream-based analysis using oledump
* Macro inspection using olevba
* Detection of AutoExec triggers
* Base64 obfuscation analysis
* Living-Off-The-Land (LOTL) techniques
* WMI-based process execution
* Threat intelligence correlation via hash lookup

---

# 🛡 Why This Matters for Blue Teams

SOC analysts often receive suspicious Office documents.

Instead of executing them, analysts must:

* Extract macro logic
* Identify obfuscation
* Determine execution method
* Extract IOCs
* Identify malware family
* Produce an incident report

This lab simulates a real-world phishing-based malware delivery scenario.

---

# ✅ Conclusion

The MalDoc101 document was a malicious macro-enabled Office file that:

* Used Document_Open AutoExec trigger
* Contained obfuscated base64 payload
* Executed PowerShell
* Leveraged WMI Win32_Process
* Downloaded Emotet trojan from external domain

This exercise strengthened skills in static malware analysis and macro-based threat investigation.

---

**Author:** [Your Name]
**Lab Platform:** CyberDefenders
**Tools Used:** oledump, olevba, sha256sum, VirusTotal

