Is an attack technique where a small set of common passwords is tested across many accounts. Unlike brute-force attacks, password spraying avoids account lockouts by testing each account with only a few attempts, exploiting poor password practices common in many organisations. Password spraying is often effective because many organisations:

- Require frequent password changes, leading users to pick predictable patterns (for example, `Summer2025!`).
- Don't enforce their policies well.
- Reuse common passwords across multiple accounts.

Common password lists for spraying include:

- Seasonal passwords.
- Default passwords used by IT teams (`Password123`).
- Passwords leaked in data breaches, like `rockyou.txt`.
Password spraying is a type of cyber attack where an attacker attempts to gain unauthorized access to a large number of accounts by trying a few common passwords against many usernames. This method is effective because it avoids account lockouts that can occur with traditional brute-force attacks, where multiple incorrect attempts are made on a single account.

### Key Characteristics of Password Spraying

- **Common Passwords**: Attackers often use widely known passwords (e.g., "Password123", "123456", "qwerty") to maximize their chances of success.
- **Targeting Multiple Accounts**: Instead of focusing on one account, the attacker targets many accounts, which reduces the risk of detection.
- **Low and Slow Approach**: The attack is typically conducted slowly to avoid triggering security alerts or account lockout mechanisms.

### Tools Used in Password Spraying

Several tools can facilitate password spraying attacks, including:

1. **Hydra**: A popular password-cracking tool that supports various protocols and can perform password spraying.
2. **Medusa**: Similar to Hydra, it is designed for speed and can handle multiple protocols.
3. **Ncrack**: A network authentication cracking tool that can be used for password spraying against network services.
4. **Burp Suite**: While primarily a web application security testing tool, it can be configured to perform password spraying against web applications.
5. **PowerShell**: Scripts can be written in PowerShell to automate password spraying against Windows accounts.

### Prevention Measures

To defend against password spraying attacks, organizations can implement several strategies:

- **Account Lockout Policies**: Set thresholds for failed login attempts to lock accounts temporarily.
- **Multi-Factor Authentication (MFA)**: Require additional verification methods beyond just passwords.
- **User Education**: Train users to create strong, unique passwords and recognize phishing attempts.
- **Monitoring and Logging**: Keep track of login attempts and alert on suspicious activities.

Understanding password spraying and employing preventive measures can significantly enhance an organization's security posture.