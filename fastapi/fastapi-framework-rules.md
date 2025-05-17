# GitHub Copilot: FastAPI Framework-Specific Rules

Guidelines for effective implementation of FastAPI's core features and patterns, including dependency injection, path operations, and async functions.

**Keywords**: #fastapi #framework #dependency-injection #pydantic #routers #path-operations

---

## 🧰 Framework-Specific Rules

### FastAPI (@fastapi)

* **Pydantic Models**:
  * Use for request/response validation, serialization, documentation
  * Define strict schemas with proper types, constraints, defaults, examples
  * Utilize specialized fields (`Field`, `EmailStr`, etc.)
  * Create clear separation between API schemas and domain models
* **Dependency Injection (`Depends`)**:
  * Use for database sessions, configuration, authentication, reusable logic
  * Make dependencies async for I/O-bound operations
  * Implement scoped dependencies for resource management
  * Design for testability with possible replacements during testing
  * Example:

  ```python
  from fastapi import Depends, FastAPI, HTTPException
  from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
  from sqlalchemy.orm import sessionmaker
  from typing import Annotated, AsyncGenerator
  
  DATABASE_URL = "postgresql+asyncpg://postgres:password@localhost/db"
  
  engine = create_async_engine(DATABASE_URL)
  async_session_maker = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
  
  async def get_db_session() -> AsyncGenerator[AsyncSession, None]:
      """Dependency that provides a database session."""
      async with async_session_maker() as session:
          try:
              yield session
              await session.commit()
          except Exception:
              await session.rollback()
              raise
          finally:
              await session.close()
  
  # Create a reusable dependency
  async def get_user_by_id(
      user_id: int, 
      db: AsyncSession = Depends(get_db_session)
  ):
      """Fetch user by ID. Reusable dependency."""
      user = await db.get(User, user_id)
      if not user:
          raise HTTPException(status_code=404, detail="User not found")
      return user
  
  # Use multiple levels of dependencies
  async def get_current_active_user(
      current_user: User = Depends(get_current_user)
  ):
      """Verify user is active. Uses get_current_user dependency."""
      if not current_user.is_active:
          raise HTTPException(status_code=400, detail="Inactive user")
      return current_user
  
  # Define a route with dependencies
  @app.get("/users/me/items/")
  async def read_own_items(
      current_user: Annotated[User, Depends(get_current_active_user)],
      db: Annotated[AsyncSession, Depends(get_db_session)]
  ):
      """Get items for the current user."""
      return await get_items_for_user(db, current_user.id)
  ```

* **Path Operations**:
  * Follow RESTful principles
  * Always specify `response_model` to prevent data leaks
  * Set appropriate `status_code`
  * Include `tags`, `summary`, `description` for documentation
  * Example:

  ```python
  @router.post(
      "/items/",
      response_model=schemas.Item,
      status_code=status.HTTP_201_CREATED,
      summary="Create a new item",
      description="Create a new item with the provided details",
      tags=["items"]
  )
  async def create_item(
      item: schemas.ItemCreate,
      current_user: User = Depends(get_current_user),
      db: AsyncSession = Depends(get_db)
  ):
      return await item_service.create_item(db, item, current_user.id)
  ```

* **Async Functions**:
  * Use `async def` for I/O-bound path operations and dependencies
  * Ensure proper `await` usage
  * Handle `asyncio.TimeoutError` and cancellation
  * Prevent blocking the event loop with CPU-intensive tasks

* **Routers (`APIRouter`)**:
  * Organize large applications into modular routers
  * Use router prefixes and tags for logical grouping
  * Apply common dependencies at router level when appropriate

* **Background Tasks**:
  * Use for non-blocking operations after response (email sending, logging)
  * Avoid long-running tasks; use proper job queues instead

* **Middleware**:
  * Implement for cross-cutting concerns (auth, logging, CORS)
  * Be mindful of middleware execution order
  * Use built-in middleware when available

* **WebSockets**:
  * Use for real-time bidirectional communication
  * Implement proper connection management
  * Handle connection errors and timeouts

  ```python
  @app.websocket("/ws/{client_id}")
  async def websocket_endpoint(websocket: WebSocket, client_id: str):
      await websocket.accept()
      try:
          # Add to connection manager
          manager.add_connection(client_id, websocket)
          
          # Process messages in a loop
          while True:
              data = await websocket.receive_text()
              # Process the received data
              await websocket.send_text(f"Message processed: {data}")
      except WebSocketDisconnect:
          # Handle disconnection
          manager.remove_connection(client_id)
      except Exception as e:
          # Handle other errors
          logger.error(f"WebSocket error: {str(e)}")
          manager.remove_connection(client_id)
  ```

* **Application Lifecycle Events**:
  * Use startup/shutdown events for resource initialization/cleanup
  * Set up database connections, logging, and background services

  ```python
  @app.on_event("startup")
  async def startup_event():
      # Initialize resources
      await initialize_database()
      # Start background services
      app.background_tasks = set()
      task = asyncio.create_task(background_service_loop())
      app.background_tasks.add(task)
      
  @app.on_event("shutdown")
  async def shutdown_event():
      # Clean up resources
      await close_database_connections()
      # Cancel background tasks
      for task in app.background_tasks:
          task.cancel()
  ```

---

## See Also
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [FastAPI Error Handling](/fastapi/fastapi-error-handling.md)
- [FastAPI Advanced Patterns](/fastapi/fastapi-advanced-patterns.md)
