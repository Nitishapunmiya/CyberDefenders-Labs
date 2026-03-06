# 🔐 BTLO — RDP Brute Force Attack | Lab Writeup



---

## 🧪 Scenario

A system administrator identified a large number of **Audit Failure** events in the Windows Security Event log. The goal was to analyze the logs from an attempted RDP brute force attack and answer investigative questions.

---

## 🛠️ Tools Used

- Linux CLI: `head`, `grep`, `wc`, `sort`, `uniq`

---

## 🔍 Investigation Walkthrough

### Step 1 — Inspect Log Structure
```bash
head BTLO_Bruteforce_Challenge.txt
```

This revealed the log fields: `Keywords`, `Date and Time`, `Source`, `Event ID`, `Task Category`, and the event message.

Key details from the first entry:
- **Keywords:** Audit Failure
- **Source:** Microsoft-Windows-Security-Auditing
- **Event ID:** 4625 — *An account failed to log on*
- **Subject Security ID:** NULL SID
- **Logon Type:** 3 (Network logon — consistent with RDP/SMB brute force)
<img width="1920" height="922" alt="Screenshot_2026-03-05_23_30_13" src="https://github.com/user-attachments/assets/7f871c25-9b4c-4c2f-a84e-2f36e4f6c1a9" />

---

### Step 2 — Count Failed Logon Attempts (Event ID 4625)
```bash
grep "4625" BTLO_Bruteforce_Challenge.txt | wc -l
# Output: 3104

grep "Audit Failure" BTLO_Bruteforce_Challenge.txt | wc -l
# Output: 3103
```

The 1-line difference is due to the header row matching the `4625` search.  
✅ **Total failed logon attempts: 3,103**
<img width="1920" height="922" alt="Screenshot_2026-03-05_23_32_14" src="https://github.com/user-attachments/assets/9570e1f3-f32f-4b17-bd1b-d0e6ecd0dd93" />



---

### Step 3 — Check for Successful Logins (Event ID 4624)
```bash
grep "4624" BTLO_Bruteforce_Challenge.txt
```

**4 successful logon events found:**

| Timestamp | Event ID | Result |
|---|---|---|
| 2/12/2022 6:06:14 AM | 4624 | ✅ Successful logon |
| 2/12/2022 6:11:27 AM | 4624 | ✅ Successful logon |
| 2/12/2022 6:51:38 AM | 4624 | ✅ Successful logon |
| 2/12/2022 7:11:39 AM | 4624 | ✅ Successful logon |

⚠️ **The attacker successfully authenticated 4 times.**
<img width="1920" height="922" alt="Screenshot_2026-03-05_23_34_11" src="https://github.com/user-attachments/assets/a8617ce2-cef6-459e-85b5-5f3fb846f3b4" />


---

### Step 4 — Check for Privilege Escalation (Event ID 4672)
```bash
grep "4672" BTLO_Bruteforce_Challenge.txt
```

Event ID `4672` = *Special Privileges Assigned to New Logon* — this fires when an account with admin-level rights logs on.

All 4 successful logins were **paired with a 4672 event** at the exact same timestamps, confirming the attacker cracked an **administrator account**.

---

### Step 5 — Analyze Source Ports
```bash
cat BTLO_Bruteforce_Challenge.txt | grep "Source Port" | sort | uniq -c
```

Every source port appeared only **once**, across the ephemeral range `49162–49322+`.  
This means the attacker's tool opened a **fresh TCP connection per attempt** — typical behavior of automated brute force tools.
<img width="1920" height="922" alt="Screenshot_2026-03-05_23_49_54" src="https://github.com/user-attachments/assets/24726497-4565-49f4-aff2-1298658a3d74" />

---

## 📊 Key Findings

| Event ID | Count | Type | Meaning |
|---|---|---|---|
| 4625 | 3,103 | ❌ Audit Failure | Failed logon attempts |
| 4624 | 4 | ✅ Audit Success | Attacker gained access |
| 4672 | 4 | ✅ Audit Success | Admin privileges confirmed |

---

## 🕐 Attack Timeline

| Time | Activity |
|---|---|
| ~6:06 AM | First successful login + admin privileges assigned |
| 6:06 AM – 7:22 AM | 3,103 brute force attempts logged |
| 6:11 AM, 6:51 AM | Additional successful logins |
| 7:11 AM | Final successful login with admin privileges |

---

## ✅ Conclusions

- **3,103 failed attempts** in ~76 minutes = ~40 attempts/minute → automated tooling
- Attacker **successfully authenticated 4 times** using an **administrator account**
- Every success paired with Event 4672 confirms **full admin access was obtained**
- Unique source ports per attempt fingerprints the brute force tool behavior

---

## 🛡️ Defensive Recommendations

- Enable **account lockout policies** after N failed attempts
- Restrict RDP to known IPs or require **VPN** for remote access
- Enforce **MFA** on all RDP-exposed accounts
- Create **SIEM alerts** on high volumes of Event ID 4625 in short windows
- Consider **geo-blocking** for RDP if foreign access is unexpected
