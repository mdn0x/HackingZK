- - - 
created : 23-09-2025 

Tags : #medium 

Released on 05 Jul 2025 (Season 8)
- - - 
# Machine Information

As is common in real life Windows pentests, you will start the Voleur box with credentials for the following account:

`ryan.naylor / HollowOct31Nyt`

# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur]
└─$ rustscan -a voleur.htb -- -A

Open 10.10.11.76:53
Open 10.10.11.76:88
Open 10.10.11.76:135
Open 10.10.11.76:139
Open 10.10.11.76:389
Open 10.10.11.76:445
Open 10.10.11.76:464
Open 10.10.11.76:593
Open 10.10.11.76:636
Open 10.10.11.76:2222
Open 10.10.11.76:3268
Open 10.10.11.76:3269
Open 10.10.11.76:5985
Open 10.10.11.76:9389
Open 10.10.11.76:49664
Open 10.10.11.76:49668
Open 10.10.11.76:65121
Open 10.10.11.76:65120
Open 10.10.11.76:65133
Open 10.10.11.76:65138
Open 10.10.11.76:65151

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-23 20:38:39Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65120/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
65121/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65133/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65138/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
65151/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)

Host script results:
| smb2-time: 
|   date: 2025-09-23T20:39:53
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h59m59s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 48495/tcp): CLEAN (Timeout)
|   Check 2 (port 49360/tcp): CLEAN (Timeout)
|   Check 3 (port 60782/udp): CLEAN (Timeout)
|   Check 4 (port 21812/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

We have [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md). we add Add `dc.voleur.htb`to our `/etc/hosts` file.
# Enumeration

Since we aren't able to use password authentication directly, we can try to request a ticket.

First update time:

```bash
$ sudo timedatectl set-ntp off
$ sudo rdate -n 10.10.11.76
Tue Sep 23 23:12:24 CEST 2025
```

We request the certificate:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ impacket-getTGT voleur.htb/'ryan.naylor':'HollowOct31Nyt'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in ryan.naylor.ccache

$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Voleur/user/ryan.naylor.ccache

nxc ldap voleur.htb -u ryan.naylor -p HollowOct31Nyt -k
LDAP        voleur.htb      389    DC               [*] None (name:DC) (domain:voleur.htb)
LDAP        voleur.htb      389    DC               [+] voleur.htb\ryan.naylor:HollowOct31Nyt  
```

Now collect with [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Voleur/user]
└─$ bloodhound-python -u ryan.naylor -p HollowOct31Nyt -k -ns 10.10.11.76 -c All -d voleur.htb --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: voleur.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 12 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.voleur.htb
INFO: Done in 00M 16S
INFO: Compressing output into 20250923231923_bloodhound.zip
```

We open the GUI and upload:

![Pasted image 20250923232129.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923232129.png)

Then search for paths:

![Pasted image 20250923232745.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923232745.png)

![Pasted image 20250923232903.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923232903.png)

![Pasted image 20250923233040.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250923233040.png)

So we are member of this `Technicians` group but we don't have special permissions.
## SMB (445)
### NetExec

We have to try the given credentials for files, we can use [NetExec](../../../3%20-%20Tags/Hacking%20Tools/NetExec.md): 


# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)