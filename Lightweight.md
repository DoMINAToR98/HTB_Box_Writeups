# Lightweight
### Info:
OS: Linux<br>
IP: 10.10.10.119<br>
Difficulty : Medium<br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.119``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/XDtmKQT.png)
We cannot run dirb etc because, it bans us for 5 minutes. <br>
Rather we can just ssh in via our ip address in the box.
<br>
### Exploitation:
Running `getcap` gives us :
![alt text](https://i.imgur.com/zUbCFqg.png) <br>
Via tcpdump :
```
tcpdump -i any -w packets.pcap
```
![alt text](https://i.imgur.com/hGVmxBE.png)
To export the pcap file :
```
scp 10.10.16.5@10.10.10.119:/home/10.10.16.5/temp/all.pcap /root/Desktop/boxes/LightWeight/
```
And viewing those packets in wireshark gives us the password for ldapuser2.
![alt text](https://i.imgur.com/sJCUKF6.png)
Via these creds we can su into the ldapuser2.
```
Username: ldapuser2
Password: 8bc8251332abe1d7f105d3e53ad39ac2
```
### Privilege Escalation (for user : ldapuser1):
We see a backup.7z file, which cannot we transfered directly. We can either user msf or base64 technique.
```
$ cat backup.7z | base 64 (on the box)
and to decrypt
$ base64 –d enc_backup > backup.7z 
```
Using https://www.lostmypass.com/file-types/7z/ we get the password for the 7z as `delete`  <br>
We can also use https://raw.githubusercontent.com/Seyptoo/7z-BruteForce/master/latex.py break the 7z. <br>
Now running strings we get the creds for ldapuser1 :
```
root@kali:~/Desktop/boxes/LightWeight/files# strings status.php | grep password
$password = 'f3ca9d298a553da117442deeb6fa932d';
```
we find a openssl present inside this user, which was owned by root. <br>
Therefore we encrypted the root.txt and saved it on the user and then decrypted it to get the flag.
![alt text](https://i.imgur.com/2BajMIP.png)
We can also get root on this shell (didnot try after getting the hash). 
### Gained knowledge:
ldap enumerations <br>
base64 decode encode to retrieve files <br>
getcap <br>
openssl <br>
