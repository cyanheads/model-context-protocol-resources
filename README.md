# Model Context Protocol Resources & Guides

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/TypeScript-1.12.1-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/Python-1.9.2-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/Kotlin-0.5.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![MCP SDK - Java](https://img.shields.io/badge/Java-0.10.0-blue.svg)](https://github.com/modelcontextprotocol/java-sdk)
[![MCP SDK - C#](https://img.shields.io/badge/C%23-0.2.0-blue.svg)](https://github.com/modelcontextprotocol/csharp-sdk)
[![Guide Last Updated](https://img.shields.io/badge/Guide%20Last%20Updated-May%202025-brightgreen.svg)]()

Welcome! This repository is a collection of guides, utilities, and server implementations for the **Model Context Protocol (MCP)** created while learning MCP. It reflects my ongoing exploration and development with this exciting new standard for creating powerful Agent capabilities. Questions and feedback are welcome! 🚀

> **Disclaimer:** The resources in this repository (guides, utilities, server implementations, and the MCP TypeScript Template) were developed independently by [cyanheads](https://github.com/cyanheads) while exploring the Model Context Protocol. This project is not officially affiliated with the Model Context Protocol organization. [Links to official MCP resources are provided below.](#official-resources) I appreciate the work of the official MCP team on the specification, SDKs, and documentation!

## 📋 Table of Contents

[Introduction](#-introduction-to-mcp) | [Guides](#-mcp-guides) | [Utilities](#-mcp-utilities) | [Servers](#-mcp-servers) | [Getting Started](#-getting-started) | [Official Resources](#-official-resources)

| Category      | Items                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Template**  | [mcp-ts-template](#-mcp-typescript-template-repo)                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Guides**    | [MCP Client Development](guides/mcp-client-development-guide.md)<br>[MCP Server Development](guides/mcp-server-development-guide.md)<br>[cyanheads-custom-llms.txt](guides/cyanheads-custom-mcp-llms-full.md)                                                                                                                                                                                                                                                                           |
| **Servers**   | [atlas-mcp-server](#-mcp-servers), [filesystem-mcp-server](#-mcp-servers), [git-mcp-server](#-mcp-servers)<br>[github-mcp-server](#-mcp-servers), [mcp-ts-template](#-mcp-servers), [mentor-mcp-server](#-mcp-servers)<br>[ntfy-mcp-server](#-mcp-servers), [obsidian-mcp-server](#-mcp-servers), [perplexity-mcp-server](#-mcp-servers)<br>[pubmed-mcp-server](#-mcp-servers), [toolkit-mcp-server](#-mcp-servers), [mcp-ts-template (src/mcp-server)](#-mcp-typescript-template-repo) |
| **Clients**   | [mcp-ts-template (src/mcp-client)](#-mcp-typescript-template-repo)                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Utilities** | [mcp-reporter](#-mcp-utilities)                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

## 🔍 Introduction to MCP

The **Model Context Protocol (MCP)** is an open standard designed to standardize how AI applications (clients/hosts) connect to and interact with external data sources and tools (servers).<br>Think of it like USB-C for AI: a universal way to plug capabilities into LLM applications.

**Key Benefits:**

- **Consistent Interface**: Standardized methods for LLMs to access tools and resources.
- **Enhanced Capabilities**: Empowers LLMs to interact with databases, APIs, local systems, and more.
- **Security & Control**: Provides structured access patterns with built-in validation and clear boundaries.
- **Extensibility**: Easily add new capabilities via servers without modifying core LLM applications.
- **Modularity**: Develop and maintain specialized functionalities in isolated, reusable server components.

For a more in-depth introduction to MCP, including its design philosophy and technical details, visit the official site: [modelcontextprotocol.io](https://modelcontextprotocol.io/).

## 🚀 MCP TypeScript Template Repo

| Project                                                             | Description                                                                                                                                                                         |
| :------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**mcp-ts-template**](https://github.com/cyanheads/mcp-ts-template) | A beginner-friendly, production-ready TypeScript template for building MCP servers and clients. Includes essential utilities, examples, and type safety for a solid starting point. |

## 📚 MCP Guides

| Guide                                                                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**MCP Client Development Guide**](guides/mcp-client-development-guide.md) | Learn how to build applications that consume MCP server capabilities. Covers core architecture, lifecycle, tools/resources, security, examples, and advanced topics. (Note: Needs update for latest spec changes)                                                                                                                                                                                                                             |
| [**MCP Server Development Guide**](guides/mcp-server-development-guide.md) | Comprehensive guide to building MCP servers. Covers core architecture, protocol fundamentals, server lifecycle, transports (Stdio, Streamable HTTP), building with the TypeScript SDK, defining Tools/Resources/Prompts, advanced features (sampling, roots, streaming, progress, subscriptions, completions), security best practices (updated for Auth Spec `2025-03-26`), troubleshooting, and example implementations. (Updated May 2025) |
| [**Cyanhead's MCP 'llms.txt'**](guides/cyanheads-custom-mcp-llms-full.md)  | A custom llms.txt for faster TypeScript MCP server development using the high-level SDK (`McpServer`). Tailored for LLM consumption, it covers key concepts, high-level examples, security, and dynamic capabilities, updated for Spec `2025-03-26` & TS SDK `v1.11.0`.                                                                                                                                                                       |

## 🔧 MCP Utilities

| Project                                                       | Description                                                                                                                                                                                              |
| :------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**mcp-reporter**](https://github.com/cyanheads/mcp-reporter) | A streamlined utility that generates comprehensive capability reports for MCP servers. Helps developers understand available functionality across their MCP ecosystem for documentation and integration. |

## 🔌 MCP Servers

This repository hosts several example MCP server implementations, showcasing different capabilities:

| Project                                                                         | Description                                                                                                                                                                                                                                         |
| :------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**atlas-mcp-server**](https://github.com/cyanheads/atlas-mcp-server)           | ATLAS (Adaptive Task & Logic Automation System), a Neo4j-powered task management system designed for LLM Agents. It uses a three-tier architecture (Projects, Tasks, Knowledge) to manage complex workflows and includes Deep Research scaffolding. |
| [**filesystem-mcp-server**](https://github.com/cyanheads/filesystem-mcp-server) | Offers platform-agnostic file system capabilities for AI agents via MCP. Enables reading, writing, updating, and managing files/directories, featuring advanced search/replace and directory traversal.                                             |
| [**git-mcp-server**](https://github.com/cyanheads/git-mcp-server)               | Provides an enterprise-ready MCP interface for Git operations. Allows LLM agents to initialize, clone, branch, commit, and manage repositories via STDIO & Streamable HTTP.                                                                         |
| [**github-mcp-server**](https://github.com/cyanheads/github-mcp-server)         | Integrates with the GitHub API via MCP, built in TypeScript. Provides a structured interface for LLM agents to manage repositories, issues, pull requests, code, files, and releases.                                                               |
| [**mentor-mcp-server**](https://github.com/cyanheads/mentor-mcp-server)         | Offers AI-powered mentorship via MCP using Deepseek-Reasoning R1 through the Deepseek API. Provides LLM agents with a 'second opinion' for code review, design critique, writing feedback, and brainstorming.                                       |
| [**ntfy-mcp-server**](https://github.com/cyanheads/ntfy-mcp-server)             | Integrates with the ntfy.sh push notification service via MCP. Enables LLM agents to send highly customizable notifications to external devices.                                                                                                    |
| [**obsidian-mcp-server**](https://github.com/cyanheads/obsidian-mcp-server)     | Enables LLMs to interact securely with Obsidian vaults via MCP. Offers token-aware tools for searching, navigating, and updating Obsidian notes, facilitating seamless knowledge base management with Properties management.                        |
| [**perplexity-mcp-server**](https://github.com/cyanheads/perplexity-mcp-server) | Unlocks Perplexity's search-augmented AI capabilities for LLM agents via MCP. Provides access to real-time web information with robust error handling, secure validation, and optional reasoning transparency (showThinking).                       |
| [**pubmed-mcp-server**](https://github.com/cyanheads/pubmed-mcp-server)         | Connects AI agents to NCBI's PubMed and E-utilities via MCP, enabling search, retrieval, and analysis of biomedical literature.                                                                                                                     |
| [**toolkit-mcp-server**](https://github.com/cyanheads/toolkit-mcp-server)       | Provides essential system utilities and tools for LLM agents via MCP, including IP geolocation, network diagnostics, system monitoring, cryptographic operations, and QR code generation.                                                           |

## 🚀 Getting Started

1.  **Explore the Guides**: Understand MCP concepts and development approaches using the [Client](guides/mcp-client-development-guide.md) and [Server](guides/mcp-server-development-guide.md) guides.
2.  **Select a Server**: Choose one relevant to your needs from the [MCP Servers](#-mcp-servers) section and follow its specific setup instructions in its repository.
3.  **Connect a Client**: Use an existing MCP-compatible client (like Claude Desktop, Cline, etc.) or build your own using the [Client Development Guide](guides/mcp-client-development-guide.md).
4.  **Experiment & Contribute**: Try out the tools and consider contributing via issues or pull requests on the respective project repositories.

## 🔗 Official Resources

Key links to official Model Context Protocol documentation, specifications, and community resources ([modelcontextprotocol.io](https://modelcontextprotocol.io/)):

| Page / Section            | Link Path                                                                                                           |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------ |
| Introduction              | [`/introduction`](https://modelcontextprotocol.io/introduction)                                                     |
| Server Quickstart         | [`/quickstart/server`](https://modelcontextprotocol.io/quickstart/server)                                           |
| Specification Home        | [`/specification`](https://modelcontextprotocol.io/specification)                                                   |
| ↳ Architecture            | [`/specification/architecture`](https://modelcontextprotocol.io/specification/architecture)                         |
| ↳ Base Protocol           | [`/specification/basic`](https://modelcontextprotocol.io/specification/basic)                                       |
| ↳ Server Features         | [`/specification/server`](https://modelcontextprotocol.io/specification/server)                                     |
| ↳ Client Features         | [`/specification/client`](https://modelcontextprotocol.io/specification/client)                                     |
| ↳ Auth Spec (2025-03-26)  | [`/.../authorization`](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization)                |
| Schema Definition (TS)    | [`spec/.../schema.ts`](https://github.com/modelcontextprotocol/specification/blob/main/schema/2025-03-26/schema.ts) |
| Contributing Guide        | [`CONTRIBUTING.md`](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/CONTRIBUTING.md)         |
| GitHub Organization       | [`github.com/...`](https://github.com/modelcontextprotocol)                                                         |
| Specification Discussions | [`spec/discussions`](https://github.com/modelcontextprotocol/specification/discussions)                             |
| Organization Discussions  | [`orgs/discussions`](https://github.com/orgs/modelcontextprotocol/discussions)                                      |
| JSON-RPC 2.0 Spec         | [`jsonrpc.org`](https://www.jsonrpc.org/specification)                                                              |

**SDKs & Tools:**

| Language                       | SDK Repository                                                           |
| :----------------------------- | :----------------------------------------------------------------------- |
| TypeScript                     | [typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) |
| Python                         | [python-sdk](https://github.com/modelcontextprotocol/python-sdk)         |
| Kotlin                         | [kotlin-sdk](https://github.com/modelcontextprotocol/kotlin-sdk)         |
| Java                           | [java-sdk](https://github.com/modelcontextprotocol/java-sdk)             |
| C#                             | [csharp-sdk](https://github.com/modelcontextprotocol/csharp-sdk)         |
| **Tool**                       | **Repository**                                                           |
| MCP Inspector (Debugging tool) | [inspector](https://github.com/modelcontextprotocol/inspector)           |

---

<div align="center">
Created by <a href="https://github.com/cyanheads">cyanheads</a> and the Model Context Protocol (MCP)
</div>
