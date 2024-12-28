# Model Context Protocol Tools

[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.0.4-green.svg)](https://modelcontextprotocol.io/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

A collection of powerful Model Context Protocol (MCP) servers created by [cyanheads](https://github.com/cyanheads) that extend Large Language Model capabilities through standardized interfaces.

## 🌟 Featured Servers

### [Atlas MCP Server](https://github.com/cyanheads/atlas-mcp-server)

[![Status](https://img.shields.io/badge/Status-Stable-green.svg)]()

ATLAS (Adaptive Task & Logic Automation System) empowers LLMs with hierarchical task management capabilities.

#### ✨ Features

- 📋 **Task Organization**
  - Hierarchical structure with parent-child relationships
  - Strong type validation (TASK, MILESTONE)
  - Status management with strict transition rules
  - Rich metadata support with schema validation
  - Dependency validation and cycle detection
  - Automatic subtask management
  - Bulk operations with transaction support

- 🛡️ **Validation & Safety**
  - Path validation and sanitization
  - Directory traversal prevention
  - Special character validation
  - Parent-child path validation
  - Path depth limits
  - Project name validation
  - Consistent path formatting

- ⚡ **Performance**
  - SQLite backend with Write-Ahead Logging (WAL)
  - Advanced caching system with TTL and LRU eviction
  - Memory pressure monitoring
  - Query optimization
  - Connection pooling
  - Transaction batching
  - Index-based fast retrieval

- 🔍 **Monitoring & Maintenance**
  - Comprehensive event system
  - Memory usage monitoring
  - Database optimization tools
  - Relationship repair utilities
  - Cache statistics tracking
  - Health monitoring
  - Performance metrics collection
  - Structured logging support
  - Automated backup scheduling

- 🌐 **Platform Support**
  - Cross-platform path resolution
  - Platform-specific file permissions
  - Intelligent symlink support
  - Process signal handling
  - Directory structure support
  - Atomic file operations

- ⚠️ **Error Handling**
  - Error severity classification
  - Comprehensive error context
  - Transaction rollback
  - Automatic retry with backoff
  - Cache and memory recovery
  - Connection pool management
  - Batch operation recovery

### [Git MCP Server](https://github.com/cyanheads/git-mcp-server)

[![Status](https://img.shields.io/badge/Status-Beta-orange.svg)]()

A secure interface enabling LLMs to perform Git operations through standardized protocols.

#### ✨ Features

- 🔄 **Core Operations**
  - Repository management
  - Commit handling
  - Branch operations
  - Remote synchronization
  - Tag management

- 🛡️ **Safety Features**
  - Path validation
  - State verification
  - Error handling
  - Atomic operations

- 🚀 **Efficiency**
  - Bulk operations
  - Smart defaults
  - Sequential execution
  - Operation batching

## 🔗 Links

- [Atlas MCP Server Repository](https://github.com/cyanheads/atlas-mcp-server)
- [Git MCP Server Repository](https://github.com/cyanheads/git-mcp-server)

## 📄 License

Apache License 2.0

---

<div align="center">
Created by <a href="https://github.com/cyanheads">cyanheads</a> with the Model Context Protocol
</div>
