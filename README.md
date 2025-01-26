# Model Context Protocol Resource: Guides and Servers

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/MCP%20SDK-TypeScript%201.4.1-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/MCP%20SDK-Python%201.2.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk) 
[![MCP SDK - Kotlin](https://img.shields.io/badge/MCP%20SDK-Kotlin%200.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-January%202025-brightgreen.svg)]()

A collection of Model Context Protocol (MCP) guides and servers created by [cyanheads](https://github.com/cyanheads) that extend Large Language Model (LLM) agent capabilities through standardized interfaces.

## Guides

### 📚 MCP Client Development

Looking to build your own MCP client? Check out my comprehensive [MCP Client Development Guide](guides/mcp-client-development-guide.md) that covers:

- Core architecture and components
- Connection lifecycle management
- Tool and resource handling
- Error handling and security best practices
- Step-by-step implementation examples in Python and TypeScript
- Advanced topics like sampling and multi-server connections

## MCP Servers

### [Atlas MCP Server](https://github.com/cyanheads/atlas-mcp-server) [![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

ATLAS (Adaptive Task & Logic Automation System) is a robust task management system designed for LLMs, featuring:

- 📋 **Hierarchical Task Management**
  - Parent-child task relationships with dependency tracking
  - Milestone and task type support with status transitions
  - Rich metadata and schema validation
  - Bulk operations with transactional safety

- ⚡ **Enterprise-Grade Performance**
  - SQLite backend with advanced caching
  - Optimized queries and connection pooling
  - Comprehensive monitoring and backup systems
  - Cross-platform support with atomic operations

- 🛡️ **Built-in Safety**
  - Strict path and input validation
  - Automatic error recovery
  - Transaction management
  - Comprehensive audit logging

### [Toolkit MCP Server](https://github.com/cyanheads/toolkit-mcp-server) [![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A comprehensive system utilities toolkit for LLM Agents featuring:

- 🌍 **Network & System Tools**
  - IP geolocation with smart caching
  - Network diagnostics and connectivity testing
  - System resource monitoring
  - Performance metrics tracking

- 🔐 **Security & Generation**
  - Cryptographic operations (MD5, SHA-256, etc.)
  - UUID generation
  - QR code generation in multiple formats
  - Secure hash comparisons

### [Obsidian MCP Server](https://github.com/cyanheads/obsidian-mcp-server) [![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A powerful interface for LLMs to interact with Obsidian vaults featuring:

- 📝 **Knowledge Base Management**
  - Atomic file operations and content manipulation
  - YAML frontmatter handling with intelligent merging
  - Comprehensive tag and property management
  - Automatic timestamp tracking

- 🔍 **Advanced Search Capabilities**
  - Full-text search with configurable context
  - Complex queries using JsonLogic
  - Tag and metadata filtering
  - Glob pattern support

### [Git MCP Server](https://github.com/cyanheads/git-mcp-server) [![Status](https://img.shields.io/badge/Status-Beta-orange.svg)]()

A secure Git operations interface for LLMs that provides:

- 🔄 **Complete Git Workflow Support**
  - Repository, branch, and tag management
  - Commit operations and remote synchronization
  - Safe state handling and atomic operations

- 🚀 **Smart Operations**
  - Bulk action support with batching
  - Intelligent defaults
  - Built-in safety validations

## 📄 License

[![Apache 2.0 License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

---

<div align="center">
Created by <a href="https://github.com/cyanheads">cyanheads</a> with the Model Context Protocol
</div>
