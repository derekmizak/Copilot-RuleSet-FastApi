# GitHub Copilot: General Security Practices & Compliance

Coding-focused general security practices, including input validation, secrets management, API security, and code-level compliance considerations.

**Keywords**: #security #best-practices #validation #secrets #api-security #compliance #privacy

---

## 🛠️ General Security Practices

* **Input Validation**:
  * Implement server-side validation as the primary defense
  * Validate type, length, format, and range of all inputs
  * Use allowlists rather than denylists for validation
  * In FastAPI, leverage Pydantic for strict schema validation

* **Output Encoding**:
  * Apply context-aware encoding to prevent XSS
  * Sanitize data before inclusion in HTTP responses, emails, and storage

* **Secrets Management** (Code-Focused):
  * Never hardcode secrets or credentials in source code
  * Use environment variables or configuration services in code
  * For local development, load environment variables from `.env` files that are gitignored
  * Implement proper error handling that doesn't expose secrets in logs or error messages
  * Use appropriate abstraction patterns for credential handling

* **API Security**:
  * Implement robust authentication (OAuth2, JWT, API Keys)
  * Apply proper authorization checks
  * Implement rate limiting and throttling
  * Prevent mass assignment vulnerabilities
  * Include security headers in API responses

* **File Uploads**:
  * Validate file type, size, and name
  * Scan for malware
  * Store files outside the web root in non-executable locations
  * Generate new filenames to prevent path traversal
  * Process files asynchronously when possible

* **Dependency Management** (Code-Focused):
  * Use specific version constraints in requirements files
  * Remove unused dependencies from your project files
  * Implement code that gracefully handles library failures
  * Follow best practices for importing and using external packages
  * Write code that isn't tightly coupled to specific library versions

* **Container Security** (Code-Level Concerns):
  * Write code that handles permissions appropriately inside containers
  * Don't rely on root access in your application code
  * Implement proper error handling for containerized environments
  * Design your code to be environment-agnostic
  * Use non-elevated users in your application logic

* **Security Headers** (Code Implementation):
  * Add code to set appropriate security headers in your web applications
  * Implement Content-Security-Policy headers in a way that matches your application's needs
  * Set X-Content-Type-Options and similar headers in your HTTP responses
  * Programmatically add CORS headers that follow the principle of least privilege
  * Include proper cache control headers for sensitive responses

* **Security Testing** (Code Level):
  * Write unit tests for security-critical components
  * Implement tests for input validation and output encoding
  * Test boundary cases that might lead to security issues
  * Add tests for authentication and authorization logic
  * Verify that error handling doesn't leak sensitive information

---

## 📊 Code-Level Compliance & Privacy

* **Data Protection in Code**:
  * Implement data validation before storing or processing
  * Apply field-level encryption for sensitive data
  * Mask or redact sensitive information in logs and outputs
  * Follow data minimization principles in your data models
  * Add appropriate comments for regulatory compliance context

* **Privacy by Design**:
  * Build consent mechanisms into user flows
  * Implement proper data deletion functionality
  * Create audit logging for access to sensitive data
  * Design with data portability in mind
  * Include features to allow users to view and export their data

* **Audit & Logging**:
  * Implement detailed logging for security-related events
  * Include relevant contextual information in log entries
  * Use structured logging formats (JSON) for easier analysis
  * Store logs securely and for appropriate retention periods
  * Ensure log integrity through append-only mechanisms

* **Secure Communication**:
  * Enforce TLS for all external communications
  * Implement certificate validation for client-server connections
  * Use secure protocols for internal service communication
  * Validate certificates and check for revocation
  * Implement proper certificate management in your code

* **Access Control Implementation**:
  * Implement attribute-based or role-based access control
  * Enforce principle of least privilege in your code
  * Create fine-grained permissions for API endpoints
  * Validate permissions on both client and server sides
  * Implement proper session management with timeouts

---

## See Also
- [Security Principles & OWASP Top 10](/security/security-principles.md)
- [Security Implementation Examples](/security/security-implementation-examples.md)
- [Secure API Design Patterns](/security/security-api-design.md)
