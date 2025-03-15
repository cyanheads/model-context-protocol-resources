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
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.6.1-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-2.0.7-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A Model Context Protocol (MCP) server that provides a comprehensive project management system for LLMs:

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

> ATLAS (Adaptive Task & Logic Automation System) features comprehensive project lifecycle management.
> Version 2.0+ uses Neo4j as the database (self-hosted Docker or AuraDB cloud), replacing SQLite used in earlier versions.

### Toolkit MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-toolkit--mcp--server-blue.svg)](https://github.com/cyanheads/toolkit-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A Model Context Protocol (MCP) server that provides system utilities and networking tools for LLM agents:

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

> Features optimized caching for network operations, reducing latency and API usage.
> All cryptographic operations use Node.js native crypto module for security and performance.

### Mentor MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-mentor--mcp--server-blue.svg)](https://github.com/cyanheads/mentor-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A Model Context Protocol (MCP) server that provides a second opinion and mentorship system powered by advanced AI models:

- 🧠 **Code Analysis**

  - Comprehensive code reviews
  - Bug detection and prevention
  - Style and best practices evaluation
  - Performance optimization recommendations

- 🎨 **Design & Architecture**

  - UI/UX critiques and improvements
  - Architectural recommendations
  - Pattern identification and suggestions
  - Technical debt assessment

- 📝 **Content Enhancement**
  - Writing feedback and improvements
  - Documentation review and structuring
  - Clarity and completeness assessment
  - Terminology consistency checking

> Powered by Deepseek-Reasoning (R1) for high-quality technical feedback and mentorship.
> Implements request rate limiting and response caching to ensure efficient operation.

### Obsidian MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-obsidian--mcp--server-blue.svg)](https://github.com/cyanheads/obsidian-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A Model Context Protocol (MCP) server that provides interfaces for interacting with Obsidian knowledge vaults:

- 📝 **Knowledge Base Management**

  - File creation, reading, and updating
  - YAML frontmatter handling and validation
  - Content appending and patching
  - Directory structure navigation

- 🔍 **Advanced Search**

  - Full-text search with context highlighting
  - JsonLogic query support for complex filters
  - Tag and property-based filtering
  - Metadata extraction and analysis

- 🔄 **Integration**
  - Seamless connection to other MCP servers
  - Enhanced knowledge management workflows
  - Content transformation capabilities
  - Version history tracking

> Features secure vault access with proper permission boundaries and validation.
> Supports all Obsidian markdown features including links, embeds, and custom syntax.

### Git MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-git--mcp--server-blue.svg)](https://github.com/cyanheads/git-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()

A Model Context Protocol (MCP) server that provides tools for interacting with Git repositories through a standardized interface:

- 📦 **Repository Management**

  - Initialize, clone, and check repository status
  - Browse repository content and structure
  - Manage configuration and settings
  - Access repository history and logs

- 🌿 **Branch Operations**

  - Create, list, checkout, delete, and merge branches
  - Compare branches and visualize differences
  - Manage branch protection and permissions
  - Resolve merge conflicts and issues

- 📝 **Working Directory**

  - Stage files, commit changes, create diffs
  - Restore and reset working tree
  - View file history and blame information
  - Manage ignored files and patterns

- 🔄 **Remote Operations**

  - Add remotes, fetch, pull, push
  - Track upstream branches
  - Configure authentication and access
  - Manage remote references

- 🔧 **Advanced Git Commands**
  - Manage tags, stash changes, cherry-pick, rebase
  - Perform interactive rebasing
  - Handle submodules and worktrees
  - Execute advanced git operations

> Exposes Git operations as MCP resources and tools while maintaining proper security boundaries.
> Features standardized error handling, comprehensive validation, and consistent resource URIs
> for accessing repository information, branches, remotes, tags, files, diffs, and commits.

### GitHub MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-github--mcp--server-blue.svg)](https://github.com/cyanheads/github-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-green.svg)]()

A Model Context Protocol (MCP) server that provides a GitHub API interface for LLMs with standardized tools and resources:

- 📁 **Repository Management**

  - Create, list, and get repository information
  - Configure repository settings and properties
  - Manage visibility and access controls
  - Track repository statistics and activity

- 🌿 **Branch Management**

  - Create, delete, and list branches
  - Configure protected branch settings
  - Set default branches and merge rules
  - Compare branch contents and history

- 🔖 **Issue Management**

  - Create and list issues with filtering
  - Apply labels, milestones, and assignments
  - Track issue status and history
  - Manage comments and reactions

- 🔄 **Pull Request Management**

  - Full PR lifecycle with merge options
  - Review comment management
  - CI/CD status checking and integration
  - Draft PR support and workflow controls

- 📝 **File Management**

  - Create and update repository content
  - Manage file paths and structure
  - Support for commit messages and references
  - Batch operations and directory handling

- 🚀 **Release Management**
  - Create tagged releases with custom options
  - Draft and prerelease support
  - Asset management and publishing
  - Release notes and documentation

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
