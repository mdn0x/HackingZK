- - - 
created : 30-11-2025 

Tags : #medium 

Released on DD.month.AA (Season x)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Gavel/user]
└─$ rustscan -a gavel.htb -- -A 
Open 10.10.11.97:22
Open 10.10.11.97:80

```
# Enumeration
## HTTP (80)

Register your account:
### Dirsearch

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)