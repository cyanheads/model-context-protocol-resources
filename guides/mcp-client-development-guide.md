# Model Context Protocol (MCP) Client Development Guide: Building Robust and Flexible LLM Integrations

[![modelcontextprotocol.io](https://img.shields.io/badge/modelcontextprotocol.io-orange.svg)](https://modelcontextprotocol.io/)
[![MCP SDK - TypeScript](https://img.shields.io/badge/MCP%20SDK-TypeScript%201.6.1-blue.svg)](https://github.com/modelcontextprotocol/typescript-sdk)
[![MCP SDK - Python](https://img.shields.io/badge/MCP%20SDK-Python%201.3.0-blue.svg)](https://github.com/modelcontextprotocol/python-sdk)
[![MCP SDK - Kotlin](https://img.shields.io/badge/MCP%20SDK-Kotlin%200.3.0-blue.svg)](https://github.com/modelcontextprotocol/kotlin-sdk)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-March%202025-brightgreen.svg)]()

This guide provides a comprehensive overview of building clients for the **Model Context Protocol (MCP)** using the TypeScript SDK. It covers everything from basic setup to advanced topics, with practical examples and best practices.

Note: This guide focuses on TypeScript implementation details.

## Table of Contents

1. [Introduction to MCP Clients](#1-introduction-to-mcp-clients)
   * [What is MCP?](#what-is-mcp)
   * [The Role of Clients in the MCP Ecosystem](#the-role-of-clients-in-the-mcp-ecosystem)
   * [Benefits of Using MCP Clients](#benefits-of-using-mcp-clients)
   * [Client vs. Server: Understanding the Relationship](#client-vs-server-understanding-the-relationship)
2. [Core Client Architecture](#2-core-client-architecture)
   * [Key Components of an MCP Client](#key-components-of-an-mcp-client)
   * [Protocol Fundamentals](#protocol-fundamentals)
   * [Client Lifecycle: Connect, Exchange, Terminate](#client-lifecycle-connect-exchange-terminate)
   * [Message Format and Transport](#message-format-and-transport)
3. [Building Your First MCP Client](#3-building-your-first-mcp-client)
   * [Setting Up Your Development Environment](#setting-up-your-development-environment)
   * [Choosing an SDK](#choosing-an-sdk)
   * [Implementing the Core Client Interface](#implementing-the-core-client-interface)
   * [Handling Connections and Authentication](#handling-connections-and-authentication)
   * [Processing Server Responses](#processing-server-responses)
4. [Discovering and Using Server Capabilities](#4-discovering-and-using-server-capabilities)
   * [Working with Tools](#working-with-tools)
   * [Accessing Resources](#accessing-resources)
   * [Utilizing Prompts](#utilizing-prompts)
   * [Schema Validation and Type Safety](#schema-validation-and-type-safety)
5. [Integrating with LLMs](#5-integrating-with-llms)
   * [Connecting MCP Clients with LLM Services](#connecting-mcp-clients-with-llm-services)
   * [Providing Tool Definitions to LLMs](#providing-tool-definitions-to-llms)
   * [Handling Tool Execution and Results](#handling-tool-execution-and-results)
   * [Context Management and Re-injection](#context-management-and-re-injection)
   * [Human-in-the-Loop Approval](#human-in-the-loop-approval)
6. [Advanced Client Features](#6-advanced-client-features)
   * [Sampling](#sampling)
   * [Multi-Server Connections](#multi-server-connections)
   * [Multi-Step (Agentic) Flows](#multi-step-agentic-flows)
   * [Resource Subscriptions](#resource-subscriptions)
   * [Roots](#roots)
   * [Progress Reporting](#progress-reporting)
   * [Handling List Endpoint Updates](#handling-list-endpoint-updates)
7. [Security and Best Practices](#7-security-and-best-practices)
   * [Authentication and Authorization](#authentication-and-authorization)
   * [Data Security](#data-security)
   * [Error Handling](#error-handling)
   * [General Best Practices](#general-best-practices)
8. [Troubleshooting and Resources](#8-troubleshooting-and-resources)
   * [Debugging Tools](#debugging-tools)
   * [Common Issues and Solutions](#common-issues-and-solutions)
   * [Implementing Logging](#implementing-logging)
   * [Debugging Workflow](#debugging-workflow)
   * [Community Resources and Support](#community-resources-and-support)
9. [Example Implementations](#9-example-implementations)
   * [Basic LLM Integration](#basic-llm-integration)
   * [Multi-Server Client Manager](#multi-server-client-manager)
   * [Streaming Response Handler](#streaming-response-handler)
   * [Complete LLM Application](#complete-llm-application)

## 1. Introduction to MCP Clients

### What is MCP?

The **Model Context Protocol (MCP)** is an open standard that defines how Large Language Model (LLM) applications access data, prompts, and tools. It acts as a universal connector, enabling your LLM-based application (chatbot, IDE, or any AI system) to communicate seamlessly with any MCP server. Within the application, an **MCP client** component manages these connections. Each client maintains a 1:1 connection with a specific MCP server.

You can find an official introduction to MCP [here](https://modelcontextprotocol.io/introduction).

**Analogy:** MCP is like a universal adapter for AI applications, allowing them to connect to a wide range of services and data sources without requiring custom integration code for each one.

### The Role of Clients in the MCP Ecosystem

Clients are the bridge between LLM applications and MCP servers. They handle the communication, discovery, and utilization of server capabilities such as:

* **Tools**: Executable functions exposed by servers (e.g., API calls, calculations, system operations)
* **Resources**: Data sources exposed by servers (e.g., files, database records, live system data)
* **Prompts**: Pre-defined templates for LLM interactions

Clients are responsible for:
* Discovering what capabilities servers offer
* Requesting and receiving data from servers
* Executing tools on behalf of the LLM
* Managing connections and error handling
* Implementing security measures like human approval for tool execution

### Benefits of Using MCP Clients

1. **Single Integration Point:** Add capabilities from any MCP server (filesystems, Git, Slack, or custom solutions) through a single, consistent protocol.
2. **LLM Independence:** Switch between LLM providers without altering your data integration strategy.
3. **Security & Human Control:** MCP incorporates built-in patterns for human approval and robust security checks. Tool calls can be approved by the user before execution.
4. **Extensibility:** Easily integrate new tools, data sources, and prompts as your application evolves.
5. **Standardization:** Follow industry best practices for LLM context integration without reinventing patterns.
6. **Modularity:** Keep your LLM interaction logic separate from your data and tool access logic.

### Client vs. Server: Understanding the Relationship

In the MCP architecture:

* **Clients** are typically embedded within LLM applications (like Claude Desktop, VS Code plugins, or custom applications) that initiate connections to servers and request services.
* **Servers** are independent processes that listen for client connections, process requests, and provide responses through exposed capabilities.

A single client can connect to multiple servers, and a single server can serve multiple clients. This many-to-many relationship allows for flexible and powerful integrations.

You can find the official quickstart documentation [here](https://modelcontextprotocol.io/quickstart/client).

## 2. Core Client Architecture

### Key Components of an MCP Client

An MCP client consists of several key components working together:

1. **Protocol Layer:** This layer handles the high-level communication patterns, including message framing, request/response linking, and notification handling. It uses the `Client` class to manage the protocol details.

2. **Transport Layer:** This layer manages the actual communication between clients and servers. MCP supports multiple transport mechanisms, such as Stdio and HTTP with Server-Sent Events (SSE). All transports use JSON-RPC 2.0 for message exchange.

3. **Capability Interfaces:** These are the APIs that clients use to interact with server capabilities (tools, resources, and prompts).

### Protocol Fundamentals

MCP is built on a client-server architecture:

* **Host Applications** (like Claude Desktop, VS Code, or custom applications) embed MCP clients
* **Clients** maintain 1:1 connections with servers, inside the host application
* **Servers** provide context, tools, and prompts to clients

```
graph LR
    subgraph "Host Application"
        LLM
        Client1[MCP Client 1]
        Client2[MCP Client 2]
    end
    ServerA[MCP Server A]
    ServerB[MCP Server B]

    Client1 --> ServerA
    Client2 --> ServerB
```

### Client Lifecycle: Connect, Exchange, Terminate

The lifecycle of an MCP connection involves three main stages:

1. **Initialization:**
   * The client sends an `initialize` request, including its protocol version and capabilities.
   * The server responds with its protocol version and capabilities.
   * The client sends an `initialized` notification to acknowledge.
   * Normal message exchange begins.

2. **Message Exchange:** After initialization, clients and servers can exchange messages using these patterns:
   * **Request-Response:** The client sends a request, and the server responds.
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

MCP clients can use different transport mechanisms for communication with servers:

1. **Standard Input/Output (stdio):** This transport launches the server as a child process and communicates via standard input and output streams. It's ideal for local integrations and command-line tools.

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const client = new Client(
  { name: "example-client", version: "1.0.0" },
  { capabilities: { tools: {}, resources: {}, prompts: {} } }
);

const transport = new StdioClientTransport({
  command: "./server", // Path to the server executable
  args: ["--option", "value"] // Optional arguments
});

await client.connect(transport);
await client.initialize();
```

2. **HTTP with Server-Sent Events (SSE):** This transport uses HTTP POST requests for client-to-server communication and Server-Sent Events for server-to-client streaming. It's suitable for remote connections.

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

const client = new Client(
  { name: "example-client", version: "1.0.0" },
  { capabilities: { tools: {}, resources: {}, prompts: {} } }
);

const transport = new SSEClientTransport(
  new URL("http://localhost:3000/sse")
);

await client.connect(transport);
await client.initialize();
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

## 3. Building Your First MCP Client

### Setting Up Your Development Environment

1. **Install Node.js and npm:** Ensure you have Node.js (version 18 or later) and npm (Node Package Manager) installed on your system.

2. **Create a Project Directory:**

```bash
mkdir my-mcp-client
cd my-mcp-client
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

The `@modelcontextprotocol/sdk` provides a convenient way to build MCP clients in TypeScript. It handles the protocol details, allowing you to focus on integrating server capabilities with your application.

### Implementing the Core Client Interface

Let's create a simple client that connects to a server, discovers its capabilities, and calls a tool. Create `src/index.ts` with the following content:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function main() {
  // Create an MCP client instance
  const client = new Client(
    { name: 'ExampleClient', version: '1.0.0' },
    { capabilities: { tools: {}, resources: {}, prompts: {} } }
  );

  try {
    // Create a transport
    const transport = new StdioClientTransport({
      command: './server.js', // Path to your server executable
      args: []
    });

    // Connect to the server
    console.log('Connecting to server...');
    await client.connect(transport);
    
    // Initialize the connection
    console.log('Initializing connection...');
    await client.initialize();
    console.log('Connection established successfully');

    // Discover server capabilities
    const tools = await client.listTools();
    console.log('Available tools:', tools.tools);

    const resources = await client.listResources();
    console.log('Available resources:', resources.resources);
    console.log('Available resource templates:', resources.resourceTemplates);

    const prompts = await client.listPrompts();
    console.log('Available prompts:', prompts.prompts);

    // If the server has an 'echo' tool, call it
    if (tools.tools.some(tool => tool.name === 'echo')) {
      console.log('Calling echo tool...');
      const result = await client.callTool('echo', { message: 'Hello, World!' });
      console.log('Echo result:', result.content[0].text);
    }

    // Close the connection
    await client.close();
    console.log('Connection closed');
  } catch (error) {
    console.error('Error:', error);
  }
}

main();
```

### Handling Connections and Authentication

For the stdio transport, connections are handled automatically. For SSE transport, you might need to handle authentication by adding headers to your requests:

```typescript
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

// Create an SSE transport with authentication
const transport = new SSEClientTransport(
  new URL("https://api.example.com/mcp/sse")
);

// Add authentication headers
transport.setHeaders({
  'Authorization': `Bearer ${apiToken}`
});

await client.connect(transport);
await client.initialize();
```

### Processing Server Responses

When interacting with a server, you'll need to process its responses. Here's how to handle different types of responses:

```typescript
// Calling a tool and handling the response
async function callTool(client, toolName, args) {
  try {
    const result = await client.callTool(toolName, args);
    
    // Check if the tool execution resulted in an error
    if (result.isError) {
      console.error(`Tool error: ${result.content[0].text}`);
      return null;
    }
    
    // Process successful result
    console.log(`Tool result: ${result.content[0].text}`);
    return result.content;
  } catch (error) {
    // Handle protocol or transport errors
    console.error(`Failed to call tool ${toolName}:`, error);
    throw error;
  }
}

// Reading a resource and handling the response
async function readResource(client, resourceUri) {
  try {
    const result = await client.readResource(resourceUri);
    
    // Process the resource contents
    for (const content of result.contents) {
      if (content.text) {
        console.log(`Resource text: ${content.text}`);
      } else if (content.blob) {
        console.log(`Resource binary data (base64): ${content.blob.substring(0, 50)}...`);
      }
    }
    
    return result.contents;
  } catch (error) {
    console.error(`Failed to read resource ${resourceUri}:`, error);
    throw error;
  }
}
```

## 4. Discovering and Using Server Capabilities

### Working with Tools

Tools are executable functions exposed by servers. Here's how to discover and use them:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { z } from 'zod';

async function discoverAndUseTool(client: Client) {
  // Discover available tools
  const toolsResult = await client.listTools();
  console.log('Available tools:', toolsResult.tools);
  
  // Find a specific tool
  const calculatorTool = toolsResult.tools.find(tool => tool.name === 'calculate');
  
  if (calculatorTool) {
    console.log('Found calculator tool:', calculatorTool);
    console.log('Input schema:', calculatorTool.inputSchema);
    
    // Create a zod schema from the tool's input schema
    const inputSchema = createZodSchema(calculatorTool.inputSchema);
    
    // Validate input against the schema
    try {
      const input = { a: 5, b: 3, operation: 'add' };
      const validatedInput = inputSchema.parse(input);
      
      // Call the tool with validated input
      const result = await client.callTool('calculate', validatedInput);
      console.log('Calculation result:', result.content[0].text);
    } catch (error) {
      console.error('Invalid input:', error);
    }
  } else {
    console.log('Calculator tool not found');
  }
}

// Helper function to create a zod schema from JSON Schema
function createZodSchema(schema: any): z.ZodTypeAny {
  // Implementation details omitted for brevity
  // See the full implementation in the schema validation section
}
```

**Key Concepts:**

* **Tool Discovery:** Use `client.listTools()` to discover available tools.
* **Tool Invocation:** Use `client.callTool(name, args)` to execute a tool.
* **Input Validation:** Validate tool inputs against the provided schema before calling.
* **Error Handling:** Check for errors in both the tool result (`isError` flag) and protocol-level errors.

### Accessing Resources

Resources are data sources exposed by servers. Here's how to discover and access them:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

async function discoverAndAccessResources(client: Client) {
  // Discover available resources
  const resourcesResult = await client.listResources();
  
  console.log('Available resources:', resourcesResult.resources);
  console.log('Available resource templates:', resourcesResult.resourceTemplates);
  
  // Access a specific resource
  if (resourcesResult.resources.length > 0) {
    const resource = resourcesResult.resources[0];
    console.log(`Reading resource: ${resource.name} (${resource.uri})`);
    
    const contentResult = await client.readResource(resource.uri);
    
    for (const content of contentResult.contents) {
      if (content.text) {
        console.log('Resource content (text):', content.text);
      } else if (content.blob) {
        console.log('Resource content (binary):', `Base64 data, ${content.blob.length} chars`);
      }
    }
  }
  
  // Use a resource template
  const templates = resourcesResult.resourceTemplates;
  if (templates.length > 0) {
    const template = templates[0];
    console.log(`Using template: ${template.name} (${template.uriTemplate})`);
    
    // Example: if template is "file:///{path}"
    const uri = template.uriTemplate.replace('{path}', 'example.txt');
    
    try {
      const contentResult = await client.readResource(uri);
      console.log('Template resource content:', contentResult.contents[0].text);
    } catch (error) {
      console.error('Error accessing template resource:', error);
    }
  }
}
```

**Key Concepts:**

* **Resource Discovery:** Use `client.listResources()` to discover available resources and templates.
* **Resource Access:** Use `client.readResource(uri)` to read resource content.
* **Content Types:** Resources can return text content or binary data (base64 encoded).
* **Templates:** Resource templates allow dynamic access to resources with parameters.

### Utilizing Prompts

Prompts are pre-defined templates for LLM interactions. Here's how to discover and use them:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

async function discoverAndUsePrompts(client: Client) {
  // Discover available prompts
  const promptsResult = await client.listPrompts();
  console.log('Available prompts:', promptsResult.prompts);
  
  // Find a specific prompt
  const codeReviewPrompt = promptsResult.prompts.find(prompt => prompt.name === 'code-review');
  
  if (codeReviewPrompt) {
    console.log('Found code review prompt:', codeReviewPrompt);
    console.log('Arguments:', codeReviewPrompt.arguments);
    
    // Use the prompt with arguments
    try {
      const promptArgs = {
        code: 'function hello() { return "world"; }',
        language: 'javascript'
      };
      
      const result = await client.getPrompt('code-review', promptArgs);
      
      console.log('Prompt result:', result);
      console.log('Messages to send to LLM:', result.messages);
      
      // Now you can send these messages to your LLM
      // const llmResponse = await sendToLLM(result.messages);
    } catch (error) {
      console.error('Error using prompt:', error);
    }
  } else {
    console.log('Code review prompt not found');
  }
}
```

**Key Concepts:**

* **Prompt Discovery:** Use `client.listPrompts()` to discover available prompts.
* **Prompt Usage:** Use `client.getPrompt(name, args)` to get a prompt with arguments.
* **Messages:** The prompt result contains messages that can be sent directly to an LLM.
* **Arguments:** Prompts can accept arguments to customize their content.

### Schema Validation and Type Safety

Schema validation is crucial for ensuring that inputs to tools and prompts are valid. The MCP TypeScript SDK works well with Zod, a TypeScript-first schema validation library:

```typescript
import { z, ZodError } from 'zod';

// Create a zod schema from a JSON Schema object
function createZodSchema(schema: any): z.ZodTypeAny {
  const type = schema.type;

  if (type === 'string') {
    return z.string();
  } else if (type === 'number') {
    return z.number();
  } else if (type === 'integer') {
    return z.number().int();
  } else if (type === 'boolean') {
    return z.boolean();
  } else if (type === 'array') {
    const items = schema.items || {};
    return z.array(createZodSchema(items));
  } else if (type === 'object') {
    const properties = schema.properties || {};
    const shape: Record<string, z.ZodTypeAny> = {};
    
    for (const [key, value] of Object.entries(properties)) {
      shape[key] = createZodSchema(value as any);
    }
    
    let baseSchema = z.object(shape);
    
    // Handle required properties
    if (schema.required && Array.isArray(schema.required)) {
      const requiredShape: Record<string, z.ZodTypeAny> = {};
      
      for (const key of Object.keys(shape)) {
        const isRequired = schema.required.includes(key);
        requiredShape[key] = isRequired ? shape[key] : shape[key].optional();
      }
      
      baseSchema = z.object(requiredShape);
    }
    
    return baseSchema;
  } else {
    return z.any();
  }
}

// Example usage
async function validateAndCallTool(client, toolName, inputData) {
  // Get tool schema
  const tools = await client.listTools();
  const tool = tools.tools.find(t => t.name === toolName);
  
  if (!tool) {
    throw new Error(`Tool not found: ${toolName}`);
  }
  
  try {
    // Create schema and validate input
    const schema = createZodSchema(tool.inputSchema);
    const validatedInput = schema.parse(inputData);
    
    // Call the tool with validated input
    return await client.callTool(toolName, validatedInput);
  } catch (error) {
    if (error instanceof ZodError) {
      console.error('Validation error:', error.errors);
      throw new Error(`Invalid input for tool ${toolName}: ${error.message}`);
    }
    throw error;
  }
}
```

## 5. Integrating with LLMs

### Connecting MCP Clients with LLM Services

To create a complete LLM application with MCP, you need to connect your MCP client with an LLM service (like Anthropic Claude, OpenAI, or others). Here's a generic approach that can be adapted to any LLM provider:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
// Import your LLM client library
// import { AnthropicClient } from '@anthropic-ai/sdk';

// Hypothetical LLM client class (you'll replace this with your actual LLM client)
class LLMClient {
  async sendMessage(messages, options = {}) {
    // Implementation depends on your LLM provider
    console.log('Sending to LLM:', { messages, options });
    return {
      id: 'msg_123',
      role: 'assistant',
      content: 'This is a sample response from the LLM.',
      tool_calls: [] // Will contain tool call requests if any
    };
  }
}

async function setupLLMWithMCP() {
  // Set up MCP client
  const mcpClient = new Client(
    { name: 'ExampleClient', version: '1.0.0' },
    { capabilities: { tools: {}, resources: {}, prompts: {} } }
  );
  
  // Connect to MCP server
  const transport = new StdioClientTransport({
    command: './server.js',
    args: []
  });
  
  await mcpClient.connect(transport);
  await mcpClient.initialize();
  
  // Set up LLM client
  const llmClient = new LLMClient();
  
  return { mcpClient, llmClient };
}
```

### Providing Tool Definitions to LLMs

LLMs can use tools exposed by MCP servers, but you need to format these tools according to your LLM provider's API requirements:

```typescript
async function prepareToolsForLLM(mcpClient) {
  // Get available tools from MCP server
  const toolsResult = await mcpClient.listTools();
  
  // Format tools for your LLM provider
  // This format will vary based on your LLM provider's API
  // This example uses a generic format - adjust as needed
  const formattedTools = toolsResult.tools.map(tool => ({
    name: tool.name,
    description: tool.description || `Tool: ${tool.name}`,
    parameters: tool.inputSchema
  }));
  
  return formattedTools;
}

// Using the formatted tools with an LLM
async function queryLLMWithTools(llmClient, formattedTools, userQuery) {
  const messages = [
    { role: 'user', content: userQuery }
  ];
  
  // Pass the tools to the LLM
  // The exact API will depend on your LLM provider
  const response = await llmClient.sendMessage(messages, {
    tools: formattedTools
  });
  
  return response;
}
```

### Handling Tool Execution and Results

When an LLM wants to use a tool, it will include a tool call in its response. You need to detect this, execute the tool via MCP, and return the results to the LLM:

```typescript
async function processLLMResponse(mcpClient, llmClient, llmResponse, messages) {
  // Check if the LLM wants to call a tool
  // This detection logic will depend on your LLM provider's response format
  if (llmResponse.tool_calls && llmResponse.tool_calls.length > 0) {
    // Process each tool call
    for (const toolCall of llmResponse.tool_calls) {
      const { name, arguments: args, id } = toolCall;
      
      console.log(`LLM wants to call tool: ${name}`);
      console.log('Tool arguments:', args);
      
      // Optionally get user approval
      const userApproved = await getUserApproval(name, args);
      
      if (userApproved) {
        try {
          // Call the tool using MCP
          const toolResult = await mcpClient.callTool(name, args);
          
          // Add the tool call to the message history
          messages.push({
            role: 'assistant',
            content: null,
            tool_calls: [{ id, name, arguments: JSON.stringify(args) }]
          });
          
          // Add the tool result to the message history
          messages.push({
            role: 'tool',
            tool_call_id: id,
            content: toolResult.isError 
              ? `Error: ${toolResult.content[0].text}` 
              : toolResult.content[0].text
          });
        } catch (error) {
          console.error(`Error calling tool ${name}:`, error);
          
          // Add error information to the message history
          messages.push({
            role: 'tool',
            tool_call_id: id,
            content: `Error executing tool ${name}: ${error.message}`
          });
        }
      } else {
        // User didn't approve the tool call
        messages.push({
          role: 'system',
          content: `Tool call to ${name} was not approved by the user.`
        });
      }
    }
    
    // Send the updated conversation back to the LLM
    const newResponse = await llmClient.sendMessage(messages);
    
    // Recursively process the new response (which might contain more tool calls)
    return processLLMResponse(mcpClient, llmClient, newResponse, messages);
  }
  
  // If no tool calls, just return the LLM response
  return { llmResponse, messages };
}

// Helper function to get user approval for tool calls
async function getUserApproval(toolName, args) {
  // In a real application, you would show a UI dialog or prompt
  // For this example, we'll simulate user approval
  return new Promise(resolve => {
    console.log(`Approve tool call to ${toolName}?`);
    console.log('Arguments:', JSON.stringify(args, null, 2));
    console.log('Type "y" to approve, anything else to deny:');
    
    process.stdin.once('data', data => {
      const input = data.toString().trim().toLowerCase();
      resolve(input === 'y');
    });
  });
}
```

### Context Management and Re-injection

Managing conversation context is crucial for effective LLM interactions. Context includes:

* Previous user messages
* LLM responses
* Tool calls and their results
* Resource content that has been read

Here's how to manage context:

```typescript
async function manageConversationContext(mcpClient, llmClient, userQuery) {
  // Initialize conversation history
  let messages = [
    { role: 'user', content: userQuery }
  ];
  
  // Get available tools
  const tools = await prepareToolsForLLM(mcpClient);
  
  // Send initial request to LLM
  let llmResponse = await llmClient.sendMessage(messages, { tools });
  
  // Process response and handle any tool calls
  const result = await processLLMResponse(mcpClient, llmClient, llmResponse, messages);
  
  // The final messages array now contains the complete conversation context
  // including tool calls and results
  return result.messages;
}
```

### Human-in-the-Loop Approval

A key security feature of MCP is human-in-the-loop approval for tool execution. This prevents the LLM from taking potentially harmful actions without user consent:

```typescript
class SecurityManager {
  // Define security policies for different tools
  private toolPolicies = {
    'file-write': { requiresApproval: true, maxCallsPerMinute: 5 },
    'execute-command': { requiresApproval: true, maxCallsPerMinute: 2 },
    'read-data': { requiresApproval: false, maxCallsPerMinute: 20 }
  };
  
  // Rate limiting state
  private toolCallCounts: Record<string, { count: number, timestamp: number }> = {};
  
  // Check if a tool call should be allowed
  async checkToolCall(toolName: string, args: any): Promise<boolean> {
    // Get policy for this tool (default is to require approval)
    const policy = this.toolPolicies[toolName] || { requiresApproval: true, maxCallsPerMinute: 10 };
    
    // Check rate limits
    if (!this.checkRateLimit(toolName, policy.maxCallsPerMinute)) {
      console.error(`Rate limit exceeded for tool ${toolName}`);
      return false;
    }
    
    // If approval required, ask user
    if (policy.requiresApproval) {
      return await this.getUserApproval(toolName, args);
    }
    
    // No approval needed and rate limit not exceeded
    return true;
  }
  
  private checkRateLimit(toolName: string, maxCallsPerMinute: number): boolean {
    const now = Date.now();
    
    // Initialize count if not exists
    if (!this.toolCallCounts[toolName]) {
      this.toolCallCounts[toolName] = { count: 0, timestamp: now };
    }
    
    const record = this.toolCallCounts[toolName];
    
    // Reset counter if more than a minute has passed
    if (now - record.timestamp > 60000) {
      record.count = 0;
      record.timestamp = now;
    }
    
    // Increment counter
    record.count++;
    
    // Check if limit exceeded
    return record.count <= maxCallsPerMinute;
  }
  
  private async getUserApproval(toolName: string, args: any): Promise<boolean> {
    console.log(`Tool "${toolName}" requires approval.`);
    console.log('Arguments:', JSON.stringify(args, null, 2));
    console.log('Type "y" to approve, anything else to deny:');
    
    return new Promise(resolve => {
      process.stdin.once('data', data => {
        const input = data.toString().trim().toLowerCase();
        resolve(input === 'y');
      });
    });
  }
}
```

## 6. Advanced Client Features

### Sampling

Sampling allows MCP servers to request LLM completions through the client. This enables sophisticated AI behaviors while maintaining security and privacy:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

// Set up a handler for sampling requests
function setupSamplingHandler(client, llmClient) {
  client.setRequestHandler('sampling/complete', async (request) => {
    const { messages, options } = request.params;
    
    console.log('Server requested LLM sampling with messages:', messages);
    
    // Optionally get user approval for the sampling request
    const approved = await getUserApproval('LLM Sampling', { messages });
    
    if (!approved) {
      throw new Error('Sampling request was denied by the user');
    }
    
    // Send the messages to the LLM
    const completion = await llmClient.sendMessage(messages, options);
    
    // Return the completion to the server
    return {
      completion: {
        role: 'assistant',
        content: completion.content
      }
    };
  });
}
```

### Multi-Server Connections

A single application can maintain connections to multiple MCP servers, each providing different capabilities:

```typescript
class MCPConnectionManager {
  private connections: Map<string, Client> = new Map();
  
  async connectToServer(id: string, command: string, args: string[]): Promise<Client> {
    // Create a new client
    const client = new Client(
      { name: 'multi-server-client', version: '1.0.0' },
      { capabilities: { tools: {}, resources: {}, prompts: {} } }
    );
    
    // Connect using stdio transport
    const transport = new StdioClientTransport({ command, args });
    await client.connect(transport);
    await client.initialize();
    
    // Store the connection
    this.connections.set(id, client);
    
    console.log(`Connected to server: ${id}`);
    return client;
  }
  
  getClient(id: string): Client | undefined {
    return this.connections.get(id);
  }
  
  getAllClients(): Client[] {
    return Array.from(this.connections.values());
  }
  
  async disconnectAll(): Promise<void> {
    const disconnections = Array.from(this.connections.entries()).map(async ([id, client]) => {
      try {
        await client.close();
        console.log(`Disconnected from server: ${id}`);
      } catch (error) {
        console.error(`Error disconnecting from server ${id}:`, error);
      }
    });
    
    await Promise.all(disconnections);
    this.connections.clear();
  }
  
  // Get all available tools across all servers
  async getAllTools(): Promise<{ serverId: string, tools: any[] }[]> {
    const results = await Promise.all(
      Array.from(this.connections.entries()).map(async ([id, client]) => {
        try {
          const tools = await client.listTools();
          return { serverId: id, tools: tools.tools };
        } catch (error) {
          console.error(`Error getting tools from server ${id}:`, error);
          return { serverId: id, tools: [] };
        }
      })
    );
    
    return results;
  }
}
```

### Multi-Step (Agentic) Flows

MCP enables complex multi-step workflows where the LLM chains together multiple tool calls and resource reads:

```typescript
async function performComplexTask(mcpClient, llmClient, task: string) {
  // Prepare tools
  const tools = await prepareToolsForLLM(mcpClient);
  
  // Start conversation with task description
  let messages = [
    { role: 'user', content: `Perform this task: ${task}. You have access to tools to help you.` }
  ];
  
  // Maximum iterations to prevent infinite loops
  const MAX_ITERATIONS = 10;
  let iterations = 0;
  
  while (iterations < MAX_ITERATIONS) {
    iterations++;
    console.log(`Iteration ${iterations}/${MAX_ITERATIONS}`);
    
    // Get LLM response
    const response = await llmClient.sendMessage(messages, { tools });
    
    // Check if LLM wants to use tools
    if (response.tool_calls && response.tool_calls.length > 0) {
      // Process tool calls and update messages
      const result = await processLLMResponse(mcpClient, llmClient, response, messages);
      messages = result.messages;
    } else {
      // LLM has completed the task
      messages.push({
        role: 'assistant',
        content: response.content
      });
      
      console.log('Task completed successfully');
      break;
    }
  }
  
  if (iterations >= MAX_ITERATIONS) {
    console.log('Reached maximum iterations - task may not be complete');
    messages.push({
      role: 'system',
      content: 'Reached maximum number of iterations. Please provide a final response or summary.'
    });
    
    // Get final summary from LLM
    const finalResponse = await llmClient.sendMessage(messages);
    messages.push({
      role: 'assistant',
      content: finalResponse.content
    });
  }
  
  return {
    completed: iterations < MAX_ITERATIONS,
    finalMessages: messages,
    iterations
  };
}
```

### Resource Subscriptions

Clients can subscribe to resource updates to receive real-time notifications when resources change:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

async function setupResourceSubscription(client: Client, resourceUri: string) {
  try {
    // Subscribe to the resource
    await client.subscribeToResource(resourceUri);
    console.log(`Subscribed to resource: ${resourceUri}`);
    
    // Set up a handler for resource update notifications
    client.setNotificationHandler(
      'notifications/resources/updated',
      async (notification) => {
        console.log(`Resource updated: ${notification.uri}`);
        
        // Read the updated resource
        try {
          const updatedContent = await client.readResource(notification.uri);
          console.log('Updated content:', updatedContent.contents[0].text);
          
          // You could trigger application logic here
          // processResourceUpdate(updatedContent);
        } catch (error) {
          console.error(`Error reading updated resource ${notification.uri}:`, error);
        }
      }
    );
    
    return true;
  } catch (error) {
    console.error(`Error subscribing to resource ${resourceUri}:`, error);
    return false;
  }
}

// Later, when you want to unsubscribe
async function unsubscribeFromResource(client: Client, resourceUri: string) {
  try {
    await client.unsubscribeFromResource(resourceUri);
    console.log(`Unsubscribed from resource: ${resourceUri}`);
    return true;
  } catch (error) {
    console.error(`Error unsubscribing from resource ${resourceUri}:`, error);
    return false;
  }
}
```

### Roots

Roots allow clients to inform servers about relevant resource locations. They're declared during initialization:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function connectWithRoots() {
  const client = new Client(
    { name: 'root-aware-client', version: '1.0.0' },
    {
      capabilities: { tools: {}, resources: {}, prompts: {} },
      initializationOptions: {
        roots: [
          {
            uri: 'file:///path/to/project',
            name: 'Project Root'
          },
          {
            uri: 'git://github.com/username/repo',
            name: 'Git Repository'
          }
        ]
      }
    }
  );
  
  const transport = new StdioClientTransport({
    command: './server.js',
    args: []
  });
  
  await client.connect(transport);
  await client.initialize();
  
  return client;
}
```

### Progress Reporting

MCP supports progress reporting for long-running operations. Clients can receive progress updates through notifications:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

function setupProgressHandling(client: Client) {
  // Handler for progress notifications
  client.setNotificationHandler(
    'notifications/progress',
    (notification) => {
      const { token, value, message } = notification;
      
      console.log(`Progress for operation ${token}: ${value}% - ${message}`);
      
      // You might update a progress bar or other UI element
      // updateProgressUI(token, value, message);
    }
  );
  
  return client;
}

async function callToolWithProgress(client: Client, toolName: string, args: any) {
  // Generate a unique token for this operation
  const progressToken = `op-${Date.now()}`;
  
  // Call a tool that might take a while and report progress
  const result = await client.callTool(
    toolName,
    args,
    { 
      progress: { 
        token: progressToken // This token will be used in progress notifications
      } 
    }
  );
  
  return result;
}
```

### Handling List Endpoint Updates

Servers can notify clients when their capabilities change. Clients should handle these notifications to maintain an up-to-date view:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

class CapabilityManager {
  private tools: any[] = [];
  private resources: any[] = [];
  private resourceTemplates: any[] = [];
  private prompts: any[] = [];
  private client: Client;
  
  constructor(client: Client) {
    this.client = client;
    
    // Set up notification handlers for capability changes
    client.setNotificationHandler(
      'notifications/tools/list_changed',
      async () => {
        console.log('Tools list changed, refreshing...');
        await this.refreshTools();
      }
    );
    
    client.setNotificationHandler(
      'notifications/resources/list_changed',
      async () => {
        console.log('Resources list changed, refreshing...');
        await this.refreshResources();
      }
    );
    
    client.setNotificationHandler(
      'notifications/prompts/list_changed',
      async () => {
        console.log('Prompts list changed, refreshing...');
        await this.refreshPrompts();
      }
    );
  }
  
  async initialize() {
    // Fetch all capabilities
    await Promise.all([
      this.refreshTools(),
      this.refreshResources(),
      this.refreshPrompts()
    ]);
    
    return this;
  }
  
  async refreshTools() {
    try {
      const result = await this.client.listTools();
      this.tools = result.tools;
      return this.tools;
    } catch (error) {
      console.error('Error refreshing tools:', error);
      return this.tools;
    }
  }
  
  async refreshResources() {
    try {
      const result = await this.client.listResources();
      this.resources = result.resources;
      this.resourceTemplates = result.resourceTemplates;
      return { resources: this.resources, resourceTemplates: this.resourceTemplates };
    } catch (error) {
      console.error('Error refreshing resources:', error);
      return { resources: this.resources, resourceTemplates: this.resourceTemplates };
    }
  }
  
  async refreshPrompts() {
    try {
      const result = await this.client.listPrompts();
      this.prompts = result.prompts;
      return this.prompts;
    } catch (error) {
      console.error('Error refreshing prompts:', error);
      return this.prompts;
    }
  }
  
  getTools() { return this.tools; }
  getResources() { return this.resources; }
  getResourceTemplates() { return this.resourceTemplates; }
  getPrompts() { return this.prompts; }
}
```

## 7. Security and Best Practices

### Authentication and Authorization

When connecting to remote MCP servers, implement appropriate authentication:

```typescript
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

function createAuthenticatedTransport(serverUrl: string, apiKey: string) {
  const transport = new SSEClientTransport(new URL(serverUrl));
  
  // Add authentication headers
  transport.setHeaders({
    'Authorization': `Bearer ${apiKey}`,
    'X-Client-ID': 'my-client-app'
  });
  
  return transport;
}

// For services that use OAuth or other token-based auth
class TokenRefreshingTransport extends SSEClientTransport {
  private tokenProvider: () => Promise<string>;
  private baseHeaders: Record<string, string>;
  
  constructor(url: URL, tokenProvider: () => Promise<string>, baseHeaders: Record<string, string> = {}) {
    super(url);
    this.tokenProvider = tokenProvider;
    this.baseHeaders = baseHeaders;
  }
  
  async start(): Promise<void> {
    // Get a fresh token before starting
    const token = await this.tokenProvider();
    this.setHeaders({
      ...this.baseHeaders,
      'Authorization': `Bearer ${token}`
    });
    
    return super.start();
  }
  
  async handleTokenRefresh(): Promise<void> {
    const token = await this.tokenProvider();
    this.setHeaders({
      ...this.baseHeaders,
      'Authorization': `Bearer ${token}`
    });
  }
}
```

### Data Security

Implement robust data security practices:

1. **Input Validation:**
   * Validate all inputs (resource URIs, tool parameters) against defined schemas
   * Sanitize file paths and command arguments
   * Use strong typing and schema validation

```typescript
import { z } from 'zod';

// Define validation schemas for different types of inputs
const securitySchemas = {
  filePath: z.string().regex(/^[a-zA-Z0-9_\-./]+$/),
  command: z.string().max(100).refine(cmd => !cmd.includes("&&") && !cmd.includes(";"), {
    message: "Command contains forbidden characters"
  }),
  url: z.string().url(),
  email: z.string().email()
};

function validateSecuritySensitiveInput(type: keyof typeof securitySchemas, value: string): boolean {
  try {
    securitySchemas[type].parse(value);
    return true;
  } catch (error) {
    console.error(`Security validation failed for ${type}:`, error);
    return false;
  }
}

// Example usage for file paths
function validateAndReadResource(client: Client, resourceUri: string) {
  // Validate URI format
  if (!resourceUri.startsWith('file:///')) {
    throw new Error('Invalid file URI format');
  }
  
  // Extract file path from URI
  const filePath = resourceUri.substring('file:///'.length);
  
  // Validate file path
  if (!validateSecuritySensitiveInput('filePath', filePath)) {
    throw new Error('Invalid or potentially unsafe file path');
  }
  
  // Now safe to read
  return client.readResource(resourceUri);
}
```

2. **Sensitive Data Handling:**
   * Use TLS for network transport (especially for SSE)
   * Avoid logging sensitive information
   * Implement appropriate data retention policies

```typescript
class SecureLogger {
  private static readonly REDACTED = '[REDACTED]';
  
  // Define patterns for sensitive data
  private static readonly PATTERNS = [
    { name: 'API Key', regex: /["']?api[_-]?key["']?\s*[:=]\s*["']([a-zA-Z0-9_\-]+)["']/gi },
    { name: 'Password', regex: /["']?password["']?\s*[:=]\s*["']([^"']+)["']/gi },
    { name: 'Social Security Number', regex: /\b\d{3}[-\s]?\d{2}[-\s]?\d{4}\b/g }
  ];
  
  static sanitize(message: string): string {
    let sanitized = message;
    
    // Redact sensitive patterns
    for (const pattern of this.PATTERNS) {
      sanitized = sanitized.replace(pattern.regex, `${pattern.name}: ${this.REDACTED}`);
    }
    
    return sanitized;
  }
  
  static log(level: string, message: string, data?: any): void {
    const sanitizedMessage = this.sanitize(message);
    let sanitizedData = data;
    
    if (data) {
      if (typeof data === 'string') {
        sanitizedData = this.sanitize(data);
      } else if (typeof data === 'object') {
        sanitizedData = JSON.parse(this.sanitize(JSON.stringify(data)));
      }
    }
    
    console[level](sanitizedMessage, sanitizedData);
  }
}
```

### Error Handling

Implement robust error handling for all MCP operations:

```typescript
import { Client, ErrorCode } from '@modelcontextprotocol/sdk/client/index.js';

// Result type for error handling
interface Result<T, E = Error> {
  success: boolean;
  value?: T;
  error?: E;
}

// Wrapping client operations with proper error handling
class SafeMcpClient {
  private client: Client;
  
  constructor(client: Client) {
    this.client = client;
  }
  
  async callTool(toolName: string, args: any): Promise<Result<any>> {
    try {
      const result = await this.client.callTool(toolName, args);
      
      if (result.isError) {
        return {
          success: false,
          error: new Error(result.content[0].text)
        };
      }
      
      return {
        success: true,
        value: result.content
      };
    } catch (error: any) {
      // Handle different error types
      if (error.code === ErrorCode.MethodNotFound) {
        return {
          success: false,
          error: new Error(`Tool ${toolName} not found`)
        };
      } else if (error.code === ErrorCode.InvalidParams) {
        return {
          success: false,
          error: new Error(`Invalid parameters for tool ${toolName}: ${error.message}`)
        };
      } else {
        return {
          success: false,
          error: error
        };
      }
    }
  }
  
  async readResource(resourceUri: string): Promise<Result<any>> {
    try {
      const result = await this.client.readResource(resourceUri);
      return {
        success: true,
        value: result.contents
      };
    } catch (error: any) {
      return {
        success: false,
        error: error
      };
    }
  }
  
  // Implement other client methods with similar error handling
}

// Example usage
async function safeToolCall(client: Client, toolName: string, args: any) {
  const safeClient = new SafeMcpClient(client);
  const result = await safeClient.callTool(toolName, args);
  
  if (result.success) {
    console.log('Tool result:', result.value);
    return result.value;
  } else {
    console.error('Tool error:', result.error.message);
    // Handle error appropriately
    return null;
  }
}
```

For transport errors and disconnections, implement reconnection logic:

```typescript
class ReconnectingClient {
  private client: Client;
  private transportFactory: () => any;
  private isConnecting: boolean = false;
  private reconnectAttempts: number = 0;
  private maxReconnectAttempts: number = 5;
  private reconnectDelay: number = 1000; // 1 second
  
  constructor(client: Client, transportFactory: () => any) {
    this.client = client;
    this.transportFactory = transportFactory;
    
    // Set up error handler
    this.client.onError((error) => {
      console.error('Client error:', error);
      this.handleDisconnection();
    });
  }
  
  async connect() {
    if (this.isConnecting) return;
    
    this.isConnecting = true;
    try {
      const transport = this.transportFactory();
      await this.client.connect(transport);
      await this.client.initialize();
      
      console.log('Connected successfully');
      this.reconnectAttempts = 0;
      this.isConnecting = false;
      return true;
    } catch (error) {
      console.error('Connection error:', error);
      this.isConnecting = false;
      this.handleDisconnection();
      return false;
    }
  }
  
  private async handleDisconnection() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }
    
    this.reconnectAttempts++;
    const delay = this.reconnectDelay * Math.pow(2, this.reconnectAttempts - 1); // Exponential backoff
    
    console.log(`Attempting to reconnect in ${delay}ms (attempt ${this.reconnectAttempts})`);
    
    setTimeout(() => {
      this.connect();
    }, delay);
  }
  
  getClient() {
    return this.client;
  }
  
  async close() {
    this.maxReconnectAttempts = 0; // Prevent reconnection
    if (this.client) {
      await this.client.close();
    }
  }
}
```

### General Best Practices

1. **Principle of Least Privilege:**
   * Only request necessary capabilities from servers
   * Implement user permission checks for sensitive operations
   * Use fine-grained access controls

2. **Defense in Depth:**
   * Implement multiple layers of security
   * Don't rely solely on server-side security
   * Validate all server responses

3. **Logging and Monitoring:**
   * Implement comprehensive logging
   * Monitor connection status
   * Track resource usage and performance

4. **Error Handling:**
   * Use a consistent error handling strategy
   * Provide meaningful error messages
   * Implement proper cleanup after errors

5. **Testing and Validation:**
   * Test with a variety of MCP servers
   * Validate all inputs and outputs
   * Test error conditions and recovery

6. **Performance Considerations:**
   * Use appropriate timeouts
   * Implement caching where appropriate
   * Optimize resource usage

7. **Security Reviews:**
   * Regularly review security practices
   * Keep dependencies updated
   * Follow secure coding guidelines

## 8. Troubleshooting and Resources

### Debugging Tools

Several tools are available for debugging MCP clients:

* **MCP Inspector:** An interactive debugging interface for testing MCP servers and clients. See the [MCP Inspector guide](https://github.com/modelcontextprotocol/inspector) for details.

* **Claude Desktop Developer Tools:** Useful for integration testing, log collection, and accessing Chrome DevTools within Claude Desktop.
  * To enable developer tools, create a `developer_settings.json` file with `{"allowDevTools": true}` in `~/Library/Application Support/Claude/`.
  * Open DevTools with Command-Option-Shift-i.

* **Client Logging:** Implement custom logging within your client to track errors, performance, and other relevant information.

### Common Issues and Solutions

* **Connection Issues:**
  * **Problem:** Client fails to connect to the server.
  * **Solutions:**
    * Check if the server is running.
    * Verify the server path or URL.
    * Check for network connectivity issues.
    * Examine server logs for error messages.

* **Tool Execution Errors:**
  * **Problem:** Tool calls fail with errors.
  * **Solutions:**
    * Validate input parameters match the tool's schema.
    * Check for proper error handling in the tool implementation.
    * Verify that the tool exists on the server.
    * Look for permission or access issues.

* **Resource Access Problems:**
  * **Problem:** Unable to read resources.
  * **Solutions:**
    * Verify the resource URI format.
    * Check if the resource exists.
    * Look for permission issues.
    * Verify network connectivity for remote resources.

* **LLM Integration Issues:**
  * **Problem:** LLM isn't using tools correctly.
  * **Solutions:**
    * Ensure tool definitions are correctly formatted for your LLM provider.
    * Verify that context management is working properly.
    * Check for message format issues.
    * Look for token limit problems.

### Implementing Logging

* **Client-Side Logging:**
  * Log connection events, capability discovery, tool calls, resource access, and error conditions.
  * Use structured logging with timestamps, severity levels, and context.
  * Implement log levels (debug, info, warn, error) to control verbosity.

```typescript
class McpClientLogger {
  private logLevel: 'debug' | 'info' | 'warn' | 'error' = 'info';
  private client: Client;
  
  constructor(client: Client, logLevel?: 'debug' | 'info' | 'warn' | 'error') {
    this.client = client;
    if (logLevel) this.logLevel = logLevel;
    
    // Set up client event handlers
    this.setupLogging();
  }
  
  private setupLogging() {
    // Log all requests
    this.client.onRequest((method, params) => {
      this.log('debug', `Request: ${method}`, params);
    });
    
    // Log all responses
    this.client.onResponse((method, result) => {
      this.log('debug', `Response: ${method}`, result);
    });
    
    // Log errors
    this.client.onError((error) => {
      this.log('error', 'Client error:', error);
    });
    
    // Log notifications
    this.client.onNotification((method, params) => {
      this.log('debug', `Notification: ${method}`, params);
    });
  }
  
  private log(level: 'debug' | 'info' | 'warn' | 'error', message: string, data?: any) {
    const levels = { debug: 0, info: 1, warn: 2, error: 3 };
    if (levels[level] >= levels[this.logLevel]) {
      const timestamp = new Date().toISOString();
      const logMessage = `[${timestamp}] [${level.toUpperCase()}] ${message}`;
      
      if (data) {
        console[level](logMessage, data);
      } else {
        console[level](logMessage);
      }
    }
  }
  
  debug(message: string, data?: any) { this.log('debug', message, data); }
  info(message: string, data?: any) { this.log('info', message, data); }
  warn(message: string, data?: any) { this.log('warn', message, data); }
  error(message: string, data?: any) { this.log('error', message, data); }
  
  setLogLevel(level: 'debug' | 'info' | 'warn' | 'error') {
    this.logLevel = level;
  }
}
```

### Debugging Workflow

1. **Initial Development:** Use the MCP Inspector for basic testing, implement core functionality, and add logging.
2. **Integration Testing:** Test with actual MCP servers, monitor logs, and check error handling.
3. **LLM Integration Testing:** Test the complete flow with LLM integration, verify tool execution, and check context management.

**Testing Workflow:**

1. Start with isolated testing of MCP client functions.
2. Test connection and capability discovery.
3. Test individual tool calls and resource access.
4. Test error handling and recovery.
5. Test complete LLM integration.

### Community Resources and Support

* **GitHub Issues:** Report bugs and request features on the [MCP TypeScript SDK repository](https://github.com/modelcontextprotocol/typescript-sdk).
* **GitHub Discussions:** Ask questions and discuss MCP development.
* **Documentation:** [Model Context Protocol documentation](https://modelcontextprotocol.io)

## 9. Example Implementations

This section provides complete example implementations to help you understand how to build practical MCP clients.

### Basic LLM Integration

This example shows a simple integration between an MCP client and an LLM:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

// Hypothetical LLM client class
class LLMClient {
  async sendMessage(messages, options = {}) {
    // Implementation depends on your LLM provider
    console.log('Sending to LLM:', { messages, options });
    // Simulated response
    return {
      id: 'msg_123',
      role: 'assistant',
      content: 'This is a sample response from the LLM.',
      tool_calls: [] // Will contain tool call requests if any
    };
  }
}

async function runBasicLLMIntegration() {
  // Set up MCP client
  const mcpClient = new Client(
    { name: 'BasicIntegrationClient', version: '1.0.0' },
    { capabilities: { tools: {}, resources: {}, prompts: {} } }
  );
  
  // Connect to MCP server
  const transport = new StdioClientTransport({
    command: './server.js',
    args: []
  });
  
  await mcpClient.connect(transport);
  await mcpClient.initialize();
  
  // Set up LLM client
  const llmClient = new LLMClient();
  
  // Discover available tools
  const toolsResult = await mcpClient.listTools();
  console.log('Available tools:', toolsResult.tools);
  
  // Format tools for LLM
  const formattedTools = toolsResult.tools.map(tool => ({
    name: tool.name,
    description: tool.description || `Tool: ${tool.name}`,
    parameters: tool.inputSchema
  }));
  
  // Start conversation
  const userQuery = 'What tools are available?';
  const messages = [
    { role: 'user', content: userQuery }
  ];
  
  // Send to LLM with tool definitions
  const llmResponse = await llmClient.sendMessage(messages, {
    tools: formattedTools
  });
  
  // Process LLM response
  console.log('LLM response:', llmResponse);
  
  // Handle any tool calls (if present)
  if (llmResponse.tool_calls && llmResponse.tool_calls.length > 0) {
    for (const toolCall of llmResponse.tool_calls) {
      console.log(`LLM wants to call tool: ${toolCall.name}`);
      
      // Execute the tool
      const toolResult = await mcpClient.callTool(toolCall.name, toolCall.arguments);
      console.log('Tool result:', toolResult);
      
      // Add to messages and continue conversation
      messages.push({
        role: 'assistant',
        content: null,
        tool_calls: [toolCall]
      });
      
      messages.push({
        role: 'tool',
        tool_call_id: toolCall.id,
        content: toolResult.content[0].text
      });
      
      // Send updated messages to LLM
      const followUpResponse = await llmClient.sendMessage(messages, {
        tools: formattedTools
      });
      
      console.log('Follow-up response:', followUpResponse);
    }
  }
  
  // Clean up
  await mcpClient.close();
}

runBasicLLMIntegration().catch(console.error);
```

### Multi-Server Client Manager

This example demonstrates managing connections to multiple MCP servers:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

class MultiServerManager {
  private servers: Map<string, { 
    client: Client, 
    config: any,
    capabilities: { 
      tools: any[], 
      resources: any[], 
      resourceTemplates: any[],
      prompts: any[] 
    }
  }> = new Map();
  
  async addServer(id: string, config: any): Promise<boolean> {
    try {
      // Create client
      const client = new Client(
        { name: 'MultiServerClient', version: '1.0.0' },
        { capabilities: { tools: {}, resources: {}, prompts: {} } }
      );
      
      // Create appropriate transport
      let transport;
      if (config.type === 'stdio') {
        transport = new StdioClientTransport({
          command: config.command,
          args: config.args || []
        });
      } else if (config.type === 'sse') {
        transport = new SSEClientTransport(new URL(config.url));
        if (config.headers) {
          transport.setHeaders(config.headers);
        }
      } else {
        throw new Error(`Unsupported transport type: ${config.type}`);
      }
      
      // Connect and initialize
      await client.connect(transport);
      await client.initialize();
      
      // Discover capabilities
      const [tools, resources, prompts] = await Promise.all([
        client.listTools(),
        client.listResources(),
        client.listPrompts()
      ]);
      
      // Store server information
      this.servers.set(id, {
        client,
        config,
        capabilities: {
          tools: tools.tools,
          resources: resources.resources,
          resourceTemplates: resources.resourceTemplates,
          prompts: prompts.prompts
        }
      });
      
      console.log(`Server ${id} added successfully`);
      return true;
    } catch (error) {
      console.error(`Error adding server ${id}:`, error);
      return false;
    }
  }
  
  getClient(id: string): Client | null {
    return this.servers.get(id)?.client || null;
  }
  
  getServerInfo(id: string): any {
    return this.servers.get(id);
  }
  
  getServers(): string[] {
    return Array.from(this.servers.keys());
  }
  
  async removeServer(id: string): Promise<boolean> {
    const server = this.servers.get(id);
    if (!server) return false;
    
    try {
      await server.client.close();
      this.servers.delete(id);
      console.log(`Server ${id} removed successfully`);
      return true;
    } catch (error) {
      console.error(`Error removing server ${id}:`, error);
      return false;
    }
  }
  
  // Get all tools across all servers
  getAllTools(): { serverId: string, tool: any }[] {
    const allTools: { serverId: string, tool: any }[] = [];
    
    for (const [id, server] of this.servers.entries()) {
      for (const tool of server.capabilities.tools) {
        allTools.push({ serverId: id, tool });
      }
    }
    
    return allTools;
  }
  
  // Find a specific tool across all servers
  findTool(toolName: string): { serverId: string, tool: any } | null {
    for (const [id, server] of this.servers.entries()) {
      const tool = server.capabilities.tools.find(t => t.name === toolName);
      if (tool) return { serverId: id, tool };
    }
    return null;
  }
  
  // Execute a tool on the appropriate server
  async executeTool(toolName: string, args: any): Promise<any> {
    const toolInfo = this.findTool(toolName);
    if (!toolInfo) {
      throw new Error(`Tool ${toolName} not found on any server`);
    }
    
    const client = this.getClient(toolInfo.serverId);
    if (!client) {
      throw new Error(`Client for server ${toolInfo.serverId} not found`);
    }
    
    console.log(`Executing tool ${toolName} on server ${toolInfo.serverId}`);
    return await client.callTool(toolName, args);
  }
  
  async close(): Promise<void> {
    const closePromises = Array.from(this.servers.entries()).map(async ([id, server]) => {
      try {
        await server.client.close();
        console.log(`Server ${id} closed successfully`);
      } catch (error) {
        console.error(`Error closing server ${id}:`, error);
      }
    });
    
    await Promise.all(closePromises);
    this.servers.clear();
  }
}

// Example usage
async function runMultiServerExample() {
  const manager = new MultiServerManager();
  
  // Add servers
  await manager.addServer('calculator', {
    type: 'stdio',
    command: './calculator-server.js'
  });
  
  await manager.addServer('fileSystem', {
    type: 'stdio',
    command: './filesystem-server.js'
  });
  
  // List all available tools
  const allTools = manager.getAllTools();
  console.log('All available tools:', allTools);
  
  // Execute a tool
  try {
    const result = await manager.executeTool('add', { a: 5, b: 3 });
    console.log('Result:', result);
  } catch (error) {
    console.error('Error executing tool:', error);
  }
  
  // Clean up
  await manager.close();
}

runMultiServerExample().catch(console.error);
```

### Streaming Response Handler

This example shows how to handle streaming responses for long-running operations:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';
import { EventEmitter } from 'events';

class StreamingResponseHandler extends EventEmitter {
  private client: Client;
  private activeOperations: Map<string, {
    progress: number,
    status: string,
    startTime: number
  }> = new Map();
  
  constructor(client: Client) {
    super();
    this.client = client;
    
    // Set up notification handler for progress updates
    this.client.setNotificationHandler(
      'notifications/progress',
      (notification) => {
        const { token, value, message } = notification;
        
        // Update operation status
        const operation = this.activeOperations.get(token);
        if (operation) {
          operation.progress = value;
          operation.status = message || operation.status;
          
          // Emit progress event
          this.emit('progress', {
            token,
            progress: value,
            status: message,
            elapsedMs: Date.now() - operation.startTime
          });
          
          // If complete, emit completion event
          if (value === 100) {
            this.emit('complete', {
              token,
              elapsedMs: Date.now() - operation.startTime
            });
            
            // Clean up
            this.activeOperations.delete(token);
          }
        }
      }
    );
  }
  
  async callToolWithProgress(toolName: string, args: any): Promise<any> {
    // Generate operation token
    const token = `op-${Date.now()}-${Math.random().toString(36).substring(2, 7)}`;
    
    // Register the operation
    this.activeOperations.set(token, {
      progress: 0,
      status: 'Starting',
      startTime: Date.now()
    });
    
    // Emit start event
    this.emit('start', { token, toolName, args });
    
    try {
      // Call the tool with progress tracking
      const result = await this.client.callTool(
        toolName,
        args,
        { progress: { token } }
      );
      
      // Emit result event
      this.emit('result', {
        token,
        result,
        elapsedMs: Date.now() - this.activeOperations.get(token)?.startTime || 0
      });
      
      // Clean up if still active
      if (this.activeOperations.has(token)) {
        this.activeOperations.delete(token);
      }
      
      return result;
    } catch (error) {
      // Emit error event
      this.emit('error', {
        token,
        error,
        elapsedMs: Date.now() - this.activeOperations.get(token)?.startTime || 0
      });
      
      // Clean up
      this.activeOperations.delete(token);
      
      throw error;
    }
  }
  
  getActiveOperations(): Array<{ token: string, progress: number, status: string, elapsedMs: number }> {
    return Array.from(this.activeOperations.entries()).map(([token, op]) => ({
      token,
      progress: op.progress,
      status: op.status,
      elapsedMs: Date.now() - op.startTime
    }));
  }
}

// Example usage
async function runStreamingExample() {
  // Set up client
  const client = new Client(
    { name: 'StreamingClient', version: '1.0.0' },
    { capabilities: { tools: {}, resources: {}, prompts: {} } }
  );
  
  const transport = new StdioClientTransport({
    command: './long-running-server.js',
    args: []
  });
  
  await client.connect(transport);
  await client.initialize();
  
  // Create streaming handler
  const streamingHandler = new StreamingResponseHandler(client);
  
  // Set up event handlers
  streamingHandler.on('start', (data) => {
    console.log(`Operation ${data.token} started for tool ${data.toolName}`);
  });
  
  streamingHandler.on('progress', (data) => {
    console.log(`Operation ${data.token}: ${data.progress}% - ${data.status} (${data.elapsedMs}ms)`);
  });
  
  streamingHandler.on('result', (data) => {
    console.log(`Operation ${data.token} completed in ${data.elapsedMs}ms`);
    console.log('Result:', data.result);
  });
  
  streamingHandler.on('error', (data) => {
    console.error(`Operation ${data.token} failed after ${data.elapsedMs}ms`);
    console.error('Error:', data.error);
  });
  
  // Call a long-running tool
  try {
    const result = await streamingHandler.callToolWithProgress('long-process', {
      duration: 5000, // 5 seconds
      steps: 10
    });
    
    console.log('Final result:', result);
  } catch (error) {
    console.error('Error:', error);
  } finally {
    await client.close();
  }
}

runStreamingExample().catch(console.error);
```

### Complete LLM Application

This example shows a complete LLM application that integrates MCP with an LLM service and includes user interaction:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';
import readline from 'readline';

// Hypothetical LLM client
class LLMClient {
  async sendMessage(messages, options = {}) {
    console.log('Sending to LLM...');
    
    // Simulate LLM response
    // In a real application, you would call your LLM API here
    if (messages[messages.length - 1].role === 'user' && 
        messages[messages.length - 1].content.toLowerCase().includes('list')) {
      // Simulate a tool call request
      return {
        id: 'msg_123',
        role: 'assistant',
        content: null,
        tool_calls: [
          {
            id: 'call_1',
            name: 'list_files',
            arguments: { directory: '.' }
          }
        ]
      };
    }
    
    // Normal response
    return {
      id: 'msg_123',
      role: 'assistant',
      content: 'I\'m a simulated LLM response. Ask about listing files to see tool usage.'
    };
  }
}

class MCPLLMApplication {
  private mcpClient: Client;
  private llmClient: LLMClient;
  private messages: any[] = [];
  private toolDefinitions: any[] = [];
  private rl: readline.Interface;
  
  constructor() {
    this.rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });
  }
  
  async initialize() {
    console.log('Initializing MCP client...');
    
    // Create and connect MCP client
    this.mcpClient = new Client(
      { name: 'CompleteLLMApp', version: '1.0.0' },
      { capabilities: { tools: {}, resources: {}, prompts: {} } }
    );
    
    const transport = new StdioClientTransport({
      command: './file-server.js',
      args: []
    });
    
    await this.mcpClient.connect(transport);
    await this.mcpClient.initialize();
    
    // Create LLM client
    this.llmClient = new LLMClient();
    
    // Discover tools
    console.log('Discovering tools...');
    const toolsResult = await this.mcpClient.listTools();
    
    // Format tools for LLM
    this.toolDefinitions = toolsResult.tools.map(tool => ({
      name: tool.name,
      description: tool.description || `Tool: ${tool.name}`,
      parameters: tool.inputSchema
    }));
    
    console.log(`Discovered ${this.toolDefinitions.length} tools`);
    
    return this;
  }
  
  async runConversation() {
    console.log('\nWelcome to the MCP LLM Application!');
    console.log('Type your messages below. Type "exit" to quit.\n');
    
    while (true) {
      const userMessage = await this.promptUser('You: ');
      
      if (userMessage.toLowerCase() === 'exit') {
        break;
      }
      
      // Add user message to history
      this.messages.push({
        role: 'user',
        content: userMessage
      });
      
      // Send to LLM
      const llmResponse = await this.llmClient.sendMessage(this.messages, {
        tools: this.toolDefinitions
      });
      
      // Check for tool calls
      if (llmResponse.tool_calls && llmResponse.tool_calls.length > 0) {
        console.log('\nAssistant wants to use tools:');
        
        // Process each tool call
        for (const toolCall of llmResponse.tool_calls) {
          console.log(`- ${toolCall.name} with args: ${JSON.stringify(toolCall.arguments)}`);
          
          // Ask for approval
          const approved = await this.promptYesNo('Approve tool execution? (y/n): ');
          
          if (approved) {
            console.log(`Executing tool ${toolCall.name}...`);
            
            try {
              // Call the tool
              const toolResult = await this.mcpClient.callTool(toolCall.name, toolCall.arguments);
              
              // Add tool call to history
              this.messages.push({
                role: 'assistant',
                content: null,
                tool_calls: [
                  {
                    id: toolCall.id,
                    name: toolCall.name,
                    arguments: JSON.stringify(toolCall.arguments)
                  }
                ]
              });
              
              // Add tool result to history
              this.messages.push({
                role: 'tool',
                tool_call_id: toolCall.id,
                content: toolResult.isError ? 
                  `Error: ${toolResult.content[0].text}` : 
                  toolResult.content[0].text
              });
              
              console.log('Tool result:', toolResult.content[0].text);
            } catch (error) {
              console.error(`Error executing tool: ${error.message}`);
              
              // Add error to history
              this.messages.push({
                role: 'tool',
                tool_call_id: toolCall.id,
                content: `Error: ${error.message}`
              });
            }
          } else {
            console.log('Tool execution denied.');
            
            // Add denial to history
            this.messages.push({
              role: 'system',
              content: `Tool call to ${toolCall.name} was denied by the user.`
            });
          }
        }
        
        // Get final response with tool results
        const finalResponse = await this.llmClient.sendMessage(this.messages, {
          tools: this.toolDefinitions
        });
        
        console.log('\nAssistant:', finalResponse.content);
        
        // Add final response to history
        this.messages.push({
          role: 'assistant',
          content: finalResponse.content
        });
      } else {
        // Normal response
        console.log('\nAssistant:', llmResponse.content);
        
        // Add response to history
        this.messages.push({
          role: 'assistant',
          content: llmResponse.content
        });
      }
      
      console.log(); // Empty line for readability
    }
    
    console.log('Goodbye!');
  }
  
  async close() {
    this.rl.close();
    await this.mcpClient.close();
  }
  
  private promptUser(prompt: string): Promise<string> {
    return new Promise(resolve => {
      this.rl.question(prompt, answer => {
        resolve(answer);
      });
    });
  }
  
  private async promptYesNo(prompt: string): Promise<boolean> {
    const answer = await this.promptUser(prompt);
    return answer.toLowerCase().startsWith('y');
  }
}

// Run the application
async function main() {
  const app = new MCPLLMApplication();
  
  try {
    await app.initialize();
    await app.runConversation();
  } catch (error) {
    console.error('Application error:', error);
  } finally {
    await app.close();
  }
}

main();
```

These example implementations demonstrate how to build sophisticated MCP clients that integrate with LLMs, manage multiple servers, handle streaming responses, and provide complete user experiences.