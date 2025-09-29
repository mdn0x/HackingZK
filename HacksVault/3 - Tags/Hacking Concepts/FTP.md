File Transfer Protocol (`FTP`) is an Application Layer protocol that enables quick data transfer between computing devices. FTP can be utilized from the command-line, web browser, or through a graphical FTP client such as FileZilla. FTP itself is established as an insecure protocol, and most users have moved to utilize tools such as SFTP to transfer files through secure channels. As a note moving into the future, most modern web browsers have phased out support for FTP as of 2020.

When we think about communication between hosts, we typically think about a client and server talking over a single socket. Through this socket, both the client and server send commands and data over the same link. In this aspect, FTP is unique since it utilizes multiple ports at a time. FTP uses ports 20 and 21 over TCP. Port 20 is used for data transfer, while port 21 is utilized for issuing commands controlling the FTP session. In regards to authentication, FTP supports user authentication as well as allowing anonymous access if configured.

FTP is capable of running in two different modes, `active` or `passive`. Active is the default operational method utilized by FTP, meaning that the server listens for a control command `PORT` from the client, stating what port to use for data transfer. Passive mode enables us to access FTP servers located behind firewalls or a NAT-enabled link that makes direct TCP connections impossible. In this instance, the client would send the `PASV` command and wait for a response from the server informing the client what IP and port to utilize for the data transfer channel connection.

### Key Characteristics of FTP

1. **Client-Server Model**: FTP operates on a client-server model, where the client initiates a connection to the server to request file transfers.

2. **Two Modes of Operation**:
   - **Active Mode**: The client opens a random port and informs the server to connect back to that port for data transfer.
   - **Passive Mode**: The server opens a random port and the client connects to that port for data transfer. This mode is often used to bypass firewall restrictions.

3. **Control and Data Connections**: FTP uses two separate channels:
   - **Control Connection**: Established on port 21, this connection is used for sending commands and receiving responses.
   - **Data Connection**: Used for transferring files, this connection can be established on a different port.

4. **Authentication**: FTP typically requires a username and password for authentication, although anonymous access is also possible, allowing users to connect without credentials.

5. **File Management**: FTP allows users to perform various file management tasks, such as uploading, downloading, renaming, deleting, and listing files and directories.

The image above shows several examples of requests issued over the FTP command channel `green arrows`, and the responses sent back from the FTP server `blue arrows`. This is all pretty standard stuff. For a list of each command and what it is doing, check out the table below.

When looking at FTP traffic, some common commands we can see passed over port 21 include:

#### FTP Commands

|**Command**|**Description**|
|---|---|
|`USER`|specifies the user to log in as.|
|`PASS`|sends the password for the user attempting to log in.|
|`PORT`|when in active mode, this will change the data port used.|
|`PASV`|switches the connection to the server from active mode to passive.|
|`LIST`|displays a list of the files in the current directory.|
|`CWD`|will change the current working directory to one specified.|
|`PWD`|prints out the directory you are currently working in.|
|`SIZE`|will return the size of a file specified.|
|`RETR`|retrieves the file from the FTP server.|
|`QUIT`|ends the session.|

This is not an exhaustive list of the possible FTP control commands that could be seen. These can vary based on the FTP application or shell in use. For more information on FTP, see `RFC:959`.

### Security Considerations

While FTP is widely used, it has several security vulnerabilities:

1. **Unencrypted Data**: FTP transmits data, including usernames and passwords, in plain text, making it susceptible to eavesdropping and man-in-the-middle attacks.

2. **Lack of Integrity Checks**: FTP does not provide built-in mechanisms for ensuring data integrity during transfer.

### Secure Alternatives to FTP

To address the security concerns associated with FTP, several secure alternatives have been developed:

1. **FTPS (FTP Secure)**: This is an extension of FTP that adds support for the Transport Layer Security (TLS) and Secure Sockets Layer (SSL) protocols, providing encryption for data transfers.

2. **SFTP (SSH File Transfer Protocol)**: This protocol is part of the SSH (Secure Shell) protocol suite and provides secure file transfer capabilities over an encrypted connection.

3. **SCP (Secure Copy Protocol)**: Also part of the SSH protocol suite, SCP allows for secure file transfers between hosts on a network.

### Conclusion

File Transfer Protocol (FTP) is a fundamental protocol for transferring files over a network. While it is widely used for its simplicity and functionality, its lack of security features makes it vulnerable to various attacks. Organizations and individuals should consider using secure alternatives like FTPS or SFTP to protect sensitive data during file transfers. Understanding the characteristics, commands, and security implications of FTP is essential for effective file management and secure data handling.