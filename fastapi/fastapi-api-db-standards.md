# GitHub Copilot: FastAPI API & DB Standards

Guidelines for API design and database operations in FastAPI applications, focusing on RESTful design, response standards, and database best practices.

**Keywords**: #fastapi #api-design #restful #database #orm #migrations

---

## 🌐 API & DB Standards

### API Design (@api)

* **RESTful Design**:
  * Use nouns for resource URLs
  * Apply standard HTTP methods (GET, POST, PUT, DELETE, PATCH)
  * Ensure idempotency for appropriate operations
* **Versioning**:
  * Implement via URL path (`/api/v1/resource`)
  * Consider header-based versioning for advanced scenarios
* **Response Standards**:
  * Use appropriate HTTP status codes
  * Maintain consistent response structure
  * Include pagination metadata for list endpoints
* **Error Responses**:
  * Consistent JSON format with error code and message
  * Avoid leaking sensitive information
  * Include request ID for traceability
* **Advanced Features**:
  * Implement pagination, filtering, and sorting
  * Support partial responses when appropriate
  * Apply rate limiting and throttling

### Database Operations (@db)

* **Security Best Practices**:
  * Use parameterized queries or ORMs to prevent SQL injection
  * Apply principle of least privilege for database users
  * Implement row-level security when needed
* **Connection Management**:
  * Use connection pooling for efficiency
  * Handle connection errors gracefully
  * Close connections properly
* **Transaction Handling**:
  * Ensure ACID properties where needed
  * Implement proper error handling and rollbacks
  * Use FastAPI dependency injection for transaction scoping
* **Data Migration**:
  * Use Alembic for schema migrations
  * Version migrations and make them reversible
  * Test migrations before production deployment

* **API Pagination Pattern**:
  * Standard pagination implementation:

  ```python
  from fastapi import FastAPI, Query, Depends
  from sqlalchemy.ext.asyncio import AsyncSession
  from sqlalchemy import select, func
  from typing import List, Dict, Any, TypeVar, Generic
  from pydantic import BaseModel, Field
  
  T = TypeVar('T')
  
  class PaginatedResponse(BaseModel, Generic[T]):
      items: List[T]
      total: int
      page: int
      size: int
      pages: int
      
      @property
      def has_more(self) -> bool:
          return self.page < self.pages
  
  async def paginate_query(
      query,
      db: AsyncSession,
      page: int = Query(1, ge=1, description="Page number"),
      size: int = Query(20, ge=1, le=100, description="Items per page")
  ) -> Dict[str, Any]:
      # Count total items
      count_query = select(func.count()).select_from(query.subquery())
      total = await db.scalar(count_query)
      
      # Calculate pagination values
      pages = (total + size - 1) // size
      
      # Apply pagination
      query = query.limit(size).offset((page - 1) * size)
      
      # Execute query
      result = await db.execute(query)
      items = result.scalars().all()
      
      return {
          "items": items,
          "total": total,
          "page": page,
          "size": size,
          "pages": pages
      }
  
  # Example usage in a route
  @app.get("/users/", response_model=PaginatedResponse[UserSchema])
  async def get_users(
      pagination: Dict = Depends(paginate_query),
      db: AsyncSession = Depends(get_db)
  ):
      query = select(User).order_by(User.id)
      pagination_data = await paginate_query(query, db)
      return pagination_data
  ```

* **Database Connection Setup**:
  * Proper SQLAlchemy async connection:

  ```python
  from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
  from sqlalchemy.orm import sessionmaker
  from sqlalchemy.pool import QueuePool
  
  # Database URL from environment
  DATABASE_URL = "postgresql+asyncpg://user:password@localhost/dbname"
  
  # Create engine with proper configuration
  engine = create_async_engine(
      DATABASE_URL,
      echo=False,  # Set to True for SQL logging in development
      pool_size=5,  # Adjust based on expected load
      max_overflow=10,
      pool_timeout=30,  # 30 seconds
      pool_recycle=1800,  # 30 minutes
      pool_pre_ping=True,  # Enable connection health checks
      poolclass=QueuePool
  )
  
  # Create session factory
  AsyncSessionLocal = sessionmaker(
      engine, 
      class_=AsyncSession, 
      expire_on_commit=False,
      autoflush=False
  )
  
  # Dependency for routes
  async def get_db() -> AsyncSession:
      async with AsyncSessionLocal() as session:
          try:
              yield session
              await session.commit()
          except Exception:
              await session.rollback()
              raise
  ```

* **Advanced Filtering Pattern**:
  * Implement flexible query filtering:

  ```python
  from fastapi import APIRouter, Query, Depends
  from typing import List, Optional
  from sqlalchemy import select
  from sqlalchemy.ext.asyncio import AsyncSession
  
  router = APIRouter()
  
  @router.get("/products/", response_model=List[ProductSchema])
  async def get_products(
      name: Optional[str] = Query(None, description="Filter by product name"),
      category: Optional[str] = Query(None, description="Filter by category"),
      min_price: Optional[float] = Query(None, ge=0, description="Minimum price"),
      max_price: Optional[float] = Query(None, ge=0, description="Maximum price"),
      sort_by: str = Query("id", description="Field to sort by"),
      sort_order: str = Query("asc", description="Sort order (asc or desc)"),
      db: AsyncSession = Depends(get_db)
  ):
      # Start building the query
      query = select(Product)
      
      # Apply filters conditionally
      if name:
          query = query.filter(Product.name.ilike(f"%{name}%"))
          
      if category:
          query = query.filter(Product.category == category)
          
      if min_price is not None:
          query = query.filter(Product.price >= min_price)
          
      if max_price is not None:
          query = query.filter(Product.price <= max_price)
      
      # Apply sorting
      if sort_order.lower() == "desc":
          query = query.order_by(getattr(Product, sort_by).desc())
      else:
          query = query.order_by(getattr(Product, sort_by))
      
      # Execute query
      result = await db.execute(query)
      products = result.scalars().all()
      
      return products
  ```

---

## See Also
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [FastAPI Advanced Patterns](/fastapi/fastapi-advanced-patterns.md)
- [FastAPI GraphQL](/fastapi/fastapi-graphql.md)
