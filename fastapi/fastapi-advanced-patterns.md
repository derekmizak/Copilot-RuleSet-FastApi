# GitHub Copilot: Advanced FastAPI Patterns

Guidelines for implementing advanced patterns in FastAPI applications, including WebSockets, background tasks, and file uploads.

**Keywords**: #fastapi #advanced #websockets #background-tasks #file-upload #real-time

---

## 🏅 Advanced FastAPI Patterns

* **WebSockets Implementation**:
  * Create real-time communication endpoints:

  ```python
  from fastapi import FastAPI, WebSocket, WebSocketDisconnect
  from typing import Dict, List

  app = FastAPI()

  class ConnectionManager:
      def __init__(self):
          self.active_connections: Dict[str, List[WebSocket]] = {}

      async def connect(self, websocket: WebSocket, room_id: str):
          await websocket.accept()
          if room_id not in self.active_connections:
              self.active_connections[room_id] = []
          self.active_connections[room_id].append(websocket)

      def disconnect(self, websocket: WebSocket, room_id: str):
          if room_id in self.active_connections:
              self.active_connections[room_id].remove(websocket)
              if not self.active_connections[room_id]:
                  del self.active_connections[room_id]

      async def broadcast(self, message: str, room_id: str):
          if room_id in self.active_connections:
              for connection in self.active_connections[room_id]:
                  await connection.send_text(message)

  manager = ConnectionManager()

  @app.websocket("/ws/{room_id}")
  async def websocket_endpoint(websocket: WebSocket, room_id: str):
      await manager.connect(websocket, room_id)
      try:
          while True:
              data = await websocket.receive_text()
              await manager.broadcast(f"Client in room {room_id}: {data}", room_id)
      except WebSocketDisconnect:
          manager.disconnect(websocket, room_id)
          await manager.broadcast(f"Client left the room {room_id}", room_id)
  ```

* **Background Tasks with Status Updates**:
  * Implement long-running tasks with status tracking:

  ```python
  from fastapi import FastAPI, BackgroundTasks, HTTPException, Depends
  from pydantic import BaseModel, UUID4
  from typing import Dict, Optional
  import uuid
  import time
  import asyncio

  app = FastAPI()

  # Simple in-memory task storage with status
  task_status: Dict[str, Dict] = {}

  class TaskStatus(BaseModel):
      id: str
      status: str
      progress: int = 0
      result: Optional[Dict] = None
      error: Optional[str] = None

  async def long_running_process(task_id: str, params: Dict):
      try:
          # Update task status to "running"
          task_status[task_id]["status"] = "running"
          
          # Simulate work with progress updates
          total_steps = 10
          for step in range(1, total_steps + 1):
              # Do actual work here
              await asyncio.sleep(1)  # Simulate work
              
              # Update progress
              progress = int(step * 100 / total_steps)
              task_status[task_id]["progress"] = progress
          
          # Task completed successfully
          task_status[task_id]["status"] = "completed"
          task_status[task_id]["result"] = {"message": "Task completed successfully"}
      except Exception as e:
          # Handle errors
          task_status[task_id]["status"] = "failed"
          task_status[task_id]["error"] = str(e)

  @app.post("/tasks/", response_model=TaskStatus)
  async def create_task(background_tasks: BackgroundTasks):
      task_id = str(uuid.uuid4())
      task_status[task_id] = {"id": task_id, "status": "pending", "progress": 0}
      
      # Start the task in the background
      background_tasks.add_task(long_running_process, task_id, {})
      
      return TaskStatus(**task_status[task_id])

  @app.get("/tasks/{task_id}", response_model=TaskStatus)
  async def get_task_status(task_id: str):
      if task_id not in task_status:
          raise HTTPException(status_code=404, detail="Task not found")
      return TaskStatus(**task_status[task_id])
  ```

* **File Upload with Validation**:
  * Handle file uploads with type and size validation:

  ```python
  from fastapi import FastAPI, File, UploadFile, HTTPException
  from fastapi.responses import JSONResponse
  import aiofiles
  import os
  import uuid
  import magic  # python-magic library for MIME type detection

  app = FastAPI()

  UPLOAD_DIR = "uploads"
  MAX_SIZE = 5 * 1024 * 1024  # 5MB
  ALLOWED_TYPES = ["image/jpeg", "image/png", "application/pdf"]

  os.makedirs(UPLOAD_DIR, exist_ok=True)

  @app.post("/upload/")
  async def upload_file(file: UploadFile = File(...)):
      # Check file size
      contents = await file.read()
      if len(contents) > MAX_SIZE:
          raise HTTPException(status_code=413, detail="File too large")
          
      # Check file type using python-magic
      mime_type = magic.from_buffer(contents, mime=True)
      if mime_type not in ALLOWED_TYPES:
          raise HTTPException(
              status_code=415, 
              detail=f"Unsupported file type: {mime_type}. Allowed types: {ALLOWED_TYPES}"
          )
          
      # Generate a secure filename
      file_ext = os.path.splitext(file.filename)[1]
      secure_filename = f"{uuid.uuid4()}{file_ext}"
      file_path = os.path.join(UPLOAD_DIR, secure_filename)
      
      # Save the file
      async with aiofiles.open(file_path, "wb") as out_file:
          await out_file.write(contents)
          
      return {"filename": secure_filename, "size": len(contents), "type": mime_type}
  ```

* **Custom Response Streaming**:
  * Stream large datasets to clients:

  ```python
  from fastapi import FastAPI
  from fastapi.responses import StreamingResponse
  import io
  import csv
  import asyncio
  from typing import AsyncGenerator, List, Dict

  app = FastAPI()

  async def get_data_chunks() -> AsyncGenerator[List[Dict], None]:
      """Simulate fetching data in chunks from a database or other source."""
      # In a real app, this would fetch from a database in chunks
      for i in range(5):  # Simulate 5 batches
          # Simulate db query time
          await asyncio.sleep(0.5)
          
          # Generate some example data
          chunk = [
              {"id": i * 1000 + j, "name": f"Item {i * 1000 + j}", "value": i * j}
              for j in range(1000)  # 1000 items per chunk
          ]
          yield chunk

  @app.get("/stream-csv/")
  async def stream_large_csv():
      async def generate_csv():
          # Write CSV header
          buffer = io.StringIO()
          writer = csv.DictWriter(buffer, fieldnames=["id", "name", "value"])
          writer.writeheader()
          yield buffer.getvalue()
          buffer.seek(0)
          buffer.truncate(0)
          
          # Generate data in chunks
          async for chunk in get_data_chunks():
              for item in chunk:
                  writer.writerow(item)
              # Yield the current buffer content
              yield buffer.getvalue()
              # Clear the buffer for the next chunk
              buffer.seek(0)
              buffer.truncate(0)
              
      return StreamingResponse(
          generate_csv(),
          media_type="text/csv",
          headers={"Content-Disposition": "attachment; filename=large_data.csv"}
      )
  ```

* **API Versioning Strategy**:
  * Implement API versioning with router organization:

  ```python
  from fastapi import FastAPI, APIRouter

  app = FastAPI()

  # Create versioned routers
  v1_router = APIRouter(prefix="/api/v1")
  v2_router = APIRouter(prefix="/api/v2")

  # Define v1 endpoints
  @v1_router.get("/users/", tags=["v1"])
  async def get_users_v1():
      return [{"id": 1, "name": "User 1 from V1"}]

  # Define v2 endpoints with schema improvements
  @v2_router.get("/users/", tags=["v2"])
  async def get_users_v2():
      return [{"id": 1, "name": "User 1", "email": "user1@example.com"}]

  # Include both versions in the app
  app.include_router(v1_router, tags=["v1"])
  app.include_router(v2_router, tags=["v2"])
  ```

---

## See Also
- [FastAPI Framework Rules](/fastapi/fastapi-framework-rules.md)
- [FastAPI Error Handling](/fastapi/fastapi-error-handling.md)
- [FastAPI GraphQL](/fastapi/fastapi-graphql.md)
