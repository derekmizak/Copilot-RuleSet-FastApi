# GitHub Copilot: Security Principles & OWASP Top 10

Coding-focused security guidelines focusing on core security principles and addressing the OWASP Top 10 vulnerabilities.

**Keywords**: #security #principles #owasp #top10 #secure-coding

---

## 🛡️ Coding Security Principles

* **Least Privilege in Code**: Only request and use the minimum permissions needed
* **Defense in Depth**: Implement multiple validation layers in your code
* **Secure Defaults**: Write code with secure default parameters and settings
* **Fail Securely**: Ensure exceptions and errors leave the application in a secure state
* **Clean Code**: Remove debugging code, commented-out code, and unnecessary functions
* **Validate All Inputs**: Never trust data from external sources, including APIs and users

---

## 🔑 OWASP Top 10 Focus (Apply to Web/APIs)

* **A01: Broken Access Control**
  * Enforce access control on every request
  * Implement deny-by-default principle for authorizations
  * Check record ownership before operations
  * Prevent Insecure Direct Object References (IDOR)
  * Implement proper Cross-Origin Resource Sharing (CORS) policies
  * Use rate limiting to prevent brute force attacks

* **A02: Cryptographic Failures** (Code-Focused)
  * Use strong, modern cryptographic algorithms and libraries in code
  * For sensitive data, use proper encryption libraries with secure defaults
  * In code, use environment variables or configuration services to retrieve secrets
  * Never hardcode cryptographic keys, passwords or sensitive values in source code
  * When working with cookies, set secure attributes programmatically
  * Implement token expiration and validation in authentication code

* **A03: Injection**
  * **SQLi**: Use parameterized queries or ORMs; never use dynamic SQL with user input
  * **NoSQLi**: Implement proper data sanitization and validation
  * **XSS**: Apply contextual output encoding and server-side input sanitization
    * Use safe templating engines with auto-escaping (e.g., Jinja2)
    * Implement Content Security Policy (CSP)
  * **OS Command Injection**: Avoid OS commands; if necessary, use safe APIs with strict validation
  * **LDAP/XML Injection**: Apply input sanitization and validation

* **A04: Insecure Design** (Code Implementation)
  * Code using established security design patterns
  * Implement proper separation of concerns in your code
  * Add parameter validation at the beginning of functions
  * Create reusable security components rather than ad-hoc implementations

* **A05: Security Misconfiguration** (Code-Level)
  * Remove default accounts and credentials from code
  * Implement robust error handling that limits exposure
  * Use up-to-date dependencies and libraries
  * Run applications with least privilege
  * Set secure values for HTTP headers programmatically

* **A06: Vulnerable and Outdated Components**
  * Create a process to keep dependencies updated in your package manager
  * Subscribe to security bulletins for used components
  * Remove unused dependencies and features from code and configurations
  * Only use official/trusted component sources
  * Implement Content Security Policy to mitigate XSS exploits

* **A07: Identification and Authentication Failures**
  * Enforce strong password policies
  * Implement account lockout against brute force
  * Use multi-factor authentication when needed
  * Follow secure token creation/validation standards (OAUTH, JWT)
  * Encrypt/hash credentials with strong algorithms

* **A08: Software and Data Integrity Failures**
  * Verify input data meets integrity checks
  * Implement digital signatures when exchanging data between systems
  * Use build pipelines that verify component signatures
  * Prevent deserialization of untrusted data

* **A09: Logging and Monitoring Failures**
  * Log security-relevant events (login failures, access control failures)
  * Ensure logs have appropriate context for investigation
  * Follow the principle of defense in depth with multiple layers of logging
  * Implement automated alerts for suspicious activities
  * Sanitize sensitive data in logs; never log credentials

* **A10: Server-Side Request Forgery (SSRF)**
  * Sanitize and validate all user-supplied input, especially URLs
  * Implement positive allowlisting for URL validation
  * Disable HTTP redirects where possible
  * Never forward raw user input to file API calls
  * Prevent sensitive data exposure in error messages

---

## See Also
- [General Security Practices](/security/security-general-practices.md)
- [Security Implementation Examples](/security/security-implementation-examples.md)
- [Secure API Design Patterns](/security/security-api-design.md)
