# GitHub Copilot: FastAPI Foundations & Project Structure

Code-focused guidelines for effective FastAPI project setup, structure, and architecture.

**Keywords**: #fastapi #python #project-structure #architecture #foundations

---

## 🧠 Coding Foundations

* Focus on writing clean, maintainable FastAPI code
* Apply Python best practices from Python Core Principles
* Implement security patterns from Security Best Practices
* Use type hints everywhere for better IDE support and validation

---

## 🏗️ Project Structure & Architecture

* **Core Principles**:
  * Clear separation of concerns
  * Maintainable folder structure
  * RESTful design practices
* **Recommended Structure**:

  ```text
  my_fastapi_app/
  ├── app/
  │   ├── api/              # API endpoints
  │   │   ├── dependencies/ # Reusable dependencies
  │   │   ├── v1/           # API version 1 
  │   │   └── v2/           # API version 2 (if needed)
  │   ├── core/             # Core modules (config, security)
  │   ├── db/               # Database models & session
  │   ├── models/           # Pydantic models
  │   │   ├── domain/       # Domain models
  │   │   └── schemas/      # Request/response schemas
  │   ├── services/         # Business logic
  │   └── utils/            # Utilities
  ├── tests/                # Tests
  ├── alembic/              # Database migrations
  ├── .env                  # Environment variables (gitignored)
  ├── .env.example          # Example environment variables
  ├── main.py               # Application entry point
  ├── pyproject.toml        # Project dependencies
  └── README.md             # Project documentation
  ```
  
* **FastAPI application factory pattern**:

  ```python
  # app/core/config.py
  from pydantic_settings import BaseSettings

  class Settings(BaseSettings):
      APP_NAME: str = "MyFastAPI"
      API_V1_STR: str = "/api/v1"
      DEBUG: bool = False
      
      # Database
      DATABASE_URL: str
      
      # Security
      SECRET_KEY: str
      ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
      
      class Config:
          env_file = ".env"

  settings = Settings()
  ```

  ```python
  # app/main.py
  from fastapi import FastAPI
  from app.api.v1.api import api_router
  from app.core.config import settings

  def create_application() -> FastAPI:
      application = FastAPI(
          title=settings.APP_NAME,
          openapi_url=f"{settings.API_V1_STR}/openapi.json",
          debug=settings.DEBUG,
      )
      
      # Include routers
      application.include_router(
          api_router, 
          prefix=settings.API_V1_STR
      )
      
      return application

  app = create_application()
  ```

* **Router organization**:

  ```python
  # app/api/v1/api.py
  from fastapi import APIRouter
  from app.api.v1.endpoints import users, items

  api_router = APIRouter()
  api_router.include_router(users.router, prefix="/users", tags=["users"])
  api_router.include_router(items.router, prefix="/items", tags=["items"])
  ```

  ```python
  # app/api/v1/endpoints/users.py
  from fastapi import APIRouter, Depends, HTTPException
  from sqlalchemy.orm import Session
  from app.api.dependencies.db import get_db
  from app.models.schemas.user import UserCreate, UserResponse
  from app.services.user import create_user, get_user_by_id

  router = APIRouter()

  @router.post("/", response_model=UserResponse, status_code=201)
  def create_new_user(
      user_in: UserCreate,
      db: Session = Depends(get_db)
  ):
      return create_user(db=db, user_in=user_in)
  ```

* **Common Dependencies**:

  ```python
  # app/api/dependencies/db.py
  from typing import Generator
  from sqlalchemy.orm import Session
  from app.db.session import SessionLocal

  def get_db() -> Generator[Session, None, None]:
      db = SessionLocal()
      try:
          yield db
      finally:
          db.close()
  ```

* **Pydantic models**:

  ```python
  # app/models/schemas/user.py
  from pydantic import BaseModel, EmailStr, Field

  class UserBase(BaseModel):
      email: EmailStr
      full_name: str = Field(min_length=1, max_length=100)
      
  class UserCreate(UserBase):
      password: str = Field(min_length=8)
      
  class UserResponse(UserBase):
      id: int
      is_active: bool = True
      
      class Config:
          from_attributes = True
  ```

* **Database Models**:

  ```python
  # app/db/base.py
  from app.db.base_class import Base  # noqa
  from app.models.domain.user import User  # noqa
  from app.models.domain.item import Item  # noqa
  ```

  ```python
  # app/db/base_class.py
  from typing import Any
  from sqlalchemy.ext.declarative import declared_attr
  from sqlalchemy.orm import DeclarativeBase

  class Base(DeclarativeBase):
      id: Any
      __name__: str
      
      # Generate __tablename__ automatically
      @declared_attr
      def __tablename__(cls) -> str:
          return cls.__name__.lower()
  ```

---

## See Also
- [FastAPI Documentation](/fastapi/fastapi-documentation.md)
- [FastAPI Testing](/fastapi/fastapi-testing.md)
- [FastAPI Performance](/fastapi/fastapi-performance.md)
