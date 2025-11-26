- - - 
created : 23-11-2025 

Tags : #hard 

Released on 22 Nov 2025 (Season 9)
- - - 
# Machine Information

Please allow up to 7 minutes for services to load. As is common in real life Windows penetration tests, you will start the Fries box with credentials for the following account : `d.cooper@fries.htb / D4LE11maan!!`

Notes: There is a way with the certs to obtain root before user, involves find a way to get pwm before but that rely on getting a valid domain user first.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ rustscan -a fries.htb -- -A

Open 10.10.11.96:22
Open 10.10.11.96:53
Open 10.10.11.96:80
Open 10.10.11.96:88

Open 10.10.11.96:135
Open 10.10.11.96:139
Open 10.10.11.96:389
Open 10.10.11.96:445
Open 10.10.11.96:443
Open 10.10.11.96:464
Open 10.10.11.96:593
Open 10.10.11.96:636

Open 10.10.11.96:2179
Open 10.10.11.96:3268
Open 10.10.11.96:3269
Open 10.10.11.96:5985
Open 10.10.11.96:9389

Open 10.10.11.96:49667
Open 10.10.11.96:49685
Open 10.10.11.96:49686
Open 10.10.11.96:49689
Open 10.10.11.96:49688
Open 10.10.11.96:49913
Open 10.10.11.96:49945
Open 10.10.11.96:49972

PORT      STATE SERVICE       REASON          VERSION
22/tcp    open  ssh           syn-ack ttl 62  OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 62  nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://fries.htb/


88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-11-23 22:41:56Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Issuer: commonName=fries-DC01-CA/domainComponent=fries
| Public Key type: rsa
443/tcp   open  ssl/http      syn-ack ttl 62  nginx 1.18.0 (Ubuntu)
| ssl-cert: Subject: commonName=pwm.fries.htb/organizationName=Fries Foods LTD/stateOrProvinceName=Madrid/countryName=SP/organizationalUnitName=PWM Configuration/localityName=Madrid/emailAddress=web@fries.htb
| Issuer: commonName=pwm.fries.htb/organizationName=Fries Foods LTD/stateOrProvinceName=Madrid/countryName=SP/organizationalUnitName=PWM Configuration/localityName=Madrid/emailAddress=web@fries.htb
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-23T22:43:50+00:00; -1m03s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Issuer: commonName=fries-DC01-CA/domainComponent=fries

2179/tcp  open  vmrdp?        syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found


9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49685/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49686/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49688/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49689/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49913/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49945/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49972/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC

Service Info: Host: DC01; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-11-23T22:43:10
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 46245/tcp): CLEAN (Timeout)
|   Check 2 (port 47430/tcp): CLEAN (Timeout)
|   Check 3 (port 23943/udp): CLEAN (Timeout)
|   Check 4 (port 21385/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: -1m03s, deviation: 0s, median: -1m03s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   31.13 ms 10.10.14.1
2   32.37 ms dc01.fries.htb (10.10.11.96)
```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc01.fries.htb` to `/etc/hosts`.
# Enumeration

## SSH (22)

We try to access:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ssh dale@fries.htb
dale@fries.htb password: 
Permission denied, please try again.
```

Obviously, it would be too easy.
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://fries.htb 

/about                (Status: 200) [Size: 14932]
/menu                 (Status: 200) [Size: 19931]
```

Nothing really interesting.
### FFuf

We use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for subdomain enumeration:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.fries.htb' -u http://fries.htb -fs 154 -c

code                    [Status: 200, Size: 13592, Words: 1048, Lines: 272, Duration: 49ms]
```

Now this is interesting, add it to the `/etc/hosts` file:

```bash
10.10.11.96 dc01.fries.htb dc.fries.htb fries.htb code.fries.htb 
```

And visit the page:

![Pasted image 20251123233855.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251123233855.png)

Try the credentials: `d.cooper@fries.htb / D4LE11maan!!`

![Pasted image 20251123234112.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251123234112.png)

We have the site repo, let's check the docker-compose file:

```js
version: '3.8'

services:
  web:
    image: fries-web
    working_dir: /app
    restart: always
    container_name: web
    command: /usr/local/bin/python3 run.py
    ports:
      - 127.0.0.1:5000:5000
    depends_on:
      - postgres
    networks:
      vpcbr2:
        ipv4_address: 172.18.0.2

  postgres:
    image: postgres:16
    container_name: postgres
    restart: always
    environment:
      POSTGRES_USER: EXAMPLE_USER
      POSTGRES_PASSWORD: EXAMPLE_PASSWORD
      POSTGRES_DB: ps_db
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
        vpcbr2:
          ipv4_address: 172.18.0.3

volumes:
  pgdata:

networks:
  vpcbr2:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16
          gateway: 172.18.0.1
```

Check commits:

![Pasted image 20251123234651.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251123234651.png)

From **README** file:

```bash
Ensure the backend PostgreSQL database is accessible and contains the necessary schema - `ps_db`
The backend database can be managed from `http://db-mgmt05.fries.htb`. (This requires infra access, contact Dylan, Mike or Dale)
Make sure the appropriate credentials and network routing are in place.
```

This is probably the way in the box, pivoting into the container, but first we'll try other enumeration on the domain because it's an hard machine and you can't know, or maybe you can find unintended paths.
## Kerberos (88)

We have to try to request a ticket for the given user, maybe will help with enumeration of the domain:

```bash
# Syncro time

─$ sudo ntpdate 10.10.11.96     
[sudo] password di mdn0x: 
2025-11-23 22:53:51.277552 (+0000) -63.001208 +/- 0.015898 10.10.11.96 s1 no-leap
CLOCK: time stepped by -63.001208

# Ticket request

─$ impacket-getTGT fries.htb/'d.cooper':'D4LE11maan!!'                        
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Kerberos SessionError: KDC_ERR_PREAUTH_FAILED(Pre-authentication information was invalid)
```

We can't.
### Kerbrute 

Try domain users enumeration with `kerbrute` and some wordlists:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ./kerbrute userenum --dc 10.10.11.96 -d fries.htb /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt

2025/11/23 22:57:53 >  [+] VALID USERNAME:       administrator@fries.htb
2025/11/23 22:58:33 >  [+] VALID USERNAME:       Administrator@fries.htb
2025/11/23 22:59:37 >  [+] VALID USERNAME:       web@fries.htb
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ./kerbrute userenum --dc 10.10.11.96 -d fries.htb /usr/share/wordlists/john.smith.txt
```
## SMB (135,139,445) 

We check [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) using `nxc` and given credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ netexec smb 10.10.11.96 -u d.cooper -p 'D4LE11maan!!' --shares
SMB         10.10.11.96     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.96     445    DC01             [-] fries.htb\d.cooper:D4LE11maan!! STATUS_LOGON_FAILURE
```

We can't.
## LDAP (389)

Check anonymous and then credential access:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ nxc ldap dc.hercules.htb -u 'd.cooper' -p 'D4LE11maan!!' 
LDAP        10.10.11.91     389    DC               [*] None (name:DC) (domain:hercules.htb)
LDAP        10.10.11.91     389    DC               [-] hercules.htb\d.cooper:D4LE11maan!! STATUS_NOT_SUPPORTED
```

These credentials seems not to work on the domain.
# Exploit

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)