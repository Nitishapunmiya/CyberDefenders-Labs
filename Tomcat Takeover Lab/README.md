## 1. Incident Overview

The SOC team detected suspicious activity on an **Apache Tomcat web server** hosted within the company’s intranet. The alert was raised after abnormal web traffic patterns were observed, indicating a possible compromise of the web server.

To investigate the incident, the network team captured network traffic and provided a **PCAP file** for forensic analysis. The objective was to determine how the attacker gained access to the Tomcat server, what actions were performed post-compromise, and whether command execution or persistence mechanisms were established.

---

## 2. Investigation Objectives

* Identify suspicious web traffic targeting the Tomcat server
* Detect reconnaissance or brute-force attempts against web resources
* Determine how administrative access was obtained
* Identify evidence of malicious file upload or command execution
* Confirm the presence of reverse shell or command-and-control activity

---

## 3. Tools Used

* **Wireshark** – Packet analysis, filters, custom columns, and statistics


---

## 4. Investigation & Analysis

### Step 1: Web Traffic Identification

Based on the scenario indicating a web server issue, the investigation began by filtering **HTTP traffic** in Wireshark:

```
http
```

This helped isolate all web-related packets for focused analysis.

**Observation:**

* Initial traffic appeared legitimate and normal
* Requests were consistent with standard web browsing behavior

---

### Step 2: Detection of Reconnaissance Activity

While scrolling through the HTTP traffic, multiple repeated **404 Not Found** responses were observed.

Further inspection of the **User-Agent** header revealed:

```
User-Agent: gobuster
```

**Finding:**

* The attacker used **Gobuster**, a web content brute-forcing tool
* This confirms a **reconnaissance and discovery phase**, where the attacker enumerated directories and resources on the Tomcat server

---

### Step 3: Unauthorized Access and File Upload

After successful enumeration, suspicious **HTTP POST requests** were identified.

**Observations:**

* POST requests were used to upload malicious content
* The attacker successfully gained access to the **Tomcat administrator interface**

This indicates a compromise of **administrative credentials** or exploitation of weak authentication controls.

---

### Step 4: TCP Stream Analysis and Reverse Shell

To understand post-compromise activity, the related **TCP streams** were followed.

**Findings:**

* The TCP stream revealed a **reverse shell payload**
* The attacker established interactive command execution on the compromised server

This confirms **remote code execution** and active **command-and-control (C2)** communication.

---

## 5. Attack Summary

* HTTP traffic analysis revealed suspicious activity
* Directory brute-forcing detected using **Gobuster**
* Multiple 404 responses indicated reconnaissance attempts
* Malicious HTTP POST request used to upload payload
* Attacker gained **Tomcat administrator access**
* Reverse shell established via TCP stream

---

## 6. Conclusion

The investigation confirmed that the Apache Tomcat server was compromised through a web-based attack. The attacker performed reconnaissance using Gobuster, gained administrative access, uploaded a malicious payload, and successfully established a reverse shell for remote command execution.

This lab highlights the importance of:

* Monitoring web server logs and HTTP traffic
* Detecting abnormal User-Agent strings
* Securing administrative interfaces
* Inspecting TCP streams for reverse shell indicators

---

