<div align="center">
  <h1>Model Context Protocol (MCP)</h1>
  <p><b>A technical overview of the protocol, current to the 2026-07-28 specification.</b></p>
</div>

<div align="center">

[![MCP Spec](https://img.shields.io/badge/MCP%20Spec-2026--07--28-8A2BE2.svg?style=flat-square)](https://modelcontextprotocol.io/specification/2026-07-28) [![Updated](https://img.shields.io/badge/Updated-October%202026-blue.svg?style=flat-square)](#) [![License](https://img.shields.io/badge/License-Apache%202.0-orange.svg?style=flat-square)](./LICENSE)

</div>

The [MCP overview](guides/mcp-overview.md) explains how MCP works: the roles, the wire format, what a server exposes, how servers ask for input, the transports, authorization, and what changed when the protocol went stateless in revision `2026-07-28`. It is written for engineers who want a working model of the protocol before reading the [specification](https://modelcontextprotocol.io/specification/2026-07-28).

**Building a server?** [`@cyanheads/mcp-ts-core`](https://github.com/cyanheads/mcp-ts-core) is an agent-native TypeScript framework for building MCP servers on the current spec. Every scaffold ships Agent Skills that your coding agent follows to design, test, and release the server, and the server answers legacy and modern clients alike. Scaffold a project with:

```bash
bunx @cyanheads/mcp-ts-core init my-mcp-server
```

**Looking for working servers?** The servers I maintain are listed on my [GitHub profile](https://github.com/cyanheads). The [Fleet Map](https://fleet.caseyjhand.com) lists every hosted one with its tools, endpoint, and install snippets.

## Guides

| Guide | Covers |
|:--|:--|
| [MCP overview](guides/mcp-overview.md) | Hosts, clients, and servers; messages and statelessness; tools, resources, and prompts; multi round-trip requests; notifications and caching; stdio and Streamable HTTP; authorization; extensions; security |
| [Legacy vs. modern servers](guides/legacy-vs-modern.md) | Revision history from `2024-11-05`, what changed in `2026-07-28`, deprecations, era compatibility and detection, and a migration checklist for server authors |

## Official resources

| Resource | Link |
|:--|:--|
| Specification (`2026-07-28`) | [modelcontextprotocol.io/specification/2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28) |
| Changelog for this revision | [Key changes](https://modelcontextprotocol.io/specification/2026-07-28/changelog) |
| Deprecated features registry | [Deprecated](https://modelcontextprotocol.io/specification/2026-07-28/deprecated) |
| Schema (source of truth) | [`schema.ts`](https://github.com/modelcontextprotocol/specification/blob/main/schema/2026-07-28/schema.ts) |
| Extensions | [Extensions overview](https://modelcontextprotocol.io/extensions/overview) |
| Enhancement proposals | [SEPs](https://modelcontextprotocol.io/seps) |
| Server registry | [MCP Registry](https://modelcontextprotocol.io/registry/about) |
| Debugging | [MCP Inspector](https://github.com/modelcontextprotocol/inspector) |

Official SDKs: [TypeScript](https://github.com/modelcontextprotocol/typescript-sdk) · [Python](https://github.com/modelcontextprotocol/python-sdk) · [Go](https://github.com/modelcontextprotocol/go-sdk) · [Java](https://github.com/modelcontextprotocol/java-sdk) · [Kotlin](https://github.com/modelcontextprotocol/kotlin-sdk) · [C#](https://github.com/modelcontextprotocol/csharp-sdk) · [Rust](https://github.com/modelcontextprotocol/rust-sdk) · [Swift](https://github.com/modelcontextprotocol/swift-sdk) · [Ruby](https://github.com/modelcontextprotocol/ruby-sdk) · [PHP](https://github.com/modelcontextprotocol/php-sdk)

---

<div align="center">
Maintained by <a href="https://github.com/cyanheads">cyanheads</a>. Independent of the Model Context Protocol project. Apache 2.0.
</div>
