# Security Implementation Examples

This file contains practical code examples for implementing common security features in your applications.

#security #implementation #code #examples #practical

---

## 🔒 Secure Password Handling

Implement password hashing with Argon2:

```python
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

# Create a password hasher with secure defaults
ph = PasswordHasher(
    time_cost=3,        # Number of iterations
    memory_cost=65536,  # Memory usage in kibibytes (64 MB)
    parallelism=4,      # Degree of parallelism
    hash_len=32,        # Length of the hash in bytes
    salt_len=16         # Length of the salt in bytes
)

def hash_password(password: str) -> str:
    """Hash a password using Argon2id."""
    return ph.hash(password)

def verify_password(stored_hash: str, provided_password: str) -> bool:
    """Verify a password against a stored hash."""
    try:
        ph.verify(stored_hash, provided_password)
        return True
    except VerifyMismatchError:
        return False

# Check if hash needs rehashing (e.g., if parameters have changed)
def check_needs_rehash(stored_hash: str) -> bool:
    """Check if a password hash needs to be rehashed."""
    return ph.check_needs_rehash(stored_hash)

# Example usage
def user_login_example(username: str, password: str) -> bool:
    """Example login function with secure password handling."""
    # Get stored user hash from database (example)
    user = get_user_by_username(username)
    if not user:
        # Always take the same time for non-existent users to prevent timing attacks
        # This simulates the verification process
        ph.verify(
            ph.hash("dummy_password"),
            "incorrect_dummy_password"
        )
        return False
    
    # Verify the password
    if verify_password(user.password_hash, password):
        # Check if we need to upgrade the hash
        if check_needs_rehash(user.password_hash):
            # Rehash with new parameters
            new_hash = hash_password(password)
            update_user_password_hash(user.id, new_hash)
        return True
    
    return False
```

## 🔑 JWT Token Security

Secure JWT implementation:

```python
from datetime import datetime, timedelta
from typing import Dict, Optional

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

# Constants
SECRET_KEY = "secure-secret-key-stored-in-env-variable"  # Use env var in production
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def create_access_token(data: Dict, expires_delta: Optional[timedelta] = None) -> str:
    """
    Create a JWT token with secure practices.
    
    Args:
        data: Payload data to encode
        expires_delta: Optional custom expiration time
        
    Returns:
        Encoded JWT token
    """
    to_encode = data.copy()
    
    # Set expiration time
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({
        "exp": expire,
        "iat": datetime.utcnow(),  # Issued at time
        "nbf": datetime.utcnow(),  # Not valid before time
    })
    
    # Encode the JWT
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def verify_token(token: str) -> Dict:
    """
    Verify and decode a JWT token.
    
    Args:
        token: JWT token to verify
        
    Returns:
        Decoded token payload
        
    Raises:
        HTTPException: If token is invalid or expired
    """
    try:
        # Decode and verify the token
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token has expired",
            headers={"WWW-Authenticate": "Bearer"},
        )
    except jwt.InvalidTokenError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token",
            headers={"WWW-Authenticate": "Bearer"},
        )

# FastAPI dependency for getting the current user
async def get_current_user(token: str = Depends(oauth2_scheme)):
    """FastAPI dependency that extracts the current user from a token."""
    payload = verify_token(token)
    username: str = payload.get("sub")
    if username is None:
        raise HTTPException(status_code=401, detail="Invalid token payload")
    
    # Get user from database
    user = get_user_by_username(username)
    if user is None:
        raise HTTPException(status_code=401, detail="User not found")
        
    return user
```

## 🛡️ SQL Injection Prevention

Correct parameterized queries:

```python
import sqlite3
from sqlalchemy import text
from sqlalchemy.ext.asyncio import AsyncSession

# BAD - SQL Injection vulnerability
def get_user_unsafe(db_conn, username: str):
    """
    VULNERABLE function - DO NOT USE!
    This function is vulnerable to SQL injection.
    """
    # NEVER do this - direct string interpolation
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor = db_conn.cursor()
    cursor.execute(query)
    return cursor.fetchone()

# GOOD - Using parameterized queries with sqlite3
def get_user_safe(db_conn, username: str):
    """Safely query a user using parameterized queries."""
    query = "SELECT * FROM users WHERE username = ?"
    cursor = db_conn.cursor()
    cursor.execute(query, (username,))
    return cursor.fetchone()

# GOOD - Using SQLAlchemy with parameters
async def get_user_sqlalchemy(db: AsyncSession, username: str):
    """Safely query a user using SQLAlchemy."""
    query = text("SELECT * FROM users WHERE username = :username")
    result = await db.execute(query, {"username": username})
    return result.fetchone()

# GOOD - Using SQLAlchemy ORM
async def get_user_orm(db: AsyncSession, username: str):
    """Safely query a user using SQLAlchemy ORM."""
    from sqlalchemy import select
    from models import User
    
    query = select(User).where(User.username == username)
    result = await db.execute(query)
    return result.scalar_one_or_none()

# GOOD - Validate input before using in queries
def get_user_with_validation(db_conn, username: str):
    """Validate input before using in a query."""
    # Basic validation
    if not isinstance(username, str) or not username.isalnum():
        raise ValueError("Invalid username format")
        
    # Additional safety with parameterized query
    query = "SELECT * FROM users WHERE username = ?"
    cursor = db_conn.cursor()
    cursor.execute(query, (username,))
    return cursor.fetchone()
```

## 🌐 XSS Prevention

Content security implementation:

```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from starlette.middleware import Middleware
from starlette.middleware.sessions import SessionMiddleware

# Setup FastAPI with security middleware
app = FastAPI(
    middleware=[
        Middleware(SessionMiddleware, secret_key="your-secure-key"),
        Middleware(TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"]),
    ]
)

# Jinja2 templates with autoescaping enabled
templates = Jinja2Templates(directory="templates")

# Add security headers middleware
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    
    # Content Security Policy
    response.headers["Content-Security-Policy"] = " ".join([
        "default-src 'self';",
        "img-src 'self' https://secure.example.com;",
        "script-src 'self' https://js.example.com;",
        "style-src 'self' https://styles.example.com;",
        "font-src 'self' https://fonts.example.com;",
        "frame-ancestors 'none';",
        "form-action 'self';",
        "base-uri 'self';",
        "object-src 'none'",
    ])
    
    # Prevent MIME type sniffing
    response.headers["X-Content-Type-Options"] = "nosniff"
    
    # Clickjacking protection
    response.headers["X-Frame-Options"] = "DENY"
    
    # XSS protection (some browsers)
    response.headers["X-XSS-Protection"] = "1; mode=block"
    
    # HSTS (HTTP Strict Transport Security)
    if request.url.scheme == "https":
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains; preload"
        
    return response

# Render a template safely
@app.get("/user/{username}", response_class=HTMLResponse)
async def get_user_profile(request: Request, username: str):
    # Get user data from database
    user = get_user_by_username(username)
    
    # Safe rendering with auto-escaping
    return templates.TemplateResponse(
        "user_profile.html",
        {"request": request, "user": user}
    )
```

## 📁 File Upload Security

Secure file upload implementation:

```python
import os
import uuid
import magic
from pathlib import Path
from fastapi import FastAPI, File, UploadFile, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# Constants
UPLOAD_DIR = Path("./uploads")
ALLOWED_MIME_TYPES = {
    "image/jpeg", "image/png", "image/gif",
    "application/pdf", "text/plain"
}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5 MB

# Ensure upload directory exists
os.makedirs(UPLOAD_DIR, exist_ok=True)

# Generate a secure filename
def get_secure_filename(original_filename: str) -> str:
    """
    Generate a secure filename with UUID to prevent path traversal attacks.
    
    Args:
        original_filename: Original user-provided filename
        
    Returns:
        Secure filename with original extension
    """
    # Extract the file extension safely
    file_extension = os.path.splitext(original_filename)[1].lower()
    
    # Validate extension (additional security)
    allowed_extensions = {".jpg", ".jpeg", ".png", ".gif", ".pdf", ".txt"}
    if file_extension not in allowed_extensions:
        file_extension = ".bin"  # Default for unrecognized types
        
    # Generate secure filename with UUID
    return f"{uuid.uuid4()}{file_extension}"

# Verify file content type
def verify_file_type(file_content: bytes) -> str:
    """
    Verify actual file content type using python-magic.
    
    Args:
        file_content: Binary content of the file
        
    Returns:
        Detected MIME type
        
    Raises:
        HTTPException: If file type is not allowed
    """
    mime = magic.Magic(mime=True)
    detected_type = mime.from_buffer(file_content)
    
    if detected_type not in ALLOWED_MIME_TYPES:
        raise HTTPException(
            status_code=415,
            detail=f"Unsupported file type: {detected_type}"
        )
            
    return detected_type

# Secure file upload endpoint
@app.post("/upload/")
async def upload_file(
    file: UploadFile = File(...),
    token: str = Depends(oauth2_scheme)
):
    """
    Securely upload and process a file.
    
    Args:
        file: The file to upload
        token: Authentication token
        
    Returns:
        Details about the processed file
        
    Raises:
        HTTPException: If the file is too large or of invalid type
    """
    # Verify authentication (through dependency)
    
    # Read file content
    content = await file.read()
    
    # Check file size
    if len(content) > MAX_FILE_SIZE:
        raise HTTPException(
            status_code=413,
            detail=f"File too large: {len(content)} bytes. Maximum size: {MAX_FILE_SIZE} bytes."
        )
    
    # Verify file type based on content, not just extension
    mime_type = verify_file_type(content)
    
    # Generate secure filename
    secure_name = get_secure_filename(file.filename)
    
    # Save to secure location
    file_path = UPLOAD_DIR / secure_name
    with open(file_path, "wb") as f:
        f.write(content)
    
    # Return details
    return {
        "filename": secure_name,
        "size": len(content),
        "mime_type": mime_type,
        "status": "success"
    }
```

---

## See Also

- [Security Principles](/security/security-principles.md)
- [Security General Practices](/security/security-general-practices.md)
- [Security API Design](/security/security-api-design.md)
