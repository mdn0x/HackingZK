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
# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)