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

We have `mssql 2022`.

Let's add `dc01.darkzero.htb` to `/etc/hosts`. 
# Enumeration

## MSSQL

We can start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) checking logging in `mssql` for given user:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Signed/user]
└─$ nxc mssql signed.htb -u scott -p 'Sm230#C5NatH'          
MSSQL       10.10.11.90     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:SIGNED.HTB)
MSSQL       10.10.11.90     1433   DC01             [-] SIGNED.HTB\scott:Sm230#C5NatH (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')
```

`impacket` shell:

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
SQL (scott  guest@master)> enum_db
name     is_trustworthy_on   
------   -----------------   
master                   0   

tempdb                   0   

model                    0   

msdb                     1   

SQL (scott  guest@master)> enum_links
SRV_NAME   SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE   SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
--------   ----------------   -----------   --------------   ------------------   ------------   -------   
DC01       SQLNCLI            SQL Server    DC01             NULL                 NULL           NULL      

Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------   
SQL (scott  guest@master)> enable_xp_cmdshell
ERROR(DC01): Line 105: User does not have permission to perform this action.
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
ERROR(DC01): Line 62: The configuration option 'xp_cmdshell' does not exist, or it may be an advanced option.
ERROR(DC01): Line 1: You do not have permission to run the RECONFIGURE statement.
```

User enumeration:

```bash
> select sp.name as login, sp.type_desc as login_type, sl.password_hash, sp.create_date, sp.modify_date, case when sp.is_disabled = 1 then 'Disabled' else 'Enabled' end as status from sys.server_principals sp left join sys.sql_logins sl on sp.principal_id = sl.principal_id where sp.type not in ('G', 'R') order by sp.name;
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

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)
## HINTS

```bash
└─$ cat hints.txt                 

User
1 . Look at ms-sql functions
2.  Find privesc

Root
- Look for special permissions on ms-sql -
```