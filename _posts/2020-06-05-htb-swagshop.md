---
layout: post
title:  "HTB - Swag Shop"
date:   2020-06-05 20:00 +0100
tags: writeups
initial: 2019-09-05
---
Writeup for the retired HTB machine Swag Shop  
Link: hhttps://www.hackthebox.eu/home/machines/profile/188  
IP: 10.10.10.140

<!--more-->

## Recon
A qick portscan reveals multiple open ports:  
```
22/tcp    open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp    open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home page
```
As a website is involved, investigation into interesting files is conducted with gobuster.

## gobuster
By investigating the dirs a bit, one can find `/app/etc/local.xml`, which holds credentials:
`root:fMVWh7bDHpgZkyfqQXreTjU9`  
as well as a crypt key:  
`b355a9e0cd018d3f7f03607141518419`
The DB name is `swagshop`.  

As we don't have any credentials that yield a successful login, we switch to running an exploit to generate them.
37977.py works after a quick code cleanup and setting the target to `$IP/index.php`.  
The admin interface is located at `/index.php/admin`.  
Thanks to the exploit we end up with an admin login `forme:forme`.  

## Admin interface & shell
The next step after login is opening a reverse shell.  
Most of this is based on the [Froghopper](https://www.foregenix.com/blog/anatomy-of-a-magento-attack-froghopper) attack, which is explained in detail in the liked article.  
Prepare a [php reverse shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) (shell.php) with your IP and Port and change the extension to .jpg to pass the upload filter.  
That can be uploaded as the image for a new Category (Catalog -> Manage Categories -> Add Category).  
If everything worked well, the file will show up under `/media/catalog/category/`.
Now, to execute the shell, it has to be called. To do that, symlinks need to be activated in System - Configuration - Developer - Template Settings.  
Then a new *Newsletter Template* has to be created with the following content:  
`{{block type="core/template" template='../../../../../../media/catalog/category/shell.jpg'}}`  
Afterwards, a click on "preview" will force the server to render the php shell. After the shell connects to the local netcat listener, the user flag can be printed.

## Road to root
Enumerating common privilege escalation paths, a sudo entry reveals a possible path:  
```
User www-data may run the following commands on swagshop:
    (root) NOPASSWD: /usr/bin/vi /var/www/html/*
```
As one can [spawn shells within vi](http://web.physics.ucsb.edu/~pcs/apps/editors/vi/vi_unix.html), this can be exploited to spawn an elevated shell.
So it's a matter of running `sudo vi /var/www/html/LICENSE.txt` and then `:!bash`.
