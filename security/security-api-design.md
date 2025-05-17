# Secure API Design Patterns

This file contains patterns and code examples for implementing secure API designs in your applications.

#security #api #design #patterns #auth #jwt #oauth

---

## 🔐 OAuth 2.0 Implementation

Code sample for implementing OAuth 2.0 with FastAPI:

```python
from datetime import datetime, timedelta
from typing import Dict, List, Optional, Union

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import (
    OAuth2PasswordBearer,
    OAuth2PasswordRequestForm,
    SecurityScopes,
)
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel, ValidationError

# Security models
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None
    scopes: List[str] = []

class User(BaseModel):
    username: str
    email: Optional[str] = None
    full_name: Optional[str] = None
    disabled: Optional[bool] = None
    scopes: List[str] = []

# Constants
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Password hashing context
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# OAuth2 configuration with scopes
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={
        "users:read": "Read user information",
        "users:write": "Create or modify users",
        "items:read": "Read items",
        "items:write": "Create or modify items",
    },
)

app = FastAPI()

# Mock user database with hashed passwords and scopes
fake_users_db = {
    "john": {
        "username": "john",
        "full_name": "John Doe",
        "email": "john@example.com",
        "hashed_password": pwd_context.hash("secret"),
        "disabled": False,
        "scopes": ["users:read", "items:read"],
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Smith",
        "email": "alice@example.com",
        "hashed_password": pwd_context.hash("password"),
        "disabled": False,
        "scopes": ["users:read", "users:write", "items:read", "items:write"],
    },
}

# Helper functions
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return User(**user_dict)

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, fake_db[username]["hashed_password"]):
        return False
    return user

def create_access_token(
    data: dict, expires_delta: Optional[timedelta] = None
):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire, "iat": datetime.utcnow()})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(
    security_scopes: SecurityScopes, 
    token: str = Depends(oauth2_scheme)
):
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
        
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": authenticate_value},
    )
    
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
            
        token_scopes = payload.get("scopes", [])
        token_data = TokenData(scopes=token_scopes, username=username)
    except (JWTError, ValidationError):
        raise credentials_exception
        
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
        
    # Check for required scopes
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Not enough permissions. Required: {scope}",
                headers={"WWW-Authenticate": authenticate_value},
            )
            
    return user

async def get_current_active_user(
    current_user: User = Depends(get_current_user),
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# Token endpoint
@app.post("/token", response_model=Token)
async def login_for_access_token(
    form_data: OAuth2PasswordRequestForm = Depends()
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
        
    # Only include scopes the user has and that were requested
    user_scopes = fake_users_db[form_data.username]["scopes"]
    scopes = [s for s in form_data.scopes if s in user_scopes]
    
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username, "scopes": scopes},
        expires_delta=access_token_expires,
    )
    
    return {"access_token": access_token, "token_type": "bearer"}

# Protected endpoints with different scope requirements
@app.get("/users/me/", response_model=User)
async def read_users_me(
    current_user: User = Depends(get_current_active_user)
):
    return current_user

@app.get("/users/", response_model=List[User])
async def read_users(
    current_user: User = Security(get_current_active_user, scopes=["users:read"])
):
    # This endpoint requires users:read scope
    return list(get_user(fake_users_db, username) for username in fake_users_db)

@app.post("/users/", response_model=User)
async def create_user(
    user: UserCreate,
    current_user: User = Security(get_current_active_user, scopes=["users:write"])
):
    # This endpoint requires users:write scope
    # Implementation details...
    return created_user
```

## 🛑 API Rate Limiting

Implementation with in-memory storage:

```python
import time
from typing import Callable, Dict, Optional, Tuple
from datetime import datetime, timedelta

from fastapi import Depends, FastAPI, HTTPException, Request

app = FastAPI()

# Simple in-memory rate limiter (for demonstration only)
# For production, consider using Redis or another distributed cache
class InMemoryRateLimiter:
    def __init__(
        self,
        times: int = 10,       # Number of requests allowed
        seconds: int = 60,     # Time window in seconds
        cleanup_interval: int = 300  # Cleanup old entries every 5 minutes
    ):
        self.times = times
        self.seconds = seconds
        self.cleanup_interval = cleanup_interval
        self._storage: Dict[str, Dict[str, any]] = {}
        self._last_cleanup = datetime.now()
        
    def _cleanup_old_entries(self) -> None:
        """Remove expired entries from storage."""
        now = datetime.now()
        if (now - self._last_cleanup).total_seconds() > self.cleanup_interval:
            to_delete = []
            for key, data in self._storage.items():
                if now > data["expiry"]:
                    to_delete.append(key)
            
            for key in to_delete:
                del self._storage[key]
            
            self._last_cleanup = now
    
    def is_rate_limited(self, identifier: str) -> Tuple[bool, int]:
        """
        Check if the request is rate limited.
        
        Args:
            identifier: Unique identifier for the client
            
        Returns:
            Tuple of (is_limited, retry_after)
        """
        # Clear old entries periodically
        self._cleanup_old_entries()
        
        now = datetime.now()
        
        # If identifier not in storage, add it
        if identifier not in self._storage:
            self._storage[identifier] = {
                "count": 1,
                "expiry": now + timedelta(seconds=self.seconds)
            }
            return False, 0
        
        # Check if window has expired
        if now > self._storage[identifier]["expiry"]:
            # Reset for new window
            self._storage[identifier] = {
                "count": 1,
                "expiry": now + timedelta(seconds=self.seconds)
            }
            return False, 0
        
        # Check if over limit
        if self._storage[identifier]["count"] >= self.times:
            # Calculate time remaining in window
            seconds_remaining = int((self._storage[identifier]["expiry"] - now).total_seconds())
            return True, seconds_remaining
        
        # Increment count and continue
        self._storage[identifier]["count"] += 1
        return False, 0

# Create limiter instances
default_limiter = InMemoryRateLimiter(times=100, seconds=60)   # 100 requests per minute
strict_limiter = InMemoryRateLimiter(times=10, seconds=60)     # 10 requests per minute

# Rate limit dependency factory
def rate_limit(
    limiter: InMemoryRateLimiter = default_limiter,
    identifier_func: Optional[Callable[[Request], str]] = None
):
    async def _rate_limit(request: Request):
        # Default identifier is client IP
        if identifier_func is None:
            identifier = request.client.host
        else:
            identifier = identifier_func(request)
        
        is_limited, retry_after = limiter.is_rate_limited(identifier)
        
        if is_limited:
            raise HTTPException(
                status_code=429,
                detail="Too Many Requests",
                headers={
                    "Retry-After": str(retry_after),
                    "X-RateLimit-Limit": str(limiter.times),
                    "X-RateLimit-Reset": str(retry_after),
                }
            )
    
    return _rate_limit

# Apply rate limiting to an endpoint
@app.get("/api/public")
async def public_endpoint(
    _=Depends(rate_limit(default_limiter))
):
    return {"message": "This is a rate-limited public endpoint"}

# Apply stricter rate limiting to sensitive endpoints
@app.post("/api/sensitive")
async def sensitive_endpoint(
    _=Depends(rate_limit(strict_limiter))
):
    return {"message": "This is a more strictly rate-limited endpoint"}

# Custom identifier function for authenticated users
def get_user_identifier(request: Request) -> str:
    # Extract user ID from authentication token
    # This is a simplified example - implement proper token extraction
    auth_header = request.headers.get("Authorization", "")
    if auth_header.startswith("Bearer "):
        token = auth_header[7:]
        # Parse token and extract user ID
        # For demo, we're just using the token as the ID
        return f"user:{token}"
    return f"anon:{request.client.host}"

# Apply rate limiting by user ID for authenticated endpoint
@app.get("/api/user-specific")
async def user_specific_endpoint(
    _=Depends(rate_limit(default_limiter, get_user_identifier))
):
    return {"message": "This endpoint is rate-limited per user"}
```

## 🔒 API Response Security Headers

Middleware implementation for secure API headers:

```python
from fastapi import FastAPI, Request
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from starlette.middleware.base import BaseHTTPMiddleware

app = FastAPI()

# Define allowed hosts
app.add_middleware(
    TrustedHostMiddleware, allowed_hosts=["api.example.com", "*.example.com"]
)

# Custom middleware for security headers
class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        
        # Add security headers
        headers = {
            # Prevent browsers from sniffing MIME types
            "X-Content-Type-Options": "nosniff",
            
            # Strict HTTPS only communication
            "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
            
            # Prevent being loaded in iframes (clickjacking protection)
            "X-Frame-Options": "DENY",
            
            # Control how much information is included in referrer header
            "Referrer-Policy": "strict-origin-when-cross-origin",
            
            # Browser features that can be used by the site
            "Permissions-Policy": "camera=(), microphone=(), geolocation=()",
            
            # Cache control for sensitive APIs
            "Cache-Control": "no-store, max-age=0",
            
            # CORS for API-only services
            "Access-Control-Allow-Origin": "https://frontend.example.com",
        }
        
        # Add headers to response
        for header_name, header_value in headers.items():
            response.headers[header_name] = header_value
            
        return response

# Add the middleware
app.add_middleware(SecurityHeadersMiddleware)

# Sample endpoint
@app.get("/api/data")
async def get_data():
    return {"message": "Secure response with proper headers"}
```

## 🛡️ API Authentication Patterns

Common authentication patterns for APIs:

### 1. API Key Authentication

```python
from fastapi import Depends, FastAPI, HTTPException, Security, status
from fastapi.security.api_key import APIKeyHeader, APIKeyCookie, APIKeyQuery
from starlette.status import HTTP_403_FORBIDDEN

app = FastAPI()

# API keys can be in header, query parameter, or cookie
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)
api_key_query = APIKeyQuery(name="api-key", auto_error=False)
api_key_cookie = APIKeyCookie(name="api-key", auto_error=False)

# In a real application, store this securely
API_KEYS = {
    "valid_api_key_1": {"user": "service1", "scope": "read"},
    "valid_api_key_2": {"user": "service2", "scope": "read:write"}
}

async def get_api_key(
    api_key_header: str = Security(api_key_header),
    api_key_query: str = Security(api_key_query),
    api_key_cookie: str = Security(api_key_cookie),
):
    # Check multiple locations for the API key
    if api_key_header in API_KEYS:
        return API_KEYS[api_key_header]
    elif api_key_query in API_KEYS:
        return API_KEYS[api_key_query]
    elif api_key_cookie in API_KEYS:
        return API_KEYS[api_key_cookie]
    
    raise HTTPException(
        status_code=HTTP_403_FORBIDDEN, detail="Invalid or missing API Key"
    )

@app.get("/api/resource")
async def read_resource(api_key_info: dict = Depends(get_api_key)):
    return {
        "resource": "secure data",
        "owner": api_key_info["user"],
        "scope": api_key_info["scope"]
    }
```

### 2. mTLS (Mutual TLS) for Service-to-Service Auth

Configuration for Uvicorn (Python web server) to use mTLS:

```python
import uvicorn
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/service-endpoint")
async def service_endpoint(request: Request):
    # In mTLS, client cert info is available
    client_cert = request.scope.get("client_cert", {})
    
    return {
        "message": "Authenticated via mTLS",
        "client": client_cert.get("subject", {}).get("commonName")
    }

# Run with mTLS configuration
if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8443,
        ssl_keyfile="./server.key",
        ssl_certfile="./server.crt",
        ssl_ca_certs="./ca.crt",  # CA that signed client certs
        ssl_cert_reqs=2,  # ssl.CERT_REQUIRED
    )
```

---

## 🌐 Best Practices for API Design

1. **Versioning Your API**

```python
from fastapi import FastAPI, APIRouter

app = FastAPI()

# Version 1 router
v1_router = APIRouter(prefix="/api/v1")

@v1_router.get("/users")
async def get_users_v1():
    # V1 implementation
    return {"version": "1", "users": ["user1", "user2"]}

# Version 2 router  
v2_router = APIRouter(prefix="/api/v2")

@v2_router.get("/users")
async def get_users_v2():
    # V2 implementation with more data
    return {
        "version": "2", 
        "users": [
            {"id": 1, "name": "user1", "role": "admin"},
            {"id": 2, "name": "user2", "role": "user"}
        ]
    }

# Include routers
app.include_router(v1_router)
app.include_router(v2_router)
```

2. **Input Validation with Pydantic**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr, Field, validator
from typing import List, Optional
import re

app = FastAPI()

# Strict input validation model
class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)
    roles: List[str] = []
    
    # Custom validators
    @validator('username')
    def username_alphanumeric(cls, v):
        if not re.match(r'^[a-zA-Z0-9_-]+$', v):
            raise ValueError('Username must be alphanumeric')
        return v
        
    @validator('password')
    def password_strength(cls, v):
        if not re.search(r'[A-Z]', v):
            raise ValueError('Password must contain an uppercase letter')
        if not re.search(r'[a-z]', v):
            raise ValueError('Password must contain a lowercase letter')
        if not re.search(r'\d', v):
            raise ValueError('Password must contain a digit')
        if not re.search(r'[^A-Za-z0-9]', v):
            raise ValueError('Password must contain a special character')
        return v
        
    @validator('roles')
    def validate_roles(cls, v):
        allowed_roles = {"admin", "user", "moderator"}
        for role in v:
            if role not in allowed_roles:
                raise ValueError(f'Role {role} is not valid')
        return v

@app.post("/users/")
async def create_user(user: UserCreate):
    # With Pydantic, validation happens automatically
    # Process the validated data
    return {"message": "User created successfully"}
```

---

## See Also

- [Security Principles](/security/security-principles.md)
- [Security General Practices](/security/security-general-practices.md)
- [Security Implementation Examples](/security/security-implementation-examples.md)
