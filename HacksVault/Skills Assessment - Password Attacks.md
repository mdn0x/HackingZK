From [Password Attacks](./Hacking%20Zettelkasten/HackTheBox/Academy/Password%20Attacks.md):

# The Credential Theft Shuffle

[The Credential Theft Shuffle](https://adsecurity.org/?p=2362), as coined by `Sean Metcalf`, is a systematic approach attackers use to compromise Active Directory environments by exploiting `stolen credentials`. The process begins with gaining initial access, often through phishing, followed by obtaining local administrator privileges on a machine. Attackers then extract credentials from memory using tools like Mimikatz and leverage these credentials to `move laterally across the network`. Techniques such as pass-the-hash (PtH) and tools like NetExec facilitate this lateral movement and further credential harvesting. The ultimate goal is to escalate privileges and `gain control over the domain`, often by compromising Domain Admin accounts or performing DCSync attacks. Sean emphasizes the importance of implementing security measures such as the `Local Administrator Password Solution (LAPS)`, enforcing `multi-factor authentication`, and `restricting administrative privileges` to mitigate such attacks.
## Skills Assessment

`Betty Jayde` works at `Nexura LLC`. We know she uses the password `Texas123!@#` on multiple websites, and we believe she may reuse it at work. Infiltrate Nexura's network and gain command execution on the domain controller. The following hosts are in-scope for this assessment:
### Pivoting Primer

The internal hosts (`JUMP01`, `FILE01`, `DC01`) reside on a private subnet that is not directly accessible from our attack host. The only externally reachable system is `DMZ01`, which has a second interface connected to the internal network. This segmentation reflects a classic DMZ setup, where public-facing services are isolated from internal infrastructure.

To access these internal systems, we must first gain a foothold on `DMZ01`. From there, we can `pivot` — that is, route our traffic through the compromised host into the private network. This enables our tools to communicate with internal hosts as if they were directly accessible. After compromising the DMZ, refer to the module `cheatsheet` for the necessary commands to set up the pivot and continue your assessment.
### Objective

`NTLM` hash of `NEXURA\Administrator`

# Scan / Recon

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ rustscan -a 10.129.234.116 -- -A  

Open 10.129.234.116:22
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   234.63 ms 10.10.14.1
2   347.88 ms 10.129.234.116
```

Betty Jayde
Texas123!@#
# User Enumeration

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ./kerbrute userenum --dc 10.129.234.116 --domain inlanefreight.htb /usr/share/wordlists/john.smith.txt 

```