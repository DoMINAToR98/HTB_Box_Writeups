# Popcorn
### Info:
OS: Linux<br>
IP: 10.10.10.6<br>
Difficulty : Easy-Medium<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.6
```
gives us the results : <br>
![alt text]()
Via nmap scan we find 2 ports open http and ssh.
We ran dirb and found test.php and /torrent/ dir. 
Going on torrent we found user login system where we can upload files.
<br>
### Exploitation:
Vulnerability Exploited: Torrent Hoster- LFI with image header. <br>
Vulnerability Explanation: Injected php code with current image header and gained a shell while listening on the given port <br>
### Privilege Escalation:
Privilege Escalation Vulnerability: (MOTD ROOT) <br>
Linux PAM 1.1.0 (Ubuntu 9.10/10.04) - MOTD File Tamperi | exploits/linux/local/14339.sh <br>
(DIrty C0w Root) <br>
### Gained knowledge:
MOTD vulnerability on Linux Pam 1.1 for privesc <br>
Shell code File inclusion inside a png file by intercepting the request for user gain <br>
