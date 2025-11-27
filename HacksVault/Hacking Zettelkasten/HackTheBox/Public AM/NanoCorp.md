- - - 
created : 27-11-2025 

Tags : #hard 

Released on 08 Nov 2025 (Season 9)
- - - 
# CHANGELOG

19TH NOVEMBER, 2025

Patched CVE-2025-54918 & CVE-2025-33073
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ rustscan -a nanocorp.htb -- -A  
Open 10.10.11.93:53
Open 10.10.11.93:80
Open 10.10.11.93:88
Open 10.10.11.93:593
Open 10.10.11.93:636
Open 10.10.11.93:3269
Open 10.10.11.93:3268
Open 10.10.11.93:5986
Open 10.10.11.93:9389
Open 10.10.11.93:49664
Open 10.10.11.93:49668
Open 10.10.11.93:50603
Open 10.10.11.93:50615
Open 10.10.11.93:50620
Open 10.10.11.93:50637

PORT      STATE SERVICE           REASON          VERSION
53/tcp    open  domain            syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http              syn-ack ttl 127 Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
| http-methods: 
|_  Supported Methods: HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://nanocorp.htb/
88/tcp    open  kerberos-sec      syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-11-27 20:48:41Z)
593/tcp   open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?          syn-ack ttl 127
3268/tcp  open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl? syn-ack ttl 127
5986/tcp  open  ssl/http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.nanocorp.htb
| Subject Alternative Name: DNS:dc01.nanocorp.htb

9389/tcp  open  mc-nmf            syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
50603/tcp open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
50615/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
50620/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
50637/tcp open  msrpc             syn-ack ttl 127 Microsoft Window

TRACEROUTE (using port 53/tcp)
HOP RTT       ADDRESS
1   185.11 ms 10.10.14.1
2   185.40 ms dc01.nanocorp.htb (10.10.11.93)
```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc01.nanocorp.htb` to `/etc/hosts`.
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://nanocorp.htb
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 301]
/.htpasswd            (Status: 403) [Size: 301]
/aux                  (Status: 403) [Size: 301]
/cgi-bin/             (Status: 403) [Size: 301]
/com1                 (Status: 403) [Size: 301]
/com2                 (Status: 403) [Size: 301]
/com3                 (Status: 403) [Size: 301]
/com4                 (Status: 403) [Size: 301]
/con                  (Status: 403) [Size: 301]
/css                  (Status: 301) [Size: 334] [--> http://nanocorp.htb/css/]
/examples             (Status: 503) [Size: 401]
/img                  (Status: 301) [Size: 334] [--> http://nanocorp.htb/img/]
/js                   (Status: 301) [Size: 333] [--> http://nanocorp.htb/js/]
/licenses             (Status: 403) [Size: 420]
/lpt1                 (Status: 403) [Size: 301]
/lpt2                 (Status: 403) [Size: 301]
/nul                  (Status: 403) [Size: 301]
/phpmyadmin           (Status: 403) [Size: 301]
/prn                  (Status: 403) [Size: 301]
/server-status        (Status: 403) [Size: 420]
/server-info          (Status: 403) [Size: 420]
/webalizer            (Status: 403) [Size: 301]
Progress: 20478 / 20478 (100.00%)
===============================================================
Finished
===============================================================
```

Visit the page:

![Pasted image 20251127140433.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127140433.png)

Add `hire.nanocorp.htb` to `/etc/hosts`:

![Pasted image 20251127140544.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127140544.png)
## Kerberos (88)
### Kerbrute

We can use `kerbrute` with some wordlists to start users enumeration on the domain:

```bash
└─$ /home/mdn0x/Documenti/GitTools/AD/kerbrute userenum --dc 10.10.11.93 -d nanocorp.htb /usr/share/wordlists/john.smith.txt

[+] VALID USERNAME:       Administrator@nanocorp.htb

```

```go
 /home/mdn0x/Documenti/GitTools/AD/kerbrute userenum --dc 10.10.11.93 -d nanocorp.htb /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt 
```
# Exploit
# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)