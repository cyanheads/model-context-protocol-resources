# Model Context Protocol (MCP) Server Development Guide: Building Powerful Tools for LLMs

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/TypeScript-1.11.0-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/Python-1.6.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/Kotlin-0.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![MCP SDK - Java](https://img.shields.io/badge/Java-0.4.0-blue.svg)](https://github.com/modelcontextprotocol/java-sdk)
[![MCP SDK - C#](https://img.shields.io/badge/C%23-SDK-blue.svg)](https://github.com/modelcontextprotocol/csharp-sdk)
[![Guide Last Updated](https://img.shields.io/badge/Last%20Updated-May%202025-brightgreen.svg)]()

## Table of Contents

1.  [Introduction to MCP Servers](#1-introduction-to-mcp-servers)
2.  [Core Server Architecture](#2-core-server-architecture)
    - [Key Components of an MCP Server](#key-components-of-an-mcp-server)
    - [Protocol Fundamentals](#protocol-fundamentals)
    - [Server Lifecycle: Connect, Exchange, Terminate](#server-lifecycle-connect-exchange-terminate)
    - [Message Format and Transport](#message-format-and-transport)
3.  [Building Your First MCP Server (TypeScript)](#3-building-your-first-mcp-server-typescript)
    - [Setting Up Your Development Environment](#setting-up-your-development-environment)
    - [Choosing an SDK](#choosing-an-sdk)
    - [Implementing the Core Server Interface](#implementing-the-core-server-interface)
    - [Handling Connections and Authentication](#handling-connections-and-authentication)
    - [Processing Client Requests](#processing-client-requests)
4.  [Exposing Capabilities](#4-exposing-capabilities)
    - [Defining and Implementing Tools](#defining-and-implementing-tools)
    - [Managing Resources](#managing-resources)
    - [Creating and Sharing Prompts](#creating-and-sharing-prompts)
    - [Schema Validation and Documentation](#schema-validation-and-documentation)
    - [Modular Capability Structure (Personal Recommended Practice)](#modular-capability-structure-personal-recommended-practice)
5.  [Advanced Server Features](#5-advanced-server-features)
    - [Sampling (Client Capability)](#sampling-client-capability)
    - [Roots (Client Capability)](#roots-client-capability)
    - [Streaming Responses (Transport Feature)](#streaming-responses-transport-feature)
    - [Progress Reporting](#progress-reporting)
    - [Resource Subscriptions](#resource-subscriptions)
    - [Completions](#completions)
    - [Logging](#logging)
    - [Performance Optimization](#performance-optimization)
    - [Dynamic Server Capabilities](#dynamic-server-capabilities)
6.  [Security and Best Practices](#6-security-and-best-practices)
    - [Authentication and Authorization](#authentication-and-authorization)
    - [Data Security](#data-security)
    - [Secure Configuration and Credentials](#secure-configuration-and-credentials)
    - [Error Handling](#error-handling)
    - [General Best Practices](#general-best-practices)
    - [Example: Basic Origin Check Middleware (Express)](#example-basic-origin-check-middleware-express)
7.  [Troubleshooting and Resources](#7-troubleshooting-and-resources)
    - [Debugging Tools](#debugging-tools)
    - [Viewing Logs (Example for Claude Desktop - check other client documentation for their log locations):](#viewing-logs-example-for-claude-desktop---check-other-client-documentation-for-their-log-locations)
    - [Common Issues and Solutions](#common-issues-and-solutions)
    - [Implementing Logging Effectively](#implementing-logging-effectively)
    - [Community Resources and Support](#community-resources-and-support)
8.  [Example Implementations](#8-example-implementations)
    - [Echo Server](#echo-server)
    - [SQLite Explorer](#sqlite-explorer)
    - [Low-Level Server Implementation (Manual Handlers)](#low-level-server-implementation-manual-handlers)

## 1. Introduction to MCP Servers

**What is the Model Context Protocol?**

The Model Context Protocol (MCP) is an open standard designed to standardize how AI applications (clients/hosts) connect to and interact with external data sources and tools (servers). Think of it like USB-C for AI: a universal way to plug capabilities into LLM applications. MCP enables a clean separation of concerns, allowing LLM applications to focus on core AI functionality while delegating tasks like data retrieval, external API access, and specialized computations to dedicated, reusable servers.

You can find the official introduction to MCP [here](https://modelcontextprotocol.io/introduction).

**The Role of Servers in the MCP Ecosystem**

Servers are the backbone of the MCP ecosystem. They act as bridges between LLM applications and the external world (local files, databases, web APIs, etc.). A server exposes specific capabilities through the MCP standard:

- **Providing access to data:** Fetching information from databases, APIs, filesystems, or other sources (via **Resources**).
- **Exposing executable functions:** Offering functionalities like API calls, code execution, calculations, or file manipulation (via **Tools**).
- **Offering guided interactions:** Providing pre-defined prompt templates or workflows for users (via **Prompts**).
- **Connecting to external systems:** Integrating with other applications, services, or platforms.

**Benefits of Implementing an MCP Server**

Creating an MCP server offers several advantages:

- **Extensibility:** Easily add new capabilities to any MCP-compatible client without modifying the client's core code.
- **Modularity:** Develop and maintain specialized functionalities in isolated, reusable components.
- **Interoperability:** Enable different LLM applications (clients) to share and use the same servers (context sources and tools).
- **Focus:** Concentrate on building unique capabilities and integrations, leveraging the standardized protocol.
- **Security:** Keep sensitive credentials and complex logic contained within the server, often running in a user's trusted environment.

**Server vs. Client: Understanding the Relationship**

In the MCP architecture:

- **Hosts:** Applications like Claude Desktop, VS Code extensions (e.g., Copilot, Continue), or custom applications that manage MCP clients and interact with the user/LLM. (Note: "Host" is a helpful conceptual term used in this guide, not formally defined in the spec itself).
- **Clients:** Protocol handlers _within_ the host application that initiate and manage stateful 1:1 connections to servers.
- **Servers:** Independent processes (local or remote) that listen for client connections, expose capabilities (Tools, Resources, Prompts), and process requests.

A single host can manage multiple clients connecting to different servers simultaneously. A single server can potentially serve multiple clients (depending on the transport and server implementation).

You can find the official server quickstart documentation [here](https://modelcontextprotocol.io/quickstart/server).

## 2. Core Server Architecture

### Key Components of an MCP Server

An MCP server implementation typically involves:

1.  **Protocol Handling:** Logic to manage the MCP connection lifecycle, negotiate capabilities, and handle incoming/outgoing JSON-RPC messages (Requests, Responses, Notifications). SDKs abstract much of this.
2.  **Transport Layer:** The mechanism for sending/receiving messages (e.g., Stdio, Streamable HTTP). The SDK provides implementations.
3.  **Capability Implementation:** The core logic defining the specific Tools, Resources, and/or Prompts the server offers.
4.  **Schema Definitions:** Clear definitions (e.g., using JSON Schema or libraries like `zod`) for the inputs and outputs of capabilities.

### Protocol Fundamentals

MCP uses JSON-RPC 2.0 for all communication. It's a stateful protocol, meaning connections are established and maintained. The core interaction involves capability negotiation followed by message exchange based on those capabilities.

### Server Lifecycle: Connect, Exchange, Terminate

The lifecycle of an MCP connection involves three main stages:

1.  **Initialization:**

    - The client sends an `initialize` request with its supported `protocolVersion`, `capabilities`, and `clientInfo`.
    - The server responds with its chosen `protocolVersion`, its `capabilities`, `serverInfo`, and optional `instructions`. Version negotiation occurs here.
    - The client sends an `initialized` notification to confirm readiness.
    - _Crucially, only `ping` requests and server `logging` notifications are permitted before initialization is complete; all other requests **ARE FORBIDDEN** to be sent._ The `ping` utility can be used by either party during this phase (and later) to check connection health without violating the initialization sequence.

2.  **Message Exchange:** After initialization, clients and servers exchange messages based on negotiated capabilities:

    - **Request-Response:** For operations like `tools/call`, `resources/read`, `prompts/get`, `sampling/createMessage`, etc.
    - **Notifications:** For updates like `listChanged`, `progress`, `cancelled`, `logging`, etc.
    - The `ping` utility can be used as a keep-alive or health check during this phase.

3.  **Termination:** The connection ends when:
    - The transport is closed (e.g., client closes stdin for stdio, HTTP connection drops).
    - An unrecoverable error occurs.
    - Explicit shutdown logic is triggered (though MCP doesn't define a specific `shutdown` message; it relies on transport closure).

#### Timeouts and Cancellation

Implementations **SHOULD** establish timeouts for sent requests. When a timeout occurs, the sender **SHOULD** issue a `$/cancelRequest` notification (as defined in the spec's Utilities section) for that request ID and stop waiting. SDK implementations can abstract timeout and cancellation logic, but the underlying requirement remains. Implementations **MAY** reset timeouts upon receiving relevant `notifications/progress` but **SHOULD** enforce a maximum overall timeout.

### Message Format and Transport

MCP uses **JSON-RPC 2.0**. Key message types are summarized below:

| Type         | `jsonrpc` | `id`                     | `method` | `params` | `result` | `error` | Description                                  |
| :----------- | :-------- | :----------------------- | :------- | :------- | :------- | :------ | :------------------------------------------- |
| Request      | `"2.0"`   | Required (string/number) | Required | Optional | N/A      | N/A     | Client initiates an action on the server.    |
| Response     | `"2.0"`   | Required (matches req)   | N/A      | N/A      | Required | XOR     | Server replies to a Request (success/error). |
| Notification | `"2.0"`   | Forbidden                | Required | Optional | N/A      | N/A     | One-way message (no response expected).      |

1.  **Requests:** Must have `jsonrpc: "2.0"`, a unique `id` (string or number, not null), and a `method`. May have `params`.

```typescript
// Example Request
{
  "jsonrpc": "2.0",
  "id": 123,
  "method": "tools/call",
  "params": { "name": "myTool", "arguments": { "arg1": "value" } }
}
```

2.  **Responses:** Must have `jsonrpc: "2.0"` and the same `id` as the request. Must contain _either_ `result` (any JSON value) _or_ `error`.

```typescript
// Example Success Response
{
  "jsonrpc": "2.0",
  "id": 123,
  "result": { "output": "Success!" }
}

// Example Error Response
{
  "jsonrpc": "2.0",
  "id": 123,
  "error": {
    "code": -32602, // JSON-RPC error code
    "message": "Invalid parameters",
    "data": { "details": "Missing required argument 'arg1'" }
  }
}
```

3.  **Notifications:** Must have `jsonrpc: "2.0"` and a `method`. Must _not_ have an `id`. May have `params`.

```typescript
// Example Notification
{
  "jsonrpc": "2.0",
  "method": "notifications/tools/list_changed"
}
```

#### Transports

MCP defines standard ways to transport these messages:

| Feature           | Standard Input/Output (stdio)                     | Streamable HTTP                                                                        |
| :---------------- | :------------------------------------------------ | :------------------------------------------------------------------------------------- |
| **Use Case**      | Local servers launched as subprocesses by client  | Local or remote servers, potentially multi-client                                      |
| **Communication** | Newline-delimited JSON-RPC on stdin/stdout        | Single endpoint (`/mcp`) using POST (client->server) & GET (server->client SSE)        |
| **Logging**       | stderr                                            | stderr or dedicated log files; MCP logging via `notifications/message`                 |
| **Lifecycle**     | Managed by client process                         | Independent server process; requires explicit start/stop                               |
| **Security**      | OS process security; Env vars for secrets         | **HTTPS Mandatory**; OAuth 2.0 (MCP Auth Spec); Origin/CORS validation; Bind localhost |
| **State**         | Implicitly 1:1 client-server                      | Supports `Mcp-Session-Id` for stateful sessions                                        |
| **Resumability**  | N/A (process restart)                             | Supported via SSE `id` and `Last-Event-ID` header                                      |
| **Pros**          | Lower overhead, simpler setup for local tools     | Flexible (local/remote), multi-client capable, web-standard protocols                  |
| **Cons**          | Local only, tied to client lifecycle, less secure | Higher overhead, requires HTTP server setup, complex security implementation           |

1.  **Standard Input/Output (stdio):** Ideal for local servers launched as subprocesses by the client.

    - **Communication:** Server reads JSON-RPC from `stdin`, writes to `stdout`. Messages are newline-delimited.
    - **Logging:** Server can write logs to `stderr`.
    - **Restrictions:** `stdout` is _only_ for MCP messages. `stdin` _only_ receives MCP messages.
    - **Lifecycle:** Client manages the server process lifecycle (start, stop).

    _Example (Server - Basic Setup):_

    ```typescript
    import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
    import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

    async function startServer() {
      const server = new McpServer(
        {
          name: "my-stdio-server",
          version: "1.0.0",
        },
        {
          // Define server capabilities here, e.g., tools: {}
        }
      );

      // Add tool/resource/prompt implementations here...

      const transport = new StdioServerTransport();
      await server.connect(transport);
      console.error("MCP Server connected via stdio."); // Log to stderr
    }

    startServer().catch((err) => {
      console.error("Server failed to start:", err);
      process.exit(1);
    });
    ```

2.  **Streamable HTTP:** Suitable for servers running as independent processes (local or remote) that might handle multiple clients. Uses HTTP POST for client messages and can use Server-Sent Events (SSE) for streaming server messages.

    - **Endpoint:** Server provides a single HTTP endpoint path supporting POST (client messages) and GET (for server-initiated streams).
    - **Client POST:** Sends JSON-RPC request(s)/notification(s)/response(s). Server responds with `202 Accepted` (for notifications/responses) or initiates a response stream (SSE or single JSON) for requests.
    - **Client GET:** Initiates an SSE stream for server-initiated messages (requests/notifications).
    - **SSE:** Server can send multiple JSON-RPC messages (requests, notifications, responses) over an SSE stream.
    - **Security:** Requires careful handling of `Origin` headers, binding to `localhost` for local servers, and authentication.
    - **Session Management:** Supports optional `Mcp-Session-Id` header for stateful sessions.
    - **Resumability:** Supports resuming broken SSE streams using SSE `id` fields and the `Last-Event-ID` header (see below).
    - **Backwards Compatibility:** Guidance exists for interoperating with older HTTP+SSE clients/servers (see below).

    _Example (Server - Basic Express Setup):_

    ```typescript
    import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
    // The MCP SDK provides transport implementations that integrate with frameworks
    // like Express or can be used with Node's built-in http module. Consult the
    // specific SDK documentation for available transport classes and integration helpers.
    // This example shows a conceptual integration with Express.
    import express, { Request, Response } from "express";

    async function startServer() {
      const server = new McpServer(
        {
          name: "my-http-server",
          version: "1.0.0",
        },
        {
          // Define server capabilities here
        }
      );

      // Add tool/resource/prompt implementations here...

      const app = express();
      // Middleware for raw body parsing is often needed for SDK transport handlers
      app.use(express.raw({ type: "*/*" }));

      // Instantiate the appropriate SDK HTTP transport helper based on SDK documentation.
      // This helper integrates the McpServer instance with the HTTP framework.

      // Define the single MCP endpoint
      app.all("/mcp", (req: Request, res: Response) => {
        // The SDK's transport handler manages the HTTP request lifecycle.
        // It handles:
        // - Differentiating GET/POST/DELETE methods.
        // - Session management ('Mcp-Session-Id' header).
        // - Origin validation.
        // - JSON-RPC parsing.
        // - SSE stream initiation and management.
        // - Sending correct HTTP status codes (200, 202, 4xx, 5xx).
        // - Routing messages to/from the McpServer instance.

        // If implementing manually without SDK helpers, refer to the MCP spec
        // for detailed HTTP handling rules. Manual implementation requires
        // careful state management for concurrent streams and request types.
        // The SDK aims to abstract this complexity.
        res
          .status(501)
          .send(
            "HTTP transport handler logic (using SDK or manual) goes here."
          );
      });

      const port = 3000;
      app.listen(port, "127.0.0.1", () => {
        // Bind to localhost for security in local development
        console.log(`MCP Server listening on http://127.0.0.1:${port}/mcp`);
      });
    }

    startServer().catch((err) => {
      console.error("Server failed to start:", err);
      process.exit(1);
    });
    ```

    **Streamable HTTP - Resumability and Redelivery:**
    To handle potentially unreliable network connections, the Streamable HTTP transport supports resuming broken SSE streams:

    - Servers **MAY** include a unique `id` field with each SSE event.
    - If an SSE connection drops, the client **SHOULD** attempt to reconnect by sending a GET request to the MCP endpoint with the `Last-Event-ID` header set to the ID of the last event it successfully received.
    - The server **MAY** use this `Last-Event-ID` to replay messages sent after that ID on the original stream, allowing the client to catch up without missing messages.

    **Streamable HTTP - Backwards Compatibility:**
    The Streamable HTTP transport replaces the older HTTP+SSE transport from protocol version 2024-11-05. To maintain compatibility:

    - **Servers** supporting older clients **SHOULD** continue to host the old SSE (`/`) and POST (`/mcp`) endpoints alongside the new single Streamable HTTP endpoint (`/mcp` supporting GET/POST/DELETE).
    - **Clients** supporting older servers **SHOULD** first attempt the new Streamable HTTP POST to `/mcp`. If this fails (e.g., 404/405), they **SHOULD** fall back to the old mechanism (GET `/` for SSE, POST `/mcp`).

#### Custom Transports

You can implement custom transports if needed, adhering to the `Transport` interface (or equivalent in other SDKs) and ensuring JSON-RPC compliance.

#### Error Handling

Transports must handle connection errors, parsing errors, timeouts, etc., and propagate them appropriately (e.g., via the `onerror` callback).

## 3. Building Your First MCP Server (TypeScript)

This section guides you through creating a basic MCP server using the TypeScript SDK.

### Setting Up Your Development Environment

1.  **Install Node.js:** Ensure Node.js (LTS version, e.g., 18.x or 20.x) and npm are installed.
2.  **Create Project:**
    ```bash
    mkdir my-mcp-server
    cd my-mcp-server
    npm init -y
    ```
3.  **Install Dependencies:**
    ```bash
    # Check for the latest stable versions before installing
    npm install @modelcontextprotocol/sdk zod
    npm install -D typescript @types/node
    ```
4.  **Configure TypeScript (`tsconfig.json`):**
    ```json
    {
      "compilerOptions": {
        "target": "ES2022", // Target modern Node.js features
        "module": "NodeNext", // Use modern ES Modules
        "moduleResolution": "NodeNext",
        "esModuleInterop": true,
        "forceConsistentCasingInFileNames": true,
        "strict": true, // Enable strict type checking
        "skipLibCheck": true,
        "outDir": "./build", // Output directory for compiled JS
        "rootDir": "./src", // Source directory
        "sourceMap": true // Generate source maps for debugging
      },
      "include": ["src/**/*"], // Compile files in src
      "exclude": ["node_modules"]
    }
    ```
5.  **Update `package.json`:** Add `"type": "module"` for ES Module support and scripts.
    ```json
    {
      "name": "my-mcp-server",
      "version": "1.0.0",
      "description": "",
      "main": "build/index.js",
      "type": "module", // Enable ES Modules
      "scripts": {
        "build": "tsc",
        "start": "node build/index.js",
        "dev": "tsc --watch & node --watch build/index.js" // Optional: for development
      },
      "keywords": [],
      "author": "",
      "license": "ISC",
      "dependencies": {
        "@modelcontextprotocol/sdk": "^1.11.0", // Example: Use specific version
        "zod": "^3.24.1" // Example: Use specific version
      },
      "devDependencies": {
        "@types/node": "^20.12.2", // Example: Use specific version
        "typescript": "^5.4.5" // Example: Use specific version
      }
    }
    // Note: Always check for and use the latest stable versions of dependencies.
    ```
6.  **Create Source File:**
    ```bash
    mkdir src
    touch src/index.ts
    ```

### Choosing an SDK

MCP offers official SDKs for several popular programming languages, allowing you to build servers in the environment you're most comfortable with. The primary SDKs available are:

| Language      | Package/Repository                                                                    | Notes                                                                                                                                         |
| :------------ | :------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| TypeScript/JS | [`@modelcontextprotocol/sdk`](https://github.com/modelcontextprotocol/typescript-sdk) | Mature, feature-rich, primary SDK for examples.                                                                                               |
| Python        | [`modelcontextprotocol-python`](https://github.com/modelcontextprotocol/python-sdk)   | Robust, good for Python ecosystem integration.                                                                                                |
| Kotlin        | [`modelcontextprotocol-kotlin`](https://github.com/modelcontextprotocol/kotlin-sdk)   | Modern JVM language, Java interop. [Server Template](https://github.com/modelcontextprotocol/kotlin-sdk/tree/main/samples/kotlin-mcp-server). |
| Java          | [`modelcontextprotocol-java`](https://github.com/modelcontextprotocol/java-sdk)       | Suitable for enterprise Java environments.                                                                                                    |
| C#            | [`ModelContextProtocol.Sdk`](https://github.com/modelcontextprotocol/csharp-sdk)      | Enables development in the .NET ecosystem.                                                                                                    |

**Factors to Consider:**

- **Language Familiarity:** Choose the SDK that best matches your team's expertise.
- **Ecosystem Integration:** Consider the libraries and external systems you need your server to interact with. Choose an SDK that integrates well with that ecosystem (e.g., Python for data science libraries, Java/.NET for enterprise systems).
- **Platform:** Ensure the SDK is compatible with your target deployment platform.
- **Feature Parity:** While all SDKs aim to implement the core MCP specification, feature sets and helper utilities might vary slightly. Check the specific SDK's documentation for details.
- **Community & Support:** Look at the activity and documentation for the specific SDK.

The TypeScript SDK (`@modelcontextprotocol/sdk`) is used for most examples in this guide due to its maturity and common use in web-related development environments. However, the core concepts apply across all official SDKs.

### Implementing the Core Server Interface

Let's create a simple server that provides a "greeting" tool.

`src/index.ts`:

```typescript
#!/usr/bin/env node

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod"; // Import zod for schema validation

async function main() {
  // 1. Create an MCP server instance
  const server = new McpServer(
    {
      // Server identification
      name: "GreetingServer",
      version: "1.0.1",
    },
    {
      // Declare server capabilities
      capabilities: {
        tools: { listChanged: false }, // We support tools, no dynamic changes
        // resources: {}, // Uncomment if supporting resources
        // prompts: {},   // Uncomment if supporting prompts
      },
    }
  );

  // 2. Define the input schema for the 'greet' tool using zod
  const greetInputSchema = z.object({
    name: z.string().min(1).describe("The name of the person to greet"),
  });

  // 3. Add the 'greet' tool implementation
  server.tool(
    "greet", // Tool name
    greetInputSchema, // Use the zod schema for input validation
    async (input) => {
      // Input is automatically validated against the schema
      const message = `Hello, ${input.name}! Welcome to MCP.`;
      console.error(`Tool 'greet' called with name: ${input.name}`); // Log to stderr

      // Return the result conforming to CallToolResultContent
      return {
        content: [{ type: "text", text: message }],
        // isError: false, // Default is false
      };
    }
  );

  // 4. Create a transport (stdio for this example)
  const transport = new StdioServerTransport();

  // 5. Connect the server to the transport and start listening
  try {
    await server.connect(transport);
    console.error("Greeting MCP Server is running and connected via stdio."); // Log to stderr
  } catch (error) {
    console.error("Failed to connect server:", error);
    process.exit(1);
  }

  // Keep the server running (for stdio, it runs until stdin closes)
  // For other transports like HTTP, you'd typically have a server.listen() call
}

// Run the main function
main().catch((error) => {
  console.error("Unhandled error during server startup:", error);
  process.exit(1);
});
```

### Handling Connections and Authentication

- **Stdio:** Connection is implicit when the client starts the server process. Authentication typically relies on the OS user context or environment variables passed by the client.
- **Streamable HTTP:** Requires explicit connection handling (e.g., using Express, Koa, or Node's `http` module). Authentication should be implemented using standard HTTP mechanisms (e.g., Bearer tokens via the MCP [Authorization spec](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization), API keys, OAuth).

### Processing Client Requests

The `McpServer` class (or the lower-level `Server` class) handles parsing incoming JSON-RPC messages and routing requests to the appropriate handlers (like the one defined with `server.tool()`). The SDK manages matching the `method` field (`tools/call`, `resources/read`, etc.) and invoking your registered functions with validated parameters.

**To run this server:**

1.  **Compile:** `npm run build`
2.  **Run:** `npm start` or `node build/index.js`

**To test it manually:**

Run the server. In another terminal, send a JSON-RPC request to its stdin:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"greet","arguments":{"name":"World"}}}' | node build/index.js
```

You should see the response printed to stdout (and the log message on stderr).

## 4. Exposing Capabilities

Servers bring value by exposing Tools, Resources, and Prompts.

### Defining and Implementing Tools

Tools are functions the LLM can invoke (with user approval) to perform actions. They are **model-controlled**.

**Key Features & Structure:**

- **Definition:** Defined using `server.tool()`. Provide a `name`, `description`, input schema (using Zod shape), optional `annotations`, and an async handler function.
- **Annotations (Optional):** Provide hints about tool behavior using a standard set of keys defined by the MCP specification.

  | Annotation        | Type      | Description                                                                                               | Trust Model    |
  | :---------------- | :-------- | :-------------------------------------------------------------------------------------------------------- | :------------- |
  | `title`           | `string`  | Human-readable title for UI display.                                                                      | Untrusted Hint |
  | `readOnlyHint`    | `boolean` | Suggests the tool does not modify state (e.g., like a GET request).                                       | Untrusted Hint |
  | `destructiveHint` | `boolean` | Suggests the tool might make significant, potentially irreversible changes or deletions.                  | Untrusted Hint |
  | `idempotentHint`  | `boolean` | Suggests calling multiple times with the same arguments has the same effect as calling once.              | Untrusted Hint |
  | `openWorldHint`   | `boolean` | Suggests interaction with external systems/data that can change unpredictably between calls (e.g., APIs). | Untrusted Hint |

- **Trust Model (CRITICAL):** Clients **MUST** treat these annotations purely as **untrusted hints** unless the server providing them is explicitly trusted by the host application or user. Servers **SHOULD NOT** rely on clients strictly adhering to these hints for enforcing security or correctness. The hints are primarily for informing the client/LLM's _strategy_ or the user interface presentation.
- **Custom Annotations:** While the protocol allows adding custom key-value pairs to the annotations object beyond the standard ones, clients are not guaranteed to understand or utilize them. The same trust model applies: custom annotations are untrusted hints. Stick to standard annotations for broadest compatibility.
- **Invocation:** The LLM decides when to call tools; the client mediates and sends a `tools/call` request. The SDK handles routing to your handler.
- **Response:** Your handler should return a `CallToolResult` object (`{ content: [...], isError?: boolean }`). Execution errors are reported via `isError: true` and details in `content`.
- **Schema Importance:** Detailed Zod schemas significantly improve the LLM's ability to use the tool correctly.

**Implementation Example (using `McpServer` helper):**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod"; // For schema definition
import { logger } from "../utils/logger.js"; // Assuming a logger utility
import { TextContent } from "@modelcontextprotocol/sdk/types.js"; // Specific content type

// Assume 'server' is an initialized McpServer instance

// Simple tool with parameters defined using Zod
server.tool(
  "calculate-bmi", // Tool name
  "Calculates Body Mass Index.", // Tool description
  {
    // Input schema shape (Zod object shape)
    weightKg: z.number().describe("Weight in kilograms"),
    heightM: z.number().positive().describe("Height in meters"),
  },
  // Optional: Tool Annotations
  { title: "BMI Calculator", readOnlyHint: true },
  // Handler function receives validated arguments
  async ({ weightKg, heightM }) => {
    logger.debug(`Executing tool: calculate-bmi`, { weightKg, heightM });
    const bmi = weightKg / (heightM * heightM);
    const resultContent: TextContent = {
      type: "text",
      text: `BMI: ${bmi.toFixed(2)}`,
    };
    return {
      content: [resultContent], // Array of content parts
      // isError: false // Default is false
    };
    // On execution error, return:
    // return { content: [{ type: 'text', text: 'Error: Invalid height' }], isError: true };
  }
);
logger.info(`Registered tool: calculate-bmi`);

// Async tool example (e.g., fetching external data)
server.tool(
  "fetch-weather",
  "Fetches weather for a given city.",
  { city: z.string().min(1).describe("The city name") },
  // Annotations: Title, read-only (doesn't change local state), open world (external API)
  { title: "Fetch Weather", readOnlyHint: true, openWorldHint: true },
  async ({ city }) => {
    logger.debug(`Executing tool: fetch-weather`, { city });
    try {
      // Replace with actual API call
      const response = await fetch(
        `https://api.example-weather.com/?city=${encodeURIComponent(city)}`
      );
      if (!response.ok) {
        throw new Error(`API request failed with status ${response.status}`);
      }
      const data = await response.text(); // Or response.json()
      const resultContent: TextContent = { type: "text", text: data };
      return { content: [resultContent] };
    } catch (error) {
      logger.error("fetch-weather tool failed", { error });
      const errorMessage =
        error instanceof Error ? error.message : "Unknown error";
      const errorContent: TextContent = {
        type: "text",
        text: `Error fetching weather: ${errorMessage}`,
      };
      return { content: [errorContent], isError: true };
    }
  }
);
logger.info(`Registered tool: fetch-weather`);

// Optional: To notify clients of dynamic tool changes:
// server.sendToolListChanged();
```

**Best Practices:**

- Use clear, descriptive names and descriptions.
- Define precise input schemas using `zod`.
- Handle errors gracefully within the tool handler (return `isError: true`). See also [Error Handling](#error-handling) in Section 6.
- Keep tools focused on a single task; complex operations might be better as multiple tools.
- Use annotations (`readOnlyHint`, `destructiveHint`, etc.) accurately to provide hints about behavior, but remember they are untrusted hints.

**Security:**

- **Validate inputs rigorously** using schemas and potentially additional checks in the handler.
- Implement access control if the tool interacts with sensitive resources or performs privileged operations.
- Sanitize outputs, especially if they incorporate data from external sources.
- Be mindful of rate limits when interacting with external APIs.

### Managing Resources

Resources expose data or content to the client/LLM. They are **application-controlled**.

**Key Features & Structure:**

- **URI:** A unique identifier for the resource (e.g., `file:///path/to/file.txt`, `db://users/123`).
- **Definition:** Resources can be defined statically or dynamically using `ResourceTemplate`.
- **Discovery:** Clients can discover resources via `resources/list` (for specific URI prefixes or templates).
- **Reading:** Clients fetch resource content using `resources/read`.
- **Content:** Can be text (`text` field) or binary (`blob` field, base64 encoded). `mimeType` and `size` should be provided.
- **Updates:** Servers can notify clients of changes using `notifications/resources/list_changed` (if the list of available resources changes) or `notifications/resources/updated` (if the content of a subscribed resource changes). See [Resource Subscriptions](#resource-subscriptions).

**Implementation Example (using `McpServer` helper):**

```typescript
import {
  McpServer,
  ResourceTemplate,
} from "@modelcontextprotocol/sdk/server/mcp.js";
import { McpSchema } from "@modelcontextprotocol/sdk/types.js"; // Import base schema types
import fs from "fs/promises";
import path from "path";
import { fileURLToPath } from "url"; // For ES Modules __dirname equivalent
import { logger } from "../utils/logger.js"; // Assuming a logger utility

// Assume 'server' is an initialized McpServer instance

// Get directory relative to the current module file
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const DATA_DIR = path.resolve(__dirname, "../data"); // Example: data dir sibling to src

// Ensure data directory exists
try {
  await fs.mkdir(DATA_DIR, { recursive: true });
  logger.info(`Data directory ensured: ${DATA_DIR}`);
} catch (error) {
  logger.error(`Failed to create data directory ${DATA_DIR}:`, error);
  // Decide if this is a fatal error for the server
}

// Static resource example
server.resource(
  "app-config", // Unique name for this resource registration
  "config://app", // The URI clients will use
  async (uri) => {
    // Handler function
    logger.debug(`Reading resource: ${uri.href}`);
    // In a real app, load config from a file or environment
    const configData = { settingA: "valueA", settingB: 123 };
    return {
      contents: [
        {
          uri: uri.href, // Echo back the requested URI
          mimeType: "application/json", // Specify the content type
          text: JSON.stringify(configData, null, 2), // The actual content
        },
      ],
    };
  }
);
logger.info(`Registered static resource: config://app`);

// Dynamic resource template for files in DATA_DIR
server.resource(
  "data-files", // Resource group name
  new ResourceTemplate("file:///data/{filename}", {
    // URI Template
    // List function: returns available resources matching the template
    list: async () => {
      logger.debug(`Listing resources for template: file:///data/{filename}`);
      try {
        const files = await fs.readdir(DATA_DIR);
        const resourceList = await Promise.all(
          files.map(async (file) => {
            const filePath = path.join(DATA_DIR, file);
            try {
              const stats = await fs.stat(filePath);
              if (stats.isFile()) {
                // Basic MIME type detection (can be improved)
                const mimeType =
                  path.extname(file) === ".txt"
                    ? "text/plain"
                    : "application/octet-stream";
                return {
                  uri: `file:///data/${file}`,
                  name: file,
                  mimeType: mimeType,
                  size: stats.size,
                };
              }
            } catch (statError: any) {
              // Ignore files that disappear or access errors
              if (statError.code !== "ENOENT") {
                logger.warn(
                  `Could not stat file ${filePath}: ${statError.message}`
                );
              }
            }
            return null;
          })
        );
        const validResources = resourceList.filter(
          (r): r is McpSchema.Resource => r !== null
        );
        logger.debug(`Found ${validResources.length} data files.`);
        return validResources;
      } catch (error: any) {
        logger.error(`Error listing data files: ${error.message}`);
        return []; // Return empty list on error
      }
    },
    // Define subscribe/unsubscribe if needed (see Advanced Features)
    // subscribe: async (uri, params) => { /* ... */ },
    // unsubscribe: async (uri, params) => { /* ... */ },
  }),
  // Read function: handles 'resources/read' for URIs matching the template
  async (uri, params) => {
    // params contains matched template variables, e.g., { filename: '...' }
    const filename = params.filename;
    logger.debug(`Reading resource: ${uri.href} (filename: ${filename})`);
    if (!filename || typeof filename !== "string") {
      throw new Error("Invalid or missing filename parameter in URI");
    }
    // IMPORTANT: Prevent path traversal attacks
    const requestedPath = path.join(DATA_DIR, filename);
    const resolvedDataDir = path.resolve(DATA_DIR);
    const resolvedRequestedPath = path.resolve(requestedPath);

    if (
      !resolvedRequestedPath.startsWith(resolvedDataDir + path.sep) &&
      resolvedRequestedPath !== resolvedDataDir
    ) {
      logger.error(`Access denied: Path traversal attempt: ${requestedPath}`);
      throw new Error("Access denied: Invalid path");
    }

    try {
      const fileContents = await fs.readFile(requestedPath); // Read as buffer
      const stats = await fs.stat(requestedPath);
      const mimeType =
        path.extname(filename) === ".txt"
          ? "text/plain"
          : "application/octet-stream";

      return {
        contents: [
          {
            uri: uri.href, // Use the full URI from the request
            mimeType: mimeType,
            blob: fileContents.toString("base64"), // Send content as base64 blob
            size: stats.size,
          },
        ],
      };
    } catch (error: any) {
      if (error.code === "ENOENT") {
        logger.warn(`Resource not found: ${uri.href}`);
        throw new Error(`Resource not found: ${uri.href}`); // Specific error
      }
      logger.error(`Error reading file ${requestedPath}: ${error.message}`);
      throw new Error(`Error reading resource: ${error.message}`);
    }
  }
);
logger.info(`Registered dynamic resource template: file:///data/{filename}`);

// Optional: To notify clients of dynamic resource list changes:
// server.sendResourceListChanged();
```

**Best Practices:**

- Use clear, hierarchical URIs (e.g., `db://users/{userId}/profile`).
- Provide accurate `mimeType` and `size` when possible.
- Implement `listChanged` notifications if the set of available resources changes dynamically.
- Consider implementing `subscribe` for resources whose content changes frequently (see [Resource Subscriptions](#resource-subscriptions)).
- **Crucially:** Sanitize and validate URIs and parameters (especially file paths) to prevent security vulnerabilities like directory traversal.

**Security:**

- Validate URIs rigorously against expected patterns.
- Implement access control based on the URI, parameters, or authenticated user context.
- Ensure path validation prevents access outside designated directories or scopes.

### Creating and Sharing Prompts

Prompts are pre-defined templates for user interactions, often surfaced as commands in the client UI. They are **user-controlled**.

**Key Features & Structure:**

- **Definition:** Defined using `server.prompt()`. Provide a `name` (often used for slash commands), `description`, argument schema (using Zod shape), and a handler function.
- **Arguments:** The handler receives validated arguments based on the schema.
- **Output:** The handler returns a `GetPromptResult` object containing `messages` (an array structured like LLM conversation history) to be sent to the LLM.
- **Discovery:** Clients discover prompts via `prompts/list`.
- **Usage:** Clients invoke prompts via `prompts/get`, providing arguments.

**Implementation Example (using High-level SDK `McpServer` helper):**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod"; // For argument schema
import { logger } from "../utils/logger.js"; // Assuming a logger utility
import { Role } from "@modelcontextprotocol/sdk/types.js"; // Role enum

// Assume 'server' is an initialized McpServer instance

server.prompt(
  "review-code", // Prompt name (often used for slash commands)
  "Generates a prompt to ask the LLM to review code.", // Description
  {
    // Argument schema shape (Zod object shape)
    code: z.string().min(10).describe("The code snippet to review"),
    focus: z
      .enum(["performance", "security", "style", "general"])
      .optional()
      .describe("Optional area of focus for the review"),
  },
  // Handler function receives validated arguments
  ({ code, focus }) => {
    logger.debug(`Generating prompt: review-code`, { focus });
    let promptText = `Please review the following code for potential issues and suggest improvements`;
    if (focus) {
      promptText += `, focusing specifically on ${focus}`;
    }
    promptText += `:\n\n\`\`\`\n${code}\n\`\`\``;

    // Construct the messages for the LLM
    return {
      // Optional description for the specific invocation
      // description: `Requesting ${focus || 'general'} review for code snippet...`,
      messages: [
        // Array of messages forming the prompt
        {
          role: Role.USER, // Or Role.ASSISTANT
          content: [
            // Content is always an array
            {
              type: "text", // Content type
              text: promptText,
            },
            // Can include other content types like images if needed
          ],
        },
      ],
    };
  }
);
logger.info(`Registered prompt: review-code`);

// Optional: To notify clients of dynamic prompt changes:
// server.sendPromptListChanged();
```

**Best Practices:**

- Use intuitive names (often mapping directly to UI commands like `/review-code`).
- Provide clear descriptions for the prompt itself and its arguments.
- Use `zod` schemas to validate arguments.
- Handle optional arguments gracefully within the prompt generation logic.

**Security:**

- Validate and sanitize all arguments provided by the user, especially if they are incorporated directly into system interactions or external calls triggered by the prompt's eventual LLM response.
- Be cautious if prompts embed resource URIs; ensure resource access control is respected when the client resolves those resources.

### Schema Validation and Documentation

- **Use Schemas:** Consistently use JSON Schema (or libraries like `zod` which generate it) to define the `inputSchema` for Tools and the structure of `arguments` for Prompts. This is crucial for interoperability, validation by the SDK, and enabling better client-side experiences (like auto-generating forms).
- **Documentation:** Provide clear `description` fields for all capabilities (Tools, Resources, Prompts) and their parameters/arguments. This helps both humans and LLMs understand how to use them correctly. Consider adding examples within descriptions where helpful.

### Modular Capability Structure (Personal Recommended Practice)

For better organization and maintainability, especially in larger servers, consider separating the logic of your resources and tools from their registration code. This pattern promotes separation of concerns.

**Example: Modular Tool Structure**

```typescript
// --- src/mcp-server/tools/myTool/myToolLogic.ts ---
// Contains the core logic and schema for the tool.
import { z } from "zod";
import { TextContent } from "@modelcontextprotocol/sdk/types.js";
import { logger } from "../../../utils/logger.js"; // Adjust path as needed

// Define input schema using Zod
export const myToolInputSchema = z.object({
  parameter1: z.string().describe("Description for parameter 1"),
  parameter2: z.number().optional().describe("Optional number parameter"),
});

// Define the handler function
export async function handleMyTool(
  args: z.infer<typeof myToolInputSchema>
): Promise<{ content: TextContent[]; isError?: boolean }> {
  logger.debug("Executing myTool logic", args);
  // --- Tool implementation goes here ---
  const resultText = `Processed ${args.parameter1} with optional ${
    args.parameter2 ?? "N/A"
  }`;
  return {
    content: [{ type: "text", text: resultText }],
  };
}

// --- src/mcp-server/tools/myTool/registration.ts ---
// Handles registering the tool with the McpServer instance.
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { myToolInputSchema, handleMyTool } from "./myToolLogic.js";
import { logger } from "../../../utils/logger.js"; // Adjust path as needed

export function registerMyTool(server: McpServer): void {
  server.tool(
    "my-tool", // Tool name
    "A description of what my-tool does.", // Description
    myToolInputSchema, // Pass the imported schema shape
    // Optional annotations
    { title: "My Awesome Tool", readOnlyHint: true },
    handleMyTool // Pass the imported handler function
  );
  logger.info("Registered tool: my-tool");
}

// --- src/mcp-server/server.ts ---
// The main server orchestration file imports and calls registration functions.
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { registerMyTool } from "./tools/myTool/registration.js"; // Import registration
import { config } from "../../config/index.js"; // Adjust path as needed
import { logger } from "../../utils/logger.js"; // Adjust path as needed
// ... other imports like transports ...

function createMcpServerInstance(): McpServer {
  const server = new McpServer(
    { name: config.mcpServerName, version: config.mcpServerVersion },
    {
      /* capabilities */
    }
  );

  // Register capabilities by calling their registration functions
  registerMyTool(server);
  // registerOtherTool(server);
  // registerMyResource(server);

  logger.info("MCP Server instance configured with capabilities.");
  return server;
}

// ... rest of server startup logic (transports, etc.) ...
```

This structure makes it easier to manage individual capabilities, test their logic independently, and keep the main `server.ts` file focused on orchestration.

## 5. Advanced Server Features

Beyond the core capabilities, MCP includes features for more sophisticated scenarios.

### Sampling (Client Capability)

While sampling (`sampling/createMessage`) is a request _sent by the server_, it relies on the _client_ supporting the `sampling` capability. Servers don't implement sampling handling themselves; they _initiate_ sampling requests if the connected client supports it.

**Use Case:** Enables agentic behavior where a server needs the LLM's help to complete a task (e.g., a Git server tool asking the LLM to write a commit message based on a diff).

**Server-Side Logic (Conceptual):**

```typescript
import { McpServerExchange } from "@modelcontextprotocol/sdk/server/index.js";
import { McpSchema } from "@modelcontextprotocol/sdk/types.js";

// Inside a tool handler or other server logic...
async function someToolHandler(input: any, exchange: McpServerExchange) {
  // 'exchange' provides access to client capabilities/requests
  if (!exchange.clientCapabilities?.sampling) {
    return {
      content: [{ type: "text", text: "Client does not support sampling." }],
      isError: true,
    };
  }

  try {
    const samplingRequest: McpSchema.CreateMessageRequest = {
      messages: [
        {
          role: "user",
          content: [
            // Content is an array
            {
              type: "text",
              text: `Analyze this data: ${JSON.stringify(input)}`,
            },
          ],
        },
      ],
      modelPreferences: { intelligencePriority: 0.7 },
      maxTokens: 500,
      // ... other params
    };
    // Send request TO the client
    const samplingResult = await exchange.createMessage(samplingRequest);

    // Process the LLM's response from samplingResult.messages
    // Assuming the response is in the first message's content
    const responseText =
      samplingResult.messages?.[0]?.content?.[0]?.text ??
      "No response text found.";

    return {
      content: [
        {
          type: "text",
          text: `Analysis complete: ${responseText}`,
        },
      ],
    };
  } catch (error: any) {
    console.error(`Sampling request failed: ${error.message}`);
    return {
      content: [
        { type: "text", text: `Failed during analysis: ${error.message}` },
      ],
      isError: true,
    };
  }
}
```

**Key Considerations:**

- Check `exchange.clientCapabilities.sampling` before calling `exchange.createMessage`.
- The client controls the actual LLM call, including model choice (guided by `modelPreferences`) and user approval.
- Handle potential errors from the sampling request.

### Roots (Client Capability)

Similar to sampling, `roots` are provided _by the client_ to the server. Servers supporting filesystem operations should check for this capability and use the `roots/list` request to understand the accessible directories.

**Server-Side Logic (Conceptual):**

```typescript
import { McpServerExchange } from "@modelcontextprotocol/sdk/server/index.js";

// Inside server initialization or a relevant handler...
async function checkRoots(exchange: McpServerExchange) {
  if (exchange.clientCapabilities?.roots) {
    try {
      const rootsResult = await exchange.listRoots();
      console.error("Client supports roots:", rootsResult.roots);
      // Use this information to constrain file operations
      // e.g., store rootsResult.roots in server state
    } catch (error) {
      console.error("Failed to list roots:", error);
    }
  } else {
    console.error("Client does not support roots.");
    // Operate without root constraints or deny file operations
  }
}
```

### Streaming Responses (Transport Feature)

The **Streamable HTTP** transport inherently supports streaming server responses via Server-Sent Events (SSE). When a server needs to send multiple messages (e.g., progress updates, multiple parts of a large result, or server-initiated requests) in response to a single client request or over a persistent connection (via GET), it uses an SSE stream.

- The SDK's transport implementation handles the mechanics of SSE.
- Your server logic decides _when_ to send multiple messages versus a single response. For long-running tools, sending progress notifications followed by a final result over an SSE stream is common.

### Progress Reporting

For long-running operations initiated by a client request (e.g., `tools/call`), the server can send `notifications/progress` messages back to the client _if_ the client included a `progressToken` in the original request's metadata.

**Client Request (Conceptual):**

```json
{
  "jsonrpc": "2.0",
  "id": 555,
  "method": "tools/call",
  "params": {
    "name": "longRunningTask",
    "arguments": {
      /* ... */
    },
    "_meta": { "progressToken": "task123" } // Client provides a token
  }
}
```

**Server Sending Progress (Conceptual):**

```typescript
import { McpServerExchange } from "@modelcontextprotocol/sdk/server/index.js";
import { McpSchema } from "@modelcontextprotocol/sdk/types.js";

// Inside the longRunningTask tool handler...
// The exact parameters available depend on the SDK's handler registration method.
// High-level helpers (like server.tool) typically provide parsed 'input',
// while lower-level handlers (like server.setRequestHandler) might provide
// the full 'request' object or just the 'exchange'.
async function longRunningTaskHandler(
  input: any, // Parsed arguments (common for high-level handlers)
  exchange: McpServerExchange, // Provides methods to interact with the client
  request?: McpSchema.CallToolRequest // Full request (common for low-level handlers)
) {
  // Access progressToken: Check request._meta first (if request is available),
  // then fall back to input._meta (if client included it there).
  const progressToken =
    request?.params._meta?.progressToken ??
    (input?._meta?.progressToken as string | undefined);

  const sendProgress = async (
    progress: number,
    total?: number,
    message?: string
  ) => {
    if (progressToken !== undefined && typeof progressToken === "string") {
      // Type check
      try {
        await exchange.sendProgress({
          progressToken,
          progress,
          total,
          message,
        });
      } catch (e) {
        console.error("Failed to send progress", e);
      }
    }
  };

  await sendProgress(0, 100, "Starting task...");
  // ... perform part 1 ...
  await sendProgress(25, 100, "Part 1 complete...");
  // ... perform part 2 ...
  await sendProgress(75, 100, "Part 2 complete...");
  // ... perform final part ...
  await sendProgress(100, 100, "Task finished.");

  // Send final result (this implicitly ends progress reporting for this token)
  return { content: [{ type: "text", text: "Task complete!" }] };
}
```

### Resource Subscriptions

If a server declares `resources: { subscribe: true }` capability, clients can send `resources/subscribe` requests for specific resource URIs. The server is then responsible for tracking these subscriptions and sending `notifications/resources/updated` when the content of a subscribed resource changes.

**Implementation Sketch:**

- Maintain a data structure mapping resource URIs to connected client sessions that subscribed (e.g., `Map<string, Set<McpServerExchange>>`).
- Monitor the underlying data source for changes (e.g., using `fs.watch` for files, database triggers, etc.).
- When a change occurs for a subscribed URI, iterate through the subscribed sessions (`Set<McpServerExchange>`) and send the `notifications/resources/updated` notification via the `exchange.sendResourceUpdated({ uri: changedUri })` method (or equivalent).
- Handle `resources/unsubscribe` requests to remove clients from the tracking structure.
- Clean up subscriptions when a client disconnects (using the transport's disconnect event).

### Completions

Servers can offer argument auto-completion for Prompts and Resource Templates by handling `completion/complete` requests.

**Note:** Unlike `tools` or `resources`, the `completions` capability isn't explicitly declared during initialization. It's an implicit capability enabled simply by implementing the handler for the `completion/complete` method.

**Implementation Sketch:**

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js"; // Use base Server for setRequestHandler
import {
  CompletionRequestSchema,
  CompletionResultSchema, // Import result schema
  McpSchema,
} from "@modelcontextprotocol/sdk/types.js";

// Assume 'server' is an initialized base Server instance

// Example: Completion for a prompt argument
server.setRequestHandler(CompletionRequestSchema, async (request, exchange) => {
  const params = request.params;
  let completionValues: string[] = [];

  // Check if it's for a prompt we know
  if (params.ref.type === "ref/prompt" && params.ref.name === "my-prompt") {
    // Check which argument is being completed
    if (params.argument.name === "targetFile") {
      const currentValue = (params.argument.value as string) || "";
      // Logic to find matching files based on currentValue
      // Replace with actual filesystem listing or other source
      completionValues = ["file1.txt", "file2.log", "another_file.txt"].filter(
        (f) => f.startsWith(currentValue)
      );
    }
  }
  // Handle other completion requests (e.g., for resource templates)

  const result: McpSchema.CompletionResult = {
    completion: {
      values: completionValues.slice(0, 100), // Max 100 results
      // total: completionValues.length, // Optional total count
      // hasMore: completionValues.length > 100 // Optional flag
    },
  };
  // Validate result (optional but good practice)
  CompletionResultSchema.parse(result);
  return result;
});
```

### Logging

Servers can send structured logs to clients using `notifications/message` if they declare the `logging` capability. Clients can optionally control the minimum level using `logging/setLevel`.

**Sending a Log Message (Conceptual):**

```typescript
import { McpServerExchange } from "@modelcontextprotocol/sdk/server/index.js";
import { McpLogLevel } from "@modelcontextprotocol/sdk/types.js"; // Import log level type

// Inside any handler where 'exchange' is available...
async function someOperation(input: any, exchange: McpServerExchange) {
  if (!exchange.clientCapabilities?.logging) {
    // Client doesn't support receiving logs via MCP
    console.error("Client does not support MCP logging.");
    // Fallback to stderr or other logging
  }

  try {
    // ... do work ...
    await exchange.sendLogMessage({
      level: McpLogLevel.Info, // Use enum or string 'info'
      logger: "MyOperationLogger", // Optional logger name
      data: { message: "Operation successful", input: input }, // Arbitrary JSON data
    });
    return {
      /* success result */
    };
  } catch (error: any) {
    await exchange.sendLogMessage({
      level: McpLogLevel.Error, // Use enum or string 'error'
      logger: "MyOperationLogger",
      data: {
        message: "Operation failed",
        error: error.message,
        // Stack traces should generally be avoided in messages sent to the client
        // unless specifically required and safe for the context. Log them server-side.
      },
    });
    return {
      /* error result */
    };
  }
}
```

### Performance Optimization

- **Caching:** Cache results of expensive operations (API calls, database queries, resource reads) where appropriate. Use time-based or event-based invalidation.
- **Concurrency:** Leverage `async/await` and Node.js's event loop to handle multiple requests concurrently without blocking. Avoid CPU-intensive synchronous tasks. Use Worker Threads for heavy computation.
- **Efficient Data Handling:** Use streams for large data transfers if the transport supports it. Process data efficiently. Send binary data as base64 `blob` in resources.
- **Transport Choice:** `stdio` is generally lower overhead than HTTP for local communication.
- **Debouncing/Throttling:** Implement server-side rate limiting for resource-intensive tools or frequent notifications.

### Dynamic Server Capabilities

You can add, remove, enable, disable, or update tools, resources, and prompts _after_ the server is connected. The `McpServer` automatically sends the necessary `listChanged` notifications when using its management methods (`.enable()`, `.disable()`, `.update()`, `.remove()`).

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { logger } from "../utils/logger.js"; // Assuming a logger utility

// Assume listMessages, putMessage, upgradeAuthAndStoreToken functions exist

const server = new McpServer({
  name: "Dynamic Example",
  version: "1.0.0",
});

// Register initial tools
const listMessageTool = server.tool(
  "listMessages",
  "Lists messages in a channel.",
  { channel: z.string().describe("Channel name") },
  // Annotations: Title, read-only
  { title: "List Channel Messages", readOnlyHint: true },
  async ({ channel }) => ({
    content: [{ type: "text", text: await listMessages(channel) }],
  })
);

const putMessageTool = server.tool(
  "putMessage",
  "Sends a message to a channel.",
  {
    channel: z.string().describe("Channel name"),
    message: z.string().min(1).describe("Message content"),
  },
  // Annotations: Title (not read-only, potentially idempotent depending on backend)
  { title: "Send Channel Message" /* idempotentHint: true (optional) */ },
  async ({ channel, message }) => ({
    content: [{ type: "text", text: await putMessage(channel, message) }],
  })
);

// Start with the 'putMessage' tool disabled
putMessageTool.disable();
logger.info(`Tool 'putMessage' initially disabled.`);

const upgradeAuthTool = server.tool(
  "upgradeAuth",
  "Upgrades authorization level.",
  {
    permission: z
      .enum(["write", "admin"])
      .describe("Permission level to upgrade to"),
  },
  // Annotations: Title (modifies state, not read-only)
  {
    title:
      "Upgrade Authorization" /* destructiveHint: false (usually not destructive) */,
  },
  // Handler receives validated arguments
  async ({ permission }) => {
    logger.info(`Attempting to upgrade auth to: ${permission}`);
    // Assume this function handles the auth logic and returns status
    const { ok, err, previousPermission } = await upgradeAuthAndStoreToken(
      permission
    );

    if (!ok) {
      logger.error(`Auth upgrade failed: ${err}`);
      return {
        content: [{ type: "text", text: `Error: ${err}` }],
        isError: true,
      };
    }

    logger.info(
      `Auth upgraded successfully from ${previousPermission} to ${permission}.`
    );

    // If we previously had read-only access (or similar), enable 'putMessage'
    if (previousPermission !== "write" && previousPermission !== "admin") {
      logger.info(`Enabling 'putMessage' tool.`);
      putMessageTool.enable(); // Automatically sends listChanged
    }

    if (permission === "write") {
      // If upgraded to 'write', update 'upgradeAuth' to only allow 'admin' next
      logger.info(`Updating 'upgradeAuth' tool to only allow 'admin' upgrade.`);
      upgradeAuthTool.update({
        // Only update the schema part
        paramSchema: {
          permission: z.enum(["admin"]).describe("Upgrade to admin permission"),
        },
      }); // Automatically sends listChanged
    } else if (permission === "admin") {
      // If upgraded to 'admin', remove the upgrade tool entirely
      logger.info(`Removing 'upgradeAuth' tool as user is now admin.`);
      upgradeAuthTool.remove(); // Automatically sends listChanged
    }

    return {
      content: [
        { type: "text", text: `Successfully upgraded auth to ${permission}` },
      ],
    };
  }
);

// Connect the server (example using Stdio)
async function startServer() {
  logger.info("Starting dynamic server example...");
  const transport = new StdioServerTransport();
  transport.onclose = () => {
    logger.info("Transport closed.");
    process.exit(0);
  };
  await server.connect(transport);
  logger.info("Server connected via Stdio.");
}

startServer();

// --- Dummy implementations for demonstration ---
async function listMessages(channel: string): Promise<string> {
  return `Messages in ${channel}`;
}
async function putMessage(channel: string, message: string): Promise<string> {
  return `Message "${message}" sent to ${channel}`;
}
async function upgradeAuthAndStoreToken(
  permission: "write" | "admin"
): Promise<{ ok: boolean; err?: string; previousPermission: string }> {
  // Simulate auth upgrade logic
  // In a real app, interact with auth system and store token
  const currentPermission = permission === "write" ? "read" : "write"; // Simulate previous state
  return { ok: true, previousPermission: currentPermission };
}
```

## 6. Security and Best Practices

Security is paramount when building MCP servers, as they often interact with sensitive data and systems. Adhere strictly to these principles, incorporating the latest [MCP Authentication Specification](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-03-26/basic/authorization.mdx):

### Authentication and Authorization

- **Transports:**
  - **Stdio:** Relies on the security context of the process execution. Ensure the client launches the server securely. Credentials might be passed via environment variables if needed, but this requires careful handling by the client and secure environment management.
  - **Streamable HTTP:** **Secure authentication is mandatory**. Implement the **MCP Authentication Specification**, which is based on **OAuth 2.0/2.1**. This includes:
    - Using secure flows like **Authorization Code with PKCE**.
    - Supporting **dynamic client registration** where applicable.
    - Enforcing scope checks based on user consent.
    - Integrating with trusted **third-party identity providers** (e.g., Auth0, Okta) is a common and recommended pattern.
    - **HTTPS Enforcement:** All HTTP-based communication **MUST** use HTTPS to ensure confidentiality and integrity. Configure your server accordingly (TLS certificates).
    - **Transport Security (Origin/CORS):** Strictly validate `Origin` headers using an allowlist (see example below). Configure CORS headers correctly. Bind to `127.0.0.1` (`localhost`) for servers intended only for local access to mitigate DNS rebinding attacks.
- **Capability Control:** Implement fine-grained authorization checks within tool/resource handlers if different clients or users should have different permissions based on their authenticated context (e.g., scopes granted via OAuth).

### Data Security

- **Input Validation & Sanitization:**
  - **Always validate and sanitize ALL inputs** from the client (tool arguments, resource URIs, prompt arguments) using robust schemas (`zod` is excellent for this).
  - Implement **context-aware semantic validation** beyond basic schema checks.
  - **Prevent Path Traversal:** If dealing with file paths, rigorously validate and normalize paths to ensure they stay within designated boundaries. Never trust client-provided paths directly. Use `path.resolve` and check if the resolved path starts with the expected base directory.
  - **Prevent Injection:** Sanitize inputs used in database queries (use parameterized queries/prepared statements), shell commands (avoid direct execution if possible, otherwise escape rigorously), or API calls.
- **Data Handling:**
  - Use TLS/HTTPS for network transports (mandatory for HTTP).
  - Encrypt sensitive data stored by the server at rest.
  - Avoid logging sensitive information. If necessary, mask or redact it. Use secure logging practices.
- **Resource Protection:**
  - Implement access controls based on authentication/authorization context.
  - **Rate Limiting:** Implement rate limiting, especially for tools interacting with external APIs or performing resource-intensive operations, to prevent Denial-of-Service (DoS) attacks and manage costs.
- **Output Sanitization:** Be cautious about the data returned in responses. Avoid leaking sensitive information, internal system details, or excessive error details that could aid attackers.

### Secure Configuration and Credentials

- **Secure Credential Handling:** Store any required credentials (e.g., API keys, database passwords) securely using system keychains, environment variables (loaded securely, e.g., via `.env` in development **only** - ensure `.env` is in `.gitignore`), or dedicated secrets managers (like HashiCorp Vault, AWS Secrets Manager, etc.). **Never expose secrets in plaintext** configuration files or logs.
- **Configuration Management:** Load configuration securely. Avoid hardcoding sensitive values.

### Error Handling

- **Be Specific but Safe:** Provide enough information for debugging but avoid leaking internal details (stack traces, sensitive paths, internal error codes) to the client in error responses. Log detailed errors server-side with context (like request IDs).
- **Tool Errors:** The recommended pattern is to return `{ isError: true, content: [...] }` for errors _within_ a tool's execution (e.g., API call failed, invalid state). Use standard JSON-RPC errors (e.g., code `-32602` for invalid params, `-32601` for method not found) for protocol-level issues. While the spec doesn't explicitly forbid a tool handler throwing an exception that results in a standard JSON-RPC error (like `-32000` Server error) for catastrophic failures before an `isError: true` response can be formed, using `isError: true` is the preferred way to signal tool-specific execution failures.
- **Resource Cleanup:** Ensure resources (file handles, network connections, DB connections) are properly closed, especially in error paths (`finally` blocks are essential).

### General Best Practices

- **Principle of Least Privilege:** Run server processes with the minimum permissions necessary. Design tools and resources to operate with the least privilege required.
- **Dependency Security:** Regularly audit dependencies (`npm audit`, `yarn audit`) and keep them updated to patch known vulnerabilities. Use lockfiles (`package-lock.json`, `yarn.lock`).
- **Code Quality:** Write clean, maintainable, and well-tested code. Simpler code is often more secure. Use linters and formatters.
- **Clear Documentation:** Document the server's purpose, capabilities, required configuration, and security considerations.
- **Testing:** Include tests for security vulnerabilities (invalid inputs, path traversal attempts, permission errors, injection attempts).
- **Annotations:** Use tool annotations (`readOnlyHint`, `destructiveHint`, etc.) accurately, but remember clients **MUST NOT** rely solely on these for security decisions. They are untrusted hints.
- **Strict Protocol Validation:** Ensure the server rigorously validates incoming MCP messages against the protocol specification (structure, field consistency, recursion depth) to prevent malformed request attacks. The SDK handles much of this, but be mindful in custom logic.

### Example: Basic Origin Check Middleware (Express)

This middleware helps enforce origin restrictions for Streamable HTTP servers.

```typescript
import { Request, Response, NextFunction } from "express";
import { logger } from "../utils/logger.js"; // Assuming a logger utility
import { config } from "../config/index.js"; // Assuming config loads env vars

// Load allowed origins from configuration (e.g., environment variable)
// Example: MCP_ALLOWED_ORIGINS="http://localhost:8080,https://my-trusted-client.com"
const ALLOWED_ORIGINS = new Set(
  (config.mcpAllowedOrigins || "").split(",").filter(Boolean)
);

export function originCheckMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  const origin = req.headers.origin;
  // Use req.socket.localAddress for a more reliable check of the binding address
  const serverBoundAddress = req.socket.localAddress;
  const isLocalhostBinding = ["127.0.0.1", "::1"].includes(serverBoundAddress);

  // Determine if the origin is allowed:
  // 1. Origin header is present AND is in the configured allowlist.
  // 2. Server is bound ONLY to localhost AND Origin header is missing OR 'null'
  //    (common for Electron/local file:// - accept ONLY if strictly localhost-bound).
  const isOriginAllowed =
    (origin && ALLOWED_ORIGINS.has(origin)) ||
    (isLocalhostBinding && (!origin || origin === "null"));

  if (isOriginAllowed) {
    logger.debug(
      `Origin allowed: ${
        origin || "(missing/null)"
      } for server bound to ${serverBoundAddress}`
    );
    if (origin) {
      // Set CORS header ONLY if origin is present and allowed
      res.setHeader("Access-Control-Allow-Origin", origin);
      // IMPORTANT: Add other necessary CORS headers
      res.setHeader(
        "Access-Control-Allow-Methods",
        "GET, POST, DELETE, OPTIONS" // Adjust as needed for MCP endpoint
      );
      res.setHeader(
        "Access-Control-Allow-Headers",
        "Content-Type, Mcp-Session-Id, Last-Event-ID, Authorization" // Include MCP headers + Auth
      );
      // res.setHeader('Access-Control-Allow-Credentials', 'true'); // Only if using cookies/credentials

      // Add other useful security headers
      res.setHeader("X-Content-Type-Options", "nosniff"); // Prevent MIME type sniffing
      res.setHeader("Referrer-Policy", "strict-origin-when-cross-origin"); // Control referrer info
      // Consider adding: 'Content-Security-Policy', 'Strict-Transport-Security' (HSTS) if applicable
    }
    // Handle OPTIONS preflight requests specifically for CORS
    if (req.method === "OPTIONS") {
      res.sendStatus(204); // No Content - Standard practice for preflight
      return;
    }
    next(); // Proceed to the next middleware/handler
  } else {
    logger.warn(`Forbidden: Origin not allowed: ${origin}`, {
      serverBoundAddress,
      allowed: [...ALLOWED_ORIGINS],
    });
    res.status(403).send("Forbidden: Invalid Origin");
  }
}

// --- Usage in Express app ---
// import express from 'express';
// const app = express();
//
// // Apply security middleware EARLY
// app.use(originCheckMiddleware);
// // Apply authentication middleware next (verifies tokens, sets user context)
// // app.use(mcpAuthMiddleware);
//
// // Then parse JSON body if needed (though SDK transports might handle raw body)
// app.use(express.json());
//
// // ... rest of your MCP routes (likely handled by SDK transport) ...
//
// app.listen(config.mcpHttpPort, config.mcpHttpHost, () => {
//   logger.info(`Server listening on ${config.mcpHttpHost}:${config.mcpHttpPort}`);
// });
```

**Note on Utility Modules:** Features like rate limiting, input sanitization beyond basic Zod validation, and complex authentication logic are often best implemented in dedicated utility modules (e.g., `src/utils/security/rateLimiter.ts`, `src/utils/auth/tokenValidator.ts`) and integrated as middleware or called directly where needed.

## 7. Troubleshooting and Resources

### Debugging Tools

- **MCP Inspector:** ([GitHub](https://github.com/modelcontextprotocol/inspector)) Essential for directly interacting with your server (especially stdio) outside a full client application. Launch it with your server command: `npx @modelcontextprotocol/inspector node build/index.js`. Use it to send requests manually and inspect responses, verifying your server's behavior and capabilities.
- **Client Logs (e.g., Claude Desktop):** Check the host application's logs for connection errors, messages sent/received, and server stderr output. These logs are crucial for understanding client-server interaction issues.
- **Server Logging:** Implement robust logging within your server. See [Implementing Logging Effectively](#implementing-logging-effectively) below.
- **Node.js Debugger:** Use standard Node.js debugging techniques (e.g., VS Code debugger, `node --inspect`) to step through your server code.

### Viewing Logs (Example for Claude Desktop - check other client documentation for their log locations):

- **macOS:** `tail -n 50 -F ~/Library/Logs/Claude/mcp*.log`
- **Windows:** Check `%APPDATA%\Claude\logs\mcp*.log` (use PowerShell `Get-Content -Path "$env:APPDATA\Claude\logs\mcp*.log" -Wait -Tail 50` for live view)
- Look for `mcp.log` (general protocol messages) and `mcp-server-SERVERNAME.log` (stderr output from your specific server).

### Common Issues and Solutions

| Issue                               | Possible Cause(s)                                                                            | Potential Solution(s)                                                                                                |
| :---------------------------------- | :------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| Server Not Starting/Connecting      | Invalid path in client config, permissions error, startup script error, port conflict (HTTP) | Verify absolute path & permissions, run server directly in terminal, check client logs, check for port conflicts.    |
| Incorrect Working Directory         | Client launches server in unexpected directory (e.g., Claude Desktop)                        | Use absolute paths, resolve paths relative to `import.meta.url` or known base directory.                             |
| Environment Variables Missing       | Client doesn't pass full environment                                                         | Configure required env vars in client's server config if possible, use config files.                                 |
| JSON-RPC: `-32700 Parse error`      | Invalid JSON sent                                                                            | Check message formatting, ensure valid JSON.                                                                         |
| JSON-RPC: `-32600 Invalid Request`  | Malformed request object                                                                     | Verify request structure against JSON-RPC 2.0 spec.                                                                  |
| JSON-RPC: `-32601 Method not found` | Method name mismatch, capability not declared/registered                                     | Check method spelling, verify server capability declaration and handler registration.                                |
| JSON-RPC: `-32602 Invalid params`   | Input doesn't match defined schema (`zod`)                                                   | Check client arguments against server's schema definition, verify schema correctness.                                |
| JSON-RPC: `-32603 Internal error`   | Unhandled exception on server-side                                                           | Check detailed server logs for stack traces and error messages.                                                      |
| Tool Calls Failing                  | Server-side error, external dependency issue, invalid input                                  | Check server logs, verify external service status, ensure input matches schema.                                      |
| Resource Access Denied              | Path traversal attempt, filesystem permissions, `roots` mismatch                             | Verify path validation logic, check server process permissions, ensure path is within allowed `roots` if applicable. |

### Implementing Logging Effectively

- **Use `console.error` for Stdio:** Simple, reliable, and captured by most clients in their server-specific logs (e.g., `mcp-server-SERVERNAME.log`). Ideal for essential operational logs and errors during development.
- **Use MCP Logging (`exchange.sendLogMessage`):** For structured logs intended to be potentially visible or filterable by the client application itself (if it supports the `logging` capability). Choose appropriate levels (`debug`, `info`, `warning`, `error`, etc.). This is useful for communicating application-level events or diagnostics back to the client context.
- **Log Key Events:** Initialization, connection establishment/closure, request received/processed, response sent, errors encountered (both internal and protocol-level), significant state changes, external API calls (request/response summaries).
- **Include Context:** Log relevant data like request IDs (if available from the request or generated server-side), method names, user identifiers (if applicable and safe), session IDs (for HTTP), and parameters (masking sensitive parts like passwords or API keys).
- **Structured Logging (Server-Side):** For more advanced server-side analysis (especially for HTTP servers or complex stdio servers), use libraries like `pino` or `winston`. Outputting structured JSON to `stderr` (for stdio) or a dedicated log file (for HTTP) makes logs easier to parse and analyze with external tools.
- **Request Context / Tracing (Advanced):** For complex servers handling concurrent requests (like HTTP), consider using Node.js `AsyncLocalStorage` to create a context per request. This allows associating logs, metrics, and tracing information (e.g., a unique request ID) throughout the lifecycle of that specific operation, greatly aiding debugging in concurrent environments.

### Community Resources and Support

- **Official Documentation:** [modelcontextprotocol.io](https://modelcontextprotocol.io/)
- **GitHub Organization:** [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol) (SDKs, Specification, Inspector, Servers)
- **GitHub Discussions:** For Q&A and community interaction ([Spec Discussions](https://github.com/modelcontextprotocol/specification/discussions), [Org Discussions](https://github.com/orgs/modelcontextprotocol/discussions)).
- **GitHub Issues:** For bug reports and feature requests on specific repositories.

## 8. Example Implementations

These examples illustrate combining different concepts using the high-level TypeScript SDK (`McpServer`).

### Echo Server

Demonstrates basic resources, tools, and prompts.

```typescript
// src/echo-server.ts
import {
  McpServer,
  ResourceTemplate,
} from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { Role } from "@modelcontextprotocol/sdk/types.js";
import { logger } from "../utils/logger.js"; // Assuming a logger utility

async function startEchoServer() {
  const server = new McpServer(
    { name: "EchoServer", version: "1.0.0" },
    {
      // Declare capabilities explicitly for clarity
      capabilities: {
        resources: { listChanged: false },
        tools: { listChanged: false },
        prompts: { listChanged: false },
      },
    }
  );

  // Echo Resource
  server.resource(
    "echo-resource", // Registration name
    new ResourceTemplate("echo://{message}", { list: undefined }), // URI Template
    async (uri, { message }) => {
      // Handler
      logger.debug(`Echo resource called: ${uri.href}`);
      return {
        contents: [
          {
            uri: uri.href,
            mimeType: "text/plain",
            text: `Resource echo: ${message}`,
          },
        ],
      };
    }
  );
  logger.info("Registered resource template: echo://{message}");

  // Echo Tool
  server.tool(
    "echo-tool", // Tool name
    "Echoes back the provided message via a tool.", // Description
    { message: z.string().min(1).describe("Message to echo") }, // Input schema
    // Annotations: Title, read-only
    { title: "Echo Message (Tool)", readOnlyHint: true },
    async ({ message }) => {
      // Handler
      logger.debug(`Echo tool called with message: ${message}`);
      return { content: [{ type: "text", text: `Tool echo: ${message}` }] };
    }
  );
  logger.info("Registered tool: echo-tool");

  // Echo Prompt
  server.prompt(
    "echo-prompt", // Prompt name
    "Generates a prompt containing the message.", // Description
    { message: z.string().min(1).describe("Message for the prompt") }, // Argument schema
    ({ message }) => {
      // Handler
      logger.debug(`Echo prompt generated for message: ${message}`);
      return {
        messages: [
          {
            role: Role.USER,
            content: [
              { type: "text", text: `Please process this message: ${message}` },
            ],
          },
        ],
      };
    }
  );
  logger.info("Registered prompt: echo-prompt");

  logger.info("Echo server configured.");
  const transport = new StdioServerTransport();
  transport.onclose = () => {
    logger.info("Echo server transport closed.");
    process.exit(0);
  };
  await server.connect(transport);
  logger.info("Echo server connected via Stdio.");
}

startEchoServer().catch((error) => {
  logger.error("Echo server failed to start:", error);
  process.exit(1);
});
```

### SQLite Explorer

Demonstrates integrating with a database, providing schema as a resource and query execution as a tool.

```typescript
// src/sqlite-explorer.ts
import {
  McpServer,
  ResourceTemplate,
} from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import sqlite3 from "sqlite3";
import { open, Database } from "sqlite"; // Use the promise-based wrapper
import { z } from "zod";
import path from "path";
import fs from "fs/promises"; // Import fs promises
import { fileURLToPath } from "url";
import { logger } from "../utils/logger.js"; // Assuming a logger utility

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
// Example: Store DB in a 'data' directory sibling to 'src'
const DB_PATH = path.resolve(__dirname, "../data/database.db");
const DATA_DIR = path.dirname(DB_PATH);

async function initializeDatabase(dbPath: string): Promise<void> {
  await fs.mkdir(path.dirname(dbPath), { recursive: true });
  const db = await open({ filename: dbPath, driver: sqlite3.Database });
  try {
    // Example schema
    await db.exec(`
      CREATE TABLE IF NOT EXISTS items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        value REAL,
        createdAt DATETIME DEFAULT CURRENT_TIMESTAMP
      );
      CREATE TABLE IF NOT EXISTS users (
        userId TEXT PRIMARY KEY,
        displayName TEXT,
        email TEXT UNIQUE
      );
    `);
    // Optional: Seed with some data if empty
    const itemCount = await db.get<{ count: number }>(
      "SELECT COUNT(*) as count FROM items"
    );
    if (itemCount?.count === 0) {
      await db.run("INSERT INTO items (name, value) VALUES (?, ?)", [
        "Sample Item 1",
        123.45,
      ]);
      await db.run("INSERT INTO items (name, value) VALUES (?, ?)", [
        "Sample Item 2",
        67.89,
      ]);
      logger.info("Database seeded with initial data.");
    }
  } finally {
    await db.close();
  }
}

async function startSQLiteServer() {
  try {
    await initializeDatabase(DB_PATH);
    logger.info(`Database ${DB_PATH} initialized/verified.`);
  } catch (error) {
    logger.error(`Failed to initialize database ${DB_PATH}:`, error);
    process.exit(1); // Exit if DB setup fails
  }

  const server = new McpServer(
    { name: "SQLite Explorer", version: "1.0.0" },
    {
      capabilities: {
        resources: { listChanged: false }, // Schema doesn't change dynamically
        tools: { listChanged: false },
      },
    }
  );

  // Helper to open DB connection (read-only recommended for safety)
  const getDb = async (): Promise<Database> => {
    return open({
      filename: DB_PATH,
      driver: sqlite3.Database,
      mode: sqlite3.OPEN_READONLY, // Open in read-only mode for safety
    });
  };

  // Resource: Database Schema
  server.resource(
    "db-schema", // Resource group name
    "db://schema", // Static URI for the schema
    async (uri) => {
      logger.debug(`Reading schema resource: ${uri.href}`);
      let db: Database | null = null;
      try {
        db = await getDb();
        const tables = await db.all<{ name: string; sql: string }>(
          "SELECT name, sql FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%';"
        );
        const schemaText = tables
          .map((t) => `-- Table: ${t.name}\n${t.sql};`)
          .join("\n\n");
        return {
          contents: [
            {
              uri: uri.href,
              mimeType: "application/sql", // More specific MIME type
              text: schemaText || "-- No tables found --",
            },
          ],
        };
      } catch (error: any) {
        logger.error(`Schema retrieval error: ${error.message}`);
        throw new Error(`Failed to retrieve schema: ${error.message}`);
      } finally {
        await db?.close();
      }
    }
  );
  logger.info("Registered resource: db://schema");

  // Tool: Execute Read-Only SQL Query
  const querySchema = z.object({
    sql: z
      .string()
      .trim()
      .min(1)
      .refine((s) => s.toLowerCase().startsWith("select"), {
        message: "Only SELECT queries are allowed.", // Zod refinement for basic check
      })
      .describe("The read-only SQL SELECT query to execute"),
  });

  server.tool(
    "sqlQuery", // Tool name
    "Executes a read-only SQL SELECT query against the database.", // Description
    querySchema, // Input schema with refinement
    { readOnlyHint: true, title: "Run SQL Query" }, // Annotations
    async ({ sql }) => {
      logger.debug(`Executing query tool: ${sql}`);
      // Additional check (belt-and-suspenders) - Zod refine should catch this
      if (!/^\s*SELECT/i.test(sql)) {
        logger.warn(`Query tool rejected non-SELECT statement: ${sql}`);
        return {
          content: [
            { type: "text", text: "Error: Only SELECT queries are allowed." },
          ],
          isError: true,
        };
      }

      let db: Database | null = null;
      try {
        db = await getDb();
        // Use prepared statements if parameters were involved for security.
        // Since the query comes directly, we execute it but rely on read-only mode.
        const results = await Promise.race([
          db.all(sql),
          new Promise((_, reject) =>
            setTimeout(() => reject(new Error("Query timeout")), 5000)
          ), // 5s timeout
        ]);

        // Limit result size for display
        const MAX_RESULTS = 50;
        const truncated = results.length > MAX_RESULTS;
        const resultText =
          JSON.stringify(results.slice(0, MAX_RESULTS), null, 2) +
          (truncated
            ? `\n\n... (${results.length} total rows, truncated)`
            : "");

        return {
          content: [{ type: "text", text: resultText }],
        };
      } catch (err: unknown) {
        const error = err as Error;
        logger.error(`Query tool error: ${error.message}`, { sql });
        return {
          content: [
            { type: "text", text: `Error executing query: ${error.message}` },
          ],
          isError: true,
        };
      } finally {
        await db?.close();
      }
    }
  );
  logger.info("Registered tool: sqlQuery");

  // --- Connection ---
  logger.info("SQLite server configured.");
  const transport = new StdioServerTransport();
  transport.onclose = () => {
    logger.info("SQLite server transport closed.");
    process.exit(0);
  };
  await server.connect(transport);
  logger.info("SQLite server connected via Stdio.");
}

startSQLiteServer().catch((error) => {
  logger.error("SQLite server failed to start:", error);
  process.exit(1);
});
```

### Low-Level Server Implementation (Manual Handlers)

This shows using the base `Server` class for finer control over request handling, bypassing the `McpServer` helpers. This approach is less common when using the TypeScript SDK but demonstrates the underlying mechanism.

```typescript
// src/low-level-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js"; // Note: different import
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  ListPromptsRequestSchema, // Import specific request schemas
  GetPromptRequestSchema,
  ListPromptsResultSchema, // Import result schemas if needed for validation
  GetPromptResultSchema,
  McpSchema, // Import base schema types
} from "@modelcontextprotocol/sdk/types.js";
import { logger } from "../utils/logger.js"; // Assuming a logger utility

async function main() {
  const server = new Server(
    {
      // Server Info
      name: "low-level-example",
      version: "1.0.0",
    },
    {
      // Server Options
      capabilities: {
        prompts: { listChanged: false }, // Declare supported capabilities
      },
    }
  );

  // Manually register a handler for 'prompts/list'
  server.setRequestHandler(
    ListPromptsRequestSchema,
    async (request, exchange) => {
      logger.debug(`Handling request: ${request.method}`);
      // Implementation for listing prompts
      const result: McpSchema.ListPromptsResult = {
        prompts: [
          {
            name: "example-prompt",
            description: "An example prompt template (low-level)",
            arguments: [
              // Use arguments array structure
              { name: "arg1", description: "Example argument", required: true },
            ],
          },
        ],
        // nextCursor: undefined // No pagination in this example
      };
      // When using low-level handlers (setRequestHandler), manually validating
      // the result against the schema (e.g., ListPromptsResultSchema.parse(result))
      // before returning is recommended to ensure protocol compliance.
      // High-level SDK helpers (like server.prompt) handle response validation.
      ListPromptsResultSchema.parse(result); // Example validation
      return result;
    }
  );
  logger.info("Registered low-level handler for: prompts/list");

  // Manually register a handler for 'prompts/get'
  server.setRequestHandler(
    GetPromptRequestSchema,
    async (request, exchange) => {
      logger.debug(
        `Handling request: ${request.method} for prompt: ${request.params.name}`
      );
      if (request.params.name !== "example-prompt") {
        // Throwing an error generates a JSON-RPC error response
        throw new Error(`Unknown prompt: ${request.params.name}`);
      }
      // Note: With low-level handlers, argument validation is manual
      const arg1Value = request.params.arguments?.arg1 ?? "[arg1 not provided]";
      // Add manual validation if needed: if (typeof arg1Value !== 'string') { ... }

      const result: McpSchema.GetPromptResult = {
        description: "Example prompt (low-level)",
        messages: [
          {
            role: "user",
            content: [
              // Content is an array
              {
                type: "text",
                text: `Low-level prompt text with arg1: ${arg1Value}`,
              },
            ],
          },
        ],
      };
      // Manually validate the result against the schema before returning.
      GetPromptResultSchema.parse(result); // Example validation
      return result;
    }
  );
  logger.info("Registered low-level handler for: prompts/get");

  // --- Connection ---
  logger.info("Low-level server configured.");
  const transport = new StdioServerTransport();
  transport.onclose = () => {
    logger.info("Low-level server transport closed.");
    process.exit(0);
  };
  await server.connect(transport);
  logger.info("Low-Level MCP Server running via stdio.");
}

main().catch((error) => {
  logger.error("Low-level server failed to start:", error);
  process.exit(1);
});
```
