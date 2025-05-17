# GitHub Copilot: FastAPI Testing

Guidelines for testing FastAPI applications effectively, including endpoint tests, dependency overrides, and async testing.

**Keywords**: #fastapi #testing #pytest #testclient #async-testing

---

## 🧪 Testing FastAPI Code

* **API Endpoint Tests**:
  * Use TestClient for endpoint testing:

  ```python
  from fastapi.testclient import TestClient
  from app.main import app

  client = TestClient(app)

  def test_read_user():
      response = client.get("/users/1")
      assert response.status_code == 200
      data = response.json()
      assert data["id"] == 1
      assert "username" in data
  ```

* **Dependency Overrides**:
  * Override dependencies for isolated testing:

  ```python
  from app.dependencies import get_db
  
  # Mock database dependency
  def override_get_db():
      db = TestingSessionLocal()
      try:
          yield db
      finally:
          db.close()
  
  app.dependency_overrides[get_db] = override_get_db
  ```

* **Async Test Functions**:
  * Write async tests for async endpoints:

  ```python
  import pytest
  from httpx import AsyncClient
  from app.main import app

  @pytest.mark.asyncio
  async def test_async_endpoint():
      async with AsyncClient(app=app, base_url="http://test") as client:
          response = await client.get("/async-endpoint")
          assert response.status_code == 200
  ```

* **Comprehensive Test Setup**:
  * Create a flexible test fixture system:

  ```python
  import pytest
  from fastapi.testclient import TestClient
  from sqlalchemy import create_engine
  from sqlalchemy.orm import sessionmaker
  from sqlalchemy.pool import StaticPool

  from app.main import app
  from app.db.base import Base
  from app.api.dependencies.db import get_db
  from app.models.domain.user import User

  # Create test database
  SQLALCHEMY_TEST_DATABASE_URL = "sqlite:///:memory:"
  engine = create_engine(
      SQLALCHEMY_TEST_DATABASE_URL,
      connect_args={"check_same_thread": False},
      poolclass=StaticPool,
  )
  TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

  # Create tables
  Base.metadata.create_all(bind=engine)

  @pytest.fixture
  def db():
      """Database session fixture"""
      db = TestingSessionLocal()
      try:
          yield db
      finally:
          db.close()

  @pytest.fixture
  def client(db):
      """Test client with db dependency override"""
      def override_get_db():
          try:
              yield db
          finally:
              pass

      app.dependency_overrides[get_db] = override_get_db
      with TestClient(app) as c:
          yield c
      app.dependency_overrides = {}

  @pytest.fixture
  def test_user(db):
      """Create a test user in the database"""
      user = User(
          email="test@example.com",
          hashed_password="$2b$12$BxYV1IfyHWQYJB.ylqKk2.GK9VadZR0LeJ3sjxpOaMqpX7MMmDEUC",  # "password"
          full_name="Test User",
          is_active=True,
      )
      db.add(user)
      db.commit()
      db.refresh(user)
      return user
  ```

* **Mock External Services**:
  * Use mocking libraries to isolate tests:

  ```python
  import pytest
  from unittest.mock import patch, MagicMock
  from app.services.external import ExternalService

  @pytest.fixture
  def mock_external_service():
      with patch("app.api.endpoints.users.get_external_service") as mock:
          service_mock = MagicMock()
          mock.return_value = service_mock
          yield service_mock

  def test_user_with_external_data(client, mock_external_service):
      # Configure the mock
      mock_external_service.get_user_profile.return_value = {
          "premium": True,
          "subscription_level": "gold"
      }
      
      # Test the endpoint
      response = client.get("/users/1/profile")
      assert response.status_code == 200
      data = response.json()
      assert data["subscription_level"] == "gold"
      
      # Verify the mock was called correctly
      mock_external_service.get_user_profile.assert_called_once_with(1)
  ```

* **Data-Driven Testing**:
  * Use parameterized tests for multiple test cases:

  ```python
  import pytest

  @pytest.mark.parametrize(
      "user_id,expected_status,expected_data",
      [
          (1, 200, {"id": 1, "name": "User 1"}),
          (999, 404, {"detail": "User not found"}),
          ("invalid", 422, {"detail": "Invalid ID format"}),
      ]
  )
  def test_get_user(client, user_id, expected_status, expected_data):
      response = client.get(f"/users/{user_id}")
      assert response.status_code == expected_status
      assert response.json() == expected_data
  ```

---

## See Also
- [FastAPI Foundations](/fastapi/fastapi-foundations.md)
- [FastAPI Documentation](/fastapi/fastapi-documentation.md)
- [FastAPI Performance](/fastapi/fastapi-performance.md)
