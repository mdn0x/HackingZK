## FFuf Overview

FFuf (Fuzz Faster U Fool) is a fast and flexible web fuzzing tool designed for discovering hidden resources on web servers. It is particularly useful for penetration testers and security researchers looking to identify vulnerabilities, such as unprotected directories, files, and parameters in web applications.

### Key Features

- **Speed**: FFuf is optimized for performance, allowing users to conduct fuzzing operations quickly, even against large target sets.

- **Customizable Payloads**: Users can specify custom wordlists for fuzzing, enabling targeted searches for specific resources or vulnerabilities.

- **Multiple Request Methods**: FFuf supports various HTTP request methods, including GET, POST, PUT, and DELETE, allowing for comprehensive testing of web applications.

- **JSON Output**: The tool can output results in JSON format, making it easy to parse and analyze the findings programmatically.

- **Filters and Matchers**: FFuf provides options to filter results based on status codes, response sizes, and other criteria, helping users focus on relevant findings.

### Installation

FFuf can be installed easily using [Go](../Programming%20Languages/Go.md), as it is written in the Go programming language. Here’s how to install it:

```bash
go install github.com/ffuf/ffuf/v2@latest
```

Make sure to add the Go binary path to your system's PATH variable if it’s not already included.
### Basic Commands

Here are some common commands and options for using FFuf:
#### 1. Basic Directory Fuzzing

To perform a basic directory fuzzing operation, use the following command:

```bash
ffuf -u http://target.com/FUZZ -w /path/to/wordlist.txt
```

- `-u`: Specifies the target URL with `FUZZ` as a placeholder for the payload.
- `-w`: Specifies the wordlist to use for fuzzing.

#### 2. Fuzzing with POST Requests

To send POST requests while fuzzing, use the `-X` option:

```bash
ffuf -u http://target.com/login -X POST -d "username=admin&password=FUZZ" -w /path/to/passwords.txt
```

#### 3. Filtering Results

To filter results based on HTTP status codes, use the `-fc` option:

```bash
ffuf -u http://target.com/FUZZ -w /path/to/wordlist.txt -fc 404
```

This command will only show results that do not return a 404 status code.

#### 4. Outputting Results

To save the results to a file, use the `-o` option:

```bash
ffuf -u http://target.com/FUZZ -w /path/to/wordlist.txt -o results.json
```

#### Usage Example

Here’s an example of a complete command that fuzzes a target URL for hidden directories:

```bash
ffuf -u http://example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc all
```

This command will search for all matching resources using the common directory wordlist.

## Recursive Fuzzing with ffuf

So far, we've focused on fuzzing directories directly under the web root and files within a single directory. But what if our target has a complex structure with multiple nested directories? Manually fuzzing each level would be tedious and time-consuming. This is where recursive fuzzing comes in handy.

### How Recursive Fuzzing Works

Recursive fuzzing is an automated way to delve into the depths of a web application's directory structure. It's a pretty basic 3 step process:

1. `Initial Fuzzing`:
    - The fuzzing process begins with the top-level directory, typically the web root (`/`).
    - The fuzzer starts sending requests based on the provided wordlist containing the potential directory and file names.
    - The fuzzer analyzes server responses, looking for successful results (e.g., HTTP 200 OK) that indicate the existence of a directory.
2. `Directory Discovery and Expansion`:
    - When a valid directory is found, the fuzzer doesn't just note it down. It creates a new branch for that directory, essentially appending the directory name to the base URL.
    - For example, if the fuzzer finds a directory named `admin` at the root level, it will create a new branch like `http://localhost/admin/`.
    - This new branch becomes the starting point for a fresh fuzzing process. The fuzzer will again iterate through the wordlist, appending each entry to the new branch's URL (e.g., `http://localhost/admin/FUZZ`).
3. `Iterative Depth`:
    - The process repeats for each discovered directory, creating further branches and expanding the fuzzing scope deeper into the web application's structure.
    - This continues until a specified depth limit is reached (e.g., a maximum of three levels deep) or no more valid directories are found.

Imagine a tree structure where the web root is the trunk, and each discovered directory is a branch. Recursive fuzzing systematically explores each branch, going deeper and deeper until it reaches the leaves (files) or encounters a predetermined stopping point.

#### Why Use Recursive Fuzzing?

Recursive fuzzing is a practical necessity when dealing with complex web applications:

- `Efficiency`: Automating the discovery of nested directories saves significant time compared to manual exploration.
- `Thoroughness`: It systematically explores every branch of the directory structure, reducing the risk of missing hidden assets.
- `Reduced Manual Effort`: You don't need to input each new directory to fuzz manually; the tool handles the entire process.
- `Scalability`: It's particularly valuable for large-scale web applications where manual exploration would be impractical.

In essence, recursive fuzzing is about `working smarter, not harder`. It allows you to efficiently and comprehensively probe the depths of a web application, uncovering potential vulnerabilities that might be lurking in its hidden corners.
### Recursive Fuzzing commands

Let's use `ffuf` to demonstrate recursive fuzzing:

```bash
└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -ic -v -u http://94.237.122.241:57697/FUZZ -e .html -recursion


[Status: 403, Size: 158, Words: 17, Lines: 11, Duration: 272ms]
| URL | http://94.237.122.241:57697/
    * FUZZ: 

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 39ms]
| URL | http://94.237.122.241:57697/level1
| --> | /level1/
    * FUZZ: level1

[INFO] Adding a new job to the queue: http://94.237.122.241:57697/level1/FUZZ

[Status: 403, Size: 158, Words: 17, Lines: 11, Duration: 39ms]
| URL | http://94.237.122.241:57697/
    * FUZZ: 
<SNIP>
```

Notice the addition of the `-recursion` flag. This tells `ffuf` to fuzz any directories it finds recursively. For example, if `ffuf` discovers an admin directory, it will automatically start a new fuzzing process on `http://localhost/admin/FUZZ`. In fuzzing scenarios where wordlists contain comments (lines starting with #), the `ffuf -ic` option proves invaluable. By enabling this option, `ffuf` intelligently ignores commented lines during fuzzing, preventing them from being treated as valid inputs.

The fuzzing commences at the web root (`http://IP:PORT/FUZZ`). Initially, `ffuf` identifies a directory named `level1`, indicated by a `301 (Moved Permanently)` response. This signifies a redirection and prompts the tool to initiate a new fuzzing process within this directory, effectively branching out its search.

As `ffuf` recursively explores `level1`, it uncovers two additional directories: `level2` and `level3`. Each is added to the fuzzing queue, expanding the search depth. Furthermore, an `index.html` file is discovered within `level1`.

The fuzzer systematically works through its queue, identifying `index.html` files in both `level2` and `level3`. Notably, the `index.html` file within `level3` stands out due to its larger file size than the others.

#### Be Responsible

While recursive fuzzing is a powerful technique, it can also be resource-intensive, especially on large web applications. Excessive requests can overwhelm the target server, potentially causing performance issues or triggering security mechanisms.

To mitigate these risks, `ffuf` provides options for fine-tuning the recursive fuzzing process:

- `-recursion-depth`: This flag allows you to set a maximum depth for recursive exploration. For example, `-recursion-depth 2` limits fuzzing to two levels deep (the starting directory and its immediate subdirectories).
- `-rate`: You can control the rate at which `ffuf` sends requests per second, preventing the server from being overloaded.
- `-timeout`: This option sets the timeout for individual requests, helping to prevent the fuzzer from hanging on unresponsive targets.


```bash
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -ic -u http://IP:PORT/FUZZ -e .html -recursion -recursion-depth 2 -rate 500

```
# Parameter and Value Fuzzing

Building upon the discovery of hidden directories and files, we now delve into parameter and value fuzzing. This technique focuses on manipulating the parameters and their values within web requests to uncover vulnerabilities in how the application processes input.

Parameters are the messengers of the web, carrying vital information between your browser and the server that hosts the web application. They're like variables in programming, holding specific values that influence how the application behaves.

## GET Parameters: Openly Sharing Information

You'll often spot `GET` parameters right in the URL, following a question mark (`?`). Multiple parameters are strung together using ampersands (`&`). For example:

```http
https://example.com/search?query=fuzzing&category=security
```

In this URL:

- `query` is a parameter with the value "fuzzing"
- `category` is another parameter with the value "security"

`GET` parameters are like postcards – their information is visible to anyone who glances at the URL. They're primarily used for actions that don't change the server's state, like searching or filtering.
### Example Usage

Parameter [Fuzzing](../Hacking%20Concepts/Fuzzing.md):

```bash
└─$ ffuf -w /home/mdn0x/HTB/Machines/Planning/fuzzDicts/paramDict/AllParam.txt -u http://94.237.61.242:52839/get.php?x=FUZZ  -c 
```
## POST Parameters: Behind-the-Scenes Communication

While `GET` parameters are like open postcards, POST parameters are more like sealed envelopes, carrying their information discreetly within the body of the HTTP request. They are not visible directly in the URL, making them the preferred method for transmitting sensitive data like login credentials, personal information, or financial details.

When you submit a form or interact with a web page that uses POST requests, the following happens:

1. `Data Collection`: The information entered into the form fields is gathered and prepared for transmission.
    
2. `Encoding`: This data is encoded into a specific format, typically `application/x-www-form-urlencoded` or `multipart/form-data`:
    
    - `application/x-www-form-urlencoded`: This format encodes the data as key-value pairs separated by ampersands (`&`), similar to GET parameters but placed within the request body instead of the URL.
    - `multipart/form-data`: This format is used when submitting files along with other data. It divides the request body into multiple parts, each containing a specific piece of data or a file.
3. `HTTP Request`: The encoded data is placed within the body of an HTTP POST request and sent to the web server.
    
4. `Server-Side Processing`: The server receives the POST request, decodes the data, and processes it according to the application's logic.
    

Here's a simplified example of how a POST request might look when submitting a login form:

```http

POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=your_username&password=your_password
```

- `POST`: Indicates the HTTP method (POST).
- `/login`: Specifies the URL path where the form data is sent.
- `Content-Type`: Specifies how the data in the request body is encoded (`application/x-www-form-urlencoded` in this case).
- `Request Body`: Contains the encoded form data as key-value pairs (`username` and `password`).

```bash
ffuf -u http://94.237.61.242:52839/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v
```

The main difference here is the use of the `-d` flag, which tells `ffuf` that the payload ("`y=FUZZ`") should be sent in the request body as `POST` data.

Again, you'll see mostly invalid parameter responses. The correct value will stand out with its `200 OK` status code.
## Why Parameters Matter for Fuzzing

Parameters are the gateways through which you can interact with a web application. By manipulating their values, you can test how the application responds to different inputs, potentially uncovering vulnerabilities. For instance:

- Altering a product ID in a shopping cart URL could reveal pricing errors or unauthorized access to other users' orders.
- Modifying a hidden parameter in a request might unlock hidden features or administrative functions.
- Injecting malicious code into a search query could expose vulnerabilities like [Cross-site Scripting (XSS)](../Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md) or [SQL Injection](../Hacking%20Concepts/SQL%20Injection.md) (SQLi).

# Virtual Host and Subdomain Fuzzing

Both virtual hosting (vhosting) and [Subdomain](../Hacking%20Concepts/Subdomain.md)s play pivotal roles in organizing and managing web content.

Virtual hosting enables multiple websites or domains to be served from a single server or [IP address](../Hacking%20Concepts/IP%20address.md). Each vhost is associated with a unique domain name or hostname. When a client sends an HTTP request, the web server examines the `Host` header to determine which vhost's content to deliver. This facilitates efficient resource utilization and cost reduction, as multiple websites can share the same server infrastructure.

Subdomains, on the other hand, are extensions of a primary domain name, creating a hierarchical structure within the domain. They are used to organize different sections or services within a website. For example, `blog.example.com` and `shop.example.com` are subdomains of the main domain `example.com`. Unlike vhosts, subdomains are resolved to specific IP addresses through DNS (Domain Name System) records.

Example:

```
$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.inlanefreight.htb' -u http://inlanefreight.htb:33197 -
```
# Conclusion

FFuf is a powerful and efficient tool for web fuzzing, providing users with the ability to discover hidden resources and potential vulnerabilities in web applications. Its speed, flexibility, and customizable options make it an essential resource for penetration testers and security researchers. By utilizing the commands and options outlined, users can effectively conduct thorough fuzzing operations to enhance their security assessments.