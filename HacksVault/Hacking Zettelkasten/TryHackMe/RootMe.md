- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Just root the machine .

# Recon
## Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan the machine IP :

```
    80/tcp open http  
    22/tcp open ssh
```
# Enumeration 
## Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to enumerate directories :

```
    /panel
```

# Exploit
## Reverse shell

We can download the reverse shell from here https://github.com/pentestmonkey/php-reverse-shell

notice that we have to modify the [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) in the code and target our [Vulnerable Machine](../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md) :


<img src="../../Flameshots/c11a62a3346b0561b978ba29613dd5e2.png" alt="c11a62a3346b0561b978ba29613dd5e2.png" width="261" height="153" class="jop-noMdConv">

## Obtaining first access

visiting /panel we can upload the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), after a few tries modifing the .php extension to fit the php version supported (**php5**)

but first we have to setup a listener on the $port you have in the shell code , i didn't change the code and execute [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) like this

         `nc -lvnp 1234`

after that we can upload our Shell on /panel and wait for the connection, then `find / -type f -name user.txt 2> /dev/null` to find the user.txt file containing the first [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) .

# Privilege Escalation

For [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we can find SUID permissions for the user `$ find / -user root -perm /4000`  and find `/usr/bin/python`

we can go to  https://gtfobins.github.io/  and get a [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) command.

then on the Target Machine we execute  `python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`  (the command for escalation)

confirm with `whoami` >> `root` and find the flag `find / -type f -name root.txt`  >>  `cat root.txt`

Rooted !!

