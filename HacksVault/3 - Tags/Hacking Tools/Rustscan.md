[Nmap](./Nmap.md)
## Rustscan Overview

Rustscan is a high-performance port scanner designed to quickly identify open ports on target systems. Written in the Rust programming language, it leverages asynchronous I/O to achieve remarkable speed and efficiency, making it a preferred choice for security professionals and penetration testers.

### Key Features

- **Speed**: Rustscan is optimized for rapid scanning, capable of scanning thousands of ports in seconds.
- **Integration with Nmap**: It can seamlessly pass open port results to Nmap for detailed service enumeration and vulnerability scanning.
- **Customizable Scans**: Users can specify port ranges, set timeouts, and adjust concurrency levels.
- **User-Friendly Output**: The output is clear and concise, displaying open ports and associated services.
- **Open Source**: Being open-source allows for community contributions and customization.

### Installation

To install Rustscan, you can use the following command if you have Rust installed:

```bash
cargo install rustscan
```

Alternatively, you can download pre-built binaries from the [Rustscan GitHub repository](https://github.com/RustScan/RustScan/releases).

### Basic Commands

Here are some common commands and options for using Rustscan:

#### 1. Basic Port Scan

To perform a basic scan on a target IP address:

```bash
rustscan -a <target_ip>
```

#### 2. Specifying Port Range

To scan a specific range of ports, use the `-p` option:

```bash
rustscan -a <target_ip> -p 1-1000
```

#### 3. Scanning Multiple Targets

You can scan multiple targets by providing a comma-separated list:

```bash
rustscan -a <target_ip1>,<target_ip2>
```

#### 4. Using Nmap Integration

To automatically pass the results to Nmap for further analysis, use the `--nmap` option:

```bash
rustscan -a <target_ip> --nmap
```

#### 5. Setting Timeout and Concurrency

You can adjust the timeout and the number of concurrent connections with the `--timeout` and `--rate` options:

```bash
rustscan -a <target_ip> --timeout 1 --rate 1000
```

### Example Usage

Here’s an example of a complete command that scans a target IP for open ports in the range of 1 to 1000 and then uses Nmap for detailed scanning:

```bash
rustscan -a 192.168.1.1 -p 1-1000 --nmap
```

### Conclusion

Rustscan is a powerful tool for quickly identifying open ports on target systems, making it an essential asset for penetration testers and security professionals. Its speed, ease of use, and integration with Nmap enhance its effectiveness in security assessments. By utilizing the commands and options outlined, users can efficiently gather valuable information about their network environments.