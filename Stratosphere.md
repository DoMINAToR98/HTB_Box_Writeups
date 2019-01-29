# STRATOSPHERE
### Info:
OS: Linux<br>
IP: 10.10.10.64<br>
Difficulty : Medium <br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.64``` <br>
Shows port 22, 80 and 443 opened. <br>
We get the tomcat version via /robots.txt  <br>
Running hydra gives no result on the /manager page.  <br>
Running Gobuster: <br>
![alt text](https://i.imgur.com/et0vOGo.png) <br>
Gives us the `monitoring` page where we get `.action` file. <br>
via stack-overflow
```
It's probably just a URL pattern for firing a Struts action. 
Most people stick with the .do convention, 
but you can make the actions fire on just about anything you want.
Struts and the box name is Stratosphere hmm.
```
### Exploitation:
Vulnerability Exploited: Apache Struts 2.3 < 2.3.34 / 2.5 < 2.5.16 - Remote Code Execution <br>
![alt text](https://i.imgur.com/9HSuSh0.png) <br>
![alt text](https://i.imgur.com/d7rsY9E.png) <br>
We see a mysqld. <br>
```
python struts-pwn.py -u http://10.10.10.64/Monitoring/example/Welcome.action -c "cat /var/lib/tomcat8/db_connect"
```
Gives us the creds for Mysql.
![alt text](https://i.imgur.com/MrpR7Dp.png) <br>
Gives us the ssh creds for user `Richard` and we get the user. <br>
### Privilege Escalation:
`sudo -l` shows one path to a different directory. <br>
We see a `Test.py` running as root. <br>
In python the flow of import libraries is: <br>
![alt text](https://i.imgur.com/hFI9edk.png) <br>
Therefore creating our own `hashlib.py` with the shell code and running the Test.py gives us the root.
![alt text](https://i.imgur.com/LhDHuxH.png)
<br>
### Gained knowledge:
Apache Struts
