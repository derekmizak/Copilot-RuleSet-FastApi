# GitHub Copilot: FastAPI Instruction Files

This directory contains specialized instruction files for FastAPI development with GitHub Copilot. Each file focuses on a specific aspect of FastAPI development to optimize token usage and provide more targeted guidance.

## Available Instruction Files

1. **[FastAPI Foundations](fastapi-foundations.md)**
   - Coding foundations for FastAPI
   - Project structure and architecture
   - Router organization
   - **Keywords**: #fastapi #python #project-structure #architecture #foundations

2. **[FastAPI Documentation](fastapi-documentation.md)**
   - API documentation in code
   - Pydantic schema documentation
   - OpenAPI customization
   - **Keywords**: #fastapi #documentation #openapi #swagger #pydantic #schema

3. **[FastAPI Testing](fastapi-testing.md)**
   - API endpoint testing
   - Dependency overrides
   - Async testing
   - Mocking
   - **Keywords**: #fastapi #testing #pytest #testclient #async-testing

4. **[FastAPI Performance](fastapi-performance.md)**
   - Async database operations
   - Efficient query patterns
   - Response streaming
   - Caching
   - **Keywords**: #fastapi #performance #optimization #async #streaming #database

5. **[FastAPI Framework Rules](fastapi-framework-rules.md)**
   - Core FastAPI patterns
   - Dependency injection
   - Path operations
   - Routers and middleware
   - **Keywords**: #fastapi #framework #dependency-injection #pydantic #routers #path-operations

6. **[FastAPI Error Handling](fastapi-error-handling.md)**
   - Centralized error handling
   - Custom exception handlers
   - Middleware for cross-cutting concerns
   - **Keywords**: #fastapi #error-handling #middleware #exception-handlers #logging #cors

7. **[FastAPI Advanced Patterns](fastapi-advanced-patterns.md)**
   - WebSockets
   - Background tasks
   - File upload with validation
   - **Keywords**: #fastapi #advanced #websockets #background-tasks #file-upload #real-time

8. **[FastAPI API & DB Standards](fastapi-api-db-standards.md)**
   - RESTful API design
   - Database operations
   - Security best practices
   - **Keywords**: #fastapi #api-design #restful #database #orm #migrations

9. **[FastAPI GraphQL](fastapi-graphql.md)**
   - GraphQL integration
   - Authentication
   - Advanced features (pagination, filtering)
   - **Keywords**: #fastapi #graphql #strawberry #api #schema #resolver

10. **[FastAPI Deployment](fastapi-deployment.md)**
    - Health checks
    - Environment configuration
    - Rate limiting
    - CORS configuration
    - **Keywords**: #fastapi #deployment #healthchecks #rate-limiting #environment-config #cors

11. **[FastAPI Rules](fastapi-rules.md)**
    - Structured rule directives
    - Format-specific guidance for GitHub Copilot
    - Concise coding standards
    - **Keywords**: #fastapi #rules #directives #guidelines #standards

## Usage Guide

Choose the most appropriate instruction file(s) based on your current development focus:

- For starting a new FastAPI project, include **FastAPI Foundations**
- When working on API documentation, include **FastAPI Documentation**
- For implementing tests, include **FastAPI Testing**
- When focusing on performance optimization, include **FastAPI Performance**
- For error handling and middleware, include **FastAPI Error Handling**
- When implementing GraphQL APIs, include **FastAPI GraphQL**
- For deployment preparation, include **FastAPI Deployment**
- When you need specific, structured guidance for GitHub Copilot, include **FastAPI Rules**

You can include multiple files if your task spans multiple areas.

### Using FastAPI Rules

The **FastAPI Rules** file contains structured directives that GitHub Copilot can follow. These rules use the format:

```markdown
@fastapi Rule - RuleName: RuleDescription
```

Example usage in your Copilot instructions:

```markdown
<instructions>
@fastapi Rule - Path Operation Function Names: Name path operation functions to clearly indicate their purpose.
@fastapi Rule - Response Models: Define explicit response models using Pydantic.
</instructions>
```

---

These instruction files are derived from the comprehensive [Python FastAPI Instructions](/python-fastapi-instructions.md) but are split for more efficient token usage and focused guidance.
