# Legacy
### Info:
OS: Windows XP <br>
IP: <br>
Difficulty : Very Easy <br>
### Enumeration:
```
nmap -sC -sV -oA nmap <ip>
``` 
gives us the results : <br>
![alt text](https://i.imgur.com/GrfzIGP.png)
Running a vuln script scan on port 445:
![alt text](https://i.imgur.com/MN2ms1w.png)
<br>
### Exploitation:
Vulnerability Exploited: MS system vuln to RCE (Ms08-67)<br>
Using msfconsole we can directory exploit this box by that particular vuln.
![alt text](https://i.imgur.com/UmZaqXk.png)
<br>
### Gained knowledge:
Ms08-67 Exploit
