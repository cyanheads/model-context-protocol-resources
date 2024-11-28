# Model Context Protocol (MCP) Tools: A Comprehensive Guide

## Table of Contents

1. [Introduction to MCP](#introduction-to-mcp)
2. [Core Architecture](#core-architecture)
3. [Resources](#resources)
4. [Prompts](#prompts)
5. [Tools](#tools)
6. [Sampling](#sampling)
7. [Transports](#transports)
8. [Example Integration](#example-integration)
9. [Best Practices](#best-practices)
10. [Conclusion](#conclusion)

## Introduction to MCP

The Model Context Protocol (MCP) is a standardized interface that enables AI models to interact with external systems and services in a controlled, secure manner. It provides a robust foundation for building tools that can be used by AI models to perform specific tasks while maintaining security, type safety, and proper error handling.

Key aspects include:
- Standardized communication patterns
- Strong type safety and validation
- Resource management
- Progress reporting
- Error handling
- Async operation support

## Core Architecture

### Protocol Layer

The protocol layer manages message framing and communication patterns:

```python
from typing import TypeVar, Generic, AsyncGenerator, Any
from dataclasses import dataclass
from abc import ABC, abstractmethod

RequestT = TypeVar("RequestT")
NotificationT = TypeVar("NotificationT")
ResultT = TypeVar("ResultT")

@dataclass
class RequestResponder(Generic[RequestT, ResultT]):
    """Handles incoming requests"""
    request: RequestT

class Session(Generic[RequestT, NotificationT, ResultT]):
    """Manages client-server communication"""
    
    async def send_request(
        self,
        request: RequestT,
        result_type: type[ResultT]
    ) -> ResultT:
        """Send request and await response"""
        ...

    async def send_notification(
        self,
        notification: NotificationT
    ) -> None:
        """Send one-way notification"""
        ...

    async def _received_request(
        self,
        responder: RequestResponder[RequestT, ResultT]
    ) -> None:
        """Handle incoming request"""
        ...

    async def _received_notification(
        self,
        notification: NotificationT
    ) -> None:
        """Handle incoming notification"""
        ...
```

### Message Types

MCP defines four primary message types:

```python
from typing import Any, Optional
from pydantic import BaseModel, Field

class Request(BaseModel):
    """Request message expecting response"""
    method: str
    params: Optional[dict[str, Any]] = Field(default_factory=dict)
    id: Optional[str] = None

class Notification(BaseModel):
    """One-way notification message"""
    method: str
    params: Optional[dict[str, Any]] = Field(default_factory=dict)

class Result(BaseModel):
    """Successful response"""
    result: Any
    id: Optional[str] = None

class Error(BaseModel):
    """Error response"""
    code: int
    message: str
    data: Optional[Any] = None
    id: Optional[str] = None
```

### Error Handling

Standard error codes and handling:

```python
from enum import IntEnum
from typing import Optional, Any

class ErrorCode(IntEnum):
    """Standard JSON-RPC error codes"""
    ParseError = -32700
    InvalidRequest = -32600
    MethodNotFound = -32601
    InvalidParams = -32602
    InternalError = -32603

    # Custom codes must be > -32000
    ResourceNotFound = -31999
    ValidationFailed = -31998
    OperationTimeout = -31997

class MCPError(Exception):
    """Base exception for MCP errors"""
    def __init__(
        self,
        message: str,
        code: int = ErrorCode.InternalError,
        data: Optional[Any] = None
    ) -> None:
        super().__init__(message)
        self.code = code
        self.data = data

    def to_response(self, id: Optional[str] = None) -> Error:
        """Convert to Error response"""
        return Error(
            code=self.code,
            message=str(self),
            data=self.data,
            id=id
        )
```

## Resources

Resources represent data sources that can be accessed through URIs:

```python
from typing import Optional, AsyncIterator
from pydantic import BaseModel, Field
from abc import ABC, abstractmethod
import aiohttp
import asyncio

class Resource(BaseModel):
    """Represents an accessible resource"""
    uri: str
    name: str
    description: Optional[str] = None
    mimeType: Optional[str] = None
    metadata: dict[str, Any] = Field(default_factory=dict)

class ResourceProvider(ABC):
    """Abstract base class for resource providers"""
    
    @abstractmethod
    async def get_resource(self, uri: str) -> Resource:
        """Retrieve resource by URI"""
        ...

    @abstractmethod
    async def list_resources(self) -> list[Resource]:
        """List available resources"""
        ...

    @abstractmethod
    async def watch_resource(
        self,
        uri: str
    ) -> AsyncIterator[Resource]:
        """Watch resource for changes"""
        ...

class ResourceManager:
    """Manages resource access and caching"""
    
    def __init__(self) -> None:
        self._providers: dict[str, ResourceProvider] = {}
        self._cache: dict[str, tuple[Resource, float]] = {}
        self._cache_ttl: float = 300  # 5 minutes
        
    def register_provider(
        self,
        scheme: str,
        provider: ResourceProvider
    ) -> None:
        """Register a resource provider for a URI scheme"""
        self._providers[scheme] = provider
        
    async def get_resource(self, uri: str) -> Resource:
        """Get resource, using cache if available"""
        now = asyncio.get_event_loop().time()
        
        # Check cache
        if uri in self._cache:
            resource, timestamp = self._cache[uri]
            if now - timestamp < self._cache_ttl:
                return resource
                
        # Get fresh resource
        scheme = uri.split("://")[0]
        provider = self._providers.get(scheme)
        if not provider:
            raise MCPError(
                f"No provider for scheme: {scheme}",
                code=ErrorCode.ResourceNotFound
            )
            
        resource = await provider.get_resource(uri)
        self._cache[uri] = (resource, now)
        return resource
```

## Prompts

Prompts enable standardized templates and workflows for LLM interactions:

```python
from typing import Optional, List, Any
from pydantic import BaseModel, Field
from abc import ABC, abstractmethod

class PromptArgument(BaseModel):
    """Defines a prompt argument"""
    name: str
    description: Optional[str] = None
    required: bool = False
    type: str = "string"
    default: Optional[Any] = None

class Prompt(BaseModel):
    """Defines a reusable prompt template"""
    name: str
    description: Optional[str] = None
    arguments: list[PromptArgument] = Field(default_factory=list)
    template: str
    version: str = "1.0.0"

class PromptMessage(BaseModel):
    """Represents a message in a prompt conversation"""
    role: str
    content: dict[str, Any]

class PromptResult(BaseModel):
    """Result of prompt resolution"""
    messages: list[PromptMessage]
    metadata: dict[str, Any] = Field(default_factory=dict)

class PromptProvider(ABC):
    """Abstract base class for prompt providers"""
    
    @abstractmethod
    async def get_prompt(
        self,
        name: str,
        arguments: Optional[dict[str, Any]] = None
    ) -> PromptResult:
        """Get resolved prompt with arguments"""
        ...

    @abstractmethod
    async def list_prompts(self) -> list[Prompt]:
        """List available prompts"""
        ...

class PromptManager:
    """Manages prompt templates and execution"""
    
    def __init__(self) -> None:
        self._providers: dict[str, PromptProvider] = {}
        self._prompts: dict[str, Prompt] = {}
        
    def register_provider(
        self,
        namespace: str,
        provider: PromptProvider
    ) -> None:
        """Register a prompt provider"""
        self._providers[namespace] = provider
        
    def register_prompt(self, prompt: Prompt) -> None:
        """Register a new prompt template"""
        self._prompts[prompt.name] = prompt
        
    async def list_prompts(self) -> list[Prompt]:
        """List all available prompts"""
        prompts = list(self._prompts.values())
        for provider in self._providers.values():
            prompts.extend(await provider.list_prompts())
        return prompts
        
    async def get_prompt(
        self,
        name: str,
        arguments: Optional[dict[str, Any]] = None
    ) -> PromptResult:
        """Get prompt with resolved arguments"""
        # Check local prompts
        if name in self._prompts:
            return await self._resolve_prompt(
                self._prompts[name],
                arguments or {}
            )
            
        # Check providers
        namespace = name.split("/")[0]
        provider = self._providers.get(namespace)
        if provider:
            return await provider.get_prompt(name, arguments)
            
        raise MCPError(
            f"Prompt not found: {name}",
            code=ErrorCode.ResourceNotFound
        )
        
    async def _resolve_prompt(
        self,
        prompt: Prompt,
        arguments: dict[str, Any]
    ) -> PromptResult:
        """Resolve prompt template with arguments"""
        # Validate required arguments
        required = {arg.name for arg in prompt.arguments if arg.required}
        provided = set(arguments.keys())
        missing = required - provided
        if missing:
            raise MCPError(
                f"Missing required arguments: {missing}",
                code=ErrorCode.InvalidParams
            )
            
        # Apply defaults
        final_args = {}
        for arg in prompt.arguments:
            if arg.name in arguments:
                final_args[arg.name] = arguments[arg.name]
            elif arg.default is not None:
                final_args[arg.name] = arg.default
                
        # Resolve template
        try:
            resolved = prompt.template.format(**final_args)
        except KeyError as e:
            raise MCPError(
                f"Invalid argument reference: {e}",
                code=ErrorCode.InvalidParams
            )
            
        return PromptResult(
            messages=[
                PromptMessage(
                    role="user",
                    content={"type": "text", "text": resolved}
                )
            ]
        )
```

Example prompt implementation:

```python
# Define prompts
code_analysis_prompt = Prompt(
    name="analyze-code",
    description="Analyze code for potential improvements",
    arguments=[
        PromptArgument(
            name="language",
            description="Programming language",
            required=True
        ),
        PromptArgument(
            name="code",
            description="Code to analyze",
            required=True
        )
    ],
    template=(
        "Please analyze this {language} code for potential improvements:\n\n"
        "```{language}\n{code}\n```"
    )
)

git_commit_prompt = Prompt(
    name="git-commit",
    description="Generate a Git commit message",
    arguments=[
        PromptArgument(
            name="changes",
            description="Git diff or description of changes",
            required=True
        )
    ],
    template=(
        "Generate a concise but descriptive commit message for these "
        "changes:\n\n{changes}"
    )
)

# Usage example
async def analyze_code(
    prompt_manager: PromptManager,
    language: str,
    code: str
) -> PromptResult:
    return await prompt_manager.get_prompt(
        "analyze-code",
        {
            "language": language,
            "code": code
        }
    )
```

## Tools

Tools are a powerful primitive in MCP that enable servers to expose executable functionality to clients. Unlike resources which represent static data, tools represent dynamic operations that can modify state or interact with external systems.

### Tool Definition

Tools are defined with a standard structure:

```python
from typing import Optional
from pydantic import BaseModel, Field

class ToolParameter(BaseModel):
    """Parameter definition for tool input schema"""
    type: str
    description: Optional[str] = None
    required: bool = False
    default: Optional[any] = None

class Tool(BaseModel):
    """Tool definition"""
    name: str
    description: Optional[str] = None
    inputSchema: dict[str, any] = Field(
        default_factory=lambda: {
            "type": "object",
            "properties": {}
        }
    )
```

### Tool Implementation Patterns

1. **System Operations**
```python
from pathlib import Path
import subprocess
from typing import List

class ExecuteCommandInput(BaseModel):
    """Input schema for command execution"""
    command: str
    args: List[str] = Field(default_factory=list)
    working_dir: Optional[str] = None

async def execute_command(
    command: str,
    args: List[str],
    working_dir: Optional[str] = None
) -> str:
    """Execute system command"""
    try:
        process = await asyncio.create_subprocess_exec(
            command,
            *args,
            cwd=working_dir,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE
        )
        stdout, stderr = await process.communicate()
        
        if process.returncode != 0:
            raise RuntimeError(f"Command failed: {stderr.decode()}")
            
        return stdout.decode()
    except Exception as e:
        raise RuntimeError(f"Command execution failed: {str(e)}")
```

2. **API Integrations**
```python
from datetime import datetime
from typing import List

class GitHubIssueInput(BaseModel):
    """Input schema for GitHub issue creation"""
    title: str
    body: str
    labels: List[str] = Field(default_factory=list)
    repository: str

async def create_github_issue(
    title: str,
    body: str,
    labels: List[str],
    repository: str
) -> dict:
    """Create GitHub issue"""
    async with aiohttp.ClientSession() as session:
        async with session.post(
            f"https://api.github.com/repos/{repository}/issues",
            json={
                "title": title,
                "body": body,
                "labels": labels
            },
            headers={"Authorization": f"token {GITHUB_TOKEN}"}
        ) as response:
            if response.status != 201:
                raise RuntimeError(
                    f"Failed to create issue: {await response.text()}"
                )
            return await response.json()
```

3. **Data Processing**
```python
import pandas as pd
from enum import Enum
from typing import List

class Operation(str, Enum):
    SUM = "sum"
    AVERAGE = "average"
    COUNT = "count"

class AnalyzeCsvInput(BaseModel):
    """Input schema for CSV analysis"""
    filepath: str
    operations: List[Operation]

async def analyze_csv(
    filepath: str,
    operations: List[Operation]
) -> dict:
    """Analyze CSV file"""
    try:
        df = pd.read_csv(filepath)
        results = {}
        
        for op in operations:
            match op:
                case Operation.SUM:
                    results["sum"] = df.sum().to_dict()
                case Operation.AVERAGE:
                    results["average"] = df.mean().to_dict()
                case Operation.COUNT:
                    results["count"] = len(df)
                    
        return results
    except Exception as e:
        raise RuntimeError(f"CSV analysis failed: {str(e)}")
```

### Tool Registration and Execution

```python
from mcp.server import Server
from mcp.types import TextContent, CallToolResult
import logging

logger = logging.getLogger(__name__)

async def serve() -> None:
    """Start MCP server"""
    server = Server(
        "example-server",
        capabilities={"tools": {}}
    )

    @server.list_tools()
    async def list_tools() -> list[Tool]:
        """List available tools"""
        return [
            Tool(
                name="execute_command",
                description="Run a system command",
                inputSchema=ExecuteCommandInput.schema()
            ),
            Tool(
                name="github_create_issue",
                description="Create a GitHub issue",
                inputSchema=GitHubIssueInput.schema()
            ),
            Tool(
                name="analyze_csv",
                description="Analyze a CSV file",
                inputSchema=AnalyzeCsvInput.schema()
            )
        ]

    @server.call_tool()
    async def call_tool(name: str, arguments: dict) -> CallToolResult:
        """Handle tool execution"""
        try:
            result = None
            match name:
                case "execute_command":
                    input_model = ExecuteCommandInput(**arguments)
                    result = await execute_command(
                        input_model.command,
                        input_model.args,
                        input_model.working_dir
                    )
                    
                case "github_create_issue":
                    input_model = GitHubIssueInput(**arguments)
                    result = await create_github_issue(
                        input_model.title,
                        input_model.body,
                        input_model.labels,
                        input_model.repository
                    )
                    
                case "analyze_csv":
                    input_model = AnalyzeCsvInput(**arguments)
                    result = await analyze_csv(
                        input_model.filepath,
                        input_model.operations
                    )
                    
                case _:
                    raise ValueError(f"Unknown tool: {name}")
                    
            return CallToolResult(
                content=[
                    TextContent(
                        type="text",
                        text=str(result)
                    )
                ]
            )
            
        except Exception as e:
            logger.error(f"Tool execution failed", exc_info=True)
            return CallToolResult(
                isError=True,
                content=[
                    TextContent(
                        type="text",
                        text=f"Error: {str(e)}"
                    )
                ]
            )

    # Tool updates notification
    async def notify_tool_changes() -> None:
        """Notify clients of tool changes"""
        if hasattr(server, "request_context"):
            await server.request_context.session.send_notification(
                "notifications/tools/list_changed"
            )

    # Initialize and run server
    options = server.create_initialization_options()
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            options,
            raise_exceptions=True
        )
```

### Security Considerations

1. **Input Validation**
```python
from pathlib import Path
import re

def validate_path(path: str, base_dir: Path) -> Path:
    """Validate and sanitize file path"""
    try:
        full_path = (base_dir / path).resolve()
        if not full_path.is_relative_to(base_dir):
            raise ValueError("Path outside base directory")
        return full_path
    except Exception as e:
        raise ValueError(f"Invalid path: {str(e)}")

def validate_command(command: str, allowed_commands: set[str]) -> str:
    """Validate system command"""
    if command not in allowed_commands:
        raise ValueError(f"Command not allowed: {command}")
    return command

def sanitize_input(value: str, pattern: str) -> str:
    """Sanitize string input"""
    if not re.match(pattern, value):
        raise ValueError(f"Invalid input format: {value}")
    return value
```

2. **Access Control**
```python
from datetime import datetime
from typing import Optional

class RateLimiter:
    """Rate limiting implementation"""
    def __init__(self, limit: int, window: int = 60) -> None:
        self._limit = limit
        self._window = window
        self._requests: dict[str, list[datetime]] = {}
        
    async def check_limit(self, key: str) -> bool:
        """Check if rate limit is exceeded"""
        now = datetime.utcnow()
        requests = self._requests.get(key, [])
        
        # Remove old requests
        requests = [
            r for r in requests
            if (now - r).total_seconds() <= self._window
        ]
        
        if len(requests) >= self._limit:
            return False
            
        requests.append(now)
        self._requests[key] = requests
        return True

class AccessControl:
    """Access control implementation"""
    def __init__(self) -> None:
        self._rate_limiter = RateLimiter(100)  # 100 requests per minute
        
    async def check_access(
        self,
        tool: str,
        user: Optional[str] = None
    ) -> bool:
        """Check access permissions"""
        if user and not await self._rate_limiter.check_limit(user):
            raise RuntimeError("Rate limit exceeded")
        return True
```

### Error Handling

```python
from typing import Optional, Any
from pydantic import BaseModel

class ToolError(BaseModel):
    """Tool error details"""
    code: str
    message: str
    details: Optional[dict[str, Any]] = None

def handle_tool_error(
    error: Exception,
    tool_name: str
) -> CallToolResult:
    """Convert exception to tool error result"""
    if isinstance(error, ValueError):
        return CallToolResult(
            isError=True,
            content=[
                TextContent(
                    type="text",
                    text=f"Invalid input: {str(error)}"
                )
            ]
        )
    elif isinstance(error, RuntimeError):
        return CallToolResult(
            isError=True,
            content=[
                TextContent(
                    type="text",
                    text=f"Operation failed: {str(error)}"
                )
            ]
        )
    else:
        logger.error(
            f"Unexpected error in tool {tool_name}",
            exc_info=True
        )
        return CallToolResult(
            isError=True,
            content=[
                TextContent(
                    type="text",
                    text="Internal server error"
                )
            ]
        )
```

## Sampling

Sampling provides a way to collect and analyze tool usage patterns:

```python
from typing import Optional, Any
from datetime import datetime
from pydantic import BaseModel
from abc import ABC, abstractmethod

class Sample(BaseModel):
    """Represents a tool usage sample"""
    tool_name: str
    timestamp: datetime
    duration: float
    success: bool
    error: Optional[str] = None
    metadata: dict[str, Any] = Field(default_factory=dict)

class SampleCollector(ABC):
    """Abstract base class for sample collectors"""
    
    @abstractmethod
    async def collect_sample(self, sample: Sample) -> None:
        """Collect a usage sample"""
        ...

    @abstractmethod
    async def get_samples(
        self,
        tool_name: Optional[str] = None,
        start_time: Optional[datetime] = None,
        end_time: Optional[datetime] = None
    ) -> list[Sample]:
        """Retrieve collected samples"""
        ...

class ToolSampler:
    """Manages tool usage sampling"""
    
    def __init__(self, collector: SampleCollector) -> None:
        self._collector = collector
        
    async def record_usage(
        self,
        tool_name: str,
        duration: float,
        success: bool,
        error: Optional[str] = None,
        metadata: Optional[dict[str, Any]] = None
    ) -> None:
        """Record tool usage"""
        sample = Sample(
            tool_name=tool_name,
            timestamp=datetime.utcnow(),
            duration=duration,
            success=success,
            error=error,
            metadata=metadata or {}
        )
        await self._collector.collect_sample(sample)
```

## Transports

MCP supports multiple transport mechanisms:

```python
from typing import AsyncIterator, Optional
from abc import ABC, abstractmethod
import asyncio
import aiohttp
from aiohttp import web
import sys
import json

class Transport(ABC):
    """Abstract base class for transports"""
    
    @abstractmethod
    async def send_message(self, message: dict[str, Any]) -> None:
        """Send message to other endpoint"""
        ...

    @abstractmethod
    async def receive_message(self) -> Optional[dict[str, Any]]:
        """Receive message from other endpoint"""
        ...

    @abstractmethod
    async def close(self) -> None:
        """Close transport"""
        ...

class StdioTransport(Transport):
    """Standard input/output transport"""
    
    def __init__(self) -> None:
        self._stdin = asyncio.StreamReader()
        self._stdout = asyncio.StreamWriter(
            sys.stdout.buffer,
            sys.stdout
        )
        
    async def send_message(self, message: dict[str, Any]) -> None:
        data = json.dumps(message).encode() + b"\n"
        self._stdout.write(data)
        await self._stdout.drain()
        
    async def receive_message(self) -> Optional[dict[str, Any]]:
        try:
            data = await self._stdin.readline()
            if not data:
                return None
            return json.loads(data.decode())
        except Exception:
            return None
            
    async def close(self) -> None:
        self._stdout.close()
        await self._stdout.wait_closed()

class HttpTransport(Transport):
    """HTTP with Server-Sent Events transport"""
    
    def __init__(
        self,
        endpoint: str,
        headers: Optional[dict[str, str]] = None
    ) -> None:
        self._endpoint = endpoint
        self._headers = headers or {}
        self._session: Optional[aiohttp.ClientSession] = None
        self._event_queue: asyncio.Queue[dict[str, Any]] = asyncio.Queue()
        
    async def connect(self) -> None:
        """Establish connection"""
        self._session = aiohttp.ClientSession(
            headers=self._headers
        )
        
        # Start SSE listener
        async def listen_events():
            async with self._session.get(
                f"{self._endpoint}/events"
            ) as response:
                async for event in response.content:
                    if event:
                        try:
                            data = json.loads(event.decode())
                            await self._event_queue.put(data)
                        except Exception:
                            continue
                            
        asyncio.create_task(listen_events())
        
    async def send_message(self, message: dict[str, Any]) -> None:
        if not self._session:
            raise RuntimeError("Transport not connected")
            
        async with self._session.post(
            self._endpoint,
            json=message
        ) as response:
            if response.status != 200:
                raise RuntimeError(
                    f"Failed to send message: {response.status}"
                )
                
    async def receive_message(self) -> Optional[dict[str, Any]]:
        try:
            return await self._event_queue.get()
        except Exception:
            return None
            
    async def close(self) -> None:
        if self._session:
            await self._session.close()
            self._session = None
```

## Example Integration

Here's a complete example of integrating all components:

```python
import asyncio
from typing import Optional, Any
from datetime import datetime

class MCPServer:
    """Complete MCP server implementation"""
    
    def __init__(self, name: str) -> None:
        self.name = name
        self.resource_manager = ResourceManager()
        self.prompt_manager = PromptManager()
        self.tool_manager = ToolManager()
        self.sampler = ToolSampler(InMemorySampleCollector())
        
    async def handle_request(
        self,
        request: Request,
        session: Session
    ) -> Result:
        """Handle incoming request"""
        start_time = datetime.utcnow()
        success = True
        error: Optional[str] = None
        
        try:
            result = await self._process_request(request, session)
            return Result(result=result, id=request.id)
        except Exception as e:
            success = False
            error = str(e)
            if isinstance(e, MCPError):
                raise
            raise MCPError(
                f"Request failed: {e}",
                code=ErrorCode.InternalError
            )
        finally:
            # Record usage sample
            duration = (
                datetime.utcnow() - start_time
            ).total_seconds()
            await self.sampler.record_usage(
                tool_name=request.method,
                duration=duration,
                success=success,
                error=error,
                metadata={"request_id": request.id}
            )
            
    async def _process_request(
        self,
        request: Request,
        session: Session
    ) -> Any:
        """Process specific request types"""
        match request.method:
            case "resources/list":
                return await self.resource_manager.list_resources()
                
            case "prompts/list":
                return await self.prompt_manager.list_prompts()
                
            case "prompts/get":
                name = request.params.get("name")
                args = request.params.get("arguments")
                if not name:
                    raise MCPError(
                        "Missing prompt name",
                        code=ErrorCode.InvalidParams
                    )
                return await self.prompt_manager.get_prompt(
                    name,
                    args
                )
                
            case "tools/list":
                return await self.tool_manager.list_tools()
                
            case "tools/call":
                name = request.params.get("name")
                args = request.params.get("arguments")
                if not name:
                    raise MCPError(
                        "Missing tool name",
                        code=ErrorCode.InvalidParams
                    )
                    
                async def progress_callback(
                    progress: Progress
                ) -> None:
                    await session.send_notification(
                        Notification(
                            method="progress",
                            params={
                                "token": request.id,
                                "progress": progress.dict()
                            }
                        )
                    )
                    
                return await self.tool_manager.call_tool(
                    name,
                    args or {},
                    progress_callback
                )
                
            case _:
                raise MCPError(
                    f"Unknown method: {request.method}",
                    code=ErrorCode.MethodNotFound
                )

# Usage example
async def main():
    # Create server
    server = MCPServer("example-server")
    
    # Register resources
    server.resource_manager.register_provider(
        "file",
        FileResourceProvider()
    )
    
    # Register prompts
    server.prompt_manager.register_prompt(code_analysis_prompt)
    server.prompt_manager.register_prompt(git_commit_prompt)
    
    # Register tools
    server.tool_manager.register_tool(
        name="read_file",
        description="Read contents of a file",
        input_schema={
            "type": "object",
            "properties": {
                "path": {
                    "type": "string",
                    "description": "Path to file"
                }
            },
            "required": ["path"]
        },
        handler=file_reader
    )
    
    # Create transport
    transport = StdioTransport()
    
    # Run server
    try:
        while True:
            message = await transport.receive_message()
            if not message:
                break
                
            try:
                request = Request.parse_obj(message)
                result = await server.handle_request(
                    request,
                    StdioSession(transport)
                )
                await transport.send_message(result.dict())
            except MCPError as e:
                await transport.send_message(
                    e.to_response(message.get("id")).dict()
                )
    finally:
        await transport.close()

if __name__ == "__main__":
    asyncio.run(main())
```
## Best Practices

1. **Tool Design**
   - Keep operations focused and atomic
   - Use clear, descriptive names
   - Provide detailed documentation
   - Include usage examples

2. **Input Validation**
   - Validate all parameters
   - Sanitize user inputs
   - Check parameter ranges
   - Prevent injection attacks

3. **Error Handling**
   - Use proper error types
   - Include helpful messages
   - Log errors with context
   - Clean up resources

4. **Security**
   - Implement rate limiting
   - Use access controls
   - Validate all inputs
   - Monitor tool usage

5. **Performance**
   - Use async operations
   - Implement timeouts
   - Cache when appropriate
   - Monitor resource usage

6. **Prompt Design**
   - Use clear, descriptive names
   - Document arguments thoroughly
   - Version prompt templates
   - Handle missing arguments gracefully
## Conclusion

Building production-ready MCP tools requires careful attention to:

1. Tool Design
   - Clear interfaces
   - Proper validation
   - Comprehensive documentation
   - Security considerations

2. Implementation
   - Error handling
   - Progress reporting
   - Resource management
   - Performance optimization

3. Operations
   - Monitoring
   - Rate limiting
   - Access control
   - Usage tracking

Following these patterns ensures reliable, secure, and maintainable tools that integrate well with AI models in production environments.