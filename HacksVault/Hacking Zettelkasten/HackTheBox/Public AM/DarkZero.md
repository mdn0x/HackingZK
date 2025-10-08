- - - 
created : 05-10-2025 

Tags : #medium 

Released on 04 Oct 2025 (Season 9)
- - - 
# Machine Information

As is common in real life pentests, you will start the DarkZero box with credentials for the following account `john.w / RFulUtONCOL!`

# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a darkzero.htb -- -A
Open 10.10.11.89:53
Open 10.10.11.89:88
Open 10.10.11.89:135
Open 10.10.11.89:139
Open 10.10.11.89:389
Open 10.10.11.89:445
Open 10.10.11.89:464
Open 10.10.11.89:593
Open 10.10.11.89:636
Open 10.10.11.89:1433
Open 10.10.11.89:2179
Open 10.10.11.89:3269
Open 10.10.11.89:3268
Open 10.10.11.89:5985
Open 10.10.11.89:9389
Open 10.10.11.89:49664
Open 10.10.11.89:49666
Open 10.10.11.89:49682
Open 10.10.11.89:49683
Open 10.10.11.89:49905
Open 10.10.11.89:49928
Open 10.10.11.89:49964
Open 10.10.11.89:63485

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-05 17:15:57Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Issuer: commonName=darkzero-DC01-CA/domainComponent=darkzero


445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb


1433/tcp  open  ms-sql-s      syn-ack ttl 127 Microsoft SQL Server 2022 16.00.1000.00; RTM
|_ssl-date: 2025-10-05T17:17:51+00:00; +7h00m00s from scanner time.
| ms-sql-info: 
|   10.10.11.89:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433

2179/tcp  open  vmrdp?        syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)

3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb


5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49682/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49683/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49905/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49928/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49964/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
63485/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC

```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md) to better understand all processes.

Let's add `dc01.darkzero.htb` to `/etc/hosts`.
# Enumeration
## SMB (135,139,445)

We try to list the shares with the credentials using [SMBclient](../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md) or [SMBmap](../../../3%20-%20Tags/Hacking%20Tools/SMBmap.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero]
└─$ smbclient -L \\\\darkzero.htb\\ -U john.w
Password for [WORKGROUP\john.w]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to darkzero.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

OR `nxc` also:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero]
└─$ nxc smb darkzero.htb -u 'john.w' -p 'RFulUtONCOL!' --shares
SMB         10.10.11.89     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:darkzero.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.89     445    DC01             [+] darkzero.htb\john.w:RFulUtONCOL! 
SMB         10.10.11.89     445    DC01             [*] Enumerated shares
SMB         10.10.11.89     445    DC01             Share           Permissions     Remark
SMB         10.10.11.89     445    DC01             -----           -----------     ------
SMB         10.10.11.89     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.89     445    DC01             C$                              Default share
SMB         10.10.11.89     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.89     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.89     445    DC01             SYSVOL          READ            Logon server share
```

Checking `SYSVOL`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ smbclient \\\\darkzero.htb\\SYSVOL -U john.w 
Password for [WORKGROUP\john.w]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 29 13:39:08 2025
  ..                                  D        0  Tue Jul 29 13:39:08 2025
  darkzero.htb                       Dr        0  Tue Jul 29 13:39:08 2025

smb: \> cd darkzero.htb\

smb: \darkzero.htb\> ls
  .                                   D        0  Tue Jul 29 13:45:53 2025
  ..                                  D        0  Tue Jul 29 13:39:08 2025
  DfsrPrivate                      DHSr        0  Tue Jul 29 13:45:53 2025
  Policies                            D        0  Tue Jul 29 13:39:19 2025
  scripts                             D        0  Tue Jul 29 13:39:08 2025

```

Nothing interesting here.
## Bloodhound

Synchronize the time:

```bash
└─$ sudo timedatectl set-ntp off

└─$ sudo rdate -n 10.10.11.89
Sun Oct  5 19:27:49 CEST 2025
```

We collect and ingest with [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ bloodhound-python -u 'john.w' -p 'RFulUtONCOL!' -d darkzero.htb -ns 10.10.11.89 -c All --zip     
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: darkzero.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.darkzero.htb
WARNING: LDAP Authentication is refused because LDAP signing is enabled. Trying to connect over LDAPS instead...
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.darkzero.htb
WARNING: LDAP Authentication is refused because LDAP signing is enabled. Trying to connect over LDAPS instead...
INFO: Found 5 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 1 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.darkzero.htb
INFO: Done in 00M 11S
INFO: Compressing output into 20251005195017_bloodhound.zip
```

We can open the GUI:

![Pasted image 20251005230349.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251005230349.png)

We are member of `Domain Users` :

![Pasted image 20251005230512.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251005230512.png)

`Administrator` and `KRBTGT` are also members of this group.

## MSSQL (1433)

We use `nxc` to check the connection to `mssql` with the given credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ nxc mssql darkzero.htb -u john.w -p 'RFulUtONCOL!'
MSSQL       10.10.11.89     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:darkzero.htb)
MSSQL       10.10.11.89     1433   DC01             [+] darkzero.htb\john.w:RFulUtONCOL! 
```
# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)
## HINTS

```bash
└─$ cat hints.txt                 

User
1 . Look at ms-sql functions
2.  Find privesc

Root
- Look for cross-controllers vulns -
```