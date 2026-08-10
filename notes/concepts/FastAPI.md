# Overview

- FastAPI is a Python web framework for building HTTP APIs with standard Python type hints

- FastAPI is built on Starlette for web behavior and Pydantic for validation and serialization

- ASGI (Asynchronous Server Gateway Interface) is the interface between the application and an async-capable server such as Uvicorn

- Type hints are used to validate request data, convert values, generate OpenAPI schemas, create interactive documentation, and improve editor support

- FastAPI automatically provides:

    Swagger UI at `/docs`

    ReDoc at `/redoc`

    OpenAPI JSON at `/openapi.json`

- A good mental model is:

    Decorator = Which HTTP request reaches a function

    Function parameters = Where input comes from and how it is validated

    Return annotation/response model = What may be returned publicly

    Dependencies = Shared request-time requirements

# Installation and First Application

- A modern project can use `uv`:

    ```bash
    uv init my-api
    cd my-api
    uv add "fastapi[standard]"
    uv run fastapi dev
    ```

- A minimal `main.py` is:

    ```python
    from fastapi import FastAPI

    app = FastAPI(title="Example API")


    @app.get("/")
    async def root():
        return {"message": "Hello World"}
    ```

- `fastapi dev` starts a development server with automatic reload

- `fastapi run` starts the application in production mode. Production still needs appropriate process, proxy, logging, and deployment configuration

- Uvicorn can run the application directly: `uvicorn main:app --reload`

- `main:app` means the `app` object inside the `main` module

# Path Operations

- A path operation combines a path and HTTP method

    ```python
    @app.get("/items")
    async def list_items():
        return []


    @app.post("/items", status_code=201)
    async def create_item():
        return {"created": True}
    ```

- Supported decorators include `get`, `post`, `put`, `patch`, `delete`, `options`, `head`, and `trace`

- Decorator options include `status_code`, `response_model`, `tags`, `summary`, `description`, `responses`, `deprecated`, and `operation_id`

- Define fixed paths before dynamic paths when both could match, such as `/users/me` before `/users/{user_id}`

# Request Parameters

- FastAPI determines the parameter source from the route, type, and explicit helper

    ```python
    from typing import Annotated
    from fastapi import FastAPI, Path, Query

    app = FastAPI()


    @app.get("/items/{item_id}")
    async def read_item(
        item_id: Annotated[int, Path(gt=0)],
        q: Annotated[str | None, Query(min_length=3, max_length=50)] = None,
        page: Annotated[int, Query(ge=1)] = 1,
    ):
        return {"item_id": item_id, "q": q, "page": page}
    ```

- A name appearing in `{}` is a path parameter

- Simple values not in the path normally become query parameters

- Pydantic models normally become JSON request bodies

- Explicit sources include `Path`, `Query`, `Header`, `Cookie`, `Body`, `Form`, and `File`

- Prefer `Annotated` because it keeps the actual Python type clear and attaches FastAPI metadata separately

- Repeated query values may use a list: `tags: Annotated[list[str] | None, Query()] = None`

- Aliases map an external name to a Python-safe name

# Pydantic Models and Validation

- Pydantic models define structured input and output

    ```python
    from pydantic import BaseModel, ConfigDict, Field


    class ItemCreate(BaseModel):
        name: str = Field(min_length=1, max_length=100)
        price: float = Field(gt=0)
        description: str | None = None
        tags: list[str] = []


    class ItemRead(BaseModel):
        model_config = ConfigDict(from_attributes=True)

        id: int
        name: str
        price: float
    ```

- Use separate input and output models so clients cannot set server-owned fields and private database fields cannot leak

- Common field constraints include `gt`, `ge`, `lt`, `le`, `min_length`, `max_length`, `pattern`, examples, and descriptions

- Pydantic supports field validators and model validators for rules beyond basic types

- `model_dump()` converts a model to a dictionary. `model_validate()` validates compatible input into a model

- Validation failures become a structured `422 Unprocessable Entity` response by default

- A `response_model` validates and filters outgoing data

    ```python
    @app.post("/items", response_model=ItemRead, status_code=201)
    async def create_item(item: ItemCreate) -> ItemRead:
        return await repository.create(item)
    ```

- Response filtering is a security feature because undeclared fields are removed

# Responses and Errors

- Returning dictionaries, lists, models, and compatible values produces JSON responses

- Return a `Response` subclass for precise behavior:

    `JSONResponse`: JSON with custom status or headers

    `PlainTextResponse`: Plain text

    `HTMLResponse`: HTML

    `FileResponse`: Stream a file

    `StreamingResponse`: Stream an iterator or async iterator

    `RedirectResponse`: Redirect

- `HTTPException` ends request processing with an expected HTTP error

    ```python
    from fastapi import HTTPException

    if item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    ```

- Add custom exception handlers with `@app.exception_handler(ExceptionType)`

- Do not expose stack traces, database errors, tokens, or internal details to clients

- Use correct HTTP status codes and a consistent error schema

# Dependency Injection

- A dependency is a callable declaring work required by a path operation

    ```python
    from typing import Annotated
    from fastapi import Depends, Header, HTTPException


    async def require_api_key(x_api_key: Annotated[str, Header()]):
        if x_api_key != "expected-secret":
            raise HTTPException(status_code=401, detail="Invalid API key")
        return x_api_key


    ApiKey = Annotated[str, Depends(require_api_key)]


    @app.get("/private")
    async def private_data(api_key: ApiKey):
        return {"allowed": True}
    ```

- Dependencies are useful for authentication, authorization, database sessions, configuration, pagination, shared validation, and request context

- Dependencies may depend on other dependencies, creating a reusable dependency tree

- A dependency using `yield` performs setup before yielding and cleanup afterward

    ```python
    async def get_session():
        async with SessionFactory() as session:
            yield session
    ```

- Dependency results are cached once per request by default. Set `use_cache=False` when the same dependency must run again

- Dependencies may be applied at parameter, decorator, router, or application level

# Routers and Application Structure

- `APIRouter` splits a large API into modules

    ```python
    # app/routers/users.py
    from fastapi import APIRouter

    router = APIRouter(prefix="/users", tags=["users"])


    @router.get("")
    async def list_users():
        return []
    ```

    ```python
    # app/main.py
    from fastapi import FastAPI
    from app.routers import users

    app = FastAPI()
    app.include_router(users.router)
    ```

- A common structure separates routers, schemas, models, services, repositories, dependencies, configuration, and database setup

- Routers should translate HTTP input/output. Business rules belong in services or domain functions, and database details belong behind repositories/data access

# Async and Concurrency

- Use `async def` when awaiting async libraries such as async database drivers or HTTP clients

- Use normal `def` for blocking libraries. FastAPI runs normal path operations and dependencies in a thread pool

- Do not call blocking I/O directly inside `async def`; it blocks the event loop and delays unrelated requests

- Async improves concurrency for waiting-heavy work. It does not make CPU-heavy computation faster

- Move large CPU work to worker processes or a task system

- Keep database pools and outbound request concurrency bounded so the application does not overload dependencies

# Databases and Transactions

- FastAPI does not require one database library. SQLAlchemy, SQLModel, database-specific drivers, and NoSQL clients may all be used

- Create one database session per request through a dependency

- Keep transaction boundaries explicit. Commit only after the complete operation succeeds and roll back after failure

- Never share a normal mutable database session across concurrent requests

- Use migrations such as Alembic for schema changes. Do not depend on automatic table creation in production

- Avoid returning ORM objects without an explicit public response model

- Prevent SQL injection through parameterized queries or ORM parameters

# Authentication and Authorization

- FastAPI includes security dependency helpers for HTTP Basic, API keys, OAuth2 flows, OpenID Connect discovery, and bearer tokens

- A typical bearer flow is:

    Credentials submitted -> Password verified -> Signed token/session created -> Client sends bearer value -> Dependency validates identity -> Endpoint checks permissions

- `OAuth2PasswordBearer` extracts a bearer token and integrates its requirement into OpenAPI. It does not validate the token by itself

- Passwords must be hashed with an appropriate password-hashing algorithm and unique salts through a maintained library

- JWT payloads are readable, not encrypted. Store only necessary claims, validate signature, issuer, audience, and expiration, and keep signing keys private

- Authentication identifies the user. Authorization must still check ownership, role, and action for each protected resource

- Prefer secure HTTP-only cookies for browser sessions when appropriate. Use HTTPS in production

# Middleware, CORS, and Background Tasks

- Middleware wraps every matching request and response

    ```python
    import time
    from fastapi import Request


    @app.middleware("http")
    async def add_process_time(request: Request, call_next):
        start = time.perf_counter()
        response = await call_next(request)
        response.headers["X-Process-Time"] = str(time.perf_counter() - start)
        return response
    ```

- Use middleware for correlation IDs, timing, logging, security headers, and cross-cutting behavior

- CORS controls which browser origins may read responses. Configure explicit production origins instead of combining credentials with a wildcard

- `BackgroundTasks` schedules small work after the response

    ```python
    from fastapi import BackgroundTasks

    @app.post("/messages")
    async def send_message(tasks: BackgroundTasks):
        tasks.add_task(write_log, "message sent")
        return {"accepted": True}
    ```

- BackgroundTasks run in the application process and are not a durable job queue. Use Celery, Dramatiq, RQ, or another worker system for retries, long jobs, durability, and separate scaling

# Files, Forms, WebSockets, and Lifespan

- Form data requires `Form`. Uploads use `File` or `UploadFile`

- `UploadFile` provides a spooled file, filename, content type, and async methods and is preferable for large uploads

- Validate upload size, actual content, destination, and permissions. Client filenames and MIME types are untrusted

- WebSockets provide a persistent bidirectional connection

    ```python
    from fastapi import WebSocket

    @app.websocket("/ws")
    async def websocket_endpoint(websocket: WebSocket):
        await websocket.accept()
        while True:
            message = await websocket.receive_text()
            await websocket.send_text(f"Echo: {message}")
    ```

- Handle disconnects, authentication, connection limits, timeouts, and multi-instance message distribution

- Lifespan manages application startup and shutdown resources

    ```python
    from contextlib import asynccontextmanager

    @asynccontextmanager
    async def lifespan(app: FastAPI):
        await open_resources()
        yield
        await close_resources()

    app = FastAPI(lifespan=lifespan)
    ```

# Testing

- FastAPI's `TestClient` uses HTTPX and works naturally with pytest

    ```python
    from fastapi.testclient import TestClient
    from app.main import app

    client = TestClient(app)


    def test_root():
        response = client.get("/")
        assert response.status_code == 200
        assert response.json() == {"message": "Hello World"}
    ```

- Override dependencies during tests with `app.dependency_overrides[original] = replacement`

- Use HTTPX `AsyncClient` for fully async tests

- Test validation, authentication, authorization, missing resources, conflicts, database rollback, and OpenAPI behavior in addition to success responses

- Keep unit tests for domain logic separate from integration tests involving the web and database layers

# Configuration, Logging, and Deployment

- Use `pydantic-settings` to validate settings loaded from environment variables

- Do not hardcode credentials. Separate development, test, and production configuration

- Log structured request IDs, status, duration, and meaningful domain events without logging passwords or tokens

- Production needs HTTPS, trusted proxy configuration, timeouts, body limits, rate limiting where needed, health checks, metrics, and error reporting

- Multiple worker processes improve CPU utilization and failure isolation but do not share in-memory state

- Containers should run a predictable command, use a non-root user where practical, expose a health endpoint, and shut down gracefully

- Run database migrations as a controlled deployment step rather than independently from every worker

- Scale the API, job workers, database pool, and downstream rate limits together

# Quick Reference

- Structured body -> Pydantic model

- URL variable -> Path parameter

- Filtering/pagination -> Query parameter

- Shared request requirement -> Dependency

- Expected client failure -> `HTTPException`

- Public output guarantee -> `response_model`

- Multiple endpoint module -> `APIRouter`

- Per-request database session -> Yield dependency

- Small post-response task -> `BackgroundTasks`

- Durable or long work -> External task queue

- Async I/O library -> `async def`

- Blocking library -> Normal `def` or deliberate thread execution

- Startup/shutdown resource -> Lifespan

- API behavior test -> `TestClient` or HTTPX
