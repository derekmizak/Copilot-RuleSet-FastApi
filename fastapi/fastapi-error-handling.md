# GitHub Copilot: FastAPI Error Handling & Middleware

Guidelines for implementing effective error handling, middleware, and cross-cutting concerns in FastAPI applications.

**Keywords**: #fastapi #error-handling #middleware #exception-handlers #logging #cors

---

## 🔄 Error Handling & Middleware

* **Centralized Error Handling**:
  * Create consistent error responses:

  ```python
  from fastapi import FastAPI, Request, status
  from fastapi.responses import JSONResponse
  from fastapi.exceptions import RequestValidationError
  from pydantic import BaseModel
  
  class ErrorResponse(BaseModel):
      status_code: int
      message: str
      details: dict | None = None
      request_id: str | None = None
  
  app = FastAPI()
  
  @app.exception_handler(RequestValidationError)
  async def validation_exception_handler(request: Request, exc: RequestValidationError):
      return JSONResponse(
          status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
          content=ErrorResponse(
              status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
              message="Validation error",
              details={"errors": exc.errors()},
              request_id=request.state.request_id if hasattr(request.state, "request_id") else None
          ).dict(),
      )
      
  # Custom exception class
  class NotFoundError(Exception):
      def __init__(self, item_type: str, item_id: str):
          self.item_type = item_type
          self.item_id = item_id
          
  @app.exception_handler(NotFoundError)
  async def not_found_exception_handler(request: Request, exc: NotFoundError):
      return JSONResponse(
          status_code=status.HTTP_404_NOT_FOUND,
          content=ErrorResponse(
              status_code=status.HTTP_404_NOT_FOUND,
              message=f"{exc.item_type} with id {exc.item_id} not found",
              request_id=request.state.request_id if hasattr(request.state, "request_id") else None
          ).dict(),
      )
  
  # Usage in route handlers
  @app.get("/items/{item_id}")
  async def read_item(item_id: str):
      item = get_item_from_db(item_id)
      if not item:
          raise NotFoundError(item_type="Item", item_id=item_id)
      return item
  ```

* **Middleware for Cross-Cutting Concerns**:
  * Implement CORS middleware:

  ```python
  from fastapi.middleware.cors import CORSMiddleware
  
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["https://frontend.example.com"],
      allow_credentials=True,
      allow_methods=["*"],
      allow_headers=["*"],
  )
  ```
  
  * Create custom middleware for timing requests:

  ```python
  import time
  from fastapi import FastAPI, Request
  from starlette.middleware.base import BaseHTTPMiddleware
  
  class TimingMiddleware(BaseHTTPMiddleware):
      async def dispatch(self, request: Request, call_next):
          start_time = time.time()
          response = await call_next(request)
          process_time = time.time() - start_time
          response.headers["X-Process-Time"] = str(process_time)
          return response
  
  app = FastAPI()
  app.add_middleware(TimingMiddleware)
  ```

* **Configuration Management**:
  * Use Pydantic's `BaseSettings` for environment variable loading
  * Separate configurations by environment (dev, test, prod)
  * Keep sensitive values in environment variables

* **Logging**:
  * Implement structured logging (JSON format)
  * Include contextual information (request ID, user ID)
  * Use appropriate log levels consistently

* **Request ID Tracking**:
  * Implement request ID middleware for tracing:

  ```python
  import uuid
  from fastapi import FastAPI, Request
  from starlette.middleware.base import BaseHTTPMiddleware
  
  class RequestIDMiddleware(BaseHTTPMiddleware):
      async def dispatch(self, request: Request, call_next):
          request_id = str(uuid.uuid4())
          # Store in request state for access in route handlers
          request.state.request_id = request_id
          
          # Add to response headers
          response = await call_next(request)
          response.headers["X-Request-ID"] = request_id
          return response
  
  app = FastAPI()
  app.add_middleware(RequestIDMiddleware)
  ```

* **Global Exception Handling**:
  * Create catch-all handler for unexpected errors:

  ```python
  @app.exception_handler(Exception)
  async def global_exception_handler(request: Request, exc: Exception):
      # Log the error with traceback
      logger.exception(f"Unexpected error: {str(exc)}")
      
      return JSONResponse(
          status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
          content=ErrorResponse(
              status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
              message="An unexpected error occurred",
              request_id=getattr(request.state, "request_id", None)
          ).dict(),
      )
  ```

* **Middleware Order**:
  * Maintain careful ordering of middleware:

  ```python
  # Order matters! Applied from top to bottom
  app.add_middleware(RequestIDMiddleware)  # Should be first to track all requests
  app.add_middleware(TimingMiddleware)
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["https://frontend.example.com"],
      allow_credentials=True,
      allow_methods=["*"],
      allow_headers=["*"],
  )
  ```

* **Authentication Middleware**:
  * Implement JWT authentication as middleware:

  ```python
  from fastapi import FastAPI, Request, HTTPException
  from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
  from starlette.middleware.base import BaseHTTPMiddleware
  import jwt
  
  security = HTTPBearer()
  
  class JWTMiddleware(BaseHTTPMiddleware):
      async def dispatch(self, request: Request, call_next):
          # Skip auth for certain paths
          if request.url.path in ["/docs", "/redoc", "/openapi.json", "/login"]:
              return await call_next(request)
          
          # Extract token
          auth_header = request.headers.get("Authorization")
          if not auth_header or not auth_header.startswith("Bearer "):
              return JSONResponse(
                  status_code=status.HTTP_401_UNAUTHORIZED,
                  content={"detail": "Missing or invalid authentication token"}
              )
          
          token = auth_header.replace("Bearer ", "")
          
          try:
              # Verify token
              payload = jwt.decode(
                  token, 
                  settings.SECRET_KEY, 
                  algorithms=[settings.JWT_ALGORITHM]
              )
              # Store user info in request state
              request.state.user_id = payload.get("sub")
          except jwt.PyJWTError:
              return JSONResponse(
                  status_code=status.HTTP_401_UNAUTHORIZED,
                  content={"detail": "Invalid authentication token"}
              )
              
          return await call_next(request)
  ```

---

## See Also
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [FastAPI Framework Rules](/fastapi/fastapi-framework-rules.md)
- [Security Implementation Examples](/security/security-implementation-examples.md)
