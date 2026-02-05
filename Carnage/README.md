# Network Analysis – Carnage Incident (Bartell Ltd)

## Platform

TryHackMe (THM)

---

## Scenario

Eric Fischer from the Purchasing Department at **Bartell Ltd** received an email from a known contact containing a malicious Microsoft Word document. Upon opening the document, Eric accidentally clicked **"Enable Content"**, which executed malicious macros.

Soon after, the SOC team received alerts indicating that Eric’s workstation was making **suspicious outbound connections**. A **PCAP file** was retrieved from the network sensor and provided for investigation.

The objective of this investigation was to analyze the network traffic, identify malicious activity, uncover attacker infrastructure, and understand post‑infection behavior.

---

## Investigation Approach

The investigation was performed entirely through **network traffic analysis** using **Wireshark**, focusing on:

* Identifying suspicious outbound connections
* Analyzing HTTP, DNS, and SMTP traffic
* Detecting malware downloads and command‑and‑control communication
* Correlating indicators using open‑source intelligence

---

## Investigation Methodology

The investigation began with a high-level review of the packet capture using **Wireshark’s Statistics tab**, which provides an immediate overview of network behavior.

### Initial Steps

1. **Statistics → Capture File Properties**

   * Reviewed capture duration and timestamps to understand the investigation window.

2. **Statistics → Protocol Hierarchy**

   * Identified dominant protocols such as HTTP, DNS, and SMTP, indicating possible malware communication and email activity.
   * <img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/0d527268-b4e6-4f52-899a-39f149966121" />


3. **Statistics → Conversations**

   * Analyzed IP-to-IP communication patterns.
   * Identified the **victim host** as:

     * **10.9.23.102**
    
       <img width="1916" height="904" alt="image" src="https://github.com/user-attachments/assets/fe82abec-97ee-479c-8378-f0e0ba11987b" />


This initial triage provided a clear picture of suspicious outbound activity originating from the victim machine.

---

## Findings: Questions, Methods & Answers

This section documents each investigative question, the methodology used to answer it, and the final finding.

---

### 1. First Malicious HTTP Connection

**Question:** What was the date and time for the first HTTP connection to the malicious IP?

**Method:**

* Applied the display filter:

```
http
```

* Reviewed packet timestamps and identified the earliest outbound HTTP request to an external suspicious IP.

**Answer:** `2021-09-24 16:44:38`
<img width="1907" height="895" alt="image" src="https://github.com/user-attachments/assets/8ee305c5-232c-40a9-a961-90491a2f9d76" />


---

### 2. Malicious ZIP File Download

**Question:** What is the name of the zip file that was downloaded?

**Method:**

* Inspected HTTP **GET** requests.
* Observed the requested file name directly in the request URI.

**Answer:** `documents.zip`


---

### 3. Hosting Domain of the ZIP File

**Question:** What was the domain hosting the malicious zip file?

**Method:**

* Expanded the HTTP protocol details in the packet pane.
* Examined the **Host** header field.

**Answer:** `attirenepal.com`
<img width="1905" height="893" alt="image" src="https://github.com/user-attachments/assets/d8c32f57-4063-4354-99b5-a868d8b0007b" />


---

### 4. File Inside the ZIP Archive

**Question:** Without downloading the file, what is the name of the file inside the zip?

**Method:**

* Followed the relevant **HTTP stream**.
* Inspected the response content to identify embedded file names.

**Answer:** `chart-1530076591.xls`
<img width="1904" height="890" alt="image" src="https://github.com/user-attachments/assets/72eba798-432a-48d5-9ae8-b7288b446ad3" />


---

### 5. Web Server Name

**Question:** What is the name of the web server hosting the malicious zip file?

**Method:**

* Reviewed HTTP response headers via **Follow HTTP Stream**.

**Answer:** `LiteSpeed`

---

### 6. Web Server Version

**Question:** What is the version of the web server?

**Method:**

* Analyzed the `Server` header in the HTTP response.

**Answer:** `PHP/7.2.34`

---

### 7. Additional Malicious Domains

**Question:** What three domains were involved in additional malicious downloads?

**Method:**

* Narrowed analysis to the timeframe **16:45:11 – 16:45:30**.
* Applied the display filter:

```
dns && (frame.time >= "2021-09-24 16:45:11" && frame.time <= "2021-09-24 16:45:30")
```

**Answer:**

* `finejewels.com.au`
* `thietbiagt.com`
* `new.americold.com`
<img width="1900" height="898" alt="image" src="https://github.com/user-attachments/assets/ac617600-5815-4442-8ca9-676cb66bf3d1" />

---

### 8. SSL Certificate Authority

**Question:** Which certificate authority issued the SSL certificate to the first domain?

**Method:**

* Followed the relevant **TCP stream**.
* Examined TLS certificate details.

**Answer:** `GoDaddy`
<img width="1907" height="954" alt="image" src="https://github.com/user-attachments/assets/44b8b5dc-484b-48b0-87c5-a2ff9427eb91" />


---

### 9. Cobalt Strike C2 IP Addresses

**Question:** What are the two Cobalt Strike command-and-control IP addresses?

**Method:**

* Identified suspicious IPs using the **Statistics tab**.
* Verified each IP using **VirusTotal (Community tab)**.

**Answer:**

* `185.106.96.158`
* `185.125.204.174`
<img width="1864" height="890" alt="image" src="https://github.com/user-attachments/assets/766fc6ab-0082-4408-bcb5-254351e11590" />

---

### 10. Host Header for First C2 Server

**Question:** What is the Host header for the first Cobalt Strike IP?

**Method:**

* Applied the display filter:

```
ip.addr == 185.106.96.158
```

* Searched for HTTP **GET** requests and reviewed headers.

**Answer:** `ocsp.verisign.com`
<img width="1907" height="902" alt="image" src="https://github.com/user-attachments/assets/8244e507-71ce-4d73-af59-012b6d67faa4" />


---

### 11. Domain of Second C2 Server

**Question:** What is the domain name of the second Cobalt Strike server?

**Method:**

* Applied DNS filter:

```
dns.a == 185.125.204.174
```

* Confirmed association using VirusTotal.

**Answer:** `securitybusinpuff.com`
<img width="1904" height="896" alt="image" src="https://github.com/user-attachments/assets/ade9ce8b-c715-4901-b0af-710ff341766f" />


---

### 12. Post-Infection Communication Domain

**Question:** What is the domain used for post-infection traffic?

**Method:**

* Filtered standard HTTP traffic.
* Identified the first **POST** request and followed the TCP stream.

**Answer:** `maldivehost.net`
<img width="1907" height="852" alt="image" src="https://github.com/user-attachments/assets/a8cc2dc2-dfd8-4fc1-8f3e-2663e685a26f" />


---

### 13. Initial Beacon Data

**Question:** What are the first eleven characters sent to the post-infection domain?

**Method:**

* Inspected the HTTP POST payload.

**Answer:** `zLIisQRWZI9`

---

### 14. C2 Packet Length

**Question:** What was the length of the first packet sent to the C2 server?

**Method:**

* Reviewed packet length in the packet details pane.

**Answer:** `281 bytes`

---

### 15. Server Header of Malicious Domain

**Question:** What was the server header for the post-infection domain?

**Method:**

* Examined HTTP response headers.

**Answer:**

```
Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4
```

---

### 16. External IP Check via API

**Question:** When did the DNS query for the IP check API occur?

**Method:**

* Applied the filter:

```
ip.addr == 10.9.23.102 && dns && frame contains "api"
```

**Answer:** `2021-09-24 17:00:04 UTC`
<img width="1907" height="856" alt="image" src="https://github.com/user-attachments/assets/d1a58043-dd27-46d5-9265-68f4381eb707" />


---

### 17. IP Check Domain

**Question:** What domain was queried to determine the victim’s IP address?

**Method:**

* Reviewed DNS query details from the same packet.

**Answer:** `api.ipify.org`

---

### 18. Malspam Activity

**Question:** What was the first MAIL FROM address observed?

**Method:**

* Filtered SMTP traffic and reviewed MAIL FROM commands.

**Answer:** `farshin@mailfa.com`

---

### 19. SMTP Packet Count

**Question:** How many SMTP packets were observed?

**Method:**

* Applied display filter:

```
smtp
```

* Reviewed packet count in the status bar.

**Answer:** `1439 packets`
<img width="1901" height="900" alt="image" src="https://github.com/user-attachments/assets/ad677027-d8ed-453a-9e49-c0ecc364181d" />


---

## Conclusion

This investigation reconstructed the complete infection lifecycle, from macro-based initial access to Cobalt Strike command-and-control communication and outbound malspam activity. The structured analysis demonstrates how effective PCAP investigation can reveal attacker infrastructure and post-exploitation behavior using only network telemetry.

This investigation successfully reconstructed the full infection lifecycle:

* Initial macro‑based infection via a Word document
* Malicious file downloads from compromised domains
* Communication with **Cobalt Strike** command‑and‑control servers
* Post‑infection beaconing and system profiling
* Evidence of outbound malspam activity

This room provided valuable hands‑on experience in **real‑world PCAP analysis**, reinforcing the importance of network forensics skills in blue‑team and SOC operations.

---

## Tools Used

* Wireshark
* VirusTotal
* TryHackMe Network Analysis Labs

---

## Badge
<img width="1599" height="730" alt="Screenshot 2026-02-03 105955" src="https://github.com/user-attachments/assets/e6e856ff-82b3-4fa4-a2fd-61c7dae1f181" />


