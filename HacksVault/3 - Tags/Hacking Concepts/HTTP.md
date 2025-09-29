Hypertext Transfer Protocol (`HTTP`) is a stateless Application Layer protocol that has been in use since 1990. HTTP enables the transfer of data in clear text between a client and server over TCP. The client would send an HTTP request to the server, asking for a resource. A session is established, and the server responds with the requested media (HTML, images, hyperlinks, video). HTTP utilizes ports 80 or 8000 over TCP during normal operations. In exceptional circumstances, it can be modified to use alternate ports, or even at times, UDP.

### Key Characteristics of HTTP

1. **Client-Server Model**: HTTP operates on a client-server model, where the client (usually a web browser) sends requests to a server, which then responds with the requested resources.

2. **Stateless Protocol**: HTTP is stateless, meaning that each request from a client to a server is treated as an independent transaction. The server does not retain any information about previous requests, which simplifies the protocol but requires additional mechanisms (like cookies or sessions) for maintaining state.

3. **Request-Response Cycle**: The communication process involves a request from the client and a response from the server. The client sends an HTTP request, and the server processes it and returns an HTTP response.

4. **Methods**: HTTP defines several request methods, each serving a different purpose:
   - **GET**: Requests data from a specified resource. It is used to retrieve information without causing any side effects.
   - **POST**: Submits data to be processed to a specified resource. It is commonly used for form submissions.
   - **PUT**: Updates a specified resource with the provided data.
   - **DELETE**: Removes a specified resource.
   - **HEAD**: Similar to GET, but it retrieves only the headers of a resource, not the body.

5. **Status Codes**: HTTP responses include status codes that indicate the result of the request. Common status codes include:
   - **200 OK**: The request was successful.
   - **404 Not Found**: The requested resource could not be found.
   - **500 Internal Server Error**: The server encountered an unexpected condition.

#### HTTP Methods

To perform operations such as fetching webpages, requesting items for download, or posting your most recent tweet all require the use of specific methods. These methods define the actions taken when requesting a URI. Methods:

|**Method**|**Description**|
|---|---|
|`HEAD`|`required` is a safe method that requests a response from the server similar to a Get request except that the message body is not included. It is a great way to acquire more information about the server and its operational status.|
|`GET`|`required` Get is the most common method used. It requests information and content from the server. For example, `GET http://10.1.1.1/Webserver/index.html` requests the index.html page from the server based on our supplied URI.|
|`POST`|`optional` Post is a way to submit information to a server based on the fields in the request. For example, submitting a message to a Facebook post or website forum is a POST action. The actual action taken can vary based on the server, and we should pay attention to the response codes sent back to validate the action.|
|`PUT`|`optional` Put will take the data appended to the message and place it under the requested URI. If an item does not exist there already, it will create one with the supplied data. If an object already exists, the new PUT will be considered the most up-to-date, and the object will be modified to match. The easiest way to visualize the differences between PUT and POST is to think of it like this; PUT will create or update an object at the URI supplied, while POST will create child entities at the provided URI. The action taken can be compared with the difference between creating a new file vs. writing comments about that file on the same page.|
|`DELETE`|`optional` Delete does as the name implies. It will remove the object at the given URI.|
|`TRACE`|`optional` Allows for remote server diagnosis. The remote server will echo the same request that was sent in its response if the TRACE method is enabled.|
|`OPTIONS`|`optional` The Options method can gather information on the supported HTTP methods the server recognizes. This way, we can determine the requirements for interacting with a specific resource or server without actually requesting data or objects from it.|
|`CONNECT`|`optional` Connect is reserved for use with Proxies or other security devices like firewalls. Connect allows for tunneling over HTTP. (`SSL tunnels`)|

Notice that we have `required` or `optional` listed beside each method. As a requirement by the standard, GET and HEAD must always work and exist with standard HTTP implementations. This is true only for them. The methods trace, options, delete, put and post are optional functionalities one can allow. An example of this is a read-only webpage like a blog post. The client PC can request a resource from the page but not modify, add, or delete the resource or resources.

For more information on HTTP as a protocol or how it operates, see `RFC:2616`.

### HTTP Headers

HTTP headers are key-value pairs sent in both requests and responses that provide additional information about the request or response. Common headers include:

- **Content-Type**: Indicates the media type of the resource (e.g., `text/html`, `application/json`).
- **User-Agent**: Identifies the client software making the request (e.g., web browser).
- **Accept**: Specifies the media types that the client is willing to receive.
- **Authorization**: Contains credentials for authenticating the client with the server.

### Security Considerations

While HTTP is widely used, it has several security vulnerabilities:

1. **Unencrypted Communication**: Data transmitted over HTTP is sent in plain text, making it susceptible to eavesdropping and man-in-the-middle attacks.

2. **Lack of Integrity Checks**: HTTP does not provide mechanisms to ensure the integrity of the data being transmitted.

### Secure Alternative: HTTPS

To address the security concerns associated with HTTP, HTTPS (HTTP Secure) was developed. HTTPS combines HTTP with Transport Layer Security (TLS) or Secure Sockets Layer (SSL) to provide encryption and secure communication over the internet. Key features of HTTPS include:

- **Encryption**: Data is encrypted during transmission, protecting it from eavesdropping.
- **Authentication**: HTTPS verifies the identity of the server, ensuring that clients are communicating with the intended server.
- **Data Integrity**: HTTPS ensures that data is not altered during transmission.

HTTP Secure (`HTTPS`) is a modification of the HTTP protocol designed to utilize Transport Layer Security (`TLS`) or Secure Sockets Layer (`SSL`) with older applications for data security. TLS is utilized as an encryption mechanism to secure the communications between a client and a server. TLS can wrap regular HTTP traffic within TLS, which means that we can encrypt our entire conversation, not just the data sent or requested. Before the TLS mechanism was in place, we were vulnerable to Man-in-the-middle attacks and other types of reconnaissance or hijacking, meaning anyone in the same LAN as the client or server could view the web traffic if they were listening on the wire. We can now have security implemented in the browser enabling everyone to encrypt their web habits, search requests, sessions or data transfers, bank transactions, and much more.

Even though it is HTTP at its base, HTTPS utilizes ports 443 and 8443 instead of the standard port 80. This is a simple way for the client to signal the server that it wishes to establish a secure connection. Let's look at an output of HTTPS traffic and discern how a `TLS handshake` functions for a minute.

Once a session is initiated via TCP, a TLS ClientHello is sent next to begin the TLS handshake. During the handshake, several parameters are agreed upon, including session identifier, peer x509 certificate, compression algorithm to be used, the cipher spec encryption algorithm, if the session is resumable, and a 48-byte master secret shared between the client and server to validate the session.

Once the session is established, all data and methods will be sent through the TLS connection and appear as TLS Application Data

To summarize the handshake:

1. Client and server exchange hello messages to agree on connection parameters.
2. Client and server exchange necessary cryptographic parameters to establish a premaster secret.
3. Client and server will exchange x.509 certificates and cryptographic information allowing for authentication within the session.
4. Generate a master secret from the premaster secret and exchanged random values.
5. Client and server issue negotiated security parameters to the record layer portion of the TLS protocol.
6. Client and server verify that their peer has calculated the same security parameters and that the handshake occurred without tampering by an attacker.

Encryption in itself is a complex and lengthy topic that deserves its own module. This section is a simple summary of how HTTP and TLS provide security within the HTTPS application protocol. For more information on how HTTPS functions and how TLS performs security operations, see `RFC:2246`.
### Conclusion

Hypertext Transfer Protocol (HTTP) is a fundamental protocol for data communication on the World Wide Web. While it enables the transfer of various types of content, its lack of security features makes it vulnerable to attacks. The adoption of HTTPS provides a secure alternative, ensuring encrypted communication and protecting sensitive data during transmission. Understanding HTTP and its characteristics is essential for web development, security, and effective data communication on the internet.