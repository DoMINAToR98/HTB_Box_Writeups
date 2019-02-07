# BOXNAME
### Info:
OS: Linux<br>
IP: 10.10.10.70<br>
Difficulty : Medium <br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.70
``` 
gives us the results : <br>
![alt text](https://i.imgur.com/FQ4FNHv.png)
Running gobuster gave us nothing.
Trying Wfuzz with filtering gives us:
![alt text](https://i.imgur.com/Ikd6Ssg.png)
Adding host for the ip as `git.canape.htb` in the `/etc/hosts` we download the `simpsons.git`(not accessible earlier). <br>
![alt text](https://i.imgur.com/3aDu4g7.png)
<br>
### Exploitation:
Vulnerability Exploited: cPickles (Python Flask App)
Vulnerability Explanation:<br> The Flask app was checking that if the characters were Simpsons characters valid or not. <br>
By including the character name as part of the os command and splitting the pickle data between `character` and `quote`, 
<br> the check will pass and the data will be recombined server-side.
![alt text](https://i.imgur.com/h4Wp8k2.png)
Running this and listening on specific port gives us a shell for www-data.

### Privilege Escalation (User):
Running ps -auxww shows us :
![alt text](https://i.imgur.com/g8yuByq.png)
Privilege Escalation Vulnerability: Apache CouchDB 1.7.0 / 2.x < 2.1.1 - Remote Privilege Escalation<br>
![alt text](https://i.imgur.com/CUzvLJH.png)
https://www.exploit-db.com/exploits/44498 --> running this as :
![alt text](https://i.imgur.com/FeWvzBw.png)
Creates a user for us. <br>
Now we can read the log files:
![alt text](https://i.imgur.com/T8GMybN.png)
We can now login to the user `homer` with the password above.
<br>
### Privilege Escalation (Root):
Privilege Escalation Vulnerability: NOPASSWD
![alt text](https://i.imgur.com/7FhbN7x.png)
Therefore creating a .py file with shellcode and listening on some port
![alt text](https://i.imgur.com/q3nJqln.png)
![alt text](https://i.imgur.com/7heEVBs.png)
gets us root!
### Gained knowledge:
Wfuzz usage with filters <br>
Apache CouchDb RCE <br>
Exploiting insecure Python Pickling <br>
Exploiting Sudo NOPASSWD <br>
