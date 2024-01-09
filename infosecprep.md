OffSec <br>
InfosecPrep
---

![image](https://github.com/xocybersec/OffSec/assets/91302698/7c6db55e-558c-43b6-a342-613907ed0f6b)


# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each. <br>
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec/assets/91302698/0bc1e403-af0d-4242-bd5e-81747b3f9cfc)

# Vulnerability assessment:
There’s a robots text file shows another text file that’s available, <i>secret.txt</i>. <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/dab24cfe-f8e8-4bb7-846f-9edadaeb9639)

A base64 encoded text file. <br>
Copy and pasted the text in a file on my local machine and decoded with base64: <br>
```
$ base64 -d code.txt > decoded.txt
```
Turns out to be a SSH private key! <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/3e8291ff-d538-4114-9397-110eedec58ea)

Upon visiting the webpage, there’s a blog entry that tells me the only user on the box! <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/c0cf266c-b82d-45a8-80f6-83977ec426a7)

That means it’s time to get a SSH session! <br>
```
$ ssh oscp@<machine_IP> -i rsa.txt
```
![image](https://github.com/xocybersec/OffSec/assets/91302698/901f3d62-4007-462a-958d-8629a6f85094)

Here are the files in the user’s home directory. <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/f8ab97ec-b541-4510-99b3-a2307fbbaf21)

Here are the contents of the ip file: <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/f72d97d8-90a9-4143-b7db-71a15437e05a)

The text file local.txt gives the user flag. <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/4f6b5f30-e082-45af-a893-58666c303880)

# Exploit:
I checked the cronjobs, no luck. I couldn’t run `sudo -l` because I didn’t have the password of the user. <br>
My next step was to see what types of files I could run with higher privileges with the SUID bit set. <br>
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/OffSec/assets/91302698/14672d94-96d0-4422-a630-3c882750b9e7)

Looks like user oscp can run the bash binary with higher privileges, perfect! <br>
I went to gtfobins[.]net to look up a quick command to use to exploit that binary. <br>
```
$ /usr/bin/bash -p
```
Now that I escalated my privileges to root, I’ll grab the root flag. <br>

![image](https://github.com/xocybersec/OffSec/assets/91302698/642d4d1e-908c-4059-b25c-3017864f24af)

# Reporting: <br>
Be sure to not have files accessible to the internet that contain private information. <br>
Use a strong encoding method or multiple encoding methods to hide contents. <br>
Have all SSH private keys passphrase protected.







