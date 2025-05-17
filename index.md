# GitHub Copilot Instruction Files Index

This is the main index file for the GitHub Copilot instruction files. It helps you navigate and select the appropriate instruction files based on your current task.

#index #github-copilot #instructions #programming #best-practices

---

## 📚 Instruction Collections

The instruction files are organized into the following categories:

### Python Development

- [Python Index](/python/index.md) - Best practices for Python development
  - [Python Core Principles](/python/python-core-principles.md)
  - [Python Code Quality](/python/python-code-quality.md)
  - [Python Performance](/python/python-performance.md)
  - [Python Rules](/python/python-rules.md) - Structured rule directives for Python

### FastAPI Development

- [FastAPI Index](/fastapi/index.md) - Instructions for FastAPI application development
  - [FastAPI Foundations](/fastapi/fastapi-foundations.md)
  - [FastAPI Documentation](/fastapi/fastapi-documentation.md)
  - [FastAPI Testing](/fastapi/fastapi-testing.md)
  - [FastAPI Performance](/fastapi/fastapi-performance.md)
  - [FastAPI Framework Rules](/fastapi/fastapi-framework-rules.md)
  - [FastAPI Error Handling](/fastapi/fastapi-error-handling.md)
  - [FastAPI Advanced Patterns](/fastapi/fastapi-advanced-patterns.md)
  - [FastAPI API & DB Standards](/fastapi/fastapi-api-db-standards.md)
  - [FastAPI GraphQL](/fastapi/fastapi-graphql.md)
  - [FastAPI Deployment](/fastapi/fastapi-deployment.md)
  - [FastAPI Rules](/fastapi/fastapi-rules.md) - Structured rule directives for FastAPI

### Security Best Practices

- [Security Index](/security/index.md) - Security-focused guidelines and implementations
  - [Security Principles](/security/security-principles.md)
  - [Security General Practices](/security/security-general-practices.md)
  - [Security Implementation Examples](/security/security-implementation-examples.md)
  - [Security API Design](/security/security-api-design.md)
  - [Security Rules](/security/security-rules.md) - Structured rule directives for security

## 🔍 How to Use This Collection

### File Selection Strategy

For optimal results with GitHub Copilot, load only the instruction files relevant to your current task. This minimizes token usage and ensures Copilot focuses on the most relevant guidance.

For example:

1. **If building a basic Python application:**
   - Load [Python Core Principles](/python/python-core-principles.md)
   - Add [Python Code Quality](/python/python-code-quality.md)

2. **If developing a FastAPI web application:**
   - Load [FastAPI Foundations](/fastapi/fastapi-foundations.md)
   - Add [FastAPI API & DB Standards](/fastapi/fastapi-api-db-standards.md)
   - Add [Security Principles](/security/security-principles.md)

3. **If focusing on API security:**
   - Load [Security API Design](/security/security-api-design.md)
   - Add [Security Implementation Examples](/security/security-implementation-examples.md)
   - Add [FastAPI Security Best Practices](/fastapi/fastapi-framework-rules.md)

### Using Structured Rule Directives

This collection includes structured rule directives in the format:

```markdown
@topicName Rule - RuleName: RuleDescription
```

These directives help GitHub Copilot understand specific guidelines for different technologies. You can:

1. **Include entire rule files** for comprehensive guidance on a topic
2. **Copy specific rules** when you need targeted guidance
3. **Combine rules from different topics** for cross-domain projects

Example rule usage in your Copilot instructions:

```markdown
<instructions>
@python Rule - Follow PEP 8: When writing Python code, always follow PEP 8 style guidelines.
@security Rule - Validate All Input: Always validate and sanitize all input from external sources.
@fastapi Rule - Response Models: Define explicit response models using Pydantic.
</instructions>
```

### Task-Based File Selection Examples

| Task | Recommended Instruction Files |
|------|------------------------------|
| Building a new Python script | Python Core Principles, Python Code Quality |
| Creating a FastAPI application | FastAPI Foundations, FastAPI Documentation, Security Principles |
| Implementing user authentication | Security Implementation Examples, Security API Design |
| Optimizing Python code | Python Performance |
| Setting up API endpoints | FastAPI API & DB Standards, Security API Design |
| Deploying a FastAPI app | FastAPI Deployment |
| Securing database operations | Security Implementation Examples, FastAPI API & DB Standards |
| Adding GraphQL to FastAPI | FastAPI GraphQL |
| Error handling in FastAPI | FastAPI Error Handling |

## 🏷️ Keywords and Tags

Use these keywords to find relevant instruction files quickly:

- **Python:** #python #pythonic #clean-code #performance #optimization
- **FastAPI:** #fastapi #api #async #pydantic #endpoints #documentation #openapi
- **Security:** #security #owasp #injection #authentication #authorization #encryption

---

## Maintenance Notes

- Last updated: 2025-05-17
- These instruction files are designed to be modular and focused
- Each file contains cross-references to related files
- Keywords are included to help with file selection
- Structured rule directives have been added to improve Copilot guidance
