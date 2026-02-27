

## 📖 Lab Summary

In this lab, I analyzed a PCAP file to investigate a suspicious HTTP-based malware attack.  
The objective was to identify how the attack started, what files were delivered, how the malware executed, and how persistence was established.

---

## 🎯 Objectives

- Identify suspicious network activity  
- Analyze HTTP traffic  
- Investigate downloaded files  
- Reconstruct the attack chain  
- Identify persistence mechanism  

---

## 🧰 Tools Used

- Wireshark  


---

## 🔍 Investigation Process

### 1️⃣ Protocol Analysis

Navigated to:

Statistics → Protocol Hierarchy  

Observed two main protocols:

- DNS  
- HTTP  

Since malware is commonly delivered over HTTP, I focused on HTTP traffic.

---

### 2️⃣ HTTP Traffic Filtering

Applied the filter:

http

Findings:

- Only 4 HTTP packets  
- Communication with direct IP address  
- Port 222 used (non-standard HTTP port)  

This behavior was suspicious.

---

### 3️⃣ Suspicious File Download – xlm.txt

Found this HTTP request:

GET /xlm.txt HTTP/1.1  
Host: 45.126.209.4:222  

Server responded:

HTTP/1.1 200 OK  

This confirms that the file was successfully downloaded.

---

### 4️⃣ Analysis of xlm.txt (Stage 1 Loader)

The file:

- Executes PowerShell  
- Hides the PowerShell window  
- Bypasses execution policy  
- Downloads another file  

This means `xlm.txt` acts as a first-stage loader.

---

### 5️⃣ Analysis of mdm.jpg (Stage 2 Payload)

The file `mdm.jpg` looks like an image but is not a real image.

Findings:

- Contains obfuscated PowerShell  
- Contains hex-encoded data  
- Converts hex into binary  
- Loads the binary directly into memory  

No executable file is written to disk.

This technique is known as fileless malware execution.

---

## 🧠 Complete Attack Flow

1. Victim downloads `xlm.txt`  
2. `xlm.txt` runs hidden PowerShell  
3. PowerShell downloads `mdm.jpg`  
4. `mdm.jpg` converts hex to binary  
5. Malware loads into memory  
6. Malware begins execution  

---

## 🔁 Persistence Mechanism

The malware creates:

- Conted.ps1  
- Conted.bat  
- Conted.vbs  

It then creates a Scheduled Task:

Task Name: Update Edge  
Runs every 2 minutes  

This ensures the malware runs repeatedly and survives reboot.

---

## 🚨 Indicators of Compromise (IOCs)

IP Address: 45.126.209.4  
Port: 222  
Files: xlm.txt, mdm.jpg  
Scheduled Task: Update Edge  

---

## ✅ Conclusion

This was a multi-stage fileless malware attack delivered over HTTP.

The attacker:

- Used HTTP on a non-standard port  
- Delivered staged payloads  
- Executed malware directly in memory  
- Established persistence via Scheduled Task  

This lab improved my understanding of network-based malware investigation and real-world attack reconstruction.
