

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

<img width="1907" height="1005" alt="Screenshot 2026-02-27 092613" src="https://github.com/user-attachments/assets/ca80d628-5cd2-496f-8157-d6d453894126" />
<img width="1898" height="1010" alt="Screenshot 2026-02-27 092722" src="https://github.com/user-attachments/assets/666d4924-a896-48ec-b3fe-aca639fc5c7b" />



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

## 🔎 Threat Intelligence & Hash Analysis

After extracting the payload, I performed additional validation using threat intelligence tools.

### 1️⃣ Hash Generation

- Extracted the malicious payload
- Generated SHA256 hash using CyberChef
- Used the hash for reputation analysis
<img width="1798" height="1012" alt="Screenshot 2026-02-27 102343" src="https://github.com/user-attachments/assets/83c1e01d-3843-4807-bc64-4303e35404fe" />


### 2️⃣ VirusTotal Analysis

- Searched the SHA256 hash on VirusTotal
- Observed multiple antivirus detections
- Identified the malware family classification
- Reviewed behavior and threat score

This confirmed that the file was malicious.
<img width="1901" height="894" alt="Screenshot 2026-02-27 102500" src="https://github.com/user-attachments/assets/5688bb55-47d3-4812-9a31-43f714139c3e" />
<img width="1907" height="881" alt="Screenshot 2026-02-27 102545" src="https://github.com/user-attachments/assets/e25ec160-8146-4e58-b7ee-514f6ebaf3ee" />

### 3️⃣ IP Reputation Check

I also investigated the attacker IP address:

45.126.209.4

Using public threat intelligence sources, I checked:

- Reputation score  
- Malicious activity reports  
- Associated domains  
- Previous abuse reports  

The IP showed suspicious reputation indicators, supporting the malicious conclusion.
<img width="1408" height="826" alt="Screenshot 2026-02-27 101919" src="https://github.com/user-attachments/assets/301fd33f-f734-41cc-a8a5-55d98c72c99c" />

---

This step helped validate the malware using external intelligence sources and strengthened the investigation.

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

---
## Badge
<a href="https://cyberdefenders.org/blueteam-ctf-challenges/achievements/nitishapunmiya16406/xlmrat/">
  <img src="https://img.shields.io/badge/-XLMRat_Lab-0A1AFF?&style=for-the-badge&logo=CyberDefenders&logoColor=white" />
</a>
