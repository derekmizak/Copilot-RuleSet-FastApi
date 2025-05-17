# GitHub Copilot: GraphQL with FastAPI

Guidelines for implementing GraphQL APIs with FastAPI using the Strawberry GraphQL library, including authentication, pagination, filtering, and sorting.

**Keywords**: #fastapi #graphql #strawberry #api #schema #resolver

---

## 🔄 GraphQL with FastAPI

* **Strawberry GraphQL Integration**:
  * Implement GraphQL APIs with FastAPI:

  ```python
  from fastapi import FastAPI
  from strawberry.fastapi import GraphQLRouter
  import strawberry
  from typing import List, Optional
  
  # Define GraphQL types with Strawberry
  @strawberry.type
  class Author:
      id: int
      name: str
      books: List["Book"]
  
  @strawberry.type
  class Book:
      id: int
      title: str
      author: Author
  
  # Sample data
  books_db = [
      {"id": 1, "title": "The Great Gatsby", "author_id": 1},
      {"id": 2, "title": "To Kill a Mockingbird", "author_id": 2},
  ]
  
  authors_db = [
      {"id": 1, "name": "F. Scott Fitzgerald"},
      {"id": 2, "name": "Harper Lee"},
  ]
  
  # Resolver functions
  def get_books() -> List[Book]:
      return [
          Book(
              id=book["id"],
              title=book["title"],
              author=Author(
                  id=next(a["id"] for a in authors_db if a["id"] == book["author_id"]),
                  name=next(a["name"] for a in authors_db if a["id"] == book["author_id"]),
                  books=[]
              )
          )
          for book in books_db
      ]
  
  def get_book(book_id: int) -> Optional[Book]:
      for book in books_db:
          if book["id"] == book_id:
              return Book(
                  id=book["id"],
                  title=book["title"],
                  author=Author(
                      id=next(a["id"] for a in authors_db if a["id"] == book["author_id"]),
                      name=next(a["name"] for a in authors_db if a["id"] == book["author_id"]),
                      books=[]
                  )
              )
      return None
  
  # Define GraphQL schema
  @strawberry.type
  class Query:
      books: List[Book] = strawberry.field(resolver=get_books)
      book: Optional[Book] = strawberry.field(resolver=get_book)
  
  schema = strawberry.Schema(query=Query)
  
  # Create FastAPI app with GraphQL endpoint
  app = FastAPI()
  graphql_app = GraphQLRouter(schema)
  
  app.include_router(graphql_app, prefix="/graphql")
  ```

* **Authentication in GraphQL**:
  * Implementing JWT authentication with Strawberry:

  ```python
  from fastapi import FastAPI, Depends, HTTPException, status
  from fastapi.security import OAuth2PasswordBearer
  import strawberry
  from strawberry.fastapi import GraphQLRouter
  from typing import List, Optional, Annotated
  from jose import JWTError, jwt
  from datetime import datetime, timedelta
  from pydantic import BaseModel
  
  # Security configuration
  oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
  SECRET_KEY = "your-secret-key"
  ALGORITHM = "HS256"
  
  # Pydantic models
  class User(BaseModel):
      username: str
      email: str
      disabled: bool = False
  
  # Mock database
  fake_users_db = {
      "johndoe": {
          "username": "johndoe",
          "email": "john@example.com",
          "disabled": False
      }
  }
  
  # Authentication logic
  async def get_current_user(token: str = Depends(oauth2_scheme)):
      credentials_exception = HTTPException(
          status_code=status.HTTP_401_UNAUTHORIZED,
          detail="Could not validate credentials",
          headers={"WWW-Authenticate": "Bearer"},
      )
      try:
          payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
          username: str = payload.get("sub")
          if username is None:
              raise credentials_exception
      except JWTError:
          raise credentials_exception
      
      user = fake_users_db.get(username)
      if user is None:
          raise credentials_exception
      
      return User(**user)
  
  # Strawberry context
  class Context:
      def __init__(self, user: Optional[User] = None):
          self.user = user
  
  # GraphQL types
  @strawberry.type
  class UserType:
      username: str
      email: str
  
  @strawberry.type
  class Query:
      @strawberry.field
      def me(self, info) -> UserType:
          # Access user from context
          user = info.context.user
          if not user:
              raise Exception("Not authenticated")
          return UserType(username=user.username, email=user.email)
  
  schema = strawberry.Schema(query=Query)
  
  # FastAPI app setup
  app = FastAPI()
  
  # Custom GraphQL router with auth
  async def get_context(token: Optional[str] = Depends(oauth2_scheme)):
      if token:
          try:
              user = await get_current_user(token)
              return Context(user=user)
          except HTTPException:
              return Context()
      return Context()
  
  graphql_app = GraphQLRouter(
      schema,
      context_getter=get_context
  )
  
  app.include_router(graphql_app, prefix="/graphql")
  ```

* **Advanced GraphQL Features**:
  * Implement pagination, filtering, and sorting:

  ```python
  import strawberry
  from enum import Enum
  from typing import List, Optional
  
  # Sort direction enum
  @strawberry.enum
  class SortDirection(Enum):
      ASC = "asc"
      DESC = "desc"
  
  # Filter input types
  @strawberry.input
  class BookFilter:
      title_contains: Optional[str] = None
      author_id: Optional[int] = None
  
  # Pagination input
  @strawberry.input
  class PaginationInput:
      page: int = 1
      page_size: int = 10
  
  # Sort input
  @strawberry.input
  class BookSortInput:
      field: str
      direction: SortDirection = SortDirection.ASC
  
  # Query implementation
  @strawberry.type
  class Query:
      @strawberry.field
      def books(
          self, 
          info, 
          filter: Optional[BookFilter] = None,
          pagination: Optional[PaginationInput] = None,
          sort: Optional[BookSortInput] = None
      ) -> List[Book]:
          # Start with all books
          result = books_db.copy()
          
          # Apply filters
          if filter:
              if filter.title_contains:
                  result = [b for b in result if filter.title_contains.lower() in b["title"].lower()]
              if filter.author_id is not None:
                  result = [b for b in result if b["author_id"] == filter.author_id]
          
          # Apply sorting
          if sort:
              reverse = sort.direction == SortDirection.DESC
              result.sort(key=lambda b: b[sort.field], reverse=reverse)
          
          # Apply pagination
          if pagination:
              start = (pagination.page - 1) * pagination.page_size
              end = start + pagination.page_size
              result = result[start:end]
              
          # Convert to GraphQL types
          return [
              Book(
                  id=book["id"],
                  title=book["title"],
                  author=get_author_by_id(book["author_id"])
              )
              for book in result
          ]
  ```

* **GraphQL Mutations**:
  * Implement data modifications with GraphQL:

  ```python
  import strawberry
  from typing import List, Optional
  
  # Input type for creating a new book
  @strawberry.input
  class BookInput:
      title: str
      author_id: int
  
  # Mutation type for data modifications
  @strawberry.type
  class Mutation:
      @strawberry.mutation
      def create_book(self, input: BookInput) -> Book:
          # Get the next available ID
          next_id = max(book["id"] for book in books_db) + 1
          
          # Create the new book
          new_book = {
              "id": next_id,
              "title": input.title,
              "author_id": input.author_id
          }
          
          # Add to the database
          books_db.append(new_book)
          
          # Return the created book
          return Book(
              id=new_book["id"],
              title=new_book["title"],
              author=get_author_by_id(new_book["author_id"])
          )
      
      @strawberry.mutation
      def update_book(self, id: int, input: BookInput) -> Optional[Book]:
          # Find the book
          for book in books_db:
              if book["id"] == id:
                  # Update the book
                  book["title"] = input.title
                  book["author_id"] = input.author_id
                  
                  # Return the updated book
                  return Book(
                      id=book["id"],
                      title=book["title"],
                      author=get_author_by_id(book["author_id"])
                  )
          
          return None
      
      @strawberry.mutation
      def delete_book(self, id: int) -> bool:
          global books_db
          original_length = len(books_db)
          books_db = [book for book in books_db if book["id"] != id]
          return len(books_db) < original_length
  
  # Update the schema to include mutations
  schema = strawberry.Schema(query=Query, mutation=Mutation)
  ```

---

## See Also
- [FastAPI Advanced Patterns](/fastapi/fastapi-advanced-patterns.md)
- [FastAPI API & DB Standards](/fastapi/fastapi-api-db-standards.md)
- [FastAPI Deployment](/fastapi/fastapi-deployment.md)
