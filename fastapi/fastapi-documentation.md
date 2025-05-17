# GitHub Copilot: FastAPI Documentation

Guidelines for implementing effective API documentation in FastAPI applications.

**Keywords**: #fastapi #documentation #openapi #swagger #pydantic #schema

---

## 📝 API Documentation in Code

* **Pydantic Schema Documentation**:
  * Add descriptions to all fields to generate clear API docs:

  ```python
  class UserCreate(BaseModel):
      username: str = Field(
          ..., 
          description="Unique username between 3-50 characters",
          min_length=3, 
          max_length=50,
          example="johndoe"
      )
      email: EmailStr = Field(
          ..., 
          description="Valid email address for account verification",
          example="john.doe@example.com"
      )
      password: str = Field(
          ..., 
          description="Secure password, minimum 8 characters",
          min_length=8,
          example="SecureP@ss123"
      )
  ```

* **Route Documentation**:
  * Document each endpoint with purpose and response details:

  ```python
  @router.get(
      "/users/{user_id}",
      summary="Get user details",
      description="Retrieve details for a specific user by their ID.",
      response_model=schemas.User,
      responses={
          404: {"model": schemas.HTTPError, "description": "User not found"},
          401: {"model": schemas.HTTPError, "description": "Unauthorized"}
      },
      tags=["users"]
  )
  ```

* **OpenAPI Organization**:
  * Group related endpoints with consistent tags:

  ```python
  # In your main.py or app creation file
  app = FastAPI(
      title="My API",
      openapi_tags=[
          {"name": "users", "description": "User management operations"},
          {"name": "items", "description": "Item CRUD operations"},
          {"name": "auth", "description": "Authentication operations"}
      ]
  )
  ```

* **Comprehensive API Documentation**:
  * Document the entire API with consistent patterns:

  ```python
  app = FastAPI(
      title="User Management API",
      description="""
      This API provides user management functionality with the following features:
      
      * User account creation and management
      * Authentication using JWT tokens
      * Password reset functionality
      * Role-based access control
      
      For support, contact api-support@example.com
      """,
      version="1.0.0",
      contact={
          "name": "API Support Team",
          "email": "api-support@example.com",
          "url": "https://example.com/support",
      },
      license_info={
          "name": "MIT License",
          "url": "https://opensource.org/licenses/MIT",
      }
  )
  ```

* **Response Examples**:
  * Include example responses for better documentation:

  ```python
  @router.get(
      "/users/{user_id}",
      response_model=UserResponse,
      responses={
          200: {
              "description": "Successful response",
              "content": {
                  "application/json": {
                      "example": {
                          "id": 1,
                          "username": "johndoe",
                          "email": "john.doe@example.com",
                          "full_name": "John Doe",
                          "created_at": "2023-04-01T12:00:00Z"
                      }
                  }
              }
          },
          404: {
              "description": "User not found",
              "content": {
                  "application/json": {
                      "example": {"detail": "User with ID 1 not found"}
                  }
              }
          }
      }
  )
  ```

* **Custom OpenAPI Schema**:
  * Customize the OpenAPI schema for better documentation:

  ```python
  def custom_openapi():
      if app.openapi_schema:
          return app.openapi_schema
      
      openapi_schema = get_openapi(
          title="Custom title",
          version="2.5.0",
          description="Custom API description",
          routes=app.routes,
      )
      
      # Add custom documentation for authentication
      openapi_schema["components"]["securitySchemes"] = {
          "JWT": {
              "type": "http",
              "scheme": "bearer",
              "bearerFormat": "JWT",
              "description": "Enter the JWT token obtained from the login endpoint"
          }
      }
      
      # Apply security globally
      openapi_schema["security"] = [{"JWT": []}]
      
      app.openapi_schema = openapi_schema
      return app.openapi_schema

  app.openapi = custom_openapi
  ```

---

## See Also
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [FastAPI Testing](/fastapi/fastapi-testing.md)
- [FastAPI Performance](/fastapi/fastapi-performance.md)
