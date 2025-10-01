# Install

```bash
sudo apt install bloodhound
[sudo] password di mdn0x: 
Installazione:                             
  bloodhound

Installazione dipendenze: 
  azurehound  binfmt-support  fastjar  jarwrapper  neo4j  sharphound

Riepilogo:
  Aggiornamento: 0, Installazione: 7, Rimozione: 0, Non aggiornati: 0
  Dimensione scaricamento: 180 MB
  Spazio richiesto: 342 MB / 27,4 GB disponibile

Continuare? [S/n] s
```

# Usage

Start

```bash
$ bloodhound
[sudo] password di mdn0x: 

 It seems it's the first time you run bloodhound

 Please run bloodhound-setup first

Do you want to run bloodhound-setup now? [Y/n] Y

 [*] Starting PostgreSQL service

 [*] Creating Database

 Creating database user

 Creating database
ALTER ROLE

 [*] Starting neo4j
Neo4j is not running.
Directories in use:
home:         /usr/share/neo4j
config:       /usr/share/neo4j/conf
logs:         /etc/neo4j/logs
plugins:      /usr/share/neo4j/plugins
import:       /usr/share/neo4j/import
data:         /etc/neo4j/data
certificates: /usr/share/neo4j/certificates
licenses:     /usr/share/neo4j/licenses
run:          /var/lib/neo4j/run
Starting Neo4j.
Started neo4j (pid:395001). It is available at http://localhost:7474
There may be a short delay until the server is ready.

 [i] You need to change the default password for neo4j
     Default credentials are user:neo4j password:neo4j

 [!] IMPORTANT: Once you have setup the new password, please update /etc/bhapi/bhapi.json with the new password before running bloodhound
...........................
 opening http://localhost:7474/
```

We log in with `neo4j:neo4j` user:pass combo and change passwd:
![Pasted image 20250922204056.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922204056.png)

Update the file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ sudo nano /etc/bhapi/bhapi.json

{
  "database": {
    "addr": "localhost:5432",
    "username": "_bloodhound",
    "secret": "bloodhound",
    "database": "bloodhound"
  },
  "neo4j": {
    "addr": "localhost:7687",
    "username": "neo4j",
    "secret": "YOURPASSWD"
  },
  "default_admin": {
    "principal_name": "admin",
    "password": "admin",
    "first_name": "Bloodhound",
    "last_name": "Kali"
  }
}
```

Run Bloodhound and login with the creds `admin/admin` and change the password.

Here we can upload our files:

![Pasted image 20250922212126.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922212126.png)

![Pasted image 20250922213043.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922213043.png)

Now we can visualize and query in the Explore page.

![Pasted image 20250922220934.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250922220934.png)

### Collect

First synchronize the time:

```bash
sudo ntpdate tombwatcher.htb
```

Now collect:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/TombWatcher/user]
└─$ bloodhound-python -u 'user' -p 'passwd'  -d domain.name -ns Victim_IP -c All --zip
```