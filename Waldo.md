# WALDO
### Info:
OS: Linux <br>
IP: 10.10.10.87<br>
Difficulty :  Medium-hard<br>
### Enumeration:
```
nmap -sC -sV -oA nmap 10.10.10.87
```
gives us the results : <br>
![alt text](https://i.imgur.com/MrQGkzk.png)
<br>
On port 80 we find a webpage on which we can find different .php files. <br>
![alt text](https://i.imgur.com/blKIlhT.png)
### Exploitation:
Make a post request via curl on FireRead, gets us the php code of the file.
![alt text](https://i.imgur.com/89QrX43.png)
Burp Suite
![alt text](https://i.imgur.com/EmGR4pb.png)
```
str_replace( array("../", "..""), "", $_POST['file']);
```
The above function doesn't work recursively therefore we can use ....// instead of ../ so that we can let it replace one of the (../) <br>
![alt text](https://i.imgur.com/ZEIyGyF.png)
We are inside a docker container.
![alt text](https://i.imgur.com/hvwYQAw.png)
We get the ssh key and now we can log into user and get the hash via:
```
ssh -i monitor.ssh nobody@10.10.10.87
```
We are now in a docker (Alpine) container. <br>
When we perform ssh following happens : <br>
10.10.16.16 ---> 10.10.10.87:22 (SSH) ---> 10.10.10.87:8888(Docker) <br>
![alt text](https://i.imgur.com/cJhrzEL.png)
User monitor is present there <br>
Therefore when we make a local ssh request from the nobody user we get the monitor user shell on port 22.
![alt text](https://i.imgur.com/BZDTriW.png)

### Lateral Movement :
We are in a restricted shell now.
<b>Rbash Escape</b>
We can escape the rbash via:
![alt text](https://i.imgur.com/zzi0a92.png)
Which gets us the complete user ` Monitor ` shell.
<br>
### Privilege Escalation:
Running LinEnum.sh gives us a file named `logMonitor-0.1`
https://blog.m0noc.com/2016/05/linux-capabilities-friend-and-foe.html
https://www.insecure.ws/linux/getcap_setcap.html
Using the above blogs as :
```
getcap -r * 2>/dev/null
/usr/bin/tac -s @ /root/.ssh/id_rsa
/usr/bin/tac | /root/.ssh/id_rsa | /usr/bin/tac
tac /root/root.txt
```
and we get the contents of root user, though no root shell.
### Gained knowledge:
Docker environment <br>
Rbash Escape <br>
Getcap <br>
PHP steg_replace <br>
