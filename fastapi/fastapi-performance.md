# GitHub Copilot: FastAPI Performance

Guidelines for optimizing performance in FastAPI applications, including async operations, efficient queries, and response streaming.

**Keywords**: #fastapi #performance #optimization #async #streaming #database

---

## ⚡ FastAPI Performance Coding

* **Async Database Operations**:
  * Use async ORM libraries with FastAPI:

  ```python
  from sqlalchemy.ext.asyncio import AsyncSession
  
  @router.get("/items")
  async def get_items(db: AsyncSession = Depends(get_async_db)):
      query = select(Item)
      result = await db.execute(query)
      return result.scalars().all()
  ```

* **Efficient Query Patterns**:
  * Avoid N+1 query problems with joins:

  ```python
  # Inefficient - causes N+1 queries
  @router.get("/users-with-posts")
  async def get_users_with_posts_inefficient(db: AsyncSession = Depends(get_db)):
      users = await db.execute(select(User))
      result = []
      for user in users.scalars().all():
          # This causes an additional query for each user
          posts = await db.execute(select(Post).where(Post.user_id == user.id))
          user_data = user.__dict__
          user_data["posts"] = posts.scalars().all()
          result.append(user_data)
      return result
  
  # Efficient - single query with join
      @router.get("/users-with-posts")
  async def get_users_with_posts_efficient(db: AsyncSession = Depends(get_db)):
      query = select(User).options(selectinload(User.posts))
      result = await db.execute(query)
      return result.scalars().all()
  ```

* **Response Streaming**:
  * Stream large responses to reduce memory usage:

  ```python
  from fastapi.responses import StreamingResponse
  import csv
  from io import StringIO
  
  @router.get("/large-data-export")
  async def export_large_data():
      async def generate_csv():
          # Create a buffer for CSV writing
          buffer = StringIO()
          writer = csv.writer(buffer)
          
          # Write header
          writer.writerow(["id", "name", "value"])
          yield buffer.getvalue()
          buffer.seek(0)
          buffer.truncate(0)
          
          # Stream rows in chunks
          for chunk in await get_data_in_chunks():
              for item in chunk:
                  writer.writerow([item.id, item.name, item.value])
              yield buffer.getvalue()
              buffer.seek(0)
              buffer.truncate(0)
      
      return StreamingResponse(
          generate_csv(),
          media_type="text/csv",
          headers={"Content-Disposition": "attachment; filename=data.csv"}
      )
  ```

* **Background Tasks**:
  * Use background tasks for offloading work:

  ```python
  from fastapi import BackgroundTasks

  def process_data_after_request(data: dict):
      # This runs after the response is sent
      # Process data, send notifications, etc.
      time.sleep(10)  # Simulate long-running task
      print(f"Processed data: {data}")

  @router.post("/items/")
  async def create_item(
      item: Item, 
      background_tasks: BackgroundTasks
  ):
      # Store the item first
      db_item = store_item_in_db(item)
      
      # Queue background processing
      background_tasks.add_task(process_data_after_request, item.dict())
      
      # Return immediately
      return {"status": "Item received", "id": db_item.id}
  ```

* **Caching Responses**:
  * Implement caching for frequently accessed data:

  ```python
  from fastapi import Depends, HTTPException, status
  from fastapi_cache import FastAPICache
  from fastapi_cache.decorator import cache
  from fastapi_cache.backends.inmemory import InMemoryBackend

  # Initialize cache in main.py
  @app.on_event("startup")
  async def startup():
      FastAPICache.init(InMemoryBackend())

  # Use caching on endpoints
  @router.get("/items/{item_id}")
  @cache(expire=60)  # Cache for 60 seconds
  async def get_item(item_id: int):
      # This will only be executed if not in cache
      return await get_item_from_db(item_id)
  ```

* **Optimized Pydantic Models**:
  * Use pydantic efficiently for better performance:

  ```python
  from pydantic import BaseModel, ConfigDict

  class Item(BaseModel):
      # Use ConfigDict for performance optimizations
      model_config = ConfigDict(
          validate_assignment=False,  # Disable validation on attribute assignment
          extra="ignore",  # Ignore extra fields for faster validation
          frozen=False,  # Allow modification after creation
          populate_by_name=True  # Enable alias population
      )
      
      id: int
      name: str
      description: str | None = None
  ```

* **Pagination for Large Datasets**:
  * Always implement pagination for large collections:

  ```python
  from fastapi import Query
  from typing import List

  @router.get("/items/", response_model=List[Item])
  async def list_items(
      skip: int = Query(0, ge=0, description="Skip N items"),
      limit: int = Query(100, ge=1, le=1000, description="Limit to N items")
  ):
      return await get_items_from_db(skip=skip, limit=limit)
  ```

---

## See Also
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [FastAPI Documentation](/fastapi/fastapi-documentation.md)
- [FastAPI Testing](/fastapi/fastapi-testing.md)
