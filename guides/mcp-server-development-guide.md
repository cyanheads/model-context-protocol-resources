# Model Context Protocol (MCP) Server Development Guide: Building Powerful Tools for LLMs

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/MCP%20SDK-TypeScript%201.6.1-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/MCP%20SDK-Python%201.3.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/MCP%20SDK-Kotlin%200.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-March%202025-brightgreen.svg)]()

## Table of Contents

1. [Introduction to MCP Servers](#1-introduction-to-mcp-servers)
2. [Core Server Architecture](#2-core-server-architecture)
3. [Building Your First MCP Server](#3-building-your-first-mcp-server)
4. [Exposing Capabilities](#4-exposing-capabilities)
   * [Defining and Implementing Tools](#defining-and-implementing-tools)
   * [Managing Resources](#managing-resources)
   * [Creating and Sharing Prompts](#creating-and-sharing-prompts)
5. [Advanced Server Features](#5-advanced-server-features)
   * [Sampling](#sampling)
   * [Roots](#roots)
   * [Streaming Responses](#streaming-responses)
   * [Progress Reporting](#progress-reporting)
   * [Resource Subscriptions](#resource-subscriptions)
   * [Multi-Server Coordination](#multi-server-coordination)
   * [Performance Optimization](#performance-optimization)
6. [Security and Best Practices](#6-security-and-best-practices)
7. [Troubleshooting and Resources](#7-troubleshooting-and-resources)
8. [Example Implementations](#8-example-implementations)

## 1. Introduction to MCP Servers

**What is the Model Context Protocol?**

The Model Context Protocol (MCP) is a standardized communication protocol designed to facilitate interactions between Large Language Model (LLM) applications (clients) and external services (servers). These servers provide contextual information, tools, and resources. MCP enables a clean separation of concerns, allowing LLM applications to focus on core functionality while delegating tasks like data retrieval, external API access, and specialized computations to dedicated servers.

You can find an official introduction to MCP [here](https://modelcontextprotocol.io/introduction).

**The Role of Servers in the MCP Ecosystem**

Servers are the backbone of the MCP ecosystem, acting as intermediaries between LLM applications and the external world. A server can provide a wide range of capabilities, including:

* **Providing access to real-time data:** Fetching information from databases, APIs, or other sources.
* **Exposing specialized tools:** Offering functionalities like image processing, code execution, or data analysis.
* **Managing and sharing prompts:** Storing and providing pre-defined prompts or prompt templates.
* **Connecting to external systems:** Integrating with other applications, services, or platforms.

**Benefits of Implementing an MCP Server**

Creating an MCP server offers several advantages:

* **Extensibility:** Easily add new capabilities to LLM applications without modifying their core code.
* **Modularity:** Develop and maintain specialized functionalities in isolated, reusable components.
* **Interoperability:** Enable different LLM applications to share the same context sources and tools.
* **Scalability:** Distribute workloads and handle complex tasks efficiently.
* **Innovation:** Focus on developing unique capabilities and integrations, leveraging the power of LLMs.

**Server vs. Client: Understanding the Relationship**

In the MCP architecture:

* **Clients** are typically LLM applications (like Claude Desktop, VS Code, or custom applications) that initiate connections to servers and request services.
* **Servers** are independent processes that listen for client connections, process requests, and provide responses. They expose capabilities like tools, resources, and prompts.

A single client can connect to multiple servers, and a single server can serve multiple clients. This many-to-many relationship allows for flexible and powerful integrations.

You can find the official quickstart documentation [here](https://modelcontextprotocol.io/quickstart/server).

## 2. Core Server Architecture

### Key Components of an MCP Server

An MCP server consists of several key components working together:

1. **Protocol Layer:** This layer handles the high-level communication patterns, including message framing, request/response linking, and notification handling. It uses classes like `Protocol`, `Client`, and `Server`.
2. **Transport Layer:** This layer manages the actual communication between clients and servers. MCP supports multiple transport mechanisms, such as Stdio and HTTP with Server-Sent Events (SSE). All transports use JSON-RPC 2.0 for message exchange.
3. **Capabilities:** These define what the server can do. Capabilities include tools (executable functions), resources (data sources), and prompts (pre-defined text inputs for LLMs).

### Protocol Fundamentals

MCP is built on a client-server architecture. LLM applications (hosts) initiate connections. Clients maintain 1:1 connections with servers within the host application. Servers provide context, tools, and prompts to clients.

### Server Lifecycle: Connect, Exchange, Terminate

The lifecycle of an MCP connection involves three main stages:

1. **Initialization:**
   * The client sends an `initialize` request, including its protocol version and capabilities.
   * The server responds with its protocol version and capabilities.
   * The client sends an `initialized` notification to acknowledge.
   * Normal message exchange begins.

2. **Message Exchange:** After initialization, clients and servers can exchange messages using these patterns:
   * **Request-Response:** Either side sends a request, and the other responds.
   * **Notifications:** Either side sends one-way messages (no response expected).

3. **Termination:** The connection can be terminated in several ways:
   * Clean shutdown via a `close()` method.
   * Transport disconnection.
   * Error conditions.

### Message Format and Transport

MCP uses JSON-RPC 2.0 as its message format. There are three main types of messages:

1. **Requests:** These expect a response. They include a `method` and optional `params`.

```typescript
{
  jsonrpc: "2.0",
  id: number | string,
  method: string,
  params?: object
}
```

2. **Responses:** These are sent in response to requests. They include either a `result` (on success) or an `error` (on failure).

```typescript
{
  jsonrpc: "2.0",
  id: number | string,
  result?: object,
  error?: {
    code: number,
    message: string,
    data?: unknown
  }
}
```

3. **Notifications:** These are one-way messages that don't expect a response. Like requests, they have a `method` and optional `params`.

```typescript
{
  jsonrpc: "2.0",
  method: string,
  params?: object
}
```

#### Transports

MCP supports different transport mechanisms for communication. The two built-in transports are:

1. **Standard Input/Output (stdio):** This transport uses standard input and output streams, suitable for local integrations and command-line tools.

*Example (Server):*

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
}, { capabilities: {} });

const transport = new StdioServerTransport();
await server.connect(transport);
```

*Example (Client):*

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const client = new Client({
  name: "example-client",
  version: "1.0.0"
}, { capabilities: {} });

const transport = new StdioClientTransport({
  command: "./server", //  Path to your server executable
  args: ["--option", "value"] // Optional arguments
});
await client.connect(transport);
```

When using the stdio transport:
* The client launches the MCP server as a subprocess.
* The server receives JSON-RPC messages via `stdin` and responds via `stdout`.
* Messages must be delimited by newlines.
* The server may use `stderr` for logging.
* **Important:** The server must *not* write anything to `stdout` that isn't a valid MCP message, and the client must *not* write anything to the server's `stdin` that isn't a valid MCP message.

2. **Server-Sent Events (SSE):** This transport uses HTTP POST requests for client-to-server communication and Server-Sent Events for server-to-client streaming.

*Example (Server):*

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { SSEServerTransport } from '@modelcontextprotocol/sdk/server/sse.js';
import express, { Response } from 'express';

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
}, { capabilities: {} });

const app = express();
app.use(express.json());

app.get("/sse", async (req, res) => {
  const transport = new SSEServerTransport("/messages", res as Response);
  await server.connect(transport);
  // Store the transport instance for later use. For simplicity, we assume a single client here.
  app.locals.transport = transport;
});

app.post("/messages", async (req, res) => {
  const transport = app.locals.transport;
  await transport.handlePostMessage(req, res);
});

app.listen(3000, () => {
  console.log('Server listening on port 3000');
});

// Note: For simplicity, this example doesn't handle routing for multiple connections.
// In a production environment, you'd need to route messages to the correct transport instance 
// based on some identifier (e.g., a session ID).
```

*Example (Client):*

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

const client = new Client({
  name: "example-client",
  version: "1.0.0"
}, { capabilities: {} });

const transport = new SSEClientTransport(
  new URL("http://localhost:3000/sse")
);
await client.connect(transport);
```

#### Custom Transports

MCP allows for custom transport implementations. A custom transport must implement the `Transport` interface:

```typescript
interface Transport {
  // Start processing messages
  start(): Promise<void>;

  // Send a JSON-RPC message
  send(message: JSONRPCMessage): Promise<void>;

  // Close the connection
  close(): Promise<void>;

  // Callbacks
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
}
```

#### Error Handling

Transport implementations should handle connection errors, message parsing errors, protocol errors, network timeouts, and resource cleanup. An example:

```typescript
class ExampleTransport implements Transport {
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;

  async start() {
    try {
      // Connection logic
    } catch (error) {
      this.onerror?.(new Error(`Failed to connect: ${error}`));
      throw error;
    }
  }

  async send(message: JSONRPCMessage) {
    try {
      // Sending logic
    } catch (error) {
      this.onerror?.(new Error(`Failed to send message: ${error}`));
      throw error;
    }
  }
  
  async close(): Promise<void> {
    // Close logic
  }
}
```

## 3. Building Your First MCP Server

This section guides you through creating a basic MCP server using the TypeScript SDK.

### Setting Up Your Development Environment

1. **Install Node.js and npm:** Ensure you have Node.js (version 18 or later) and npm (Node Package Manager) installed on your system.
2. **Create a Project Directory:**

```bash
mkdir my-mcp-server
cd my-mcp-server
```

3. **Initialize a Node.js Project:**

```bash
npm init -y
```

4. **Install the MCP TypeScript SDK:**

```bash
npm install @modelcontextprotocol/sdk
```

5. **Install TypeScript and other dependencies:**

```bash
npm install typescript zod
```

6. **Create a `tsconfig.json` file:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "NodeNext",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "outDir": "./build"
  },
  "include": ["src/**/*"]
}
```

7. **Create a `src` directory and an `index.ts` file:**

```bash
mkdir src
touch src/index.ts
```

### Choosing an SDK

The `@modelcontextprotocol/sdk` provides a convenient way to build MCP servers in TypeScript. It handles the protocol details, allowing you to focus on implementing your server's capabilities.

### Implementing the Core Server Interface

Let's create a simple server that echoes back any input it receives. Create `src/index.ts` with the following content:

```typescript
#!/usr/bin/env node

import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { z } from 'zod';

// Create an MCP server instance
const server = new McpServer({
  name: 'EchoServer',
  version: '1.0.0',
});

// Add a tool that echoes back the input
server.tool(
  'echo',
  { message: z.string() }, // Define input schema using zod
  async ({ message }) => ({
    content: [{ type: 'text', text: `Echo: ${message}` }],
  })
);

// Create a transport (stdio for this example)
const transport = new StdioServerTransport();

// Connect the server to the transport
await server.connect(transport);
```

### Handling Connections and Authentication

In this simple example, we're using the `StdioServerTransport`, which handles connections automatically through standard input and output. For more complex scenarios, you might need to implement custom connection handling and authentication, especially when using other transport mechanisms like SSE.

### Processing Client Requests

The `server.tool()` method defines a tool named "echo" that the server can handle. The second argument, `{ message: z.string() }`, uses the `zod` library to define the expected input schema for the tool. The third argument is an asynchronous function that takes the input (validated against the schema) and returns the result. The `McpServer` automatically handles routing client requests to the appropriate tool based on the request's `method`.

To run this server:

1. **Compile the TypeScript code:**

```bash
npx tsc
```

2. **Run the compiled JavaScript:**

```bash
node build/index.js
```

Add the following to the `scripts` section of your `package.json`:

```json
"scripts": {
  "build": "tsc && node -e \"require('fs').chmodSync('build/index.js', '755')\"",
  "start": "node build/index.js"
}
```

Now you can run `npm run build` and `npm start`. You can test it by running `./build/index.js` and typing in a JSON-RPC 2.0 message.

Here's an example of a valid JSON-RPC 2.0 message you could send to the `echo` tool:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": { "name": "echo", "arguments": { "message": "Hello, world!" } }
}
```

## 4. Exposing Capabilities

MCP servers expose their functionalities through three primary mechanisms: Tools, Resources, and Prompts. These capabilities allow LLMs to interact with external systems, access data, and leverage pre-defined interaction patterns.

### Defining and Implementing Tools

Tools enable LLMs to perform actions through your server. They are similar to POST endpoints in a REST API and are designed to be *model-controlled*.

**Key Features:**

* **Discovery:** Clients can discover available tools using the `tools/list` endpoint.
* **Invocation:** Tools are called using the `tools/call` endpoint.
* **Flexibility:** Tools can range from simple calculations to complex API interactions.

**Tool Definition Structure:**

```typescript
{
  name: string;          // Unique identifier
  description?: string;  // Human-readable description
  inputSchema: {         // JSON Schema for input parameters
    type: "object",
    properties: { ... }  // Tool-specific parameters
  }
}
```

**Implementation Example:**

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { z } from 'zod';

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// Add a tool that calculates the sum of two numbers
server.tool(
  'calculate_sum',
  { a: z.number(), b: z.number() }, // Define input schema using zod
  async ({ a, b }) => ({
    content: [{ type: 'text', text: String(a + b) }],
  })
);
```

**Best Practices:**

* Use clear names and descriptions.
* Provide detailed JSON Schema definitions using tools like `zod`.
* Implement proper error handling.
* Keep tool operations focused.
* Consider rate limiting.

**Security Considerations:**

* Validate all inputs.
* Implement access control.
* Handle errors securely.

### Managing Resources

Resources allow servers to expose data and content to LLMs. They are similar to GET endpoints in a REST API and are designed to be *application-controlled*.

**Key Features:**

* **Resource URIs:** Resources are identified by URIs (e.g., `file:///path/to/file.txt`).
* **Text and Binary Resources:** Resources can contain text (UTF-8) or binary data (base64 encoded).
* **Discovery:**
  * **Direct Resources:** Servers expose a list of concrete resources.
  * **Resource Templates:** Servers expose URI templates for dynamic resources.
* **Reading Resources:** Clients use the `resources/read` request.
* **Resource Updates:**
  * **List Changes:** Servers notify clients of changes to the resource list.
  * **Content Changes:** Clients can subscribe to updates for specific resources.

**Implementation Example:**

```typescript
import { McpServer, ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js';
import { z } from 'zod';
import fs from 'fs/promises';
import path from 'path';

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    resources: {}
  }
});

const LOGS_DIR = './logs'; // Or an appropriate subdirectory within your server project

server.resource(
  "app-logs",
  new ResourceTemplate("file:///logs/{filename}", {
    list: async () => {
      const files = await fs.readdir(LOGS_DIR);
      return files.map(file => ({
        uri: `file:///logs/${file}`,
        name: file,
        mimeType: 'text/plain' // Or determine based on file extension
      }));
    }
  }),
  async (uri, { filename }) => {
    const filePath = path.join(LOGS_DIR, filename);

    // Basic sanitization and path traversal prevention
    if (!filePath.startsWith(LOGS_DIR) || path.relative(LOGS_DIR, filePath).startsWith('..')) {
      throw new Error("Access denied");
    }

    try {
      const logContents = await fs.readFile(filePath, 'utf-8');
      return {
        contents: [{
          uri: uri.href,
          mimeType: "text/plain",
          text: logContents
        }]
      };
    } catch (error: any) {
      throw new Error(`Error reading file: ${error.message}`);
    }
  }
);
```

**Best Practices:**

* Use clear resource names and URIs.
* Set appropriate MIME types.
* Use subscriptions for frequently changing resources.
* Validate URIs before processing.

**Security Considerations:**

* Validate all resource URIs.
* Implement access controls.
* Sanitize file paths.

### Creating and Sharing Prompts

Prompts are reusable prompt templates and workflows that clients can surface to users and LLMs. They are designed to be *user-controlled*.

**Key Features:**

* **Dynamic Arguments:** Prompts can accept dynamic arguments.
* **Context from Resources:** Prompts can include context from resources.
* **Multi-step Workflows:** Prompts can chain multiple interactions.
* **UI Integration:** Prompts can be surfaced as UI elements (e.g., slash commands).

**Prompt Structure:**

```typescript
{
  name: string;              // Unique identifier
  description?: string;      // Human-readable description
  arguments?: [              // Optional arguments
    {
      name: string;          // Argument identifier
      description?: string;  // Argument description
      required?: boolean;    // Is argument required?
    }
  ]
}
```

**Discovering Prompts:** Clients use the `prompts/list` endpoint.

**Using Prompts:** Clients use the `prompts/get` request.

**Implementation Example:**

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { z } from 'zod';

const PROMPTS = {
  "git-commit": {
    name: "git-commit",
    description: "Generate a Git commit message",
    arguments: [{
      name: "changes",
      description: "Git diff or description of changes",
      required: true
    }]
  }
};

const server = new McpServer({
  name: "example-prompts-server",
  version: "1.0.0"
}, {
  capabilities: {
    prompts: {}
  }
});

server.prompt(
  "git-commit",
  { changes: z.string() },
  ({ changes }) => ({
    messages: [{
      role: "user",
      content: {
        type: "text",
        text: `Generate a concise but descriptive commit message for these changes:\n\n${changes}`
      }
    }]
  })
);
```

**Best Practices:**

* Use clear prompt names and descriptions.
* Validate all arguments.
* Handle missing arguments gracefully.
* Consider versioning for prompt templates.

**Security Considerations:**

* Validate all arguments.
* Sanitize user input.
* Consider rate limiting.
* Implement access controls.

### Schema Validation and Documentation

All capabilities (Tools, Resources, and Prompts) should have clear and well-defined schemas using JSON Schema. This ensures that clients can understand the expected input and output formats, enabling reliable communication and preventing errors. The TypeScript SDK provides utilities for working with JSON Schema, and you should leverage these to define your schemas. The examples above demonstrate using `zod` for schema definition, which is a good practice.

Good documentation is crucial for making your server's capabilities discoverable and understandable. Use descriptive names and descriptions for all capabilities and their parameters. Consider providing examples in your documentation to illustrate how the capabilities should be used.

## 5. Advanced Server Features

MCP offers several advanced features that enhance server capabilities and enable more complex interactions.

### Sampling

Sampling allows servers to request LLM completions through the client. This enables agentic behaviors while maintaining security and privacy through a human-in-the-loop design.

**Key Concepts:**

* **Human-in-the-Loop:** The client reviews and can modify both the sampling request and the completion.
* **Message Format:** Sampling requests use a standardized message format with `messages`, `modelPreferences`, `systemPrompt`, `includeContext`, and other sampling parameters.
* **Model Preferences:** Servers can specify preferences for model selection (hints, cost, speed, intelligence).
* **Context Inclusion:** Servers can specify which MCP context to include (`none`, `thisServer`, `allServers`).

**Example Request:**

```json
{
  "method": "sampling/createMessage",
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "What files are in the current directory?"
        }
      }
    ],
    "systemPrompt": "You are a helpful file system assistant.",
    "includeContext": "thisServer",
    "maxTokens": 100
  }
}
```

**Best Practices:**

The [zod](https://zod.dev/) library is a TypeScript-first schema declaration and validation library that's excellent for defining these schemas concisely and with good type safety.

* Provide clear prompts.
* Handle both text and image content.
* Set reasonable token limits.
* Include relevant context.
* Validate responses.

### Roots

Roots define the boundaries where servers can operate. Clients suggest roots (URIs) to inform servers about relevant resources and their locations.

**Key Concepts:**

* **Guidance, Not Enforcement:** Roots are informational, not strictly enforcing.
* **Common Use Cases:** Project directories, repository locations, API endpoints.

**Example:**

```json
{
  "roots": [
    {
      "uri": "file:///home/user/projects/frontend",
      "name": "Frontend Repository"
    },
    {
      "uri": "https://api.example.com/v1",
      "name": "API Endpoint"
    }
  ]
}
```

### Streaming Responses

MCP supports streaming responses for tools and resources, allowing servers to send data incrementally. This is particularly useful for large outputs or long-running operations. This is handled at the transport layer, as shown in the SSE transport examples in Section 2.

### Progress Reporting

Servers can report progress for long-running operations, providing feedback to the client and user. This is typically done through notifications, allowing the client to display progress indicators. The specific implementation depends on the server and the nature of the operation.

### Resource Subscriptions

As mentioned in Section 4, clients can subscribe to resource updates. This allows servers to notify clients when a resource's content changes, enabling real-time updates.

### Multi-Server Coordination

MCP is designed to support multiple servers working together. Clients can connect to multiple servers simultaneously, and servers can potentially interact with each other (though direct server-to-server communication is not currently part of the core MCP specification). This allows for complex workflows and distributed processing.

### Performance Optimization

Several techniques can be used to optimize MCP server performance:

* **Caching:** Cache frequently accessed data (e.g., resource contents) to reduce latency.
* **Concurrency:** Handle multiple requests concurrently using asynchronous programming.
* **Efficient Data Structures:** Use appropriate data structures for storing and retrieving information.
* **Optimized Algorithms:** Choose efficient algorithms for processing data.
* **Transport Choice:** Use `stdio` when possible for local communication as it's more efficient.

## 6. Security and Best Practices

### Authentication and Authorization

* Implement appropriate authentication mechanisms for your chosen transport (especially for remote connections).
* Implement authorization checks to control access to resources and tools when needed.

### Data Security

* **Input Validation:**
  * Validate all inputs (resource URIs, tool parameters, prompt arguments) against defined schemas. Using libraries like `zod` is strongly recommended.
  * Sanitize file paths and system commands to prevent injection attacks.
  * Validate URLs and external identifiers.
  * Check parameter sizes and ranges.
* **Data Handling:**
  * Use TLS for network transport (especially for SSE).
  * Encrypt sensitive data at rest and in transit.
  * Validate message integrity.
  * Implement message size limits.
  * Sanitize input data.
  * Be cautious with binary data handling.
* **Resource Protection:**
  * Implement access controls for resources.
  * Validate resource paths.
  * Monitor resource usage.
  * Rate limit requests.

### Error Handling

* Use a consistent error handling strategy (e.g., the Result pattern).
* Don't leak sensitive information in error messages.
* Log security-relevant errors.
* Implement proper cleanup after errors.
* Handle timeouts appropriately.
* Report tool errors within the result object, not as protocol-level errors.

### General Best Practices

* **Principle of Least Privilege:** Grant only the necessary permissions to clients and users.
* **Defense in Depth:** Implement multiple layers of security.
* **Regular Audits:** Regularly audit your server's security and dependencies.
* **Dependency Management:** Use lock files and regularly audit dependencies for vulnerabilities.
* **Keep it Simple:** Simpler code is generally easier to secure.
* **Documentation:** Thoroughly document your server's security measures and expected behavior.
* **Testing:** Include security testing as part of your overall testing strategy.
* **Transports:** Use `stdio` when possible for local communication as it's more efficient and avoids network-related security concerns.
* **Sampling:** Validate all message content and sanitize sensitive information when using sampling.

## 7. Troubleshooting and Resources

This section provides guidance on troubleshooting MCP server implementations and lists helpful resources.

### Debugging Tools

Several tools are available for debugging MCP servers:

* **MCP Inspector:** An interactive debugging interface for direct server testing. See the [MCP Inspector guide](https://github.com/modelcontextprotocol/inspector) for details.
* **Claude Desktop Developer Tools:** Useful for integration testing, log collection, and accessing Chrome DevTools within Claude Desktop.
  * To enable developer tools, create a `developer_settings.json` file with `{"allowDevTools": true}` in `~/Library/Application Support/Claude/`.
  * Open DevTools with Command-Option-Shift-i.
* **Server Logging:** Implement custom logging within your server to track errors, performance, and other relevant information.

### Viewing Logs (Claude Desktop)

You can view detailed MCP logs from Claude Desktop:

```bash
# Follow logs in real-time
tail -n 20 -F ~/Library/Logs/Claude/mcp*.log
```

These logs capture server connection events, configuration issues, runtime errors, and message exchanges.

### Common Issues and Solutions

* **Working Directory Issues (Claude Desktop):** The working directory for servers launched via `claude_desktop_config.json` may be undefined. Always use absolute paths in your configuration and `.env` files.
* **Environment Variable Issues:** MCP servers inherit only a subset of environment variables. Specify required variables explicitly in your `claude_desktop_config.json`.
* **Server Initialization Problems:**
  * Check for incorrect server executable paths, missing files, or permission problems.
  * Verify configuration files for valid JSON syntax, missing fields, or type mismatches.
  * Ensure required environment variables are set correctly.
* **Connection Problems:**
  * Check Claude Desktop logs.
  * Verify the server process is running.
  * Test the server standalone with the MCP Inspector.
  * Verify protocol compatibility.

### Implementing Logging

* **Server-Side Logging:**
  * For `stdio` transport, log messages to `stderr`. These will be captured by the host application (e.g., Claude Desktop).
  * For all transports, send log messages to the client using `server.sendLoggingMessage()` (from `@modelcontextprotocol/sdk/server`).
  * Log initialization steps, resource access, tool execution, error conditions, and performance metrics.
* **Client-Side Logging:** Enable debug logging, monitor network traffic, track message exchanges, and record error states in your client application.

### Debugging Workflow

1. **Initial Development:** Use the MCP Inspector for basic testing, implement core functionality, and add logging.
2. **Integration Testing:** Test within Claude Desktop, monitor logs, and check error handling.

**Testing Changes:**

* Configuration changes: Restart Claude Desktop.
* Server code changes: Use Command-R to reload.

### Best Practices

* **Structured Logging:** Use consistent formats, include context, add timestamps, and track request IDs.
* **Error Handling:** Log stack traces, include error context, track error patterns, and monitor recovery.
* **Performance Tracking:** Log operation timing, monitor resource usage, track message sizes, and measure latency.

### Security Considerations (Debugging)

* **Sensitive Data:** Sanitize logs, protect credentials, and mask personal information.
* **Access Control:** Verify permissions, check authentication, and monitor access patterns.

### Community Resources and Support

* **GitHub Issues:** Report bugs and request features.
* **GitHub Discussions:** Ask questions and discuss MCP development.
* **Documentation:** [Model Context Protocol documentation](https://modelcontextprotocol.io)

## 8. Example Implementations

This section provides more complex example implementations to help you understand how to build practical MCP servers.

### SQLite Explorer

This example shows how to build an MCP server that provides access to a SQLite database:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import sqlite3 from "sqlite3";
import { promisify } from "util";
import { z } from "zod";

const server = new McpServer({
  name: "SQLite Explorer",
  version: "1.0.0"
});

// Helper to create DB connection
const getDb = () => {
  const db = new sqlite3.Database("database.db");
  return {
    all: promisify<string, any[]>(db.all.bind(db)),
    close: promisify(db.close.bind(db))
  };
};

// Resource to expose database schema
server.resource(
  "schema",
  "schema://main",
  async (uri) => {
    const db = getDb();
    try {
      const tables = await db.all(
        "SELECT sql FROM sqlite_master WHERE type='table'"
      );
      return {
        contents: [{
          uri: uri.href,
          text: tables.map((t: {sql: string}) => t.sql).join("\n")
        }]
      };
    } finally {
      await db.close();
    }
  }
);

// Tool to execute SQL queries
server.tool(
  "query",
  { sql: z.string() },
  async ({ sql }) => {
    const db = getDb();
    try {
      const results = await db.all(sql);
      return {
        content: [{
          type: "text",
          text: JSON.stringify(results, null, 2)
        }]
      };
    } catch (err: unknown) {
      const error = err as Error;
      return {
        content: [{
          type: "text",
          text: `Error: ${error.message}`
        }],
        isError: true
      };
    } finally {
      await db.close();
    }
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Low-Level Server Implementation

For more control, you can use the low-level Server class directly:

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  ListPromptsRequestSchema,
  GetPromptRequestSchema
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  {
    name: "example-server",
    version: "1.0.0"
  },
  {
    capabilities: {
      prompts: {}
    }
  }
);

server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: [{
      name: "example-prompt",
      description: "An example prompt template",
      arguments: [{
        name: "arg1",
        description: "Example argument",
        required: true
      }]
    }]
  };
});

server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  if (request.params.name !== "example-prompt") {
    throw new Error("Unknown prompt");
  }
  return {
    description: "Example prompt",
    messages: [{
      role: "user",
      content: {
        type: "text",
        text: "Example prompt text"
      }
    }]
  };
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Multi-Capability Server Example

This example shows a server that combines tools, resources, and prompts:

```typescript
import { McpServer, ResourceTemplate } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import fs from "fs/promises";
import path from "path";

const server = new McpServer({
  name: "Multi-Capability Server",
  version: "1.0.0"
});

// Tool example - Weather lookup
server.tool(
  "get-weather",
  { city: z.string() },
  async ({ city }) => {
    try {
      // In a real implementation, you would call a weather API
      const mockWeather = {
        city,
        temperature: Math.round(Math.random() * 30),
        conditions: ["Sunny", "Cloudy", "Rainy"][Math.floor(Math.random() * 3)]
      };
      
      return {
        content: [{
          type: "text",
          text: JSON.stringify(mockWeather, null, 2)
        }]
      };
    } catch (error: any) {
      return {
        content: [{
          type: "text",
          text: `Error retrieving weather: ${error.message}`
        }],
        isError: true
      };
    }
  }
);

// Resource example - Project files
const PROJECT_DIR = './project';
server.resource(
  "project-files",
  new ResourceTemplate("file:///project/{filename}", {
    list: async () => {
      try {
        await fs.mkdir(PROJECT_DIR, { recursive: true });
        const files = await fs.readdir(PROJECT_DIR);
        return files.map(file => ({
          uri: `file:///project/${file}`,
          name: file,
          mimeType: 'text/plain'
        }));
      } catch (error) {
        console.error(`Error listing project files: ${error}`);
        return [];
      }
    }
  }),
  async (uri, { filename }) => {
    const filePath = path.join(PROJECT_DIR, filename);
    
    // Security validation
    if (!filePath.startsWith(PROJECT_DIR) || path.relative(PROJECT_DIR, filePath).startsWith('..')) {
      throw new Error("Access denied");
    }
    
    try {
      const fileContents = await fs.readFile(filePath, 'utf-8');
      return {
        contents: [{
          uri: uri.href,
          mimeType: "text/plain",
          text: fileContents
        }]
      };
    } catch (error: any) {
      throw new Error(`Error reading file: ${error.message}`);
    }
  }
);

// Prompt example - Code review
server.prompt(
  "code-review",
  { 
    code: z.string(),
    language: z.string().optional(),
    focus: z.enum(["security", "performance", "readability"]).optional()
  },
  ({ code, language, focus }) => ({
    messages: [{
      role: "user",
      content: {
        type: "text",
        text: `Please review this ${language || 'code'} with a focus on ${focus || 'overall quality'}:\n\n${code}`
      }
    }]
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

These examples demonstrate how to build more complex MCP servers that provide multiple capabilities and integrate with external systems. You can use them as starting points for your own implementations.