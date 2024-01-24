OffSec <br>
Katana
---
 
# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/570c4b87-1e2f-4c0d-a524-598ffff10d1d)
 
Gobuster scan of port 80:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/8bd3b9d1-d7e8-4661-b6ef-8583abd8def6)
 
Scanning /ebook
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/27a2248e-95df-44b2-a4f5-3234020d77dc)

# Vulnerability assessment:
Visiting /ebook page:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d80d2198-e0cb-4d17-9734-9f5fbbc5c6ab)
 
There’s also an Admin Login link on the bottom right of the page. <br>
With the above information it’s a good time to try some SQL injection.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/f4fd48c8-38b2-4c5f-82ff-a3b9523e50bf)
 
I tried a few things and one of them got me access!
```
Name: admin
Pass: ‘ OR 1=1 -- -
```
I tried to edit a few things and upload a reverse shell with no luck. <br>
Checking /ebook/database:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/7c5d3f58-468d-4840-9d0a-333dbf609331)
 
Contents of readme.txt.txt:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/4bff8ab5-5c43-4569-9404-0dfff72d4e49)
 
Looks like I didn’t need to do any SQLi. It’s just basic default credentials. <br>
I also grabbed the SQL file and looked for any extra information.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/52b00b22-8063-41b0-9e3a-e0978bac989f)
 
Cracked that with crackstation[.]net but it was the credential from earlier.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/2f6f3a77-9177-4d4b-a45e-0892a39bbb91)
 
The rest of the directories in /ebook had nothing of interest. <br>
Now I’ll scan port 8088 for directories.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/b9e57580-6836-4a51-8514-42fa6d85413f)

I also tried to find the version of litespeed to check for exploits, which was on the /phpinfo.php page,  <br>
but there weren’t any exploits for this version.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/2f97b355-1982-4f22-8a1d-32c94beea63c)

/docs showed a different version of litespeed, so I looked it up on searchsploit again only to find no results once more.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/20f78dbc-44ba-4539-ba20-7ab0b93772dd)

/protected brings up a login prompt:
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/ce7e04a9-c116-43fa-be5a-22f5814cbd94)

# Initial foothold:
/upload.html actually had a working upload form, /upload.php didn’t, so I uploaded a reverse shell.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d59d9173-8c33-44c8-8db6-453c55dc43a7)
 
It uploaded but not to the current server on port 8088.  <br>
So, I tried the directory at port 80, nothing, then I went to port 8715 since it’s another server.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/094bf862-6169-4c59-926c-4b9c5c2723f1)
 
Reverse shell on my netcat listener! <br>
After a quick poking around I found the first flag.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/87657dd7-fcb3-4210-8ebd-3d5c8cbfece4)

# Privilege Escalation:
Since I didn’t have the user’s password, I couldn’t run sudo -l, next up, I checked cronjobs and found nothing. <br>
The ace up my sleeve was to check to see the capabilities of this user.
```
$ getcap -r / 2>/dev/null
```
That checks in the root directory, recursively, for the user’s capabilities with executable files. <br>
I was hoping to find one with the setuid with extended permissions, meaning I can run the executable with higher privileges.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/1aaa42ec-450b-4f0d-9936-f448024289b7)
 
Bingo! Next, I checked on gtfobins for specific commands to break out of this shell and into root access.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/2d7b7f75-9c0c-45ec-b62b-3f4be0748050)
 
This means I’m about to get root access! <br>
Since I’m able to run python2.7 with the extended privs, the command looked like this.
```
$ /usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/4df80a1a-e248-4d10-9c69-31823acdb083)
 
Time to grab that root flag!
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/dc58fb5c-d37e-40cc-be10-179687a7fad3)

# Reporting:
•	Make sure to remove files from web server that have sensitive information or are not in use anymore. <br>
•	Don’t have the backend database type/version easily accessible or web facing. <br>
•	Ensure correct permissions are set for users/groups and access to files/binaries on system are checked regularly. <br>
•	Have an allow list/checker for all upload forms to prevent unrestricted file upload ability. <br>
•	Use strong cryptographic algorithms when hashing user’s passwords.

