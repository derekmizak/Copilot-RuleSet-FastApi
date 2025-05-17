# GitHub Copilot: FastAPI Deployment-Ready Code Practices

Guidelines for creating deployment-ready FastAPI applications with proper health checks, environment configuration, rate limiting, and CORS.

**Keywords**: #fastapi #deployment #healthchecks #rate-limiting #environment-config #cors

---

## 🚀 Deployment-Ready Code Practices

* **Health Checks**:
  * Implement health check endpoints:

  ```python
  from fastapi import FastAPI, Depends, status
  from sqlalchemy.ext.asyncio import AsyncSession

  app = FastAPI()

  @app.get("/health", tags=["health"])
  async def health_check():
      return {"status": "ok"}

  @app.get("/health/db", tags=["health"])
  async def db_health_check(db: AsyncSession = Depends(get_db_session)):
      try:
          # Execute a simple query to check database connection
          result = await db.execute("SELECT 1")
          if result:
              return {"status": "ok", "database": "connected"}
      except Exception as e:
          return {"status": "error", "database": "disconnected", "details": str(e)}, status.HTTP_503_SERVICE_UNAVAILABLE
  ```

* **Environment Configuration**:
  * Write code to handle multiple deployment environments:

  ```python
  from enum import Enum
  from pydantic import BaseSettings, Field

  class Environment(str, Enum):
      DEVELOPMENT = "development"
      STAGING = "staging"
      PRODUCTION = "production"

  class Settings(BaseSettings):
      ENVIRONMENT: Environment = Environment.DEVELOPMENT
      DEBUG: bool = Field(default=False)
      
      # Database settings with different defaults per environment
      DATABASE_HOST: str
      DATABASE_PORT: int = 5432
      DATABASE_USER: str
      DATABASE_PASSWORD: str
      DATABASE_NAME: str
      
      # Computed property for SQLAlchemy URL
      @property
      def database_url(self) -> str:
          return f"postgresql+asyncpg://{self.DATABASE_USER}:{self.DATABASE_PASSWORD}@{self.DATABASE_HOST}:{self.DATABASE_PORT}/{self.DATABASE_NAME}"
      
      # Specific environment overrides
      @property
      def is_production(self) -> bool:
          return self.ENVIRONMENT == Environment.PRODUCTION
      
      # SSL settings based on environment
      @property
      def db_connection_args(self) -> dict:
          if self.is_production:
              return {"ssl": True, "ssl_mode": "require"}
          return {}
      
      class Config:
          env_file = ".env"
          env_file_encoding = "utf-8"
          
  # Load settings once as a singleton
  settings = Settings()
  
  # Use in other modules
  from app.core.config import settings
  
  # Database setup using settings
  engine = create_async_engine(
      settings.database_url,
      connect_args=settings.db_connection_args,
      echo=settings.DEBUG,
  )
  ```

* **Rate Limiting Implementation**:
  * Add rate limiting to protect your API:

  ```python
  from fastapi import FastAPI, Request, Response, Depends, HTTPException
  import time
  from datetime import datetime, timedelta
  from typing import Dict, Callable, Optional, Tuple

  app = FastAPI()

  # In-memory cache for rate limiting
  # Note: For production, consider a distributed solution
  class InMemoryRateLimit:
      def __init__(self):
          self.cache: Dict[str, Dict] = {}
          self.last_cleanup = datetime.now()
          self.cleanup_interval = timedelta(minutes=5)  # Cleanup every 5 minutes
      
      def cleanup(self):
          """Remove expired entries from cache"""
          now = datetime.now()
          if now - self.last_cleanup > self.cleanup_interval:
              expired_keys = []
              for key, data in self.cache.items():
                  if now > data["expires"]:
                      expired_keys.append(key)
              
              for key in expired_keys:
                  del self.cache[key]
              
              self.last_cleanup = now
      
      def get(self, key: str) -> Optional[Dict]:
          """Get rate limit data if exists and not expired"""
          self.cleanup()
          if key in self.cache and datetime.now() <= self.cache[key]["expires"]:
              return self.cache[key]
          return None
      
      def set(self, key: str, count: int, window: int):
          """Set rate limit data with expiration"""
          self.cache[key] = {
              "count": count,
              "expires": datetime.now() + timedelta(seconds=window)
          }
      
      def increment(self, key: str) -> int:
          """Increment request count"""
          if key in self.cache:
              self.cache[key]["count"] += 1
              return self.cache[key]["count"]
          return 0
      
      def get_ttl(self, key: str) -> int:
          """Get remaining TTL in seconds"""
          if key in self.cache:
              remaining = (self.cache[key]["expires"] - datetime.now()).total_seconds()
              return max(0, int(remaining))
          return 0

  # Create rate limiter
  rate_limiter = InMemoryRateLimit()

  # Rate limiter dependency
  async def rate_limit(
      request: Request,
      response: Response,
      limit: int = 100,
      window: int = 60,  # in seconds
      key_func: Optional[Callable] = None
  ):
      # Default key function uses client IP
      if key_func is None:
          key_func = lambda req: req.client.host
          
      # Generate rate limit key
      key = f"ratelimit:{key_func(request)}"
      
      # Check if key exists
      data = rate_limiter.get(key)
      
      # Set headers
      response.headers["X-RateLimit-Limit"] = str(limit)
      
      if data is None:
          # First request in window
          rate_limiter.set(key, 1, window)
          response.headers["X-RateLimit-Remaining"] = str(limit - 1)
          return
          
      count = data["count"]
      if count >= limit:
          # Rate limit exceeded
          ttl = rate_limiter.get_ttl(key)
          response.headers["X-RateLimit-Remaining"] = "0"
          response.headers["Retry-After"] = str(ttl)
          raise HTTPException(
              status_code=429,
              detail="Rate limit exceeded. Try again later."
          )
          
      # Increment counter
      current = rate_limiter.increment(key)
      response.headers["X-RateLimit-Remaining"] = str(limit - current)

  # Apply to specific endpoints
  @app.get("/api/public", dependencies=[Depends(rate_limit)])
  async def public_endpoint():
      return {"message": "This is a rate-limited endpoint"}
      
  # Apply with custom parameters
  @app.get("/api/sensitive")
  async def sensitive_endpoint(
      _: None = Depends(lambda req, res: rate_limit(req, res, limit=10, window=60))
  ):
      return {"message": "This is a more strictly rate-limited endpoint"}
  ```

* **CORS Configuration**:
  * Properly configure CORS for production:

  ```python
  from fastapi import FastAPI
  from fastapi.middleware.cors import CORSMiddleware
  from app.core.config import settings

  app = FastAPI()

  # Development vs Production CORS settings
  origins = ["*"] if not settings.is_production else [
      "https://frontend.example.com",
      "https://app.example.com",
  ]

  app.add_middleware(
      CORSMiddleware,
      allow_origins=origins,
      allow_credentials=True,
      allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"],
      allow_headers=["Authorization", "Content-Type", "Accept"],
      expose_headers=["X-Request-ID", "X-RateLimit-Limit", "X-RateLimit-Remaining"],
      max_age=3600,  # Cache preflight requests for 1 hour
  )
  ```

* **Production Server Configuration**:
  * Configure UVicorn and Gunicorn for production:

  ```python
  # gunicorn_conf.py
  import multiprocessing
  import os

  # Get the number of workers based on CPU cores
  # Formula: (2 * CPU cores) + 1
  workers_per_core_str = os.getenv("WORKERS_PER_CORE", "2")
  workers_per_core = int(workers_per_core_str)
  cores = multiprocessing.cpu_count()
  default_workers = (workers_per_core * cores) + 1
  max_workers = int(os.getenv("MAX_WORKERS", default_workers))

  # Gunicorn configurations
  bind = os.getenv("BIND", "0.0.0.0:8000")
  worker_class = os.getenv("WORKER_CLASS", "uvicorn.workers.UvicornWorker")
  loglevel = os.getenv("LOG_LEVEL", "info")
  accesslog = os.getenv("ACCESS_LOG", "-")  # stderr
  errorlog = os.getenv("ERROR_LOG", "-")    # stderr
  worker_tmp_dir = os.getenv("WORKER_TMP_DIR", "/dev/shm")
  graceful_timeout = int(os.getenv("GRACEFUL_TIMEOUT", "120"))
  timeout = int(os.getenv("TIMEOUT", "120"))
  keepalive = int(os.getenv("KEEPALIVE", "5"))

  # Run command:
  # gunicorn -c gunicorn_conf.py app.main:app
  ```

* **Containerization**:
  * Create a production-ready Dockerfile:

  ```dockerfile
  # Use official Python image
  FROM python:3.12-slim as builder

  # Set environment variables
  ENV PYTHONDONTWRITEBYTECODE=1 \
      PYTHONUNBUFFERED=1 \
      PIP_NO_CACHE_DIR=off \
      PIP_DISABLE_PIP_VERSION_CHECK=on

  # Install system dependencies
  RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential \
      libpq-dev \
      && rm -rf /var/lib/apt/lists/*

  # Create a virtual environment
  RUN python -m venv /opt/venv
  ENV PATH="/opt/venv/bin:$PATH"

  # Install Python dependencies
  COPY requirements.txt .
  RUN pip install --upgrade pip && \
      pip install -r requirements.txt && \
      pip install gunicorn uvicorn

  # Create a lightweight final image
  FROM python:3.12-slim

  # Copy virtual environment from builder
  COPY --from=builder /opt/venv /opt/venv
  ENV PATH="/opt/venv/bin:$PATH"

  # Install runtime dependencies
  RUN apt-get update && apt-get install -y --no-install-recommends \
      libpq5 \
      && rm -rf /var/lib/apt/lists/*

  # Create a non-root user
  RUN useradd -m appuser
  WORKDIR /app
  
  # Copy application code
  COPY --chown=appuser:appuser . /app/

  # Switch to non-root user
  USER appuser

  # Set environment
  ENV ENVIRONMENT=production \
      PORT=8000

  # Run the application with Gunicorn
  CMD gunicorn -c gunicorn_conf.py app.main:app
  ```

---

## See Also
- [FastAPI API & DB Standards](/fastapi/fastapi-api-db-standards.md)
- [FastAPI Error Handling](/fastapi/fastapi-error-handling.md)
- [FastAPI Advanced Patterns](/fastapi/fastapi-advanced-patterns.md)
