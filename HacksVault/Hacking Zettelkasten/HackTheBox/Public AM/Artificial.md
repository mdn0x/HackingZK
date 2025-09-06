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

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

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
# 🔒 Locked Content

For full guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)