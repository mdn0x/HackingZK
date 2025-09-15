## Description

**Gobuster** is a powerful tool used for directory and file brute-forcing on web servers. It is particularly useful for penetration testers and security researchers who want to discover hidden resources, such as directories, files, and virtual hosts, that may not be directly accessible through a web browser. Gobuster operates by making HTTP requests to a target server and checking for valid responses.

### Key Features

- **Directory and File Brute-Forcing**: Gobuster can enumerate directories and files on a web server using a wordlist, helping to uncover hidden resources.
- **DNS Subdomain Enumeration**: It can also be used to discover subdomains of a target domain, which is useful for identifying additional attack surfaces.
- **Customizable Options**: Users can customize various parameters, such as the number of threads, timeout settings, and HTTP methods.
- **Support for Multiple Protocols**: Gobuster supports both HTTP and HTTPS protocols, making it versatile for different web applications.
### Installation

Gobuster is written in [Go](../Programming%20Languages/Go.md), and you can install it using the following command:

```bash
go install github.com/OJ/gobuster/v3@latest
```

Make sure you have Go installed on your system before running this command.
### Common Commands

#### 1. Directory and File Brute-Forcing

To perform a directory and file brute-force attack, use the following command structure:

```
gobuster dir -u <target_url> -w <wordlist>
```

- **Example**:
  ```
  gobuster dir -u http://example.com -w /path/to/wordlist.txt
  ```

#### 2. Subdomain Fuzzing

While often associated with vhost and directory discovery, `Gobuster` also excels at subdomain enumeration, a crucial step in mapping the attack surface of a target domain. By systematically testing variations of potential subdomain names, `Gobuster` can uncover hidden or forgotten subdomains that might host valuable information or vulnerabilities.

Let's break down the `Gobuster` subdomain fuzzing command:

```bash
gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

- `gobuster dns`: Activates `Gobuster's` DNS fuzzing mode, directing it to focus on discovering subdomains.
- `-d inlanefreight.com`: Specifies the target domain (e.g., `inlanefreight.com`) for which you want to discover subdomains.
- `-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`: This points to the wordlist file that `Gobuster` will use to generate potential subdomain names. In this example, we're using a wordlist containing the top 5000 most common subdomains.

Under the hood, `Gobuster` works by generating subdomain names based on the wordlist, appending them to the target domain, and then attempting to resolve those subdomains using DNS queries. If a subdomain resolves to an IP address, it is considered valid and included in the output.
#### 3. VHost Fuzzing

While `gobuster` is primarily known for directory and file enumeration, its capabilities extend to virtual host (vhost) discovery, making it a valuable tool in assessing the security posture of a web server.

Let's dissect the `Gobuster` vhost fuzzing command:

```bash
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain
```

- `gobuster vhost`: This flag activates `Gobuster's vhost fuzzing mode`, instructing it to focus on discovering virtual hosts rather than directories or files.
- `-u http://inlanefreight.htb:81`: This specifies the base URL of the target server. `Gobuster` will use this URL as the foundation for constructing requests with different vhost names. In this example, the target server is located at `inlanefreight.htb` and listens on port 81.
- `-w /usr/share/seclists/Discovery/Web-Content/common.txt`: This points to the wordlist file that `Gobuster` will use to generate potential vhost names. The `common.txt` wordlist from SecLists contains a collection of commonly used vhost names and subdomains.
- `--append-domain`: This crucial flag instructs `Gobuster` to append the base domain (`inlanefreight.htb`) to each word in the wordlist. This ensures that the `Host` header in each request includes a complete domain name (e.g., `admin.inlanefreight.htb`), which is essential for vhost discovery.

In essence, `Gobuster` takes each word from the wordlist, appends the base domain to it, and then sends an HTTP request to the target URL with that modified `Host` header. By analyzing the server's responses (e.g., status codes, response size), `Gobuster` can identify valid `vhosts` that might not be publicly advertised or documented.

Running the command will execute a `vhost scan` against the target.
### 3. Customizing Options

You can customize various options to enhance your scans:

- **-t**: Set the number of concurrent threads (default is 10).
- **-r**: Follow redirects.
- **-s**: Specify which status codes to display (e.g., `200,204,301,302,403`).
- **-o**: Output results to a file.



## Conclusion
### Benefits

- **Efficiency**: Gobuster is designed for speed and can quickly enumerate directories and files, making it a valuable tool for penetration testing.
- **Flexibility**: The ability to customize options allows users to tailor their scans to specific needs and environments.
- **Open Source**: Being an open-source tool, Gobuster is freely available and can be modified or extended by users.

### Considerations

- **Ethical Use**: It is essential to use Gobuster responsibly and only on systems for which you have explicit permission to test.
- **Detection**: Some web servers may have security measures in place to detect and block brute-force attacks, so users should be aware of potential countermeasures.

Gobuster is an essential tool for cybersecurity professionals, providing robust capabilities for discovering hidden resources on web servers. Its speed, flexibility, and ease of use make it a go-to choice for many penetration testers and security researchers.