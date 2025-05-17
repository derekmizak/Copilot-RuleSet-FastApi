# Security Best Practices Index

This file serves as an index for the security best practices instruction files. Based on your task, you can choose the most relevant files to load.

#security #index #best-practices #owasp #secure-coding

---

## 📑 Available Security Instruction Files

### Core Security Concepts

- [Security Principles](/security/security-principles.md) - Core security principles and OWASP Top 10 vulnerabilities.
  - Keywords: `#security #principles #owasp #top10 #vulnerabilities`
  - When to use: For foundational security knowledge and understanding the most critical security risks.

- [Security General Practices](/security/security-general-practices.md) - General security practices applicable to most applications.
  - Keywords: `#security #general #practices #input-validation #secrets-management #compliance #privacy`
  - When to use: For implementation of common security practices like input validation, output encoding, and compliance considerations.

### Implementation-Focused Guides

- [Security Implementation Examples](/security/security-implementation-examples.md) - Practical code examples for implementing security features.
  - Keywords: `#security #implementation #code #examples #practical`
  - When to use: When you need ready-to-use code patterns for common security implementations like password hashing, JWT handling, or SQL injection prevention.

- [Security API Design](/security/security-api-design.md) - Patterns and code examples for secure API design.
  - Keywords: `#security #api #design #patterns #auth #jwt #oauth`
  - When to use: When building APIs and needing patterns for authentication, authorization, rate limiting, and secure API responses.

- [Security Rules](/security/security-rules.md) - Structured rule directives for GitHub Copilot to follow.
  - Keywords: `#security #rules #directives #guidelines #standards`
  - When to use: When you need specific, structured guidance for GitHub Copilot when implementing security features.

## 🎯 How to Choose the Right Files

1. **For general security guidance:**
   - Start with [Security Principles](/security/security-principles.md)
   - Then add [Security General Practices](/security/security-general-practices.md)

2. **When implementing specific security features:**
   - Load [Security Implementation Examples](/security/security-implementation-examples.md)
   - Reference the specific section within the file that corresponds to your task

3. **When building APIs:**
   - Include [Security API Design](/security/security-api-design.md)
   - For complete API security, also include [Security Principles](/security/security-principles.md) for OWASP awareness

4. **For web application security:**
   - Start with [Security Principles](/security/security-principles.md) (OWASP Top 10)
   - Add [Security Implementation Examples](/security/security-implementation-examples.md) (XSS Prevention and File Upload sections)

5. **For database access security:**
   - Load [Security Implementation Examples](/security/security-implementation-examples.md) (SQL Injection Prevention section)
   - Add [Security General Practices](/security/security-general-practices.md) (for general input validation)

6. **For specific GitHub Copilot guidance on security:**
   - Load [Security Rules](/security/security-rules.md) for structured directives
   - Combine with specific implementation examples as needed

## 🔍 Security Topics Overview

| Topic | Covered In | Key Concepts |
|-------|------------|--------------|
| OWASP Top 10 | Security Principles | Broken Access Control, Cryptographic Failures, Injection, etc. |
| Authentication | Security Implementation Examples, Security API Design | Password hashing, JWT, OAuth 2.0 |
| Authorization | Security API Design | Role-based access, Scopes, API permissions |
| Input Validation | Security General Practices, Security Implementation Examples | Server-side validation, Allowlists |
| SQL Injection | Security Implementation Examples | Parameterized queries, ORM usage |
| XSS Prevention | Security Implementation Examples | Content Security Policy, Output encoding |
| File Upload Security | Security Implementation Examples | Content validation, Secure storage |
| API Security | Security API Design | Rate limiting, Security headers, Versioning |
| Secrets Management | Security General Practices | Environment variables, Configuration services |
| Compliance & Privacy | Security General Practices | Data protection, Privacy by design |

## Using Security Rules

The **Security Rules** file contains structured directives that GitHub Copilot can follow. These rules use the format:

```markdown
@security Rule - RuleName: RuleDescription
```

Example usage in your Copilot instructions:

```markdown
<instructions>
@security Rule - Validate All Input: Always validate and sanitize all input from external sources.
@security Rule - SQL Injection Prevention: Always use parameterized queries or ORMs instead of string concatenation.
</instructions>
```

You can combine security rules with other topic rules for cross-domain projects:

```markdown
<instructions>
@security Rule - Validate All Input: Always validate and sanitize all input from external sources.
@fastapi Rule - Response Models: Define explicit response models using Pydantic.
</instructions>
```

---

## See Also

- [Python Core Principles](/python/python-core-principles.md)
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [Main Index](/index.md)
