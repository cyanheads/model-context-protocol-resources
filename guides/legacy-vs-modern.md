# Legacy vs. Modern MCP Servers

MCP revision `2026-07-28` removed the `initialize` handshake and protocol-level sessions. A server written for an earlier revision and a client written for this one can't talk to each other unless one side supports both. This page covers how the protocol got here, what changed, how the two eras interoperate, and what a server needs to move across.

For the protocol itself, start with the [overview](../README.md).

## Terminology

The [specification](https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning) defines three terms:

| Term | Meaning |
|:--|:--|
| **Legacy** | Revisions that open with an `initialize` handshake and hold a session: `2025-11-25` and earlier |
| **Modern** | Revisions that carry version, identity, and capabilities on every request: `2026-07-28` and later |
| **Dual-era** | An implementation that speaks both |

MCP revisions are named by date, not by semver. When people talk about "v1" and "v2" servers, they usually mean SDK release lines. In TypeScript, `@modelcontextprotocol/sdk` 1.x implements the legacy revisions, and the v2 packages (`@modelcontextprotocol/server`, `@modelcontextprotocol/client`) implement `2026-07-28`. Other SDKs version independently, so check each one's release notes for the revisions it supports.

## Revision history

| Revision | Era | Notable changes |
|:--|:--|:--|
| [`2024-11-05`](https://modelcontextprotocol.io/specification/2024-11-05) | Legacy | First release. stdio and HTTP+SSE transports; tools, resources, prompts; sampling, roots, and logging. |
| [`2025-03-26`](https://modelcontextprotocol.io/specification/2025-03-26/changelog) | Legacy | OAuth 2.1 authorization framework. Streamable HTTP replaces HTTP+SSE. Tool annotations, audio content, and the `completions` capability. JSON-RPC batching (removed in the next revision). |
| [`2025-06-18`](https://modelcontextprotocol.io/specification/2025-06-18/changelog) | Legacy | Structured tool output (`outputSchema`, `structuredContent`). Elicitation. Resource links in tool results. Servers classified as OAuth resource servers, and clients required to send RFC 8707 resource indicators. `MCP-Protocol-Version` header required on HTTP. `title` fields for display names. Batching removed. |
| [`2025-11-25`](https://modelcontextprotocol.io/specification/2025-11-25/changelog) | Legacy | Icons. URL-mode elicitation. Experimental tasks. Client ID Metadata Documents. Incremental scope consent through `WWW-Authenticate`. Tool name guidance. JSON Schema 2020-12 as the default dialect. Input validation failures returned as tool execution errors. Tool calling inside sampling. |
| [`2026-07-28`](https://modelcontextprotocol.io/specification/2026-07-28/changelog) | Modern | Stateless protocol: no `initialize` and no sessions. `server/discover`. Multi round-trip requests replace server-initiated requests. `subscriptions/listen`. Required `resultType` and cache hints. Tasks moved to an extension. Roots, Sampling, Logging, and Dynamic Client Registration deprecated. |

## What changed in 2026-07-28

| Area | Legacy | Modern |
|:--|:--|:--|
| Connection setup | `initialize` / `notifications/initialized` negotiates version and capabilities once | None. Every request carries `io.modelcontextprotocol/protocolVersion` and `clientCapabilities` in `_meta`. |
| Version mismatch | Settled during `initialize` | Per request: `UnsupportedProtocolVersionError` (`-32022`) lists supported versions, and the client retries. |
| Discovery | Server info and capabilities returned by `initialize` | `server/discover` (servers must implement it; clients may skip it) |
| Sessions | `Mcp-Session-Id` header; HTTP `DELETE` ends a session | None. State spanning calls uses explicit handles passed as tool arguments. |
| Lists | Could differ per session | Must not vary per connection; may vary by authorization |
| Server needs input | Server sends `sampling/createMessage`, `elicitation/create`, `roots/list` as its own requests | Server returns an `InputRequiredResult`, and the client retries with `inputResponses` and `requestState` (MRTR). |
| URL elicitation completion | `notifications/elicitation/complete` and `elicitationId` | Removed. The client learns the outcome by retrying, and servers correlate through `requestState`. |
| Change notifications | Standalone HTTP GET stream; `resources/subscribe` / `unsubscribe` | `subscriptions/listen` with an explicit filter, acknowledged by the server, each notification tagged with `subscriptionId` |
| Stream recovery | SSE event IDs and `Last-Event-ID` resumption | Not resumable. A broken stream loses the request, and the client re-issues it with a new ID. |
| Liveness | `ping` | Removed |
| Log level | `logging/setLevel` for the session | Per request through `io.modelcontextprotocol/logLevel` in `_meta`. Logging itself is deprecated. |
| Roots changes | `notifications/roots/list_changed` | Removed |
| Long-running work | Experimental core tasks with blocking `tasks/result` and `tasks/list` | `io.modelcontextprotocol/tasks` extension: poll `tasks/get`, send input with `tasks/update`; `tasks/list` removed |
| Results | No type field | Required `resultType`: `"complete"` or `"input_required"`. Clients treat an absent field as `"complete"`. |
| Caching | Not specified | `ttlMs` and `cacheScope` required on `server/discover`, list results, and `resources/read` |
| HTTP headers | `MCP-Protocol-Version` (from `2025-06-18`) | Also `Mcp-Method` on every request and `Mcp-Name` on `tools/call` / `resources/read` / `prompts/get`, validated against the body. `x-mcp-header` mirrors chosen tool parameters. |
| Error codes | Resource not found was `-32002` | Resource not found is `-32602`. `-32020` to `-32099` is reserved for the spec, and `-32000` to `-32019` is closed to new allocations. |
| Schemas | Restricted keyword set | Any JSON Schema 2020-12 keyword. `structuredContent` can be any JSON value. Network `$ref`s are not fetched by default. |
| Authorization | DCR as a registration path | Client ID Metadata Documents preferred and DCR deprecated. Clients validate `iss` (RFC 9207) and key stored credentials by issuer. |

## Deprecations

Deprecated features keep working for at least twelve months under the [feature lifecycle policy](https://modelcontextprotocol.io/community/feature-lifecycle). New implementations shouldn't adopt them. The authoritative list is the [deprecated features registry](https://modelcontextprotocol.io/specification/2026-07-28/deprecated).

| Feature | Deprecated in | Migrate to | Earliest removal |
|:--|:--|:--|:--|
| Roots | `2026-07-28` | Paths passed as tool parameters, resource URIs, or server configuration | First revision on or after 2027-07-28 |
| Sampling | `2026-07-28` | Calling an LLM provider API directly | First revision on or after 2027-07-28 |
| Logging | `2026-07-28` | `stderr` on stdio; OpenTelemetry for observability | First revision on or after 2027-07-28 |
| Dynamic Client Registration | `2026-07-28` | Client ID Metadata Documents | First revision on or after 2027-07-28 |
| `includeContext: "thisServer"` / `"allServers"` | `2025-11-25` | Omit it or use `"none"` | With Sampling |
| HTTP+SSE transport (`2024-11-05`) | `2025-03-26` | Streamable HTTP | Three months after SEP-2596 reaches Final |

## Compatibility

| Client | Server | Result |
|:--|:--|:--|
| Modern | Modern | Works. A version mismatch returns `UnsupportedProtocolVersionError`, and the client retries with a supported version. |
| Modern | Legacy | Fails. The client should detect the legacy server and show an actionable error. |
| Dual-era | Modern | Works in modern mode |
| Dual-era | Legacy | Works. The client falls back to `initialize`. |
| Legacy | Modern-only | Fails. stdio rejects `initialize`, and HTTP returns `400` for the missing headers. |
| Legacy | Dual-era | Works. `initialize` selects legacy behavior for that process or session. |
| Legacy | Legacy | Works under the negotiated legacy revision |

Legacy clients can't move forward on their own. Whether an older host keeps working depends entirely on the server staying dual-era. A modern-only server should name the versions it supports in the error it returns to `initialize`, because that message may be the only thing the user sees.

### How a dual-era client detects a server's era

- **stdio:** send `server/discover` with the preferred modern version before anything else. A `DiscoverResult` means modern. A recognized modern error such as `UnsupportedProtocolVersionError` means modern on a different version, so retry with a supported one and don't fall back. Any other error, or a timeout, means legacy, so fall back to `initialize`. Don't key the fallback to one specific error code, because legacy servers answer unknown pre-`initialize` methods inconsistently.
- **Streamable HTTP:** send a modern request. On `400`, read the body. A recognized modern JSON-RPC error means the server is modern, so fix the request or switch versions. An empty or unrecognized body means legacy, so fall back to `initialize` (and, past that, possibly the deprecated HTTP+SSE transport).
- Cache the result per server process (stdio) or origin (HTTP), and probe again if the assumption later fails.

## Migrating a server

1. **Stay dual-era during the transition.** Serve modern requests statelessly and keep answering `initialize` for legacy clients. A dual-era server can serve both eras on the same endpoint at once.
2. **Move session state into handles.** Anything keyed by a session ID or connection becomes an opaque handle returned by a tool and passed back as an argument. Check authorization against the handle on every call, generate it with enough entropy if the server is unauthenticated, state its lifetime in the creating tool's description, and return a tool error that names the expiry when it lapses.
3. **Make lists connection-independent.** `tools/list`, `resources/list`, and `prompts/list` may vary by the caller's authorization but nothing else. Return tools in a deterministic order.
4. **Convert server-initiated requests to MRTR.** Wherever a handler used to call elicitation, sampling, or roots mid-execution, return an `InputRequiredResult` and resume on the retry. Put the continuation in `requestState`, protect its integrity when it affects authorization or business logic, and bind it to the principal, a short expiry, and the originating request. Don't assume the client will retry. If the retry is missing something, ask again rather than returning an error.
5. **Replace subscriptions.** Drop `resources/subscribe` and the GET stream, and implement `subscriptions/listen`: acknowledge first, honor only the requested notification types, tag each with `subscriptionId`, and send periodic SSE comment lines as keep-alives on quiet streams.
6. **Implement discovery and caching.** Answer `server/discover` with supported versions, capabilities, `serverInfo`, and `instructions`. Put `ttlMs` and `cacheScope` on discover, list, and `resources/read` results.
7. **Update the HTTP layer.** Require and validate `MCP-Protocol-Version`, `Mcp-Method`, and `Mcp-Name` against the body (`400` with `-32020` on mismatch). Answer `GET` and `DELETE` with `405`. Ignore `Mcp-Session-Id` and `Last-Event-ID` on modern requests. Send `X-Accel-Buffering: no` on SSE responses so reverse proxies don't buffer them.
8. **Update error codes.** Return `-32602` for a missing resource, and use the spec-defined codes for header mismatch, missing client capability, and unsupported version.
9. **Retire deprecated features.** Log to `stderr` or OpenTelemetry instead of `notifications/message`. Call an LLM provider directly instead of sampling. Take paths as parameters or configuration instead of roots. Move long-running work to the Tasks extension.

In TypeScript, the SDK's [v2 migration guide](https://ts.sdk.modelcontextprotocol.io/v2/migration/upgrade-to-v2) covers the package split and API changes. [`@cyanheads/mcp-ts-core`](https://github.com/cyanheads/mcp-ts-core) builds on the v2 SDK and serves both eras from one server.
