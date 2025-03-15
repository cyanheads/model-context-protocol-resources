# Model Context Protocol Resources & Guides

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/MCP%20SDK-TypeScript%201.7.0-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/MCP%20SDK-Python%201.3.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/MCP%20SDK-Kotlin%200.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-March%202025-brightgreen.svg)]()

A collection of guides, clients, and servers for the Model Context Protocol (MCP) that I've developed while exploring and implementing this powerful standard. It's a work in progress, and I'll be adding more resources and servers as I continue to experiment and learn. Feel free to reach out if you have any questions or suggestions! Thanks for stopping by! 🚀

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
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-2.1.3-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()
[![GitHub](https://img.shields.io/github/stars/cyanheads/atlas-mcp-server?style=social)](https://github.com/cyanheads/atlas-mcp-server)

ATLAS (Adaptive Task & Logic Automation System) is a Model Context Protocol server designed for LLMs to manage complex projects. Built with TypeScript and featuring Neo4j graph database integration, efficient project management, and collaborative features, ATLAS provides LLM Agents project management capabilities through a clean, flexible tool interface.

- 📋 **Project Management**

  - Lifecycle management with metadata and status tracking
  - Dependency handling with automatic validation
  - Rich content support with bulk operations
  - Comprehensive tracking with project metadata and statuses
  - Bulk operations support for efficient management

- 🤝 **Collaboration**

  - Member & role management with permission controls
  - Resource sharing and activity tracking
  - Real-time collaborative whiteboard with version history
  - Role-based access control (owner, admin, member, viewer)

- 🔍 **Graph Database**

  - Neo4j-powered relationship management
  - ACID-compliant transactions
  - Optimized caching and batch operations
  - Advanced property-based search with fuzzy matching

- 🖌️ **Whiteboard System**

  - Real-time collaboration capabilities
  - Version control with history tracking
  - Schema validation for data integrity
  - Seamless project integration

- 🧠 **ATLAS Skills**
  - Modular knowledge and best practices system
  - Hierarchical organization with dependency resolution
  - Base, language/framework, and tool-specific categories
  - Customizable parameters for project needs

> **Important Version Note**: Version 1.5.4 is the last version that uses SQLite as the database. Version 2.0 and onwards has been completely rewritten to use Neo4j, which requires either self-hosting using Docker (docker-compose included in repository) or using Neo4j AuraDB cloud service.

### Toolkit MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-toolkit--mcp--server-blue.svg)](https://github.com/cyanheads/toolkit-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.4.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.1-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()
[![GitHub](https://img.shields.io/github/stars/cyanheads/toolkit-mcp-server?style=social)](https://github.com/cyanheads/toolkit-mcp-server)

A Model Context Protocol server providing LLM Agents with system utilities and tools, including IP geolocation, network diagnostics, system monitoring, cryptographic operations, and QR code generation.

- 🌍 **Network & System Tools**

  - IP geolocation with smart caching
  - Network diagnostics and connectivity testing
  - Ping and traceroute utilities
  - Public IP detection
  - System resource monitoring
  - Performance metrics tracking
  - Load average tracking
  - Network interface details

- 🔐 **Security & Generation**

  - Cryptographic operations (MD5, SHA-256, etc.)
  - Constant-time hash comparison
  - UUID generation
  - QR code generation in multiple formats
  - SVG and Base64 encoded outputs

- ⏱️ **Time & Date Management**

  - Current time retrieval with formatting options
  - Timezone conversion utilities
  - IANA timezone listing and filtering
  - Date formatting with localization support

> Features optimized caching for geolocation to reduce API usage (45 requests/minute limit).
> All cryptographic operations use Node.js native crypto module for maximum security and performance.
> Can be easily installed via npm or from source with minimal configuration required.

### Mentor MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-mentor--mcp--server-blue.svg)](https://github.com/cyanheads/mentor-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.4.1-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()
[![GitHub](https://img.shields.io/github/stars/cyanheads/mentor-mcp-server?style=social)](https://github.com/cyanheads/mentor-mcp-server)

A Model Context Protocol server providing LLM Agents a second opinion via AI-powered Deepseek-Reasoning (R1) mentorship capabilities, including code review, design critique, writing feedback, and idea brainstorming through the Deepseek API. Set your LLM Agent up for success with expert second opinions and actionable insights.

- 🧠 **Code Analysis**

  - Comprehensive code reviews
  - Bug detection and prevention
  - Style and best practices evaluation
  - Performance optimization recommendations
  - Security vulnerability assessment

- 🎨 **Design & Architecture**

  - UI/UX design critiques
  - Architectural diagram analysis
  - Design pattern recommendations
  - Accessibility evaluation
  - Consistency checks

- 📝 **Content Enhancement**

  - Writing feedback and improvement
  - Grammar and style analysis
  - Documentation review
  - Content clarity assessment
  - Structural recommendations

- 🔮 **Strategic Planning**
  - Feature enhancement brainstorming
  - Second opinions on approaches
  - Innovation suggestions
  - Feasibility analysis
  - User value assessment

> Powered by Deepseek-Reasoning (R1) through the Deepseek API for high-quality technical feedback and mentorship.
> Configurable with timeout settings, retry mechanisms, and token limits for optimal performance.
> Implements request rate limiting and response caching to ensure efficient API usage.

### Obsidian MCP Server

[![GitHub](https://img.shields.io/badge/GitHub-obsidian--mcp--server-blue.svg)](https://github.com/cyanheads/obsidian-mcp-server)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.6.1-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-1.4.1-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-blue.svg)]()
[![GitHub](https://img.shields.io/github/stars/cyanheads/obsidian-mcp-server?style=social)](https://github.com/cyanheads/obsidian-mcp-server)

A Model Context Protocol server designed for LLMs to interact with Obsidian vaults. Built with TypeScript and featuring secure API communication, efficient file operations, and comprehensive search capabilities, it enables AI assistants to seamlessly manage knowledge bases through a clean, flexible tool interface.

- 📝 **Knowledge Base Management**

  - File creation, reading, and updating
  - YAML frontmatter handling and validation
  - Content appending and patching
  - Directory structure navigation and listing
  - Atomic file operations with error handling

- 🔍 **Advanced Search**

  - Full-text search with context highlighting
  - JsonLogic query support for complex filters
  - Tag and property-based filtering
  - Support for glob patterns and frontmatter fields

- 🔧 **Property Management**

  - YAML frontmatter parsing and intelligent merging
  - Automatic timestamps management
  - Custom field support with type validation
  - Standardized property schemas

- 🔒 **Security & Performance**
  - API key authentication with rate limiting
  - Resource monitoring and health checks
  - Configurable request limits and timeouts
  - Graceful shutdown handling

> Requires the Local REST API plugin in Obsidian to function.
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
[![Model Context Protocol](https://img.shields.io/badge/MCP-1.7.0-green.svg)](https://github.com/anthropics/modelcontextprotocol)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)]()
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Stable-green.svg)]()
[![GitHub](https://img.shields.io/github/stars/cyanheads/github-mcp-server?style=social)](https://github.com/cyanheads/github-mcp-server)

A Model Context Protocol (MCP) server that provides tools for interacting with the GitHub API. This server allows LLM agents manage GitHub repositories, issues, pull requests, branches, files, and releases through a standardized interface.

- 📁 **Repository Management**

  - Create, list, and get repository information
  - Configure repository settings and properties
  - Manage repository visibility (public/private)
  - Add detailed descriptions and configuration
  - Support for organization repositories

- 🌿 **Branch Management**

  - Create, delete, and list branches
  - Filter by protected branches
  - Support for advanced branch operations
  - Branch protection and permissions
  - Resolve merge conflicts and issues

- 🔖 **Issue Management**

  - Create and list issues with filtering
  - Apply labels to categorize issues
  - Filter by state (open, closed, all)
  - Detailed title and body content
  - Track issue status and progress

- 🔄 **Pull Request Management**

  - Full PR lifecycle with merge options
  - Support for different merge strategies (merge, squash, rebase)
  - Update existing pull requests
  - Filter and list pull requests
  - Add commit messages and references

- 📝 **File Management**

  - Create and update repository content
  - Base64 encoding support for binary files
  - Associate file changes with commits
  - Specify commit messages for changes
  - Handle file paths and content

- 🚀 **Release Management**

  - Create tagged releases with custom options
  - Draft and prerelease support
  - Asset management and publishing
  - Release notes and documentation

- ⚙️ **System Architecture**
  - Atomic feature-oriented architecture
  - Comprehensive validation with Zod schemas
  - GitHub API rate limit protection
  - Error categorization and handling
  - Consistent response formatting

> Requires a GitHub personal access token with appropriate permissions.
> Implements robust error handling with standardized error objects and detailed logging.
> Features automatic rate limit handling to prevent GitHub API throttling.

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
