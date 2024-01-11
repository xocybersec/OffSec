Offsec <br>
FunboxRookie
---
![image](https://github.com/xocybersec/OffSec/assets/91302698/0afe6dd5-caa1-4191-919d-45080196a3c9)

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each. <br>
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec/assets/91302698/2e12ffd3-6135-4231-9170-e6d7c1d9855e)

# Vulnerability assessment:
Got some nice results; an <b>FTP</b> server with anonymous login enabled, numerous files to check and the <i>robots.txt</i> file. <br>
Notice how the zip file <i>tom.zip</i> only has read permissions, I wonder why only that file has those specific permissions. <br>
Once in the FTP server I found some hidden files.

![image](https://github.com/xocybersec/OffSec/assets/91302698/e43d0bd7-583c-4ed1-88aa-f927bcc48fa6)
![image](https://github.com/xocybersec/OffSec/assets/91302698/aa8d2d8c-a6d0-48ed-8622-614005e129ea)

Since those were hidden, I wanted to check those out first. <br>
<i>.@admins</i> had jumbled text that looked like base64, so I decoded that to find:
```
$ base64 -d .@admins
```
![image](https://github.com/xocybersec/OffSec/assets/91302698/831265b3-ddd4-487e-8803-ddbe8695cf55)
 
The <i>.@users</i> file wasn’t much different.

![image](https://github.com/xocybersec/OffSec/assets/91302698/c4b3c12f-bdc6-4869-b99a-6055fac37cf7)
 
That made me think that maybe Tom was an admin.. <br>
The next file I checked was the <i>welcome.msg</i> file.

![image](https://github.com/xocybersec/OffSec/assets/91302698/f99717be-f636-4ab5-940f-ccfffe61b6ef)

# Initial foothold: 
Now to try to unzip tom’s file.

![image](https://github.com/xocybersec/OffSec/assets/91302698/b20c0bb8-e88f-4cd4-b41b-ed303f3cff03)

Could the passwords be in the disallowed <b>/logs</b> directory? I tried to access that directory but got the dreaded <i>404 error</i>. <br>
No problem, I’ll try to crack that zip file with <b>john</b>.
```
# using the tool zip2john to turn the file into a hash to further crack
$ zip2john tom.zip > tomhash.txt

# now using john to crack the hash
$ john tomhash.txt -w=/usr/share/wordlists/rockyou.txt
```
Success!

![image](https://github.com/xocybersec/OffSec/assets/91302698/3397817e-907b-4aaf-b96d-47d33154316d)
 
Now I can try to SSH into the machine as Tom. <br>
<i>NOTE: Be sure to change the permissions of the rsa file to read only before trying to get a shell.</i> `chmod 400 <filename>`

![image](https://github.com/xocybersec/OffSec/assets/91302698/cd4592b5-5e90-4094-b2ca-5ac12cb91651)
 
The user flag is located in the user’s home directory so I’ll go ahead and snag that. <br>
I wanted to see how I could escalate my privileges so I ran a find command to locate any SUID files,  <br>
only to find out I’m in a restricted shell.

![image](https://github.com/xocybersec/OffSec/assets/91302698/49f2770f-5a22-4b9c-ac6a-6fc8c7ae7634)
 
Well, before moving on I checked for any hidden files/directories.

![image](https://github.com/xocybersec/OffSec/assets/91302698/a37eba8b-1d3d-470a-b562-f2ec1b3d228e)
 
Let’s see what’s in that file and if there is any information I can use.

![image](https://github.com/xocybersec/OffSec/assets/91302698/6ab68276-4b02-4946-a729-adf80e7e8a24)
 
Looks like tom entered his credentials! Time to see if I can log into the mysql server.

![image](https://github.com/xocybersec/OffSec/assets/91302698/7de0cd47-5107-4d9d-9442-08a482d429d2)
 
I’m in, awesome!
After looking through a lot of the tables and not finding anything I went back to the shell I had and found that I went down a rabbit hole.

# Privilege escalation:
All I needed to do, instead of enumerating the sql db, was try the basic way of escalating privileges since I now had tom’s password.
```
$ sudo su
```
Just like that I escalated to root access!

![image](https://github.com/xocybersec/OffSec/assets/91302698/1c5c5573-d698-4896-adbb-ee3446febe22)

Now to grab the root flag.

![image](https://github.com/xocybersec/OffSec/assets/91302698/50ed03d7-0b73-4a82-8cc4-0d25efa3fa36)

# Reporting:
It’s highly advised to disable anonymous login on the FTP server. <br>
Check permissions of users/groups to make sure they’re configured properly. <br>
Make sure files containing sensitive information are accessible by the proper people only <br> 
and password protected with a strong password. 



 

