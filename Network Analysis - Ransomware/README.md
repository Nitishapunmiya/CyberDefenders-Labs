# Network Analysis – Ransomware Incident (ABC Industries)

## Overview

ABC Industries spent weeks preparing a critical tender document for a high‑value project. Just before submission, the organization was hit by a **ransomware attack**, and the final version of the tender document was encrypted. The attackers are believed to be a competitor attempting to sabotage the bid.

The only available evidence for investigation included:

* Network traffic capture (**PCAP**)
* The encrypted tender document
* A ransom note

The objective was to analyze the network traffic, identify the malicious activity, determine the ransomware involved, and recover the encrypted document.

---

## Investigation Approach

The investigation was performed entirely through **network traffic analysis** using **Wireshark**, focusing on identifying suspicious communication patterns, malicious downloads, and file transfers.

---

## Traffic Exploration & Initial Findings

The analysis started with Wireshark’s **Statistics** section:

* Capture File Details
* Protocol Hierarchy
* Conversations

### Key Observations

* **10.0.2.4** was identified as the **compromised internal host**
* **10.0.2.15** showed frequent interaction with the compromised system
* High packet volume and repeated connections indicated abnormal behavior
* Relevant ports used during communication were identified from conversation statistics

The Statistics tab provided a clear high‑level view of traffic flow, allowing quick identification of suspicious hosts.
<img width="1920" height="923" alt="Screenshot_2026-01-22_23_44_00" src="https://github.com/user-attachments/assets/ce0a2bb7-6da9-4fe1-8154-68d94634095c" />
<img width="1920" height="923" alt="Screenshot_2026-01-22_23_44_38" src="https://github.com/user-attachments/assets/45646b9a-6260-4be0-b4b8-d8fff529f716" />


---

## Suspicious External Communication

Deeper packet inspection revealed:

* Continuous HTTP requests from **10.0.2.4** to an external web server
* Malicious web server IP identified as **40.112.72.205**
* Repeated requests with unreachable web responses

Additionally:

* **192.168.55.1** appeared to function as a **DNS server**
* Multiple DNS queries were made to different domains, suggesting command‑and‑control or payload delivery attempts

  <img width="1920" height="923" alt="Screenshot_2026-01-22_23_41_46" src="https://github.com/user-attachments/assets/54bdb2bd-87c8-4550-9a0e-19ad3b1d537d" />
  <img width="1920" height="923" alt="Screenshot_2026-01-22_23_57_38" src="https://github.com/user-attachments/assets/2569bcb9-0240-4d96-b2ee-bd18514e32f9" />



---

## Internal Network Interaction

Further investigation showed:

* **10.0.2.4** initiating ARP requests to locate **10.0.2.15**
* Successful MAC address resolution
* A full **TCP three‑way handshake** between the two hosts

This confirmed active communication and lateral interaction inside the network.

---

## Malware Identification

Packet analysis uncovered a critical HTTP **GET request**:

* File requested: **`safecrypt.exe`**
* Destination port: **8000**

This executable was suspected to be the ransomware payload responsible for encrypting the tender document.

<img width="1920" height="923" alt="Screenshot_2026-01-23_00_03_14" src="https://github.com/user-attachments/assets/469a646e-aef2-464f-ba2b-0b5d517a6e68" />


---

## File Extraction & Malware Analysis

Using Wireshark’s **Export Objects** feature:

* The malicious executable was extracted directly from the PCAP
* An **MD5 hash** was generated for the file
* The hash was submitted to **VirusTotal**
  <img width="1920" height="923" alt="Screenshot_2026-01-23_01_07_04" src="https://github.com/user-attachments/assets/ffa9d78a-b1a0-46b7-8d55-4d16bd1ee2d8" />
<img width="1920" height="1080" alt="Screenshot 2026-01-23 105520" src="https://github.com/user-attachments/assets/bbb07648-e12b-44f9-bbd0-101735660a30" />


### Results

* File marked as **malicious**
* Ransomware family identified as **TeslaCrypt**

---

## Decryption & Recovery

Once TeslaCrypt was confirmed:

* The official **TeslaDecrypt** tool was obtained
* The encrypted tender document was successfully decrypted
* The challenge **flag was recovered**, confirming successful incident resolution

---

## How Statistics Accelerated the Investigation

The **Statistics tab** played a crucial role by:

* Providing instant visibility into active conversations
* Highlighting suspicious IPs and packet volume
* Helping narrow down relevant ports and protocols
* Reducing manual packet‑by‑packet analysis time

This significantly streamlined the investigation process.

---

## Key Learnings & Takeaways

Through this lab, I gained hands‑on experience in:

* Analyzing PCAP files methodically
* Using Wireshark filters for focused investigation
* Identifying ransomware delivery via network traffic
* Exporting malicious files from PCAPs
* Generating and analyzing file hashes
* Correlating network evidence to real‑world attacks

---

## Final Outcome

This investigation successfully:

* Identified the compromised host
* Traced malicious external communication
* Detected ransomware payload delivery
* Confirmed TeslaCrypt ransomware activity
* Recovered the encrypted tender document

A complete ransomware incident was reconstructed **purely from network traffic**, reinforcing the importance of strong network forensics skills in blue‑team operations.

---

## Badge


# Network Analysis – Ransomware Incident (ABC Industries)

## Overview

ABC Industries spent weeks preparing a critical tender document for a high‑value project. Just before submission, the organization was hit by a **ransomware attack**, and the final version of the tender document was encrypted. The attackers are believed to be a competitor attempting to sabotage the bid.

The only available evidence for investigation included:

* Network traffic capture (**PCAP**)
* The encrypted tender document
* A ransom note

The objective was to analyze the network traffic, identify the malicious activity, determine the ransomware involved, and recover the encrypted document.

---

## Investigation Approach

The investigation was performed entirely through **network traffic analysis** using **Wireshark**, focusing on identifying suspicious communication patterns, malicious downloads, and file transfers.

---

## Traffic Exploration & Initial Findings

The analysis started with Wireshark’s **Statistics** section:

* Capture File Details
* Protocol Hierarchy
* Conversations

### Key Observations

* **10.0.2.4** was identified as the **compromised internal host**
* **10.0.2.15** showed frequent interaction with the compromised system
* High packet volume and repeated connections indicated abnormal behavior
* Relevant ports used during communication were identified from conversation statistics

The Statistics tab provided a clear high‑level view of traffic flow, allowing quick identification of suspicious hosts.

---

## Suspicious External Communication

Deeper packet inspection revealed:

* Continuous HTTP requests from **10.0.2.4** to an external web server
* Malicious web server IP identified as **40.112.72.205**
* Repeated requests with unreachable web responses

Additionally:

* **192.168.55.1** appeared to function as a **DNS server**
* Multiple DNS queries were made to different domains, suggesting command‑and‑control or payload delivery attempts

---

## Internal Network Interaction

Further investigation showed:

* **10.0.2.4** initiating ARP requests to locate **10.0.2.15**
* Successful MAC address resolution
* A full **TCP three‑way handshake** between the two hosts

This confirmed active communication and lateral interaction inside the network.

---

## Malware Identification

Packet analysis uncovered a critical HTTP **GET request**:

* File requested: **`safeecrypt.exe`**
* Destination port: **8000**

This executable was suspected to be the ransomware payload responsible for encrypting the tender document.

---

## File Extraction & Malware Analysis

Using Wireshark’s **Export Objects** feature:

* The malicious executable was extracted directly from the PCAP
* An **MD5 hash** was generated for the file
* The hash was submitted to **VirusTotal**

### Results

* File marked as **malicious**
* Ransomware family identified as **TeslaCrypt**

---

## Decryption & Recovery

Once TeslaCrypt was confirmed:

* The official **TeslaDecrypt** tool was obtained
* The encrypted tender document was successfully decrypted
* The challenge **flag was recovered**, confirming successful incident resolution

---

## How Statistics Accelerated the Investigation

The **Statistics tab** played a crucial role by:

* Providing instant visibility into active conversations
* Highlighting suspicious IPs and packet volume
* Helping narrow down relevant ports and protocols
* Reducing manual packet‑by‑packet analysis time

This significantly streamlined the investigation process.

---

## Key Learnings & Takeaways

Through this lab, I gained hands‑on experience in:

* Analyzing PCAP files methodically
* Using Wireshark filters for focused investigation
* Identifying ransomware delivery via network traffic
* Exporting malicious files from PCAPs
* Generating and analyzing file hashes
* Correlating network evidence to real‑world attacks

---

## Final Outcome

This investigation successfully:

* Identified the compromised host
* Traced malicious external communication
* Detected ransomware payload delivery
* Confirmed TeslaCrypt ransomware activity
* Recovered the encrypted tender document

A complete ransomware incident was reconstructed **purely from network traffic**, reinforcing the importance of strong network forensics skills in blue‑team operations.

---

## Badge

<img width="1139" height="842" alt="Screenshot 2026-01-23 231441" src="https://github.com/user-attachments/assets/6561f2bc-cdf5-4a0c-ac5e-008345839e60" />


