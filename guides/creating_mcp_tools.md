# Creating MCP Tools: A Comprehensive Guide

This guide provides a detailed walkthrough of creating tools for the Model Context Protocol (MCP), incorporating best practices and real-world patterns from production implementations.

## Table of Contents

1. [Understanding MCP Tools](#understanding-mcp-tools)
2. [Project Setup](#project-setup)
3. [Core Components](#core-components)
4. [MCP Server Implementation](#mcp-server-implementation)
5. [Advanced Features](#advanced-features)
6. [Testing and Quality Assurance](#testing-and-quality-assurance)
7. [Deployment and Operations](#deployment-and-operations)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

## Understanding MCP Tools

### What is an MCP Tool?

An MCP tool is a standardized interface that allows AI models (like Claude) to interact with local or remote services in a secure and controlled way. Tools can:

- Execute specific operations
- Process and return data
- Handle user inputs
- Provide progress updates
- Manage resources
- Report errors consistently

### Key Concepts

1. **Resources**: Static or dynamic data sources accessible through URIs
2. **Tools**: Executable operations with defined inputs and outputs
3. **Prompts**: AI-friendly text generation templates
4. **Progress Tracking**: Real-time operation status updates
5. **Error Handling**: Standardized error reporting and recovery

## Project Setup

### 1. Directory Structure

```
my_mcp_tool/
├── src/
│   └── my_mcp_tool/
│       ├── __init__.py      # Package initialization and entry point
│       ├── __main__.py      # CLI entry point
│       ├── models.py        # Data models and types
│       ├── service.py       # Core business logic
│       └── server.py        # MCP server implementation
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_service.py
│   └── test_server.py
├── scripts/
│   ├── run_service.sh       # Service runner script
│   └── service.sh           # Service management script
├── pyproject.toml          # Project metadata and dependencies
├── README.md              # Documentation
└── .env                   # Environment configuration
```

### 2. Project Configuration

```toml
# pyproject.toml
[project]
name = "my-mcp-tool"
version = "0.1.0"
description = "MCP Tool Description"
requires-python = ">=3.10"
dependencies = [
    "mcp>=1.0.0",
    "httpx>=0.27.0",
    "python-dotenv>=1.0.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project.scripts]
my-mcp-tool = "my_mcp_tool:run"
```

### 3. Environment Setup

```bash
# Create virtual environment
uv venv .venv

# Install dependencies
uv pip install mcp httpx python-dotenv

# Install development dependencies
uv pip install --dev pytest pytest-asyncio pytest-cov
```

## Core Components

### 1. Data Models (models.py)

```python
from dataclasses import dataclass
from datetime import datetime
from enum import Enum
from typing import Dict, Any, Optional

class DataType(Enum):
    """Define standardized data types"""
    TEXT = "text"
    NUMBER = "number"
    BINARY = "binary"

@dataclass
class ProcessingResult:
    """Core result data structure"""
    type: DataType
    data: Any
    timestamp: datetime
    metadata: Optional[Dict[str, Any]] = None

    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary format"""
        return {
            "type": self.type.value,
            "data": self.data,
            "timestamp": self.timestamp.isoformat(),
            **({"metadata": self.metadata} if self.metadata else {})
        }
```

### 2. Service Layer (service.py)

```python
import logging
from typing import Dict, Any
import httpx

logger = logging.getLogger(__name__)

class ToolService:
    """Core service implementation"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.client = httpx.AsyncClient(
            timeout=10.0,
            limits=httpx.Limits(
                max_keepalive_connections=5,
                max_connections=10
            )
        )
        self._cache = {}
        
    async def process_request(self, params: Dict[str, Any]) -> ProcessingResult:
        """Process a tool request"""
        try:
            # Implementation
            pass
        except Exception as e:
            logger.error(f"Processing error: {str(e)}", exc_info=True)
            raise
            
    async def cleanup(self):
        """Clean up resources"""
        await self.client.aclose()
        self._cache.clear()
```

## MCP Server Implementation

### 1. Server Setup (server.py)

```python
import os
import logging
from typing import Dict, Any, List
from dotenv import load_dotenv
from mcp.server import Server, McpError
from mcp.types import Tool, TextContent, Resource

# Load configuration
load_dotenv()
logger = logging.getLogger(__name__)

# Initialize server
server = Server("my_tool")
service = ToolService(config={
    "api_key": os.getenv("API_KEY"),
    "base_url": os.getenv("BASE_URL")
})

@server.list_resources()
async def handle_list_resources() -> List[Resource]:
    """List available resources"""
    return [
        Resource(
            uri="myapp://default/resource",
            name="Default Resource",
            description="Description",
            mimeType="application/json"
        )
    ]

@server.list_tools()
async def handle_list_tools() -> List[Tool]:
    """List available tools"""
    return [
        Tool(
            name="my-tool",
            description="Tool description",
            inputSchema={
                "type": "object",
                "properties": {
                    "param1": {
                        "type": "string",
                        "description": "Parameter description"
                    }
                },
                "required": ["param1"]
            }
        )
    ]

@server.call_tool()
async def handle_call_tool(
    name: str,
    arguments: Dict[str, Any]
) -> List[TextContent]:
    """Handle tool execution"""
    try:
        # Validate tool name
        if name != "my-tool":
            raise McpError(f"Unknown tool: {name}")
            
        # Validate arguments
        if not arguments or "param1" not in arguments:
            raise McpError("Missing required parameter: param1")
            
        # Process request
        result = await service.process_request(arguments)
        
        return [TextContent(
            type="text",
            text=json.dumps(result.to_dict())
        )]
        
    except Exception as e:
        logger.error(f"Tool error: {str(e)}", exc_info=True)
        raise McpError(f"Tool execution failed: {str(e)}")
```

### 2. Entry Points

```python
# __main__.py
"""Main entry point for the MCP tool."""

from . import run

if __name__ == "__main__":
    run()
```

```python
# __init__.py
import argparse
import asyncio
import logging
import signal
import sys
from typing import Optional

logger = logging.getLogger(__name__)

def parse_args():
    """Parse command line arguments."""
    parser = argparse.ArgumentParser(description="MCP Tool Service")
    parser.add_argument(
        "--log-level",
        choices=["DEBUG", "INFO", "WARNING", "ERROR"],
        default="INFO",
        help="Set the logging level"
    )
    parser.add_argument(
        "--config",
        type=str,
        help="Path to config file"
    )
    return parser.parse_args()

def setup_logging(level: str):
    """Configure logging with the specified level."""
    logging.basicConfig(
        level=level,
        format="%(asctime)s.%(msecs)03d - %(name)s - %(levelname)s - %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )

def handle_signal(signum, frame):
    """Handle system signals gracefully."""
    signal_name = signal.Signals(signum).name
    logger.info(f"Received signal {signal_name} ({signum})")
    logger.info("Initiating graceful shutdown...")
    sys.exit(0)

def run():
    """
    Main entry point for running the service.
    
    This function:
    1. Parses command line arguments
    2. Sets up logging
    3. Configures signal handlers
    4. Starts the async event loop
    5. Handles cleanup on shutdown
    """
    try:
        # Parse arguments
        args = parse_args()
        
        # Setup logging
        setup_logging(args.log_level)
        
        # Set up signal handlers
        signal.signal(signal.SIGINT, handle_signal)
        signal.signal(signal.SIGTERM, handle_signal)
        
        logger.info("=== Service Starting ===")
        logger.info(f"Version: {__version__}")
        logger.info("Press Ctrl+C to stop")
        logger.info("=====================")
        
        # Run the async main function
        asyncio.run(main())
    except KeyboardInterrupt:
        logger.info("Service stopped by user")
    except Exception as e:
        logger.error("Service error:", exc_info=True)
        sys.exit(1)
    finally:
        logger.info("=== Service Stopped ===")
        logger.info("All resources cleaned up")
        logger.info("=====================")

async def main():
    """Async main function"""
    import mcp.server.stdio
    
    try:
        async with mcp.server.stdio.stdio_server() as (read, write):
            await server.run(read, write)
    finally:
        await service.cleanup()
```

## Advanced Features

### 1. Progress Tracking

```python
async def send_progress(
    progress: int,
    total: int,
    description: str
) -> None:
    """Send progress updates safely"""
    try:
        if (
            hasattr(server, "request_context") and
            server.request_context is not None and
            server.request_context.meta is not None and
            server.request_context.meta.progressToken is not None
        ):
            await server.request_context.session.send_progress_notification(
                progress_token=server.request_context.meta.progressToken,
                progress=progress,
                total=total,
                description=description
            )
    except Exception as e:
        logger.warning(f"Progress notification failed: {str(e)}")
```

### 2. Caching

```python
class CacheManager:
    """Manage cached data with timeouts"""
    
    def __init__(self, max_size: int = 100):
        self._cache = {}
        self._timestamps = {}
        self._max_size = max_size
        
    def get(self, key: str) -> Optional[Any]:
        """Get cached value if available"""
        return self._cache.get(key)
        
    def set(self, key: str, value: Any):
        """Cache a value with timestamp"""
        if len(self._cache) >= self._max_size:
            oldest = min(self._timestamps.items(), key=lambda x: x[1])[0]
            del self._cache[oldest]
            del self._timestamps[oldest]
            
        self._cache[key] = value
        self._timestamps[key] = datetime.now()
```

### 3. Resource Management

```python
class ResourceManager:
    """Manage shared resources"""
    
    def __init__(self):
        self._resources = {}
        self._locks = {}
        
    async def acquire(self, resource_id: str):
        """Acquire resource lock"""
        if resource_id not in self._locks:
            self._locks[resource_id] = asyncio.Lock()
        await self._locks[resource_id].acquire()
        
    async def release(self, resource_id: str):
        """Release resource lock"""
        if resource_id in self._locks:
            self._locks[resource_id].release()
```

## Testing and Quality Assurance

### 1. Unit Tests

```python
import pytest
from unittest.mock import Mock, AsyncMock
from mcp.server import McpError

@pytest.mark.asyncio
async def test_tool_execution():
    # Arrange
    mock_service = AsyncMock()
    mock_service.process_request.return_value = ProcessingResult(...)
    
    # Act
    result = await handle_call_tool("my-tool", {"param1": "value"})
    
    # Assert
    assert len(result) == 1
    assert result[0].type == "text"
    data = json.loads(result[0].text)
    assert "type" in data
```

### 2. Integration Tests

```python
@pytest.mark.asyncio
async def test_full_workflow():
    # Test complete workflow
    tools = await handle_list_tools()
    assert len(tools) > 0
    
    result = await handle_call_tool(
        tools[0].name,
        {"param1": "test"}
    )
    assert result is not None
```

### 3. Error Handling Tests

```python
@pytest.mark.asyncio
async def test_error_handling():
    # Test invalid tool
    with pytest.raises(McpError) as exc:
        await handle_call_tool("invalid", {})
    assert "Unknown tool" in str(exc.value)
    
    # Test missing parameter
    with pytest.raises(McpError) as exc:
        await handle_call_tool("my-tool", {})
    assert "Missing required parameter" in str(exc.value)
```

## Deployment and Operations

### 1. Service Management Scripts

#### Run Service Script (run_service.sh)
```bash
#!/bin/bash

# Get the directory where the script is located
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

# Add src directory to Python path
export PYTHONPATH="$DIR/src:$PYTHONPATH"

# Run the service using uv
cd "$DIR" && uv run -m my_mcp_tool
```

#### Service Management Script (service.sh)
```bash
#!/bin/bash

# Service Management Script
# Helps install and manage the service on macOS

SERVICE_NAME="com.my-mcp-tool"
PLIST_PATH="$HOME/Library/LaunchAgents/$SERVICE_NAME.plist"
LOG_PATH="$HOME/Library/Logs/my-mcp-tool.log"
ERROR_LOG_PATH="$HOME/Library/Logs/my-mcp-tool.error.log"

# Colors for output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

print_status() {
    echo -e "${GREEN}==>${NC} $1"
}

print_error() {
    echo -e "${RED}Error:${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}Warning:${NC} $1"
}

check_dependencies() {
    if ! command -v uv &> /dev/null; then
        print_error "uv is not installed"
        exit 1
    fi
}

install_service() {
    print_status "Installing service..."
    
    # Create logs directory
    mkdir -p "$HOME/Library/Logs"
    
    # Copy plist file
    cp "$SERVICE_NAME.plist" "$PLIST_PATH"
    if [ $? -ne 0 ]; then
        print_error "Failed to copy plist file"
        exit 1
    fi
    
    # Load the service
    launchctl load "$PLIST_PATH"
    if [ $? -ne 0 ]; then
        print_error "Failed to load service"
        exit 1
    fi
    
    print_status "Service installed successfully"
    print_status "Logs will be available at:"
    echo "  - Standard output: $LOG_PATH"
    echo "  - Error log: $ERROR_LOG_PATH"
}

uninstall_service() {
    print_status "Uninstalling service..."
    launchctl unload "$PLIST_PATH" 2>/dev/null
    rm -f "$PLIST_PATH"
    print_status "Service uninstalled successfully"
}

start_service() {
    print_status "Starting service..."
    launchctl start "$SERVICE_NAME"
    print_status "Service started"
}

stop_service() {
    print_status "Stopping service..."
    launchctl stop "$SERVICE_NAME"
    print_status "Service stopped"
}

restart_service() {
    print_status "Restarting service..."
    stop_service
    sleep 2
    start_service
}

check_status() {
    if launchctl list | grep -q "$SERVICE_NAME"; then
        print_status "Service is installed and running"
        echo "Recent logs:"
        echo "----------------------------------------"
        tail -n 5 "$LOG_PATH" 2>/dev/null || echo "No logs available yet"
        echo "----------------------------------------"
    else
        print_warning "Service is not running"
    fi
}

show_logs() {
    if [ -f "$LOG_PATH" ]; then
        tail -f "$LOG_PATH"
    else
        print_error "Log file not found"
    fi
}

show_help() {
    echo "Service Management Script"
    echo
    echo "Usage: $0 [command]"
    echo
    echo "Commands:"
    echo "  install    Install and start the service"
    echo "  uninstall  Stop and remove the service"
    echo "  start      Start the service"
    echo "  stop       Stop the service"
    echo "  restart    Restart the service"
    echo "  status     Check service status"
    echo "  logs       Show and follow service logs"
    echo "  help       Show this help message"
}

# Check dependencies
check_dependencies

# Parse command
case "$1" in
    "install")
        install_service
        ;;
    "uninstall")
        uninstall_service
        ;;
    "start")
        start_service
        ;;
    "stop")
        stop_service
        ;;
    "restart")
        restart_service
        ;;
    "status")
        check_status
        ;;
    "logs")
        show_logs
        ;;
    "help"|"")
        show_help
        ;;
    *)
        print_error "Unknown command: $1"
        echo
        show_help
        exit 1
        ;;
esac
```

### 2. Launch Configuration (com.my-mcp-tool.plist)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.my-mcp-tool</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/uv</string>
        <string>run</string>
        <string>my-mcp-tool</string>
        <string>--log-level</string>
        <string>INFO</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>WorkingDirectory</key>
    <string>/path/to/your/tool</string>
    <key>StandardOutPath</key>
    <string>~/Library/Logs/my-mcp-tool.log</string>
    <key>StandardErrorPath</key>
    <string>~/Library/Logs/my-mcp-tool.error.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin</string>
        <key>PYTHONPATH</key>
        <string>/path/to/your/tool</string>
    </dict>
</dict>
</plist>
```

## Best Practices

1. **Error Handling**
   - Use specific error types
   - Provide clear error messages
   - Include context in errors
   - Clean up resources in error cases

2. **Logging**
   - Use structured logging
   - Include request IDs
   - Log appropriate detail levels
   - Handle sensitive data

3. **Resource Management**
   - Implement proper cleanup
   - Use context managers
   - Handle timeouts
   - Manage concurrent access

4. **Security**
   - Validate all inputs
   - Sanitize output data
   - Use secure defaults
   - Implement rate limiting

5. **Performance**
   - Implement caching
   - Use connection pooling
   - Batch operations when possible
   - Monitor resource usage

## Troubleshooting

### Common Issues

1. **Tool Not Found**
   - Check tool registration
   - Verify tool name
   - Check for typos

2. **Invalid Parameters**
   - Validate input schema
   - Check parameter types
   - Verify required fields

3. **Resource Issues**
   - Check resource availability
   - Verify permissions
   - Monitor resource usage

### Debugging

1. Enable Debug Logging
```python
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
```

2. Monitor Resources
```python
async def monitor_resources():
    while True:
        memory = psutil.Process().memory_info().rss / 1024 / 1024
        logger.debug(f"Memory: {memory:.2f} MB")
        await asyncio.sleep(60)
```

3. Test Connectivity
```python
async def test_connectivity():
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(TEST_URL)
            response.raise_for_status()
        return True
    except Exception as e:
        logger.error(f"Connectivity test failed: {str(e)}")
        return False
```

Remember to follow these guidelines and best practices when creating your MCP tools. This will ensure your tools are reliable, maintainable, and provide a great user experience.
