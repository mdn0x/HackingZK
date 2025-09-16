- - - 
created : 25-08-2025 

Tags : #easy 

Released on 24 May 2025 (Season 8)
- - - 
# Machine Information

As is common in real life [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) pentests, you will start the Fluffy box with credentials for the following account: j.fleischman / J0elTHEM4n1990!
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

This takes a lot so we wait.
Notice we have [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) here.
# Enumeration
## SMB 

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
# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)