<!-- filepath: /fastapi/fastapi-rules.md -->
# GitHub Copilot: FastAPI Rules

A collection of structured rule directives for GitHub Copilot to follow when working with FastAPI applications.

**Keywords**: #fastapi #rules #best-practices #api-design #python

---

## FastAPI Rule Directives

### API Design

```
@fastapi Rule - Path Operation Function Names: Name path operation functions to clearly indicate their purpose, not HTTP method (e.g., `create_user` not `post_user`).
```

```
@fastapi Rule - Path Parameters: Use path parameters for required resource identifiers (e.g., `/users/{user_id}`).
```

```
@fastapi Rule - Query Parameters: Use query parameters for optional filters and parameters.
```

```
@fastapi Rule - HTTP Status Codes: Use appropriate HTTP status codes in responses (e.g., 201 for creation, 204 for no content, 400 for bad request).
```

### Data Validation and Models

```
@fastapi Rule - Pydantic Models: Always use Pydantic models for request and response data validation.
```

```
@fastapi Rule - Response Models: Define explicit response models using Pydantic `response_model` parameter.
```

```
@fastapi Rule - Input Validation: Implement comprehensive validation rules in Pydantic models using field validators and constraints.
```

```
@fastapi Rule - Model Inheritance: Use model inheritance and composition to avoid code duplication in Pydantic models.
```

### Dependency Injection

```
@fastapi Rule - Dependency Injection: Use FastAPI's dependency injection system for shared logic instead of global variables.
```

```
@fastapi Rule - Database Sessions: Manage database connections as dependencies with proper lifecycle management.
```

```
@fastapi Rule - Caching Dependencies: Cache heavy dependencies with `fastapi.Depends(dependency, use_cache=True)` for performance.
```

### Security

```
@fastapi Rule - Authentication: Implement authentication using OAuth2 with Password or OpenID Connect.
```

```
@fastapi Rule - Authorization: Use dependency injection for authorization checks on routes.
```

```
@fastapi Rule - CORS: Configure CORS with appropriate origins, methods, and headers.
```

### Performance and Organization

```
@fastapi Rule - Router Organization: Use APIRouter to organize related endpoints and maintain clean code structure.
```

```
@fastapi Rule - Background Tasks: Use background tasks for time-consuming operations that don't need to block the response.
```

```
@fastapi Rule - Async/Await: Leverage FastAPI's asynchronous capabilities with async/await for I/O bound operations.
```

### Documentation

```
@fastapi Rule - API Documentation: Provide comprehensive docstrings and descriptions for all endpoints and models.
```

```
@fastapi Rule - Examples: Include example requests and responses in API documentation using Pydantic's `Schema_extra`.
```

---

## Usage Instructions

To use these rules in your GitHub Copilot instructions, copy the appropriate rule directives into your instruction file or include this file using:

```markdown
<!-- include: /fastapi/fastapi-rules.md -->
```
