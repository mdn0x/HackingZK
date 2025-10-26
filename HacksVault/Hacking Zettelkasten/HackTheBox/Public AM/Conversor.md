- - - 
created : 26-10-2025 

Tags : #easy 

Released on 25 Oct 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Conversor/user]
└─$ rustscan -a conversor.htb -- -A

Open 10.10.11.92:22
Open 10.10.11.92:80
```
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
-$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://conversor.htb

/about                (Status: 200) [Size: 2842]
/convert              (Status: 405) [Size: 153]
/javascript           (Status: 301) [Size: 319] [--> http://conversor.htb/javascript/]
/login                (Status: 200) [Size: 722]
/logout               (Status: 302) [Size: 199] [--> /login]
/register             (Status: 200) [Size: 726]
```

![Pasted image 20251026100613.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251026100613.png)
**/about**
![Pasted image 20251026100648.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251026100648.png)

Extract the files:

```bash
└─$ tar -xvf source_code.tar.gz
app.py
app.wsgi
install.md
instance/
instance/users.db
scripts/
static/
static/images/
static/images/david.png
static/images/fismathack.png
static/images/arturo.png
static/nmap.xslt
static/style.css
templates/
templates/register.html
templates/about.html
templates/index.html
templates/login.html
templates/base.html
templates/result.html
uploads/
```

Reading `install.md` (read other files too):

```
To deploy Conversor, we can extract the compressed file:

"""
tar -xvf source_code.tar.gz
"""

We install flask:

"""
pip3 install flask
"""

We can run the app.py file:

"""
python3 app.py
"""

You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
"""
```
  
So every minute, the system executes any  `.py` script inside `/var/www/conversor.htb/scripts/` as the **www-data** user. 

We have a shell trigger so if we can drop Python code into that directory, it gets executed automatically after a while.
# Exploit

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)

## HINTS

```bash
└─$ cat hints.txt                 

User
1 . Look at parsing
2.  Find RCE

Root
- Look for custom configs -
```