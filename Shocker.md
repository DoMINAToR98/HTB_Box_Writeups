# SHOCKER
### Info:
OS: Linux <br>
IP: 10.10.10.56<br>
Difficulty :  Medium<br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.56``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/K9gSimY.png)
Running gobuster on it gives us user.sh
![alt text](https://i.imgur.com/MjkCd9a.png)
<br>
### Exploitation:
Vulnerability Exploited: Shellshock <br>
Using `http-shellshock.nse` we can gain access to user account <b> Shelly </b>
![alt text](https://i.imgur.com/kWGhIGG.png)
<br>
### Privilege Escalation: <br>
Privilege Escalation Vulnerability: NOPASSWD <br>
Running LinEnum.sh shows us the NOPASSWD vuln using which we can run 
```
sudo /usr/bin/perl -e 'exec "/bin/sh"'
```
to own the the box
<br>
### Gained knowledge:
Shellshock Vulnerability <br>
Gobuster file extensions <br>
Exploiting NOPASSWD 
