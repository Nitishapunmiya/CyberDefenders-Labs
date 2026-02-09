# Packet Maze 

This document contains a structured write‑up of my **Packet Maze** PCAP analysis, including methodology, filters used, questions solved, and key learnings. It is intended to be clear, reproducible, and suitable for GitHub documentation.

---

## Scenario

A company's internal server has been flagged for unusual network activity, with multiple outbound connections to an unknown external IP. Initial analysis suggests possible data exfiltration. Investigate the provided network logs to determine the source and method of compromise.
### Initial Approach

1. Started with the **Statistics tab** in Wireshark
2. Analysed **Capture Details**
3. Reviewed:

   * Protocols used
   * IP addresses
   * Ports involved
4. Identified the **victim IP** based on traffic patterns
5. Observed protocol usage frequency

The **major protocol used was TCP 443**, so analysis began with HTTPS/TLS traffic. Later, **FTP traffic** was also analysed as it was present and relevant.
<img width="1907" height="1008" alt="Screenshot 2026-02-09 222306" src="https://github.com/user-attachments/assets/275e3fda-bc50-4c1f-a409-0940d4d7f2ef" />

<img width="1906" height="998" alt="Screenshot 2026-02-09 223150" src="https://github.com/user-attachments/assets/f4b38976-4d42-4c12-9c95-b1cb4fc12c81" />



---

## Victim Identification

* Victim IP identified: **192.168.1.26**
* Reasoning:

  * High volume of outbound connections
  * Participation in multiple protocols (TLS, DNS, FTP, UDP)

---

## Question‑wise Analysis

### Q1: What is the FTP password?

**Methodology:**

* Filtered FTP traffic
* Analysed FTP command streams
* Identified username and password from plaintext FTP authentication

**Answer:**

```
AfricaCTF2021
```
<img width="1904" height="1016" alt="Screenshot 2026-02-09 225907" src="https://github.com/user-attachments/assets/737ffbf0-5975-49f3-8345-f980bd334c42" />

---

### Q2: What is the IPv6 address of the DNS server used by 192.168.1.26?

**Methodology:**

* Applied filter for `dns` and got dns server IP
* Opened **Statistics → Endpoints**
* Checked **Ethernet / IPv6 entries**

**Answer:**

```
fe80::c80b:adff:feaa:1db7
```
<img width="1907" height="1017" alt="Screenshot 2026-02-09 225943" src="https://github.com/user-attachments/assets/40552991-1c0a-44c7-9c30-79a79970b681" />

---

### Q3: What domain is the user looking up in packet 15174?

**Methodology:**

* Navigated directly to packet **15174**
* Observed DNS query details

**Answer:**

```
www.7-zip.org
```
<img width="1898" height="1013" alt="Screenshot 2026-02-09 234215" src="https://github.com/user-attachments/assets/49b35ed3-bca0-460c-a9c4-0b98cb5edd01" />

---

### Q4: How many UDP packets were sent from 192.168.1.26 to 24.39.217.246?

**Filter Used:**

```
ip.src == 192.168.1.26 && ip.dst == 24.39.217.246 && udp
```

**Answer:**

```
10
```
<img width="1907" height="1020" alt="Screenshot 2026-02-09 234415" src="https://github.com/user-attachments/assets/da179e77-0795-4a29-aa9d-bd46725d91da" />

---

### Q5: What is the MAC address of the system under investigation?

**Methodology:**

* Applied filter:

```
ip == 192.168.1.26
```

* Opened **Statistics → Endpoints → Ethernet**

**Answer:**

```
c8:09:a8:57:47:93
```
<img width="1899" height="1005" alt="Screenshot 2026-02-09 234858" src="https://github.com/user-attachments/assets/cba44b54-0d9f-484a-b916-cd8c69ed7449" />

---

### Q6: What was the camera model name used to take picture `20210429_152157.jpg`?

**Methodology:**

* Applied filter:

```
frame contains "20210429_152157.jpg"
```

* Exported object from HTTP
* Checked file metadata / properties

**Answer:**

```
LM-Q725K
```
<img width="1907" height="1004" alt="Screenshot 2026-02-09 235325" src="https://github.com/user-attachments/assets/3d9e3ade-206d-4fda-b889-36c9364da5d6" />

---

### Q7: What is the ephemeral public key provided by the server during the TLS handshake?

**Session ID:**

```
da4a0000342e4b73459d7360b4bea971cc303ac18d29b99067e46d16cc07f4ff
```

**Filter Used:**

```
tls.handshake.session_id == "da4a0000342e4b73459d7360b4bea971cc303ac18d29b99067e46d16cc07f4ff"
```

**Answer:**

```
04edcc123af7b13e90ce101a31c2f996f471a7c8f48a1b81d765085f548059a550f3f4f62ca1f0e8f74d727053074a37bceb2cbdc7ce2a8994dcd76dd6834eefc5438c3b6da929321f3a1366bd14c877cc83e5d0731b7f80a6b80916efd4a23a4d
```
<img width="1907" height="1022" alt="Screenshot 2026-02-10 001953" src="https://github.com/user-attachments/assets/6a9e4f63-8504-4435-b442-c66b99ee2613" />

---

### Q8: What is the first TLS 1.3 client random used to establish a connection with protonmail.com?

**Methodology:**

* Applied TLS filter
* Identified **TLSv1.3** packets
* Isolated traffic for `protonmail.com`
* Extracted **Client Random** from packet details

**Answer:**

```
24e92513b97a0348f733d16996929a79be21b0b1400cd7e2862a732ce7775b70
```
<img width="1895" height="1010" alt="Screenshot 2026-02-10 002436" src="https://github.com/user-attachments/assets/5e955137-1e1d-4eee-8568-770a7ba61554" />

---

### Q9: Which country is the manufacturer of the FTP server’s MAC address registered in?

**Methodology:**

* Filtered FTP traffic
* Identified FTP server IP
* Retrieved corresponding MAC address
* Performed MAC vendor lookup on google 

**Answer:**

```
United States
```
<img width="1907" height="1012" alt="Screenshot 2026-02-10 002535" src="https://github.com/user-attachments/assets/d1ed3ad6-7762-4c31-ae34-6559aea811a9" />
<img width="1907" height="1014" alt="Screenshot 2026-02-10 002914" src="https://github.com/user-attachments/assets/402662bb-c9e0-4f47-8802-284051645e56" />


---

### Q10: What time was a non‑standard folder created on the FTP server on 20th April?

**Methodology:**

* Analysed FTP directory listing timestamps
* Focused on non‑standard folder entries

**Answer:**

```
17:53 (HH:MM)
```

---

### Q11: What URL was visited by the user and connected to the IP address `104.21.89.171`?

**Status:**

* Analysis performed
* URL identification pending / not captured in provided data

  <img width="1907" height="1009" alt="Screenshot 2026-02-10 003513" src="https://github.com/user-attachments/assets/8967a94f-be51-4650-8db2-1d4b19c49e26" />


---

## Key Learnings

Through this challenge, I gained hands‑on understanding of:

* **Crypto APIs in TLS**
* **HEAD method in HTTP**
* **Ephemeral public keys** and their role in TLS handshakes
* Practical Wireshark filtering strategies
* Correlating IP, MAC, protocol, and application‑layer data

---

## Tools Used

* Wireshark
* MAC Address Lookup (Vendor Identification)
* PCAP Export Objects (HTTP)

---

## Conclusion

This Packet Maze investigation strengthened my network forensics skills, especially in protocol analysis, TLS internals, and FTP traffic inspection. The structured, statistics‑first approach proved effective for identifying victims and narrowing down relevant traffic.

---
## Badge
I successfully completed PacketMaze Blue Team Lab at @CyberDefenders!
https://cyberdefenders.org/blueteam-ctf-challenges/achievements/nitishapunmiya16406/packetmaze/
 

