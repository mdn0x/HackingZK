- - - 
created : 22-09-2025 

Tags : #medium 

Released on 07 Jun 2025 (Season 8)
- - - 
# Machine Information

As is common in real life Windows pentests, you will start the TombWatcher box with credentials for the following account: 

`henry/ H3nry_987TGV!`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ rustscan -a tombwatcher.htb -- -A 

Open 10.10.11.72:53
Open 10.10.11.72:80
Open 10.10.11.72:88
Open 10.10.11.72:135
Open 10.10.11.72:139
Open 10.10.11.72:389
Open 10.10.11.72:445
Open 10.10.11.72:464
Open 10.10.11.72:593
Open 10.10.11.72:636
Open 10.10.11.72:3268
Open 10.10.11.72:3269
Open 10.10.11.72:5985
Open 10.10.11.72:9389
Open 10.10.11.72:49691
Open 10.10.11.72:49712
Open 10.10.11.72:49694
Open 10.10.11.72:49692
Open 10.10.11.72:49731
Open 10.10.11.72:49746

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE

88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-22 19:04:44Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)

445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher


3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher

3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher

5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49691/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49694/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49712/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49731/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49746/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10

Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 20899/tcp): CLEAN (Timeout)
|   Check 2 (port 26528/tcp): CLEAN (Timeout)
|   Check 3 (port 61752/udp): CLEAN (Timeout)
|   Check 4 (port 46167/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2025-09-22T19:05:55
|_  start_date: N/A
|_clock-skew: mean: 4h00m00s, deviation: 0s, median: 4h00m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

```

We have [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md), let's add `dc01.tombwatcher.htb` to our `hosts` file.
# Enumeration
## HTTP (80) IIS

We can explore the website:

![Pasted image 20250922170417.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922170417.png)

### Dirsearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target website:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ dirsearch -u http://tombwatcher.htb

Target: http://tombwatcher.htb/

[17:08:51] Starting:                                                                                                            
[17:08:55] 404 -    2KB - /.ashx                                            
[17:08:55] 404 -    2KB - /.asmx                                              
[17:09:02] 404 -    2KB - /admin%20/                                        
[17:09:03] 404 -    2KB - /admin.                                                                              
[17:09:13] 301 -  160B  - /aspnet_client  ->  http://tombwatcher.htb/aspnet_client/
[17:09:13] 404 -    2KB - /asset..                                          
[17:09:24] 400 -    3KB - /docpicker/internal_proxy/https/127.0.0.1:9043/ibm/console
[17:09:29] 404 -    2KB - /index.php.                                       
[17:09:30] 404 -    2KB - /javax.faces.resource.../                         
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/vmSystemProperties
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/help/*
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/vmLog/disable
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/jvmtiAgentLoad/!/etc!/passwd
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/vmLog/output=!/tmp!/pwned
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/jfrStart/filename=!/tmp!/foo
[17:09:31] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/compilerDirectivesAdd/!/etc!/passwd
[17:09:31] 400 -    3KB - /jolokia/read/java.lang:type=*/HeapMemoryUsage    
[17:09:31] 400 -    3KB - /jolokia/exec/java.lang:type=Memory/gc
[17:09:31] 400 -    3KB - /jolokia/search/*:j2eeType=J2EEServer,*
[17:09:31] 400 -    3KB - /jolokia/read/java.lang:type=Memory/HeapMemoryUsage/used
[17:09:31] 400 -    3KB - /jolokia/write/java.lang:type=Memory/Verbose/true 
[17:09:33] 404 -    2KB - /login.wdm%2e                                     
[17:09:36] 404 -    2KB - /mcx/mcxservice.svc                               
[17:09:51] 404 -    2KB - /rating_over.                                     
[17:09:51] 404 -    2KB - /reach/sip.svc                                    
[17:09:56] 404 -    2KB - /service.asmx                                     
[17:10:03] 404 -    2KB - /static..                                         
[17:10:09] 403 -    2KB - /Trace.axd                                        
[17:10:10] 404 -    2KB - /umbraco/webservices/codeEditorSave.asmx          
[17:10:15] 404 -    2KB - /WEB-INF./                                        
[17:10:18] 404 -    2KB - /WebResource.axd?d=LER8t9aS                       
[17:10:18] 404 -    2KB - /webticket/webticketservice.svc                                                               
Task Completed
```

It's a dead-end.
## SMB (445)

We have to try the given credentials for files, we can use [SMBclient](../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ smbclient  -L //10.10.11.72/ -U henry
Password for [WORKGROUP\henry]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.72 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Nothing interesting.
## Bloodhound

We'll try to collect more info on the user `henry` with [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md)

First synchronize the time:

```bash
sudo ntpdate tombwatcher.htb
```

Now collect:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodhound-python -u 'henry' -p 'H3nry_987TGV!'  -d tombwatcher.htb -ns 10.10.11.72 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: tombwatcher.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc01.tombwatcher.htb:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc01.tombwatcher.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.tombwatcher.htb
INFO: Found 9 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.tombwatcher.htb
INFO: Done in 00M 19S
INFO: Compressing output into 20250922173843_bloodhound.zip
```

`unzip` it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ unzip 20250922173843_bloodhound.zip -d bloodyhound 
Archive:  20250922173843_bloodhound.zip
 extracting: bloodyhound/20250922173843_users.json  
 extracting: bloodyhound/20250922173843_ous.json  
 extracting: bloodyhound/20250922173843_computers.json  
 extracting: bloodyhound/20250922173843_containers.json  
 extracting: bloodyhound/20250922173843_domains.json  
 extracting: bloodyhound/20250922173843_gpos.json  
 extracting: bloodyhound/20250922173843_groups.json  
```

Now we can open it with the [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) GUI:

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)
