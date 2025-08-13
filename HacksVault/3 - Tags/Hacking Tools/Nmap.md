## Overview of Nmap

**Nmap** (Network Mapper) is a powerful open-source tool used for network discovery and security auditing. It is widely utilized by network administrators, security professionals, and penetration testers to discover hosts and services on a computer network, thus creating a "map" of the network. Nmap can be used to identify open ports, running services, operating systems, and potential vulnerabilities.

### Key Features

- **Host Discovery**: Identify live hosts on a network.
- **Port Scanning**: Determine which ports are open on a target host.
- **Service Version Detection**: Identify the version of services running on open ports.
- **Operating System Detection**: Determine the operating system and hardware characteristics of network devices.
- **Scripting Engine**: Use scripts to automate various tasks, including vulnerability detection and exploitation.

### Installation

Nmap can be installed on various operating systems, including Windows, Linux, and macOS. On most Linux distributions, you can install it using the package manager:

```bash
sudo apt install nmap  # For Debian/Ubuntu
sudo yum install nmap  # For CentOS/RHEL
```

For Windows, you can download the installer from the [official Nmap website](https://nmap.org/download.html).

### Common Commands and Flags

#### 1. Basic Scanning

To perform a simple scan of a target IP address or hostname:

```bash
nmap <target_ip>
```

- **Example**:
  ```bash
  nmap 192.168.1.1
  ```

#### 2. Scanning Multiple Targets

You can scan multiple targets by specifying them in a comma-separated list or using a range:

```bash
nmap <target1>,<target2>,<target3>
nmap 192.168.1.1-10
```

#### 3. Port Scanning

To scan specific ports, use the `-p` option:

```bash
nmap -p <port_range> <target_ip>
```

- **Example**:
  ```bash
  nmap -p 22,80,443 192.168.1.1
  ```

#### 4. Service Version Detection

To detect the version of services running on open ports, use the `-sV` option:

```bash
nmap -sV <target_ip>
```

- **Example**:
  ```bash
  nmap -sV 192.168.1.1
  ```

#### 5. Operating System Detection

To attempt to determine the operating system of a target, use the `-O` option:

```bash
nmap -O <target_ip>
```

- **Example**:
  ```bash
  nmap -O 192.168.1.1
  ```

#### 6. Aggressive Scan

An aggressive scan combines several options, including OS detection, version detection, script scanning, and traceroute. Use the `-A` option:

```bash
nmap -A <target_ip>
```

- **Example**:
  ```bash
  nmap -A 192.168.1.1
  ```

#### 7. Scanning a Subnet

To scan an entire subnet, specify the CIDR notation:

```bash
nmap <subnet>
```

- **Example**:
  ```bash
  nmap 192.168.1.0/24
  ```

#### 8. Stealth Scanning

To perform a stealth scan (SYN scan), use the `-sS` option. This method is less likely to be logged by the target:

```bash
nmap -sS <target_ip>
```

- **Example**:
  ```bash
  nmap -sS 192.168.1.1
  ```

#### 9. UDP Scanning

To scan for open UDP ports, use the `-sU` option:

```bash
nmap -sU <target_ip>
```

- **Example**:
  ```bash
  nmap -sU 192.168.1.1
  ```

#### 10. Timing and Performance

You can adjust the timing of your scans using the `-T` option, which ranges from 0 (paranoid) to 5 (insane):

```bash
nmap -T<0-5> <target_ip>
```

- **Example**:
  ```bash
  nmap -T4 192.168.1.1
  ```

#### 11. Output Options

Nmap allows you to save the output in various formats using the `-oN`, `-oX`, `-oG`, and `-oA` options:

- **Normal Output**:
  ```bash
  nmap -oN output.txt <target_ip>
  ```

- **