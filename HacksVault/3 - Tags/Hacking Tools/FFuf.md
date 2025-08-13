## FFuf Overview

FFuf (Fuzz Faster U Fool) is a fast and flexible web fuzzing tool designed for discovering hidden resources on web servers. It is particularly useful for penetration testers and security researchers looking to identify vulnerabilities, such as unprotected directories, files, and parameters in web applications.

### Key Features

- **Speed**: FFuf is optimized for performance, allowing users to conduct fuzzing operations quickly, even against large target sets.

- **Customizable Payloads**: Users can specify custom wordlists for fuzzing, enabling targeted searches for specific resources or vulnerabilities.

- **Multiple Request Methods**: FFuf supports various HTTP request methods, including GET, POST, PUT, and DELETE, allowing for comprehensive testing of web applications.

- **JSON Output**: The tool can output results in JSON format, making it easy to parse and analyze the findings programmatically.

- **Filters and Matchers**: FFuf provides options to filter results based on status codes, response sizes, and other criteria, helping users focus on relevant findings.

### Installation

FFuf can be installed easily using Go, as it is written in the Go programming language. Here’s how to install it:

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

### Example Usage

Here’s an example of a complete command that fuzzes a target URL for hidden directories:

```bash
ffuf -u http://example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc all
```

This command will search for all matching resources using the common directory wordlist.

### Conclusion

FFuf is a powerful and efficient tool for web fuzzing, providing users with the ability to discover hidden resources and potential vulnerabilities in web applications. Its speed, flexibility, and customizable options make it an essential resource for penetration testers and security researchers. By utilizing the commands and options outlined, users can effectively conduct thorough fuzzing operations to enhance their security assessments.