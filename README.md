# Model Context Protocol Resources & Guides

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/MCP%20SDK-TypeScript%201.7.0-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/MCP%20SDK-Python%201.3.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/MCP%20SDK-Kotlin%200.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-March%202025-brightgreen.svg)]()

A collection of guides, clients, and servers for the Model Context Protocol (MCP) that I've developed while exploring and implementing this powerful standard. It's a work in progress, and I'll be adding more resources and servers as I continue to experiment and learn. Feel free to contribute or reach out if you have any questions or suggestions! Thanks for stopping by! 🚀

## 📋 Table of Contents

- [Introduction to MCP](#introduction-to-mcp)
- [Guides](#guides)
- [MCP Servers](#mcp-servers)
  - [Atlas](#atlas-mcp-server)
  - [Toolkit](#toolkit-mcp-server)
  - [Mentor](#mentor-mcp-server)
  - [Obsidian](#obsidian-mcp-server)
  - [Git](#git-mcp-server)
  - [GitHub](#github-mcp-server)
- [Getting Started](#getting-started)
- [License](#license)

## 🔍 Introduction to MCP

The Model Context Protocol (MCP) is a standardized communication protocol enabling Large Language Models (LLMs) to interact with external systems and services. Benefits include:

- **Consistent Interface**: Standardized methods for LLMs to access tools and resources
- **Enhanced Capabilities**: Gives LLMs the ability to interact with databases, APIs, and local systems
- **Security & Control**: Provides structured access patterns with built-in validation
- **Extensibility**: Easy to implement new capabilities as system requirements evolve

## 📚 Guides

### MCP Client Development

Looking to build your own MCP client? Check out my comprehensive [MCP Client Development Guide](guides/mcp-client-development-guide.md) that covers:

- Core architecture and components
- Connection lifecycle management
- Tool and resource handling
- Error handling and security best practices
- Step-by-step implementation examples in Python and TypeScript
- Advanced topics like sampling and multi-server connections

### MCP Server Development

Looking to build your own MCP server? Check out my comprehensive [MCP Server Development Guide](guides/mcp-server-development-guide.md) that covers:

- Core server architecture
- Building your first MCP server
- Exposing capabilities (tools, resources, and prompts)
- Advanced server features (sampling, roots, streaming, etc.)
- Security and best practices
- Troubleshooting and resources
- Example implementations

## 🖥️ MCP Servers

### Atlas MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-atlas--mcp--server-blue.svg)](https://github.com/cyanheads/atlas-mcp-server)
[![Version](https://img.shields.io/badge/Version-2.0.7-blue.svg)]()
[![MCP](https://img.shields.io/badge/MCP-1.6.1-green.svg)]()
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

**ATLAS** (Adaptive Task & Logic Automation System) is a comprehensive project management system for LLMs with:

- 📋 **Project Management**

  - Lifecycle management with metadata and status tracking
  - Dependency handling with automatic validation
  - Rich content support with bulk operations
  - Advanced property-based search with fuzzy matching

- 🤝 **Collaboration**

  - Member & role management with permission controls
  - Resource sharing and activity tracking
  - Real-time collaborative whiteboard with version history
  - Relationship mapping between projects and resources

- 🔍 **Graph Database**

  - Neo4j-powered relationship management
  - ACID-compliant transactions
  - Optimized caching and batch operations
  - Scalable architecture with monitoring

> **Note**: Version 2.0+ uses Neo4j as the database (self-hosted Docker or AuraDB cloud), replacing SQLite used in earlier versions.

### Toolkit MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-toolkit--mcp--server-blue.svg)](https://github.com/cyanheads/toolkit-mcp-server)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

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

### Mentor MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-mentor--mcp--server-blue.svg)](https://github.com/cyanheads/mentor-mcp-server)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A second opinion and mentorship system powered by Deepseek-Reasoning (R1) featuring:

- 🔍 **Code Analysis**: Comprehensive reviews, bug detection, style evaluation
- 🎨 **Design & Architecture**: UI/UX critiques, architectural recommendations
- 📝 **Content Enhancement**: Writing feedback, documentation review
- 🚀 **Strategic Planning**: Feature brainstorming, approach validation

### Obsidian MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-obsidian--mcp--server-blue.svg)](https://github.com/cyanheads/obsidian-mcp-server)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A powerful interface for LLMs to interact with Obsidian vaults featuring:

- 📝 **Knowledge Base Management**: File operations, YAML frontmatter handling
- 🔍 **Advanced Search**: Full-text search, JsonLogic queries, metadata filtering
- 🔄 **Integration**: Seamless connection to other MCP servers for enhanced workflows

### Git MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-git--mcp--server-blue.svg)](https://github.com/cyanheads/git-mcp-server)
[![Status](https://img.shields.io/badge/Status-Beta-orange.svg)]()

A secure Git operations interface for LLMs that provides:

- 🔄 **Complete Git Workflow**: Repository, branch, and tag management
- 🚀 **Smart Operations**: Bulk actions, intelligent defaults, safety validations

### GitHub MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-github--mcp--server-blue.svg)](https://github.com/cyanheads/github-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://github.com/anthropics/modelcontextprotocol)
[![Status](https://img.shields.io/badge/Status-Stable-green.svg)]()

A comprehensive GitHub API interface for LLMs that provides tools for:

- 📁 **Repository Management**: Create, list, and get repository information
- 🌿 **Branch Management**: Create, delete, and list branches with protected branch support
- 🔖 **Issue Management**: Create and list issues with filtering by status and labels
- 🔄 **Pull Request Management**: Full PR lifecycle with merge options (merge, squash, rebase)
- 📝 **File Management**: Create and update repository content with commit messages
- 🚀 **Release Management**: Create tagged releases with draft and prerelease support

> Features atomic feature-oriented architecture with comprehensive validation, error handling,
> and GitHub API rate limit protection. Uses a consistent response format for all operations
> that includes proper categorization and metadata.

## 🚀 Getting Started

1. **Explore the guides** to understand MCP concepts and implementation approaches
2. **Select a server** that matches your use case and follow its installation instructions
3. **Connect with an MCP-compatible client** or build your own using the client development guide
4. **Experiment and contribute** by submitting issues or pull requests

## 📄 License

[![Apache 2.0 License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

---

<div align="center">
Created by <a href="https://github.com/cyanheads">cyanheads</a> with the Model Context Protocol
</div>
