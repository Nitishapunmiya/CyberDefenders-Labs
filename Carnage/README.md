# Carnage – TryHackMe PCAP Analysis

## Platform

TryHackMe (THM)

## Scenario Overview

Eric Fischer from the Purchasing Department at **Bartell Ltd** received a phishing email containing a malicious Microsoft Word document from a known contact. After opening the document, Eric clicked **"Enable Content"**, unknowingly executing malicious macros.

Shortly after, the SOC detected **suspicious outbound connections** from Eric’s workstation. A **PCAP file** was captured by a network sensor and provided for investigation.

**Objective:** Analyze the packet capture to uncover malicious activity, identify infrastructure, and trace post-infection behavior.

---

## Initial Analysis & Methodology

### 1. Statistics Tab Review

The investigation began with Wireshark’s **Statistics** tab to gain a high-level understanding of the traffic:

* Capture duration and timestamps
* Protocol distribution
* Endpoint conversations

This helped quickly establish:

* **Victim IP Address:** `10.9.23.102`
* Suspicious outbound HTTP, DNS, and SMTP activity

### 2. Protocol & Conversation Analysis

By reviewing protocol hierarchies and conversations:

* HTTP traffic indicated file downloads
* DNS traffic revealed suspicious domains
* SMTP traffic suggested malspam behavior

This provided a clear picture of a compromised host communicating with multiple external servers.

---

## Findings & Answers

### First Malicious HTTP Activity

**Question:** What was the date and time for the first HTTP connection to the malicious IP?

* **Answer:** `2021-09-24 16:44:38`

Method:

* Applied `http` filter
* Noted the timestamp of the first malicious HTTP packet

---

### Malicious File Download

**Zip File Name:** `documents.zip`

**Hosting Domain:** `attirenepal.com`

Method:

* Inspected HTTP GET requests
* Expanded the HTTP protocol details to identify the `Host` header

---

### Contents of the Zip File

**File Inside ZIP:** `chart-1530076591.xls`

Method:

* Followed the relevant HTTP stream
* Examined response content without extracting the file

---

### Web Server Information

* **Web Server Name:** LiteSpeed
* **Web Server Version:** PHP/7.2.34

Method:

* Analyzed HTTP response headers via stream follow

---

### Additional Malicious Domains

**Timeframe Analyzed:** `16:45:11 – 16:45:30`

**Domains Identified:**

* `finejewels.com.au`
* `thietbiagt.com`
* `new.americold.com`

Method:

* Applied DNS filter within the narrowed timeframe

---

### SSL Certificate Authority

**Domain:** `finejewels.com.au`

* **Certificate Authority:** GoDaddy

Method:

* Followed TCP stream
* Examined TLS certificate details

---

## Cobalt Strike Command-and-Control

### Identified C2 IP Addresses

Confirmed via VirusTotal (Community tab):

* `185.106.96.158`
* `185.125.204.174`

### Host Header for First C2 IP

* **IP:** `185.106.96.158`
* **Host Header:** `ocsp.verisign.com`

Method:

* Filtered traffic using `ip.addr == 185.106.96.158`
* Searched for HTTP GET requests

---

### Second Cobalt Strike Domain

* **IP:** `185.125.204.174`
* **Domain:** `securitybusinpuff.com`

Method:

* Applied DNS A-record filter for the IP
* Verified via VirusTotal

---

## Post-Infection Traffic

**Post-Infection Domain:** `maldivehost.net`

### Beacon Payload Details

* **First 11 Characters Sent:** `zLIisQRWZI9`
* **First Packet Length:** `281 bytes`

### Server Header

```
Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4
```

Method:

* Followed TCP stream of the first POST request
* Inspected HTTP headers

---

## Victim IP Discovery (External API Check)

**DNS Query Time:** `2021-09-24 17:00:04 UTC`

**Queried Domain:** `api.ipify.org`

Method:

* Applied filter:

```
ip.addr == 10.9.23.102 && dns && frame contains "api"
```

---

## Malspam Activity

**First MAIL FROM Address:** `farshin@mailfa.com`

**Total SMTP Packets Observed:** `1439`

Method:

* Applied `smtp` display filter
* Reviewed SMTP conversation flow

---

## Conclusion

This PCAP analysis revealed a full infection chain:

* Initial macro-based infection via Word document
* Malicious file download from compromised domains
* Communication with multiple Cobalt Strike C2 servers
* Post-infection beaconing and IP discovery
* Evidence of outbound malspam activity

This room significantly improved understanding of:

* Real-world PCAP analysis
* Identifying C2 traffic patterns
* Correlating DNS, HTTP, and SMTP artifacts
* Mapping attacker infrastructure using Wireshark and VirusTotal

---

## Tools Used

* Wireshark
* VirusTotal
* TryHackMe Network Analysis Labs

---

## Author

**Carnage Room – TryHackMe**
PCAP Analysis & Documentation by *[Your Name]*
