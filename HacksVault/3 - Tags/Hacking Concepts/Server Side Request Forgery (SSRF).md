
Server-Side Request Forgery (SSRF) is a type of security vulnerability that allows an attacker to send crafted requests from a vulnerable server to internal or external resources. This can lead to unauthorized access to sensitive data, internal services, or even the exploitation of other vulnerabilities within the network. SSRF typically occurs when a web application accepts user input to make requests to other servers without proper validation or sanitization.

### How SSRF Works

In an SSRF attack, the attacker manipulates a server-side application to make requests to unintended locations. This can include:

- Accessing internal services that are not exposed to the public internet (e.g., databases, metadata services).
- Making requests to external services that the server can reach.
- Bypassing security controls that restrict direct access to certain resources.

### Common Scenarios for SSRF

1. **Internal Network Access**:
   - An attacker can exploit SSRF to access internal services that are not directly accessible from the internet, such as admin panels, databases, or other sensitive endpoints.

2. **Metadata Service Access**:
   - In cloud environments (e.g., AWS, Google Cloud), SSRF can be used to access metadata services that provide sensitive information, such as instance credentials or configuration data.

3. **Port Scanning**:
   - An attacker can use SSRF to perform port scanning on internal services, identifying open ports and services running on the internal network.

### Examples of SSRF

1. **Basic SSRF Example**:
   - A web application allows users to submit a URL to fetch and display content. If the application does not validate the input, an attacker can submit a request to an internal service:
     ```
     http://vulnerable-app.com/fetch?url=http://localhost:8080/admin
     ```

2. **Accessing Metadata in AWS**:
   - An attacker can exploit SSRF to access the AWS metadata service, which provides sensitive information about the instance:
     ```
     http://vulnerable-app.com/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
     ```

3. **Accessing Internal Database**:
   - If a web application allows users to specify a database connection string, an attacker could manipulate it to connect to an internal database:
     ```
     http://vulnerable-app.com/connect?db=postgresql://user:password@internal-db:5432/dbname
     ```

### Common Payloads for SSRF

1. **Localhost Access**:
   - Accessing services running on the same server:
     ```
     http://vulnerable-app.com/fetch?url=http://127.0.0.1:80
     ```

2. **Accessing Internal IPs**:
   - Targeting internal IP addresses:
     ```
     http://vulnerable-app.com/fetch?url=http://192.168.1.1
     ```

3. **Accessing Cloud Metadata**:
   - For AWS:
     ```
     http://vulnerable-app.com/fetch?url=http://169.254.169.254/latest/meta-data/
     ```
   - For Google Cloud:
     ```
     http://vulnerable-app.com/fetch?url=http://metadata.google.internal/computeMetadata/v1/
     ```

4. **Port Scanning**:
   - Attempting to access various ports on internal services:
     ```
     http://vulnerable-app.com/fetch?url=http://192.168.1.1:22
     ```

5. **Protocol Manipulation**:
   - Using different protocols to access services:
     ```
     http://vulnerable-app.com/fetch?url=file:///etc/passwd
     ```

### Prevention Measures

To mitigate SSRF vulnerabilities, developers should implement the following best practices:

- **Input Validation**: Validate and sanitize user input to ensure that only allowed URLs or IP addresses can be accessed. Use a whitelist approach to restrict requests to known safe endpoints.

- **Network Segmentation**: Limit the server's ability to access internal resources by implementing network segmentation and firewalls.

- **Use of Outbound Request Libraries**: Use libraries that provide built-in protections against SSRF, such as restricting requests to certain domains or IP ranges.

- **Logging and Monitoring**: Implement logging and monitoring to detect unusual outbound requests that may indicate an SSRF attack.

- **Disable Unnecessary Protocols**: If the application does not require certain protocols (e.g., file://), disable them to reduce the attack surface.

### Conclusion

Server-Side Request Forgery (SSRF) is a significant security vulnerability that can lead to unauthorized access to sensitive data and internal services. By understanding how SSRF works, recognizing common payloads, and implementing robust security measures, developers can significantly reduce the risk of exploitation and protect their applications from potential attacks.