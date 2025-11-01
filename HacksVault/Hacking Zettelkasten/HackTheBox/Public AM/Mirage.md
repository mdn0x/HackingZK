- - - 
created : 20-07-2025 

Tags : #hard 

Released on 19 Jul 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a mirage.htb -- -A
Open 10.10.11.78:53
Open 10.10.11.78:88
Open 10.10.11.78:111
Open 10.10.11.78:139
Open 10.10.11.78:135
Open 10.10.11.78:389
Open 10.10.11.78:445
Open 10.10.11.78:464
Open 10.10.11.78:593
Open 10.10.11.78:636
Open 10.10.11.78:2049
Open 10.10.11.78:3268
Open 10.10.11.78:3269
Open 10.10.11.78:4222
Open 10.10.11.78:5985
Open 10.10.11.78:9389
Open 10.10.11.78:47001
Open 10.10.11.78:49664
Open 10.10.11.78:49666
Open 10.10.11.78:49665
Open 10.10.11.78:49667
Open 10.10.11.78:49668
Open 10.10.11.78:51617
Open 10.10.11.78:51626
Open 10.10.11.78:51627
Open 10.10.11.78:51642
Open 10.10.11.78:51648
Open 10.10.11.78:51671
Open 10.10.11.78:51685
Open 10.10.11.78:56442

PORT      STATE SERVICE         REASON          VERSION
53/tcp    open  domain          syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec    syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-29 14:02:07Z)
111/tcp   open  rpcbind         syn-ack ttl 127 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn     syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap            syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)

445/tcp   open  microsoft-ds?   syn-ack ttl 127
464/tcp   open  kpasswd5?       syn-ack ttl 127
593/tcp   open  ncacn_http      syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap        syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)

2049/tcp  open  nlockmgr        syn-ack ttl 127 1-4 (RPC #100021)
3268/tcp  open  ldap            syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap        syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
4222/tcp  open  vrml-multi-use? syn-ack ttl 127

5985/tcp  open  http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf          syn-ack ttl 127 .NET Message Framing
47001/tcp open  http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51617/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51626/tcp open  ncacn_http      syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
51627/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51642/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51648/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51671/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51685/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
56442/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc01.mirage.htb` to `/etc/hosts`.
# Enumeration
## RPC (2049) 

Let's check for unmounted shares:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nxc nfs 10.10.11.78 --shares
NFS         10.10.11.78     2049   10.10.11.78      [*] Supported NFS versions: (2, 3, 4) (root escape:False)                                                                                                 
NFS         10.10.11.78     2049   10.10.11.78      [*] Enumerating NFS Shares
NFS         10.10.11.78     2049   10.10.11.78      UID        Perms    Storage Usage    Share                          Access List                                                                           
NFS         10.10.11.78     2049   10.10.11.78      ---        -----    -------------    -----                          -----------                                                                           
NFS         10.10.11.78     2049   10.10.11.78      4294967294 r--      15.9GB/19.8GB    /MirageReports                 No network 
```

There is a share called `MirageReports` so let’s see the contents:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ nxc nfs 10.10.11.78 --share '/MirageReports' --ls '/' 
NFS         10.10.11.78     2049   10.10.11.78      [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.10.11.78     2049   10.10.11.78      UID        Perms  File Size     File Path
NFS         10.10.11.78     2049   10.10.11.78      ---        -----  ---------     ---------
NFS         10.10.11.78     2049   10.10.11.78      4294967294 dr--   64.0B         /MirageReports/.
NFS         10.10.11.78     2049   10.10.11.78      4294967294 dr--   64.0B         /MirageReports/..
NFS         10.10.11.78     2049   10.10.11.78      4294967294 -r-x   8.1MB         /MirageReports/Incident_Report_Missing_DNS_Record_nats-svc.pdf
NFS         10.10.11.78     2049   10.10.11.78      4294967294 -r-x   8.9MB         /MirageReports/Mirage_Authentication_Hardening_Report.pdf
```

Let's mount it on our machine:

```bash
─$ mkdir mirage_reports_files
─$ sudo mount -t nfs 10.10.11.78:/MirageReports mirage_reports_files                                                                                   
─$ cd mirage_reports_files                                                                                                      
─$ ls
Incident_Report_Missing_DNS_Record_nats-svc.pdf  Mirage_Authentication_Hardening_Report.pdf
```

We found another hostname `nats-svc.mirage.htb` in `Incident_Report_Missing_DNS_Record_nats-svc.pdf` and add it to `/etc/hosts`
# Exploit

### DNS Spoofing

Default port for NATS service is `4222` and we found that port open with the previous [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) scan.

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)