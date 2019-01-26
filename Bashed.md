# BASHED
### Info:
OS: Linux<br>
IP: 10.10.10.68<br>
Difficulty : Easy <br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.68``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/9o6lXPy.png)
Running dirb gives us :
![alt text](https://i.imgur.com/NA2rhfH.png)
<br>
### Exploitation: 
We get a shell on that url highlighted above, where we can run LinEnum and we find a NOPASSWD vuln.
<br>
### Privilege Escalation:
<b>User</b>
Uploading reverse-shell.php in uploads and then running the script gets us the shell on local machine. <br>
Now we can use the NOPASSWD as 
```
sudo -u scriptmanager bash
```
and we get the user. <br>
<b>Root</b>
We see 2 files inside the script folder `test.py` and `test.txt`. <br> 
Inserting reverse shell in test.py gives us the shell within a minute. <br>
Explanation: <br>The test.py (user owned) file is generating test.txt (root owned) every minute. 
<br>Therefore adding shellcode within test.py gives us the root shell.
### Gained knowledge:
Exploiting NOPASSWD <br>
