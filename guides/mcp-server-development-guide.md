# Model Context Protocol (MCP) Server Development Guide: Building Powerful Tools for LLMs

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/TypeScript-1.10.2-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/Python-1.6.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/Kotlin-0.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![MCP SDK - Java](https://img.shields.io/badge/Java-0.4.0-blue.svg)](https://github.com/modelcontextprotocol/java-sdk)
[![MCP SDK - C#](https://img.shields.io/badge/C%23-0.0.0-blue.svg)](https://github.com/modelcontextprotocol/csharp-sdk)
[![Guide Last Updated](https://img.shields.io/badge/Last%20Updated-April%202025-brightgreen.svg)]()

## Table of Contents

1.  [Introduction to MCP Servers](#1-introduction-to-mcp-servers)
2.  [Core Server Architecture](#2-core-server-architecture)
3.  [Building Your First MCP Server (TypeScript)](#3-building-your-first-mcp-server-typescript)
4.  [Exposing Capabilities](#4-exposing-capabilities)
    - [Defining and Implementing Tools](#defining-and-implementing-tools)
    - [Managing Resources](#managing-resources)
    - [Creating and Sharing Prompts](#creating-and-sharing-prompts)
5.  [Advanced Server Features](#5-advanced-server-features)
    - [Sampling (Client Capability)](#sampling-client-capability)
    - [Roots (Client Capability)](#roots-client-capability)
    - [Streaming Responses (Transport Feature)](#streaming-responses-transport-feature)
    - [Progress Reporting](#progress-reporting)
    - [Resource Subscriptions](#resource-subscriptions)
    - [Completions](#completions)
    - [Logging](#logging)
    - [Performance Optimization](#performance-optimization)
6.  [Security and Best Practices](#6-security-and-best-practices)
7.  [Troubleshooting and Resources](#7-troubleshooting-and-resources)
8.  [Example Implementations](#8-example-implementations)

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

- **Hosts:** Applications like Claude Desktop, VS Code extensions (e.g., Copilot, Continue), or custom applications that manage MCP clients and interact with the user/LLM.
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
    - _Crucially, no other requests (except potentially `ping` or server `logging`) should be sent before initialization is complete._

2.  **Message Exchange:** After initialization, clients and servers exchange messages based on negotiated capabilities:

    - **Request-Response:** For operations like `tools/call`, `resources/read`, `prompts/get`, `sampling/createMessage`, etc.
    - **Notifications:** For updates like `listChanged`, `progress`, `cancelled`, `logging`, etc.

3.  **Termination:** The connection ends when:
    - The transport is closed (e.g., client closes stdin for stdio, HTTP connection drops).
    - An unrecoverable error occurs.
    - Explicit shutdown logic is triggered (though MCP doesn't define a specific `shutdown` message; it relies on transport closure).

### Message Format and Transport

MCP uses **JSON-RPC 2.0**. Key message types:

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
    - **Client POST:** Sends JSON-RPC request(s)/notification(s)/response(s). Server responds with 202 (for notifications/responses) or initiates a response stream (SSE or single JSON) for requests.
    - **Client GET:** Initiates an SSE stream for server-initiated messages (requests/notifications).
    - **SSE:** Server can send multiple JSON-RPC messages (requests, notifications, responses) over an SSE stream.
    - **Security:** Requires careful handling of `Origin` headers, binding to `localhost` for local servers, and authentication.
    - **Session Management:** Supports optional `Mcp-Session-Id` header for stateful sessions.

    _Example (Server - Basic Express Setup):_

    ```typescript
    import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
    import { StreamableHttpServerTransport } from "@modelcontextprotocol/sdk/server/streamable-http.js"; // Assuming this class exists or similar helper
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
      // Middleware for raw body parsing needed by some transport helpers
      app.use(express.raw({ type: "*/*" }));

      const transport = new StreamableHttpServerTransport(server); // Conceptual

      // Define the single MCP endpoint
      app.all("/mcp", (req: Request, res: Response) => {
        // The transport handler would differentiate GET/POST/DELETE
        // and manage streams/sessions. This requires a specific
        // implementation from the SDK or a custom one.
        transport.handleHttpRequest(req, res); // Conceptual method
      });

      const port = 3000;
      app.listen(port, "127.0.0.1", () => {
        // Bind to localhost for security
        console.log(`MCP Server listening on http://127.0.0.1:${port}/mcp`);
      });
    }

    startServer().catch((err) => {
      console.error("Server failed to start:", err);
      process.exit(1);
    });
    ```

    _(Note: The exact implementation details for `StreamableHttpServerTransport` would depend on the SDK's specific helpers for frameworks like Express/Koa/Node HTTP)._

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
        "@modelcontextprotocol/sdk": "^latest", // Use actual latest version
        "zod": "^latest" // Use actual latest version
      },
      "devDependencies": {
        "@types/node": "^latest", // Use actual latest version
        "typescript": "^latest" // Use actual latest version
      }
    }
    ```
6.  **Create Source File:**
    ```bash
    mkdir src
    touch src/index.ts
    ```

### Choosing an SDK

We are using `@modelcontextprotocol/sdk` for TypeScript. It simplifies handling the protocol details.

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

**Key Features & Structure:** (As described in Section 2) - Name, Description, Input Schema (use `zod`), Annotations (`title`, `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`).

**Implementation Example (using `McpServer` helper):**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";
import { ToolAnnotation } from "@modelcontextprotocol/sdk/types.js"; // Import ToolAnnotation type

// Assume 'server' is an initialized McpServer instance

const apiLookupSchema = z.object({
  query: z.string().describe("Search query for the external API"),
  maxResults: z
    .number()
    .int()
    .positive()
    .optional()
    .default(5)
    .describe("Maximum results to return"),
});

const apiLookupAnnotations: ToolAnnotation = {
  title: "External API Lookup",
  readOnlyHint: true, // This tool only reads data
  openWorldHint: true, // Interacts with an external system
};

server.tool(
  "externalApiLookup", // Tool name
  apiLookupSchema, // Input schema
  async (input) => {
    // Async handler function
    console.error(`Performing API lookup for query: ${input.query}`);
    try {
      // Simulate API call
      const results = await fetch(
        `https://api.example.com/search?q=${encodeURIComponent(
          input.query
        )}&limit=${input.maxResults}`
      ).then((res) => {
        if (!res.ok) throw new Error(`API Error: ${res.statusText}`);
        return res.json();
      });

      return {
        content: [{ type: "text", text: JSON.stringify(results) }],
      };
    } catch (error: any) {
      console.error(`API Lookup failed: ${error.message}`);
      return {
        content: [
          { type: "text", text: `Failed to perform lookup: ${error.message}` },
        ],
        isError: true, // Indicate tool execution error
      };
    }
  },
  apiLookupAnnotations // Pass annotations
);
```

**Best Practices:**

- Clear names/descriptions.
- Use `zod` for robust schema validation.
- Handle errors gracefully within the tool (return `isError: true`).
- Keep tools focused; complex operations might be multiple tools.
- Use annotations to provide hints about behavior.

**Security:**

- **Validate inputs rigorously.**
- Implement access control if the tool accesses sensitive resources.
- Sanitize outputs if they include external data.
- Be mindful of rate limits on external APIs.

### Managing Resources

Resources expose data/content to the client/LLM. They are **application-controlled**.

**Key Features & Structure:** (As described in Section 2) - URI, Name, Description, MimeType, Size. Text/Binary content. Discovery (List, Templates). Reading (`resources/read`). Updates (List Changes, Subscriptions).

**Implementation Example (File Resource using `McpServer` helper):**

```typescript
import {
  McpServer,
  ResourceTemplate,
} from "@modelcontextprotocol/sdk/server/mcp.js";
import fs from "fs/promises";
import path from "path";
import { fileURLToPath } from "url"; // For ES Modules __dirname equivalent

// Assume 'server' is an initialized McpServer instance

// Get directory relative to the current module file
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const DATA_DIR = path.resolve(__dirname, "../data"); // Example: data dir sibling to src

// Ensure data directory exists
await fs.mkdir(DATA_DIR, { recursive: true });

// Resource Template for files in DATA_DIR
server.resource(
  "data-files", // Resource group name
  new ResourceTemplate("file:///data/{filename}", {
    // URI Template
    // List function: returns available resources matching the template
    list: async () => {
      try {
        const files = await fs.readdir(DATA_DIR);
        const resourceList = await Promise.all(
          files.map(async (file) => {
            const filePath = path.join(DATA_DIR, file);
            const stats = await fs.stat(filePath);
            if (stats.isFile()) {
              return {
                uri: `file:///data/${file}`,
                name: file,
                // Determine mimeType based on extension if needed
                mimeType: "application/octet-stream",
                size: stats.size,
              };
            }
            return null;
          })
        );
        return resourceList.filter((r) => r !== null) as McpSchema.Resource[]; // Type assertion
      } catch (error) {
        console.error(`Error listing data files: ${error}`);
        return [];
      }
    },
  }),
  // Read function: handles 'resources/read' for URIs matching the template
  async (uri, params) => {
    // params contains matched template variables, e.g., { filename: '...' }
    const filename = params.filename as string; // Type assertion
    if (!filename || typeof filename !== "string") {
      throw new Error("Invalid filename parameter");
    }
    const requestedPath = path.join(DATA_DIR, filename);

    // **CRITICAL SECURITY CHECK:** Prevent path traversal
    const resolvedDataDir = path.resolve(DATA_DIR);
    const resolvedRequestedPath = path.resolve(requestedPath);
    if (!resolvedRequestedPath.startsWith(resolvedDataDir)) {
      console.error(`Access denied: Path traversal attempt: ${requestedPath}`);
      throw new Error("Access denied: Invalid path");
    }

    try {
      const fileContents = await fs.readFile(requestedPath, "utf-8"); // Assuming text
      const stats = await fs.stat(requestedPath);
      return {
        contents: [
          {
            uri: uri.href, // Use the full URI from the request
            mimeType: "text/plain", // Or determine dynamically
            text: fileContents,
            size: stats.size,
          },
        ],
      };
    } catch (error: any) {
      if (error.code === "ENOENT") {
        throw new Error(`Resource not found: ${uri.href}`); // More specific error
      }
      console.error(`Error reading file ${requestedPath}: ${error}`);
      throw new Error(`Error reading resource: ${error.message}`);
    }
  }
);
```

**Best Practices:**

- Use clear URIs and names.
- Provide accurate `mimeType` and `size` if possible.
- Implement `listChanged` notifications if the resource list is dynamic.
- Implement `subscribe` for resources that change frequently.
- **Crucially:** Sanitize and validate paths to prevent directory traversal attacks.

**Security:**

- Validate URIs rigorously.
- Implement access control based on URI or other factors.
- Ensure path validation prevents access outside allowed directories.

### Creating and Sharing Prompts

Prompts are pre-defined templates for user interactions, often surfaced as commands. They are **user-controlled**.

**Key Features & Structure:** (As described in Section 2) - Name, Description, Arguments (with schema). Can embed resources. Discovery (`prompts/list`). Usage (`prompts/get`).

**Implementation Example (using `McpServer` helper):**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

// Assume 'server' is an initialized McpServer instance

const summarizeInputSchema = z.object({
  textToSummarize: z
    .string()
    .min(10)
    .describe("The text content to be summarized"),
  maxLength: z
    .number()
    .int()
    .positive()
    .optional()
    .describe("Optional maximum length for the summary"),
});

server.prompt(
  "summarize-text", // Prompt name (e.g., used for slash command)
  summarizeInputSchema, // Input schema
  (input) => {
    // Handler function
    let promptText = `Please summarize the following text concisely:\n\n"${input.textToSummarize}"`;
    if (input.maxLength) {
      promptText += `\n\nThe summary should be no more than ${input.maxLength} words.`;
    }

    // Return the message(s) to be sent to the LLM
    return {
      // description: "Summarizes the provided text", // Optional override
      messages: [
        {
          role: "user",
          content: { type: "text", text: promptText },
        },
      ],
    };
  }
);
```

**Best Practices:**

- Use intuitive names (often mapping to UI commands).
- Provide clear descriptions for prompts and arguments.
- Validate arguments using schemas (`zod`).
- Handle optional arguments gracefully.

**Security:**

- Validate and sanitize all arguments, especially if they are incorporated directly into system interactions or external calls triggered by the prompt's result.
- Be cautious if prompts embed resources; ensure resource access control is respected.

### Schema Validation and Documentation

- **Use Schemas:** Consistently use JSON Schema (or libraries like `zod` which generate it) to define the `inputSchema` for Tools and the structure of `arguments` for Prompts. This is crucial for interoperability and validation.
- **Documentation:** Provide clear `description` fields for all capabilities (Tools, Resources, Prompts) and their parameters/arguments. This helps both humans and LLMs understand how to use them correctly. Consider adding examples within descriptions where helpful.

## 5. Advanced Server Features

Beyond the core capabilities, MCP includes features for more sophisticated scenarios.

### Sampling (Client Capability)

While sampling (`sampling/createMessage`) is a request _sent by the server_, it relies on the _client_ supporting the `sampling` capability. Servers don't implement sampling handling themselves; they _initiate_ sampling requests if the connected client supports it.

**Use Case:** Enables agentic behavior where a server needs the LLM's help to complete a task (e.g., a Git server tool asking the LLM to write a commit message based on a diff).

**Server-Side Logic (Conceptual):**

```typescript
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
          content: {
            type: "text",
            text: `Analyze this data: ${JSON.stringify(input)}`,
          },
        },
      ],
      modelPreferences: { intelligencePriority: 0.7 },
      maxTokens: 500,
      // ... other params
    };
    // Send request TO the client
    const samplingResult = await exchange.createMessage(samplingRequest);

    // Process the LLM's response from samplingResult.content
    return {
      content: [
        {
          type: "text",
          text: `Analysis complete: ${samplingResult.content.text}`,
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
// Inside server initialization or a relevant handler...
async function checkRoots(exchange: McpServerExchange) {
  if (exchange.clientCapabilities?.roots) {
    try {
      const rootsResult = await exchange.listRoots();
      console.error("Client supports roots:", rootsResult.roots);
      // Use this information to constrain file operations
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
// Inside the longRunningTask tool handler...
async function longRunningTaskHandler(
  input: any,
  exchange: McpServerExchange,
  request: McpSchema.CallToolRequest
) {
  const progressToken = request.params._meta?.progressToken;

  const sendProgress = async (
    progress: number,
    total?: number,
    message?: string
  ) => {
    if (progressToken !== undefined) {
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

- Maintain a data structure mapping resource URIs to connected client sessions that subscribed.
- Monitor the underlying data source for changes.
- When a change occurs for a subscribed URI, iterate through the subscribed sessions and send the `notifications/resources/updated` notification via the `exchange.sendResourceUpdated()` method (or equivalent).
- Handle `resources/unsubscribe` requests to remove clients from the tracking structure.
- Clean up subscriptions when a client disconnects.

### Completions

Servers can offer argument auto-completion for Prompts and Resource Templates by declaring the `completions` capability and handling `completion/complete` requests.

**Implementation Sketch:**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";
import {
  CompletionRequestSchema,
  PromptReferenceSchema,
} from "@modelcontextprotocol/sdk/types.js"; // Import relevant types

// Assume 'server' is an initialized McpServer instance with completions capability

// Example: Completion for a prompt argument
server.setRequestHandler(CompletionRequestSchema, async (request, exchange) => {
  const params = request.params;

  // Check if it's for a prompt we know
  if (params.ref.type === "ref/prompt" && params.ref.name === "my-prompt") {
    // Check which argument is being completed
    if (params.argument.name === "targetFile") {
      const currentValue = (params.argument.value as string) || "";
      // Logic to find matching files based on currentValue
      const matchingFiles = [
        "file1.txt",
        "file2.log",
        "another_file.txt",
      ].filter((f) => f.startsWith(currentValue));

      return {
        completion: {
          values: matchingFiles.slice(0, 100), // Max 100 results
          // total: matchingFiles.length, // Optional total count
          // hasMore: matchingFiles.length > 100 // Optional flag
        },
      };
    }
  }
  // Handle other completion requests or return empty results
  return { completion: { values: [] } };
});
```

### Logging

Servers can send structured logs to clients using `notifications/message` if they declare the `logging` capability. Clients can optionally control the minimum level using `logging/setLevel`.

**Sending a Log Message (Conceptual):**

```typescript
// Inside any handler where 'exchange' is available...
async function someOperation(input: any, exchange: McpServerExchange) {
  try {
    // ... do work ...
    await exchange.sendLogMessage({
      level: "info",
      logger: "MyOperationLogger", // Optional logger name
      data: { message: "Operation successful", input: input }, // Arbitrary JSON data
    });
    return {
      /* success result */
    };
  } catch (error: any) {
    await exchange.sendLogMessage({
      level: "error",
      logger: "MyOperationLogger",
      data: {
        message: "Operation failed",
        error: error.message,
        stack: error.stack,
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
- **Concurrency:** Leverage `async/await` and Node.js's event loop to handle multiple requests concurrently without blocking. Avoid CPU-intensive synchronous tasks.
- **Efficient Data Handling:** Use streams for large data transfers if the transport supports it. Process data efficiently.
- **Transport Choice:** `stdio` is generally lower overhead than HTTP for local communication.
- **Debouncing/Throttling:** Implement server-side rate limiting for resource-intensive tools or frequent notifications.

## 6. Security and Best Practices

Security is paramount when building MCP servers, as they often interact with sensitive data and systems.

### Authentication and Authorization

- **Transports:**
  - **Stdio:** Relies on the security context of the process execution. Ensure the client launches the server securely. Credentials might be passed via environment variables if needed, but this requires careful handling by the client.
  - **Streamable HTTP:** **MUST** implement proper authentication (e.g., Bearer tokens, API Keys) and authorization. Follow the [MCP Authorization specification](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization) which is based on OAuth 2.1. Validate `Origin` headers and bind to `localhost` for local servers to prevent DNS rebinding. **MUST** use HTTPS for remote connections.
- **Capability Control:** Implement fine-grained authorization checks within tool/resource handlers if different clients should have different permissions.

### Data Security

- **Input Validation:**
  - **Always validate and sanitize ALL inputs** from the client (tool arguments, resource URIs, prompt arguments) using robust schemas (`zod` is excellent for this).
  - **Prevent Path Traversal:** If dealing with file paths, rigorously validate and normalize paths to ensure they stay within designated boundaries. Never trust client-provided paths directly.
  - **Prevent Injection:** Sanitize inputs used in database queries (use parameterized queries), shell commands (avoid direct execution if possible, otherwise escape rigorously), or API calls.
- **Data Handling:**
  - Use TLS/HTTPS for network transports.
  - Encrypt sensitive data stored by the server.
  - Avoid logging sensitive information. If necessary, mask or redact it.
- **Resource Protection:**
  - Implement access controls based on authentication/authorization context.
  - Rate limit requests to prevent abuse.

### Error Handling

- **Be Specific but Safe:** Provide enough information for debugging but avoid leaking internal details (stack traces, sensitive paths, internal error codes) to the client in error responses. Log detailed errors server-side.
- **Tool Errors:** Return `{ isError: true, content: [...] }` for errors _within_ a tool's execution (e.g., API call failed). Use standard JSON-RPC errors for protocol-level issues (e.g., tool not found, invalid params).
- **Resource Cleanup:** Ensure resources (file handles, network connections, DB connections) are properly closed, especially in error paths (`finally` blocks).

### General Best Practices

- **Principle of Least Privilege:** Servers should only have the permissions they absolutely need to function.
- **Dependency Security:** Keep dependencies updated (`npm audit`). Use lockfiles (`package-lock.json`).
- **Code Quality:** Write clean, maintainable, and well-tested code. Simpler code is often more secure.
- **Clear Documentation:** Document the server's purpose, capabilities, required configuration, and security considerations.
- **Testing:** Include tests for security vulnerabilities (invalid inputs, path traversal attempts, permission errors).
- **Annotations:** Use tool annotations (`readOnlyHint`, `destructiveHint`, etc.) accurately, but remember clients **MUST NOT** rely solely on these for security decisions.

## 7. Troubleshooting and Resources

### Debugging Tools

- **MCP Inspector:** ([GitHub](https://github.com/modelcontextprotocol/inspector)) Essential for directly interacting with your server (especially stdio) outside a full client application. Launch it with your server command: `npx @modelcontextprotocol/inspector node build/index.js`.
- **Client Logs (e.g., Claude Desktop):** Check the host application's logs for connection errors, messages sent/received, and server stderr output. (See paths in Section 3 of the quickstart).
- **Server Logging:** Implement robust logging within your server (using `console.error` for stdio, or the MCP logging capability).
- **Node.js Debugger:** Use standard Node.js debugging techniques (e.g., VS Code debugger, `node --inspect`).

### Viewing Logs (Claude Desktop Example)

- **macOS:** `tail -n 50 -F ~/Library/Logs/Claude/mcp*.log`
- **Windows:** Check `%APPDATA%\Claude\logs\mcp*.log` (use PowerShell `Get-Content -Path "$env:APPDATA\Claude\logs\mcp*.log" -Wait -Tail 50` for live view)
- Look for `mcp.log` (general) and `mcp-server-SERVERNAME.log` (stderr from your server).

### Common Issues and Solutions

- **Server Not Starting/Connecting:**
  - Check client logs for errors (path issues, command errors).
  - Verify server executable path and permissions in client config (e.g., `claude_desktop_config.json`). Use **absolute paths**.
  - Run the server command directly in your terminal to check for startup errors.
  - Ensure the server script has execute permissions (`chmod +x build/index.js`).
  - Check for port conflicts if using HTTP.
- **Incorrect Working Directory (esp. with Claude Desktop):** Servers launched by clients might not have the expected working directory. Use absolute paths for file access or resolve paths relative to `import.meta.url` or a known base directory.
- **Environment Variables Missing:** Clients may not pass the full environment. Explicitly configure required environment variables in the client's server configuration if possible, or design the server to read from a config file.
- **JSON-RPC Errors:**
  - `-32700 Parse error`: Invalid JSON sent.
  - `-32600 Invalid Request`: JSON is not a valid Request object.
  - `-32601 Method not found`: Client called a method the server doesn't support or hasn't registered. Check capabilities and method names.
  - `-32602 Invalid params`: Input doesn't match the schema defined for the tool/resource/prompt. Check `zod` schemas and client arguments.
  - `-32603 Internal error`: Server-side exception occurred during processing. Check server logs.
- **Tool Calls Failing:** Check server logs for detailed errors. Ensure external dependencies (APIs, DBs) are available. Verify input parameters match the tool's schema.
- **Resource Access Denied:** Check path validation logic and filesystem permissions. Ensure the server process has access to the requested files/directories.

### Implementing Logging Effectively

- **Use `console.error` for Stdio:** Simple and captured by most clients.
- **Use MCP Logging (`exchange.sendLogMessage`):** For structured logs visible to clients that support the logging capability. Choose appropriate levels (`debug`, `info`, `warn`, `error`).
- **Log Key Events:** Initialization, connection, disconnection, request received, response sent, errors encountered, significant state changes.
- **Include Context:** Log relevant data like request IDs, method names, user identifiers (if applicable and safe), and parameters (masking sensitive parts).
- **Structured Logging:** Consider libraries like `pino` or `winston` for more advanced server-side logging (outputting JSON to stderr for stdio is often effective).

### Community Resources and Support

- **Official Documentation:** [modelcontextprotocol.io](https://modelcontextprotocol.io/)
- **GitHub Organization:** [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol) (SDKs, Specification, Inspector, Servers)
- **GitHub Discussions:** For Q&A and community interaction ([Spec Discussions](https://github.com/modelcontextprotocol/specification/discussions), [Org Discussions](https://github.com/orgs/modelcontextprotocol/discussions)).
- **GitHub Issues:** For bug reports and feature requests on specific repositories.

## 8. Example Implementations

These examples illustrate combining different concepts.

### SQLite Explorer (Enhanced)

```typescript
// src/sqlite-explorer.ts
import {
  McpServer,
  ResourceTemplate,
} from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import sqlite3 from "sqlite3";
import { open, Database } from "sqlite"; // Use the promise-based 'sqlite' wrapper
import { z } from "zod";
import path from "path";
import { fileURLToPath } from "url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const DB_PATH = path.resolve(__dirname, "../database.db"); // Store DB alongside src

async function main() {
  const server = new McpServer(
    {
      name: "SQLiteExplorer",
      version: "1.0.1",
    },
    {
      capabilities: {
        resources: { listChanged: false },
        tools: { listChanged: false },
      },
    }
  );

  // Helper to open DB connection
  const getDb = async (): Promise<Database> => {
    // Use verbose mode for better debugging during development
    // sqlite3.verbose();
    return open({
      filename: DB_PATH,
      driver: sqlite3.Database,
    });
  };

  // Resource: Database Schema
  server.resource(
    "db-schema", // Resource group name
    "schema://main", // Static URI for the main schema
    async (uri) => {
      // Read handler
      let db: Database | null = null;
      try {
        db = await getDb();
        // Query to get schema for all tables
        const tables = await db.all<{ name: string; sql: string }>(
          "SELECT name, sql FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%';"
        );
        const schemaText = tables
          .map((t) => `-- ${t.name}\n${t.sql};`)
          .join("\n\n");
        return {
          contents: [
            {
              uri: uri.href,
              mimeType: "application/sql", // Use appropriate MIME type
              text: schemaText || "-- No tables found --",
            },
          ],
        };
      } catch (error: any) {
        console.error(`Schema retrieval error: ${error.message}`);
        throw new Error(`Failed to retrieve schema: ${error.message}`);
      } finally {
        await db?.close(); // Ensure DB is closed
      }
    }
  );

  // Tool: Execute Read-Only SQL Query
  const querySchema = z.object({
    sql: z
      .string()
      .trim()
      .min(1)
      .describe("The read-only SQL query to execute"),
  });

  server.tool(
    "sqlQuery", // Tool name
    querySchema, // Input schema
    async ({ sql }) => {
      // Handler
      // Basic validation: Prevent modification queries (improve this for production)
      const lowerSql = sql.toLowerCase();
      if (
        /\b(insert|update|delete|drop|create|alter|attach)\b/.test(lowerSql)
      ) {
        return {
          content: [
            {
              type: "text",
              text: "Error: Only read-only SELECT queries are allowed.",
            },
          ],
          isError: true,
        };
      }

      let db: Database | null = null;
      try {
        db = await getDb();
        // Use a timeout for queries
        const results = await Promise.race([
          db.all(sql),
          new Promise((_, reject) =>
            setTimeout(() => reject(new Error("Query timeout")), 5000)
          ), // 5s timeout
        ]);

        return {
          content: [
            {
              type: "text",
              // Limit result size for display
              text:
                JSON.stringify(results.slice(0, 50), null, 2) +
                (results.length > 50 ? "\n... (results truncated)" : ""),
            },
          ],
        };
      } catch (error: any) {
        console.error(`SQL Query error: ${error.message}`);
        return {
          content: [{ type: "text", text: `Query Error: ${error.message}` }],
          isError: true,
        };
      } finally {
        await db?.close();
      }
    },
    { readOnlyHint: true } // Annotation: This tool doesn't modify data
  );

  // --- Connection ---
  const transport = new StdioServerTransport();
  try {
    await server.connect(transport);
    console.error("SQLite Explorer MCP Server running via stdio.");
  } catch (error) {
    console.error("Failed to connect server:", error);
    process.exit(1);
  }
}

main().catch((error) => {
  console.error("Unhandled error during server startup:", error);
  process.exit(1);
});
```

### Low-Level Server Implementation (Manual Handlers)

This shows using the base `Server` class for finer control over request handling, bypassing the `McpServer` helpers.

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
      console.error(`Handling request: ${request.method}`);
      // Implementation for listing prompts
      const result: McpSchema.ListPromptsResult = {
        prompts: [
          {
            name: "example-prompt",
            description: "An example prompt template (low-level)",
            arguments: [
              { name: "arg1", description: "Example argument", required: true },
            ],
          },
        ],
        // nextCursor: undefined // No pagination in this example
      };
      // Validate result against schema (optional but recommended)
      ListPromptsResultSchema.parse(result);
      return result;
    }
  );

  // Manually register a handler for 'prompts/get'
  server.setRequestHandler(
    GetPromptRequestSchema,
    async (request, exchange) => {
      console.error(
        `Handling request: ${request.method} for prompt: ${request.params.name}`
      );
      if (request.params.name !== "example-prompt") {
        // Throwing an error generates a JSON-RPC error response
        throw new Error(`Unknown prompt: ${request.params.name}`);
      }
      const arg1Value = request.params.arguments?.arg1 ?? "[arg1 not provided]";

      const result: McpSchema.GetPromptResult = {
        description: "Example prompt (low-level)",
        messages: [
          {
            role: "user",
            content: {
              type: "text",
              text: `Low-level prompt text with arg1: ${arg1Value}`,
            },
          },
        ],
      };
      // Validate result against schema (optional but recommended)
      GetPromptResultSchema.parse(result);
      return result;
    }
  );

  // --- Connection ---
  const transport = new StdioServerTransport();
  try {
    await server.connect(transport);
    console.error("Low-Level MCP Server running via stdio.");
  } catch (error) {
    console.error("Failed to connect server:", error);
    process.exit(1);
  }
}

main().catch((error) => {
  console.error("Unhandled error during server startup:", error);
  process.exit(1);
});
```
