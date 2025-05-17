<!-- filepath: /security/security-rules.md -->
# GitHub Copilot: Security Rules

A collection of structured rule directives for GitHub Copilot to follow when addressing security concerns in code.

**Keywords**: #security #rules #secure-coding #owasp #best-practices

---

## Security Rule Directives

### General Security Principles

```
@security Rule - Validate All Input: Always validate and sanitize all input from external sources, including user input, API responses, and configuration files.
```

```
@security Rule - Least Privilege: Implement the principle of least privilege in all code - only request and use the minimum permissions needed.
```

```
@security Rule - Defense in Depth: Never rely on a single security control; implement multiple validation layers.
```

```
@security Rule - Secure Defaults: Write code with secure default parameters and settings that require explicit opt-out for less secure options.
```

### OWASP Top 10 Mitigations

```
@security Rule - SQL Injection Prevention: Always use parameterized queries, prepared statements, or ORMs instead of string concatenation for database queries.
```

```
@security Rule - XSS Prevention: Escape output and use context-specific encoding when displaying user-controlled data in web applications.
```

```
@security Rule - CSRF Protection: Implement anti-CSRF tokens for state-changing operations in web applications.
```

```
@security Rule - Secure Authentication: Use industry standard authentication methods and libraries; never implement custom authentication schemes.
```

### API Security

```
@security Rule - API Authorization: Implement proper authorization checks on all API endpoints, even if they seem non-sensitive.
```

```
@security Rule - Rate Limiting: Implement rate limiting on API endpoints to prevent abuse and DoS attacks.
```

```
@security Rule - Sensitive Data Exposure: Never log sensitive data and always encrypt it in transit and at rest.
```

### Implementation Security

```
@security Rule - Secret Management: Never hardcode secrets, API keys, or credentials in source code; use environment variables or dedicated secret management solutions.
```

```
@security Rule - Dependency Management: Regularly update dependencies and scan for vulnerabilities using tools like dependabot or snyk.
```

```
@security Rule - Error Handling: Implement proper error handling that doesn't expose sensitive information in error messages to users.
```

```
@security Rule - Secure Logging: Implement logging for security-relevant events but ensure sensitive data is never logged.
```

---

## Usage Instructions

To use these rules in your GitHub Copilot instructions, copy the appropriate rule directives into your instruction file or include this file using:

```markdown
<!-- include: /security/security-rules.md -->
```
