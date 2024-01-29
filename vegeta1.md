OffSec <br>
Vegeta1
---

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d7fe5f14-a025-42da-9740-5bceac01269c)


# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/3744e87e-ba1a-4bd9-b02a-a4aef549fd33)
 

Using gobuster to scan for directories:
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/88e391a4-6791-40a7-96fe-86b07baebe7e)

/robots.txt shows:

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/61494802-40ba-4783-9bab-b7d65031deac)
 
When visiting that directory there’s an html file.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/8237205b-8dc0-4bc9-a572-e4e52bc629a1)

That html looks blank but in the source code at the bottom is some base64 encoded text. <br>
I used wget to download the file and used strings to make the encoded text easier to copy.
```
# downloading the file
$ wget hxxp://<machine_IP>/find_me/find_me.html

# using strings to get the text
$ strings find_me.html

# after coping the text, now decoding the text
$ base64 -d <filename>
```
It returned some more encoded text, so I tried using base64 to decode that too and at the top of the decoded text it said PNG. <br>
I saved that output to a file with the PNG extension, viewed the directory I was in via the GUI and a QR code showed up as the output!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/816cf9c5-76f9-4966-99f9-242455019bcd)
 
When scanning the code there was some text that showed a password!  <br>
That made me think that Vegeta was the username.. I tried to SSH into the machine with the found credentials but that didn’t work. <br>

# Initial foothold:
I got stuck for a little bit after that and thought I must need to enumerate more, somehow I’m missing something. <br>
Since it’s a Dragon Ball Z reference room, I made a wordlist related to that anime and ran it in gobuster.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/d719154b-c780-4d74-9056-b6e6a4e3b1f3)
 
YES! Visiting the /bulma directory there was an audio file. Must mean there’s something hidden in it.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/8a97c5fe-bf23-44eb-9ed8-6c41623fa802)
 
I used an online audio decoder to check for a message in that file and found one!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/a5217f79-f95a-4006-82b4-487514792cb9)
 
A username and password in that file, awesome!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/0613f708-1c4f-4c78-8a90-fc4e03a3ed33)
 
I’ll grab the user flag in this user’s home directory.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/ee6949ce-8e8a-4f74-94f6-d1b97d859eb4)
 
# Exploit:
The .bash_history file looks to not delete the output so let me go ahead and view that.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/112ffed5-3ca5-44bc-99d1-d8e4ae9e90f3)
 
Looks like this user used perl to make a salted password then used that in adding a new user to the /etc/passwd file. <br>
If this user can do that then that means I have write permissions on the /etc/passwd file..!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/38bbb8f3-b8b5-4613-b08c-5db4784a0a18)
 
I do!
Since Tom doesn’t exist in that file anymore, let me go ahead and try to add that user in there using the commands from the .bash_history file.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/93c21f85-5cd0-4770-b21b-31b5ecbef9e3)


# Privilege Escalation:
Tom is now created and confirmed in the /etc/passwd file, let’s see if I can escalate to that user.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/96655fdc-9f00-4337-afce-e85236c0226e)
 
Success, escalated to the created user with root privileges! <br>
Time to go ahead and grab the root flag.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/9f5b0056-db40-4acf-9aa0-b7ba61c5531e)

# Reporting: 
•	Ensure correct permissions are set for users/groups and access to files/binaries on system are checked regularly. <br>
•	Turn off history logging for the current session and/or make sure to clear it at the end of the session so that valuable commands can’t be recalled.


 
