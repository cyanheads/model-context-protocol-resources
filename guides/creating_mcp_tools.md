# Creating MCP Tools: A Comprehensive Guide

## Table of Contents

1. [Core Concepts](creating_tools_v2.md#core-concepts)
2. [Project Structure](creating_tools_v2.md#project-structure)
3. [Implementation Guide](creating_tools_v2.md#implementation-guide)
4. [Best Practices](creating_tools_v2.md#best-practices)
5. [Integration Patterns](creating_tools_v2.md#integration-patterns)
6. [Testing & Quality Assurance](creating_tools_v2.md#testing--quality-assurance)
7. [Deployment](creating_tools_v2.md#deployment)
8. [Security & Production Readiness](creating_tools_v2.md#security--production-readiness)

## Core Concepts

### What is an MCP Tool?

An MCP (Model Context Protocol) tool is a standardized interface that allows AI models to interact with external systems and services. Key aspects include:

- Standardized input/output formats
- Strong type safety
- Error handling
- Progress reporting
- Resource management
- Async operation support

### Key Components

1. **Server**: The MCP server implementation that handles tool registration and execution
2. **Tools**: Individual operations exposed to AI models
3. **Models**: Data structures for input/output validation
4. **Session**: Handles client communication and capabilities
5. **Configuration**: System and runtime settings

## Project Structure

```
my_mcp_tool/
├── src/
│   └── my_mcp_tool/
│       ├── __init__.py      # Entry point & CLI
│       ├── __main__.py      # Script entry
│       ├── server.py        # Core MCP server
│       ├── models.py        # Data models
│       ├── exceptions.py    # Custom exceptions
│       ├── config.py        # Configuration handling
│       └── utils/           # Helper functions
│           ├── __init__.py
│           ├── logging.py   # Logging setup
│           └── security.py  # Security utilities
├── tests/
│   ├── __init__.py
│   ├── conftest.py         # Test configuration
│   ├── test_server.py
│   ├── test_models.py
│   └── integration/        # Integration tests
├── pyproject.toml          # Project metadata
├── README.md              # Documentation
├── CHANGELOG.md          # Version history
├── LICENSE              # License information
└── .gitignore          # Git ignore rules
```

### Essential Files

#### pyproject.toml
```toml
[project]
name = "my-mcp-tool"
version = "0.1.0"
description = "MCP Tool Description"
requires-python = ">=3.10"
dependencies = [
    "mcp>=1.0.0",
    "pydantic>=2.6.3",
    "click>=8.1.7",
    "structlog>=24.1.0",
    "prometheus-client>=0.20.0"
]

[project.optional-dependencies]
dev = [
    "pytest>=8.1.1",
    "pytest-asyncio>=0.23.5",
    "pytest-cov>=4.1.0",
    "black>=24.2.0",
    "ruff>=0.3.0",
    "mypy>=1.9.0"
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project.scripts]
my-mcp-tool = "my_mcp_tool:main"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]

[tool.ruff]
target-version = "py310"
select = ["E", "F", "B", "I"]

[tool.mypy]
python_version = "3.10"
strict = true
```

## Implementation Guide

### 1. Configuration Management

```python
# config.py
from pydantic import BaseModel, Field
from pathlib import Path
import tomli

class ServerConfig(BaseModel):
    host: str = Field(default="127.0.0.1")
    port: int = Field(default=8000, ge=1024, le=65535)
    log_level: str = Field(default="INFO")
    metrics_enabled: bool = Field(default=True)

def load_config(path: Path | None = None) -> ServerConfig:
    if path and path.exists():
        with open(path, "rb") as f:
            config_dict = tomli.load(f)
        return ServerConfig(**config_dict)
    return ServerConfig()
```

### 2. Logging Setup

```python
# utils/logging.py
import structlog
import logging
from typing import Any

def setup_logging(level: str = "INFO") -> None:
    structlog.configure(
        processors=[
            structlog.processors.add_log_level,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.JSONRenderer()
        ],
        wrapper_class=structlog.make_filtering_bound_logger(
            logging.getLevelName(level)
        ),
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory(),
        cache_logger_on_first_use=True
    )
```

### 3. Custom Exceptions

```python
# exceptions.py
class MCPToolError(Exception):
    """Base exception for MCP tool errors"""
    pass

class ConfigurationError(MCPToolError):
    """Configuration related errors"""
    pass

class OperationError(MCPToolError):
    """Operation execution errors"""
    pass

class ValidationError(MCPToolError):
    """Input validation errors"""
    pass
```

### 4. Server Implementation

```python
# server.py
from mcp.server import Server
from mcp.types import Tool, TextContent, Progress
from pydantic import BaseModel, ValidationError
from prometheus_client import Counter, Histogram
import structlog
from .exceptions import *
from .config import ServerConfig

logger = structlog.get_logger()

# Metrics
REQUESTS = Counter("tool_requests_total", "Total tool requests", ["tool_name"])
LATENCY = Histogram("tool_latency_seconds", "Tool execution latency", ["tool_name"])

class MyToolInput(BaseModel):
    param1: str
    param2: int = Field(default=10, ge=0, le=100)

class ToolServer:
    def __init__(self, config: ServerConfig):
        self.config = config
        self.server = Server("my-mcp-tool")
        self.setup_routes()

    def setup_routes(self):
        @self.server.list_tools()
        async def list_tools() -> list[Tool]:
            return [
                Tool(
                    name="my_tool",
                    description="Tool description",
                    inputSchema=MyToolInput.schema(),
                )
            ]

        @self.server.call_tool()
        async def call_tool(name: str, arguments: dict) -> list[TextContent]:
            try:
                REQUESTS.labels(tool_name=name).inc()
                with LATENCY.labels(tool_name=name).time():
                    match name:
                        case "my_tool":
                            # Validate input
                            try:
                                input_model = MyToolInput(**arguments)
                            except ValidationError as e:
                                raise ValidationError(f"Invalid input: {e}")

                            # Report progress
                            if hasattr(self.server, "request_context"):
                                await self.server.request_context.session.send_progress(
                                    Progress(current=0, total=100)
                                )

                            # Execute operation
                            try:
                                result = await self.process_tool(input_model)
                            except Exception as e:
                                logger.error("operation_failed", 
                                           tool=name, 
                                           error=str(e),
                                           exc_info=True)
                                raise OperationError(f"Operation failed: {e}")

                            # Final progress update
                            if hasattr(self.server, "request_context"):
                                await self.server.request_context.session.send_progress(
                                    Progress(current=100, total=100)
                                )

                            return [TextContent(
                                type="text",
                                text=str(result)
                            )]
                        case _:
                            raise ValueError(f"Unknown tool: {name}")
            except Exception as e:
                logger.error("tool_execution_failed",
                           tool=name,
                           error=str(e),
                           exc_info=True)
                raise

    async def process_tool(self, input: MyToolInput) -> str:
        # Implementation
        pass

    async def start(self):
        options = self.server.create_initialization_options()
        async with stdio_server() as (read, write):
            await self.server.run(read, write, options)
```

### 5. Entry Point

```python
# __init__.py
import click
from pathlib import Path
import asyncio
import structlog
from prometheus_client import start_http_server
from .config import load_config
from .utils.logging import setup_logging
from .server import ToolServer

logger = structlog.get_logger()

@click.command()
@click.option("--config", type=Path, help="Config file path")
@click.option("-v", "--verbose", count=True)
def main(config: Path | None, verbose: bool) -> None:
    """MCP Tool Description"""
    try:
        # Load configuration
        cfg = load_config(config)

        # Setup logging
        log_level = "DEBUG" if verbose else cfg.log_level
        setup_logging(log_level)

        # Start metrics server if enabled
        if cfg.metrics_enabled:
            start_http_server(8000)

        # Create and run server
        server = ToolServer(cfg)
        asyncio.run(server.start())
    except Exception as e:
        logger.error("startup_failed", error=str(e), exc_info=True)
        raise
```

## Best Practices

### 1. Input Validation

Enhanced Pydantic models with strict validation:

```python
from pydantic import BaseModel, Field, validator
from typing import List
import re

class ToolInput(BaseModel):
    path: Path = Field(..., description="File path")
    count: int = Field(default=10, ge=0, le=100)
    mode: str = Field(..., pattern="^(read|write)$")
    tags: List[str] = Field(default_factory=list)

    @validator("path")
    def validate_path(cls, v):
        if not v.is_absolute():
            raise ValueError("Path must be absolute")
        return v

    @validator("tags")
    def validate_tags(cls, v):
        if not all(re.match(r"^[a-zA-Z0-9_-]+$", tag) for tag in v):
            raise ValueError("Tags must be alphanumeric")
        return v

    class Config:
        frozen = True
```

### 2. Resource Management

Enhanced resource manager with timeouts and cleanup:

```python
from contextlib import asynccontextmanager
import asyncio
from typing import Dict, Any

class ResourceManager:
    def __init__(self):
        self._resources: Dict[str, Any] = {}
        self._locks: Dict[str, asyncio.Lock] = {}
        self._timeouts: Dict[str, float] = {}

    @asynccontextmanager
    async def acquire(self, resource_id: str, timeout: float = 30.0):
        if resource_id not in self._locks:
            self._locks[resource_id] = asyncio.Lock()

        try:
            await asyncio.wait_for(
                self._locks[resource_id].acquire(),
                timeout=timeout
            )
            yield self._resources.get(resource_id)
        finally:
            if resource_id in self._locks:
                self._locks[resource_id].release()

    async def cleanup(self, max_age: float = 3600.0):
        """Cleanup old resources"""
        current_time = asyncio.get_event_loop().time()
        to_remove = [
            rid for rid, timestamp in self._timeouts.items()
            if current_time - timestamp > max_age
        ]
        
        for rid in to_remove:
            if rid in self._resources:
                del self._resources[rid]
            if rid in self._timeouts:
                del self._timeouts[rid]
```

## Testing & Quality Assurance

### 1. Comprehensive Testing

```python
# conftest.py
import pytest
import asyncio
from pathlib import Path
from typing import AsyncGenerator
from my_mcp_tool.server import ToolServer
from my_mcp_tool.config import ServerConfig

@pytest.fixture
async def server() -> AsyncGenerator[ToolServer, None]:
    config = ServerConfig()
    server = ToolServer(config)
    yield server

# test_server.py
import pytest
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_tool_execution(server: ToolServer):
    # Arrange
    input_data = {
        "param1": "test",
        "param2": 42
    }
    
    # Act
    with patch("my_mcp_tool.server.process_tool", new_callable=AsyncMock) as mock:
        mock.return_value = "success"
        result = await server.call_tool("my_tool", input_data)
    
    # Assert
    assert len(result) == 1
    assert result[0].type == "text"
    assert result[0].text == "success"
    mock.assert_called_once()

@pytest.mark.asyncio
async def test_invalid_input(server: ToolServer):
    # Arrange
    invalid_input = {
        "param1": "test",
        "param2": -1  # Invalid value
    }
    
    # Act & Assert
    with pytest.raises(ValidationError):
        await server.call_tool("my_tool", invalid_input)
```

## Security & Production Readiness

### 1. Security Enhancements

```python
# utils/security.py
import secrets
from pathlib import Path
from typing import Optional

def generate_token() -> str:
    """Generate secure token"""
    return secrets.token_urlsafe(32)

def sanitize_path(path: str, base_dir: Optional[Path] = None) -> Path:
    """Sanitize and validate path"""
    try:
        clean_path = Path(path).resolve()
        if base_dir:
            if not clean_path.is_relative_to(base_dir):
                raise ValueError("Path outside base directory")
        return clean_path
    except Exception as e:
        raise ValueError(f"Invalid path: {e}")

def rate_limit(key: str, limit: int, window: int = 60) -> bool:
    """Simple rate limiting"""
    # Implementation using Redis or similar
    pass
```

### 2. Production Configuration

```python
# Production config example
server_config = {
    "host": "0.0.0.0",
    "port": 8000,
    "log_level": "INFO",
    "metrics_enabled": True,
    "rate_limit": {
        "enabled": True,
        "limit": 100,
        "window": 60
    },
    "security": {
        "token_required": True,
        "allowed_origins": ["https://api.example.com"],
        "max_request_size": 1048576
    },
    "timeouts": {
        "operation": 30.0,
        "connection": 5.0
    }
}
```

## Deployment

### 1. Container Support

```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY . .

RUN pip install uv && \
    uv pip install --system -e .

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

ENTRYPOINT ["uvx", "my-mcp-tool"]
```

## Conclusion

Building production-ready MCP tools requires attention to:

1. Core Implementation
   - Strong type safety
   - Comprehensive error handling
   - Efficient resource management
   - Async operation support

2. Quality & Testing
   - Unit and integration tests
   - Performance testing
   - Security testing
   - Code quality tools

3. Production Features
   - Monitoring & metrics
   - Health checks
   - Rate limiting
   - Security measures

4. Operations
   - Container support
   - Configuration management
   - Logging & tracing
   - Documentation

Following these patterns ensures reliable, maintainable, and secure tools that integrate well with AI models in production environments.
