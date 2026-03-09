

## 1. Executive Summary

This report documents the forensic analysis of a phishing email impersonating **Amazon**, targeting account holder `saintington73@outlook.com`. The email falsely claims the recipient's Amazon account has been locked due to suspicious activity and urges them to click a malicious link to "review" their account within 72 hours.

**This is a CONFIRMED PHISHING email.** It was sent from a fraudulent domain (`zyevantoby.cn`) masquerading as Amazon. The email body was **Base64-encoded** to evade content-based spam filters. Embedded links redirect through Microsoft SafeLinks to the attacker-controlled domain `amazn.zzyuchengzhika.cn` — a typosquatted domain designed to steal Amazon login credentials.


<img width="1920" height="922" alt="Screenshot_2026-03-09_00_13_34" src="https://github.com/user-attachments/assets/91c93d59-291f-41ce-a858-d64d907389b6" />

---

## 2. Sender & Receiver Analysis

| Field | Value |
|---|---|
| **Display Name** | `Amazn` ⚠️ (misspelled — missing letter 'o') |
| **From Address** | `amazon@zyevantoby.cn` |
| **Return-Path** | `amazon@zyevantoby.cn` |
| **Reply-To** | Not specified |
| **To Address** | `saintington73@outlook.com` |
| **Legitimate Amazon Domain** | `amazon.com` |
| **Attacker's Domain** | `zyevantoby.cn` (Chinese .cn TLD) |

### Impersonation & Typosquatting Analysis

- **Display Name Spoofing:** The sender name is `Amazn` — deliberately misspelled. Many email clients show only the display name, making this look like "Amazon" at a glance.
- **Domain Mismatch:** The From address uses `zyevantoby.cn`, a completely unrelated Chinese domain. Legitimate Amazon emails always originate from `@amazon.com` or verified Amazon subdomains.
- **Typosquatting in Body Link:** The redirect URL contains `amazn.zzyuchengzhika.cn` — "amazn" (missing 'o') combined with a random Chinese domain, designed to look like Amazon on quick inspection.
- **.cn TLD Red Flag:** The `.cn` (China) top-level domain is used for attacker infrastructure. Amazon's legitimate domains end in `.com`, `.co.uk`, `.de`, etc.

---

## 3. SPF, DKIM, and DMARC Authentication

| Protocol | Result | Analysis |
|---|---|---|
| **SPF** | ✅ PASS ⚠️ | Passed for `zyevantoby.cn` (attacker domain), **NOT** for `amazon.com`. IP `45.156.23.138` was authorized by attacker's own SPF record. |
| **DKIM** | ✅ PASS ⚠️ | Signature verified for `zyevantoby.cn`. The attacker signed their own email with their own DKIM keys. This proves authenticity of the attacker's domain — not Amazon. |
| **DMARC** | ✅ PASS ⚠️ | Passed with `action=none` on `zyevantoby.cn`. Again, this validates the attacker's domain consistency, not Amazon's identity. |

### 🧠 Why "PASS" Does NOT Mean Safe

This is one of the most important concepts in email security.

> SPF, DKIM, and DMARC **validate whether the email was legitimately sent from the stated sending domain** — they do NOT verify whether that domain has anything to do with the brand being impersonated.

In this case, the attacker:
1. Registered `zyevantoby.cn`
2. Set up proper SPF records
3. Signed emails with DKIM
4. Configured DMARC

All three checks **passed** — because the email really did come from `zyevantoby.cn`. But `zyevantoby.cn` is **not Amazon**. The attacker used these legitimate mechanisms to appear technically trustworthy while impersonating a well-known brand.

---

## 4. Email Header Routing Analysis (SMTP Path)

Reading email `Received` headers is done **bottom-to-top** — the lowest entry is the origin.
```
Raw Header Example:

Received: from mta0.zyevantoby.cn (45.156.23.138) by
  BN1NAM02FT027.mail.protection.outlook.com (10.13.2.141)
  with Microsoft SMTP Server (version=TLS1_2) id 15.20.4308.20
  via Frontend Transport; Tue, 13 Jul 2021 19:14:57 +0000

Authentication-Results: spf=pass (sender IP is 45.156.23.138)
  smtp.mailfrom=zyevantoby.cn; dkim=pass (signature was verified)
  header.d=zyevantoby.cn; dmarc=pass action=none
  header.from=zyevantoby.cn
```

### SMTP Hop Trace

| Hop | Server / Host | Significance |
|---|---|---|
| **1 (Origin)** | `mta0.zyevantoby.cn` — IP `45.156.23.138` | 🔴 **Attacker's mail server.** Email injected here from attacker-controlled Chinese domain. |
| **2** | `BN1NAM02FT027.mail.protection.outlook.com` | Microsoft's inbound EOP gateway. Receives email from attacker for delivery to Outlook victim. |
| **3** | `BN9PR03CA0911.namprd03.prod.outlook.com` | Microsoft internal relay — routes email toward victim's mailbox. |
| **4 (Delivery)** | `AM7PR06MB6609.eurprd06.prod.outlook.com` | Microsoft Exchange delivers email to the victim's inbox. |

### Key Observations

- The email originated from IP `45.156.23.138`, hosted on attacker-registered domain `zyevantoby.cn`
- After the first hop, the email transited entirely through **legitimate Microsoft infrastructure**, making it appear trustworthy
- Attackers use properly configured mail servers on cheap foreign domains (`.cn`, `.ru`, `.xyz`) to pass infrastructure checks while routing through trusted relays
- The `+0900` timezone (Japan Standard Time) in the Date header is suspicious given the `.cn` Chinese domain — attackers often use VPS servers in geographically inconsistent regions

---

## 5. IP & Domain Analysis

| Indicator | Value |
|---|---|
| **Originating IP** | `45.156.23.138` |
| **Sending Domain** | `zyevantoby.cn` |
| **MTA Hostname** | `mta0.zyevantoby.cn` |
| **Phishing Redirect Domain** | `amazn.zzyuchengzhika.cn` |
| **Secondary Social Link** | `www.facebook.com/amir.boyka.7` (wrapped in SafeLinks) |
| **SafeLinks Host** | `emea01.safelinks.protection.outlook.com` |

### Domain Indicators of Compromise

- **`zyevantoby.cn`** — Random-looking string + `.cn` TLD. No association with Amazon. Likely registered cheaply for this specific phishing campaign.
- **`amazn.zzyuchengzhika.cn`** — Classic typosquatting. `amazn` is a deliberate misspelling of `amazon`. This is the credential-harvesting server.
- **`45.156.23.138`** — Should be investigated on VirusTotal, AbuseIPDB, or Shodan. The `45.156.x.x` range is commonly associated with European VPS providers used for bulk malicious activity.
- **SafeLinks Wrapping** — Microsoft SafeLinks rewrites URLs for scanning, but attackers exploit this — the real malicious URL is still accessible via the `originalSrc` parameter.

---

## 6. Message Content Analysis

### Decoded Email Body (via CyberChef — From Base64)
```
Hello Dear Customer,

Your account access has been limited. We've noticed significant changes
in your account activity. As your payment process, We need to understand
these changes better.

This Limitation will affect your ability to:
Pay. Change your payment method. Buy or redeem gift cards. Close your account.

What to do next:

Please click the link above and follow the steps in order to Review The Account,
If we don't receive the information within 72 hours, Your account access may be lost.
```

### Social Engineering Techniques Identified

| Technique | Example from Email |
|---|---|
| **Generic Greeting** | "Hello Dear Customer" — Amazon always uses your registered name |
| **Urgency Tactic** | "within 72 hours" — prevents victim from thinking critically |
| **Fear-Based Language** | "your account access may be lost" — triggers panic |
| **Vague Threat** | "significant changes" — unspecified, impossible to verify |
| **Grammar Errors** | "As your payment process, We need to understand" — non-native writing |
| **Inconsistent Capitalization** | "Limitation", "Review The Account" — unprofessional |

---

## 7. Attachment & Link Analysis

### Base64 Encoding — Content Obfuscation

The email body used `Content-Transfer-Encoding: base64` (visible at line 131 of the raw `.eml` file). This is a common obfuscation technique to **bypass content-based spam filters** that scan for keywords like "account locked" or "click here."
```
Line 131 in raw .eml:
Content-Transfer-Encoding: base64

ICAgICAgICAgICAgICAgICAgICAgICAgIA0KICAgIA0KSGVsbG8gRGVhciBDdXN0
b21lciwNCiA...
```

Both the plain-text part (lines 133–146) and the HTML part were Base64-encoded. Using **CyberChef → From Base64** decoded both successfully.

---

### Primary Credential Harvesting Link
```
Displayed as: "Review Account" button

SafeLinks URL:
https://emea01.safelinks.protection.outlook.com/
  ?url=https%3A%2F%2Famazn.zzyuchengzhika.cn%2F
  %3Fmailtoken%3Dsaintington73%40outlook.com
  &data=04%7C01%7C...
  &reserved=0

Decoded originalSrc:
https://amazn.zzyuchengzhika.cn/?mailtoken=saintington73@outlook.com
```

> 🔴 The `mailtoken` parameter **pre-populates the victim's email address** on the phishing page — a technique to make the fake login page appear personalized and reduce suspicion.

---

### Secondary Social Engineering Link
```
Displayed as: "Amazon Support Team" (styled in orange)
Points to:    https://www.facebook.com/amir.boyka.7
              (wrapped in Microsoft SafeLinks to appear legitimate)
```

This Facebook profile is used to add false legitimacy — possibly as a secondary contact vector for the attacker.

---

### Amazon Logo — CDN Abuse
```
IMG src:
https://images.squarespace-cdn.com/content/52e2b6d3e4b06446e8bf13ed/
1500584238342-OX2L298XVSKF8AO6I3SV/amazon-logo?format=750w&content-type=image%2Fpng
```

The attacker hosted the Amazon logo on **Squarespace CDN** to avoid hosting it on their own suspicious domain. This prevents the image from triggering IP blocklists and makes the email look visually authentic.

---

### HTML Template Analysis

The HTML body used **Mailchimp-style CSS classes** (`mcnTextBlock`, `mcnButtonContent`, `templateContainer`). The attacker likely used a stolen or leaked Mailchimp email template to create a pixel-perfect Amazon-looking email without building it from scratch.

---

## 8. Red Flags Summary

| # | Indicator | Severity |
|---|---|---|
| 1 | Sender display name `Amazn` — misspelled, missing 'o' | 🔴 High |
| 2 | From address `amazon@zyevantoby.cn` — unrelated to `amazon.com` | 🔴 High |
| 3 | Sending domain `zyevantoby.cn` — Chinese TLD, random string | 🔴 High |
| 4 | Originating IP `45.156.23.138` — not Amazon infrastructure | 🔴 High |
| 5 | SPF/DKIM/DMARC passed for attacker's domain, not Amazon | 🔴 High |
| 6 | Generic greeting "Hello Dear Customer" | 🟡 Medium |
| 7 | 72-hour ultimatum — manufactured urgency | 🟡 Medium |
| 8 | Fear language: "account access may be lost" | 🟡 Medium |
| 9 | Grammar errors and inconsistent capitalization | 🟡 Medium |
| 10 | Email body Base64-encoded to evade spam filters | 🔴 High |
| 11 | Primary link leads to `amazn.zzyuchengzhika.cn` (typosquatted) | 🔴 High |
| 12 | `mailtoken` parameter pre-fills victim's email on phishing page | 🔴 High |
| 13 | Secondary link points to random Facebook profile | 🟡 Medium |
| 14 | Amazon logo hosted on Squarespace CDN — avoids IP reputation | 🟡 Medium |
| 15 | HTML template uses Mailchimp CSS — copied/stolen template | 🟡 Medium |
| 16 | Return-Path matches attacker's From domain | 🟡 Medium |
| 17 | Message-ID contains dollar signs — non-standard format | 🟠 Low |
| 18 | Date header shows +0900 timezone despite .cn domain | 🟠 Low |

---

## 9. Technical Learning Section (For Beginners)

### What I Learned From This Email

- **Authentication ≠ Legitimacy:** SPF, DKIM, DMARC passing means the email came from who it claims — but if the "who" is a fake domain, it means nothing. Always check *which domain* the authentication is for.
- **Base64 is not encryption:** It's just encoding. CyberChef can decode it in seconds. Attackers use it to hide content from automated scanners, not to protect data.
- **SafeLinks can be bypassed:** The real malicious URL is still accessible via the `originalSrc` parameter inside the SafeLinks URL.
- **Display Name Spoofing is trivial:** Anyone can set their display name to "Amazon". Always check the actual email address in angle brackets `< >`, not just the name.
- **Mailtoken pre-filling:** Phishing pages pre-fill your email to appear personalized by passing it as a URL parameter.
- **CDN abuse:** Attackers host images on Squarespace, Google, or AWS to make emails look authentic without triggering IP reputation alerts.

---

