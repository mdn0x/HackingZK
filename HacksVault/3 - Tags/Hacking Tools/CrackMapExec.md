**[CrackMapExec](CrackMapExec.md)** is a well-known network service exploitation tool that we will use throughout this module. It allows us to perform enumeration, command execution, and post-exploitation attacks in Windows environments. It supports various network protocols, such as SMB, LDAP, RDP, and SSH.
## Overview of CrackMapExec

CrackMapExec (CME) is a powerful tool designed for penetration testing, particularly in environments with Windows workstations and servers. It automates various tasks related to network security assessments, making it a versatile addition to any penetration tester's toolkit.

### Key Features

- **Multi-Protocol Support**: CME works with various network protocols, including SMB, WinRM, and RDP.
- **Credential Management**: It can store and correlate credentials, allowing users to track access across multiple systems.
- **Post-Exploitation**: CME facilitates tasks like password spraying, remote command execution, and data extraction from Active Directory.
## Installation

### Recommended Installation Methods

|Method|Command|Notes|
|---|---|---|
|**Using Pipx**|`python3 -m pip install pipx``pipx install crackmapexec`|Isolates dependencies, reducing issues.|
|**Using Docker**|`docker pull byt3bl33d3r/crackmapexec`|Quick setup in a containerized environment.|
|**From Source**|`git clone --recursive https://github.com/byt3bl33d3r/CrackMapExec``cd CrackMapExec``poetry install`|Requires additional dependencies.|
### Dependencies

- Python 3
- Various Python libraries (e.g., `impacket`, `requests`, `paramiko`)
## Usage

CME is often described as a "Swiss Army Knife" for network testing. It allows users to perform tasks such as:

- Scanning networks for vulnerabilities
- Dumping password hashes
- Executing commands on remote machines

While primarily used for offensive security, it can also assist blue teams in assessing account privileges and identifying misconfigurations.

CrackMapExec is continuously evolving, with community contributions and forks available for users seeking the latest features.