- - - 
created : 01-11-2025 

Tags : #medium 

Released on 01 Nov 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$ rustscan -a giveback.htb -- -A

Open 10.10.11.94:22
Open 10.10.11.94:80
Open 10.10.11.94:30686

PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    syn-ack ttl 62 nginx 1.28.0
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-favicon: Unknown favicon MD5: 000BF649CC8F6BF27CFB04D1BCDCD3C7
|_http-server-header: nginx/1.28.0
|_http-title: GIVING BACK IS WHAT MATTERS MOST &#8211; OBVI
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-generator: WordPress 6.8.1

30686/tcp open  http    syn-ack ttl 63 Golang net/http server
```

We have [Wordpress](../../../3%20-%20Tags/Hacking%20Concepts/Wordpress.md).
# Enumeration
## HTTP (80)

![Pasted image 20251102182523.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251102182523.png)

`v3.14.0`

Find the portal http://giveback.htb/donations/the-things-we-need/

![Pasted image 20251102182634.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251102182634.png)
## 30686

Use `cURL` to examine:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$  curl -k http://giveback.htb:30686/ --insecure
{
        "service": {
                "namespace": "default",
                "name": "wp-nginx-service"
        },
        "localEndpoints": 1,
        "serviceProxyHealthy": true
}   
```
## WPScan

Run [WPScan](../../../3%20-%20Tags/Hacking%20Tools/WPScan.md) with URL flag:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$ wpscan --url http://giveback.htb/ -e ap --plugins-detection aggressive
```

We find the CVE for a known plugin, which is vulnerable to Unauthenticated Remote Code Execution.
# Exploit
# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)