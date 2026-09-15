# Model Context Protocol (MCP) Message Format

## Overview
MCP uses JSON-RPC 2.0 as its standardized message format for all client-server communications. This ensures interoperability, extensibility, and a well-defined error handling mechanism while keeping the protocol lightweight and easy to implement.

## JSON-RPC 2.0 Foundation

### Basic Structure
All MCP messages follow the JSON-RPC 2.0 specification:

```json
{
  "jsonrpc": "2.0",
  "id": "<identifier-or-null>",
  "method": "<method-name>",
  "params": {...}
}
```

**Required Fields:**
- `jsonrpc`: Must be exactly "2.0"
- `method`: String containing the method name to invoke
- `id`: Identifier for matching requests with responses (can be null for notifications)
- `params`: Structured value (object or array) holding method parameters

### Response Messages
Successful responses:
```json
{
  "jsonrpc": "2.0",
  "id": "<matching-request-id>",
  "result": {...}
}
```

Error responses:
```json
{
  "jsonrpc": "2.0",
  "id": "<matching-request-id>",
  "error": {
    "code": <integer>,
    "message": "<string>",
    "data": {...}  // Optional
  }
}
```

## MCP-Specific Message Types

### 1. Initialization Handshake
Establishes connection parameters and negotiates capabilities.

**Client → Server (initialize):**
```json
{
  "jsonrpc": "2.0",
  "id": "1",
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {},
      "resources": {},
      "prompts": {}
    },
    "clientInfo": {
      "name": "claude-desktop",
      "version": "0.1.0"
    }
  }
}
```

**Server → Client (initialized):**
```json
{
  "jsonrpc": "2.0",
  "id": "1",
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {
        "listChanged": true
      },
      "resources": {
        "subscribe": true,
        "listChanged": true
      },
      "prompts": {
        "listChanged": true
      }
    },
    "serverInfo": {
      "name": "filesystem-server",
      "version": "0.1.0"
    }
  }
}
```

### 2. Tool Interaction Messages

#### Tool Discovery
**Client → Server (tools/list):**
```json
{
  "jsonrpc": "2.0",
  "id": "2",
  "method": "tools/list"
}
```

**Server → Client (tools/list response):**
```json
{
  "jsonrpc": "2.0",
  "id": "2",
  "result": {
    "tools": [
      {
        "name": "read_file",
        "description": "Read the contents of a file",
        "inputSchema": {
          "type": "object",
          "properties": {
            "path": {
              "type": "string",
              "description": "Path to the file to read"
            }
          },
          "required": ["path"]
        }
      },
      {
        "name": "write_file",
        "description": "Write content to a file",
        "inputSchema": {
          "type": "object",
          "properties": {
            "path": {
              "type": "string",
              "description": "Path to the file to write"
            },
            "content": {
              "type": "string",
              "description": "Content to write to the file"
            }
          },
          "required": ["path", "content"]
        }
      }
    ]
  }
}
```

#### Tool Invocation
**Client → Server (tools/call):**
```json
{
  "jsonrpc": "2.0",
  "id": "3",
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": {
      "path": "/home/user/document.txt"
    }
  }
}
```

**Server → Client (tools/call response - success):**
```json
{
  "jsonrpc": "2.0",
  "id": "3",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Hello, World! This is the file content."
      }
    ],
    "isError": false
  }
}
```

**Server → Client (tools/call response - error):**
```json
{
  "jsonrpc": "2.0",
  "id": "3",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Error: File not found at path '/home/user/document.txt'"
      }
    ],
    "isError": true
  }
}
```

### 3. Resource Interaction Messages

#### Resource Discovery
**Client → Server (resources/list):**
```json
{
  "jsonrpc": "2.0",
  "id": "4",
  "method": "resources/list"
}
```

**Server → Client (resources/list response):**
```json
{
  "jsonrpc": "2.0",
  "id": "4",
  "result": {
    "resources": [
      {
        "uri": "file:///home/user/documents/",
        "name": "User Documents",
        "description": "Personal documents folder",
        "mimeType": "application/octet-stream"
      },
      {
        "uri": "database://main/users",
        "name": "User Database",
        "description": "SQLite database containing user information",
        "mimeType": "application/x-sqlite3"
      }
    ]
  }
}
```

#### Resource Reading
**Client → Server (resources/read):**
```json
{
  "jsonrpc": "2.0",
  "id": "5",
  "method": "resources/read",
  "params": {
    "uri": "file:///home/user/document.txt"
  }
}
```

**Server → Client (resources/read response):**
```json
{
  "jsonrpc": "2.0",
  "id": "5",
  "result": {
    "contents": [
      {
        "uri": "file:///home/user/document.txt",
        "mimeType": "text/plain",
        "text": "Hello, World! This is the file content."
      }
    ]
  }
}
```

#### Resource Subscriptions
**Client → Server (resources/subscribe):**
```json
{
  "jsonrpc": "2.0",
  "id": "6",
  "method": "resources/subscribe",
  "params": {
    "uri": "file:///home/user/logs/app.log"
  }
}
```

**Server → Client (resources/notification - update):**
```json
{
  "jsonrpc": "2.0",
  "method": "resources/updated",
  "params": {
    "uri": "file:///home/user/logs/app.log"
  }
}
```

### 4. Prompt Interaction Messages

#### Prompt Discovery
**Client → Server (prompts/list):**
```json
{
  "jsonrpc": "2.0",
  "id": "7",
  "method": "prompts/list"
}
```

**Server → Client (prompts/list response):**
```json
{
  "jsonrpc": "2.0",
  "id": "7",
  "result": {
    "prompts": [
      {
        "name": "summarize-text",
        "description": "Create a concise summary of provided text",
        "arguments": [
          {
            "name": "text",
            "description": "Text to summarize",
            "required": true
          },
          {
            "name": "maxLength",
            "description": "Maximum length of summary",
            "required": false
          }
        ]
      }
    ]
  }
}
```

#### Prompt Retrieval
**Client → Server (prompts/get):**
```json
{
  "jsonrpc": "2.0",
  "id": "8",
  "method": "prompts/get",
  "params": {
    "name": "summarize-text",
    "arguments": {
      "text": "This is a long article about artificial intelligence...",
      "maxLength": 100
    }
  }
}
```

**Server → Client (prompts/get response):**
```json
{
  "jsonrpc": "2.0",
  "id": "8",
  "result": {
    "description": "Create a concise summary of provided text",
    "messages": [
      {
        "role": "user",
        "content": "Please summarize the following text in under 100 characters: This is a long article about artificial intelligence..."
      }
    ]
  }
}
```

### 5. Notification Messages

#### Server-to-Client Notifications
**Resource Updates:**
```json
{
  "jsonrpc": "2.0",
  "method": "resources/updated",
  "params": {
    "uri": "file:///home/user/data.csv"
  }
}
```

**Tool Availability Changes:**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/list_changed",
  "params": {}
}
```

**Prompt Updates:**
```json
{
  "jsonrpc": "2.0",
  "method": "prompts/list_changed",
  "params": {}
}
```

#### Client-to-Server Notifications
**Ping/Keepalive:**
```json
{
  "jsonrpc": "2.0",
  "method": "ping",
  "params": {}
}
```

**Progress Updates (for long-running tools):**
```json
{
  "jsonrpc": "2.0",
  "method": "notification/progress",
  "params": {
    "progressToken": "long-operation-123",
    "progress": 0.75,
    "total": 100
  }
}
```

## Error Handling

### Standard JSON-RPC Errors
MCP adheres to JSON-RPC 2.0 error codes:
- `-32700`: Parse error (invalid JSON)
- `-32600`: Invalid Request (missing method, invalid params)
- `-32601`: Method not found
- `-32602`: Invalid params
- `-32603`: Internal error

### MCP-Specific Error Extensions
While MCP primarily uses standard JSON-RPC errors, implementations may include:
- `-32000 to -32099`: Server error (reserved for implementation-defined)
- `-32800 to -32899`: Server error (reserved for implementation-defined)

**Example MCP Error Response:**
```json
{
  "jsonrpc": "2.0",
  "id": "9",
  "error": {
    "code": -32603,
    "message": "Failed to establish database connection",
    "data": {
      "retryAfter": 5000,
      "possibleCauses": ["network", "credentials", "server_down"]
    }
  }
}
```

## Message Sequencing and Flow

### Typical Tool Invocation Flow
1. **Connection Establishment**: Client sends `initialize`, receives `initialized`
2. **Capability Discovery**: Client sends `tools/list`, receives tool catalog
3. **Tool Execution**: Client sends `tools/call` with parameters
4. **Result Processing**: Server executes tool and returns result
5. **Error Handling**: Any errors returned in standardized format
6. **Connection Maintenance**: Periodic ping/pong or automatic reconnection

### Streaming Large Results
For large responses, servers may:
1. Send initial response with partial data
2. Follow with progress notifications
3. Send final completion message
4. Allow client to cancel via specific methods

### Batch Processing
While JSON-RPC 2.0 supports batch messages, MCP implementations typically:
- Process messages sequentially for simplicity
- Use connection pooling for parallelism
- Implement queueing systems for high-throughput scenarios
- Avoid batches due to complexity in error correlation

## Transport-Specific Considerations

### stdio Transport
- Messages delimited by newlines
- Each line contains a complete JSON-RPC message
- No framing beyond line separation
- Ideal for local, same-process communication

### HTTP Transport
- POST requests with JSON-RPC body
- Content-Type: application/json
- HTTP status codes indicate transport-level issues
- JSON-RPC errors in response body for application-level issues
- Keep-alive connections for efficiency

### WebSocket Transport
- Full-duplex bidirectional communication
- Each WebSocket message contains one JSON-RPC packet
- Heartbeat mechanism via ping/pong frames
- Automatic reconnection logic

## Implementation Best Practices

### Message Validation
- Validate JSON-RPC version field
- Ensure method names conform to allowed patterns
- Validate params against JSON Schema when available
- Check ID types for response matching
- Reject messages with extra top-level fields (strict mode)

### Performance Optimization
- Reuse JSON parsers/serializers
- Pre-allocate buffers for common message sizes
- Implement message pooling for high-frequency operations
- Use binary JSON alternatives (MessagePack, CBOR) when beneficial
- Compress large payloads when bandwidth is constrained

### Security Considerations
- Validate and sanitize all input parameters
- Implement size limits to prevent DoS attacks
- Use timeouts to prevent resource exhaustion
- Log messages for audit trails (excluding sensitive data)
- Implement rate limiting per client connection

### Debugging and Tracing
- Log message exchanges with timestamps
- Correlate requests and responses using IDs
- Provide verbose modes for development
- Implement message replay capabilities for testing
- Offer pretty-printing for human-readable logs

## Extensibility and Versioning

### Backward Compatibility
- New methods should be optional
- Existing methods should not change signature
- New parameters should be optional with sensible defaults
- Deprecation warnings for planned removals

### Version Negotiation
- Clients specify desired protocol version in initialize
- Servers respond with supported version
- Fallback mechanisms for version mismatches
- Clear documentation of version-specific features

### Custom Extensions
- Vendor-specific methods use namespacing: `vendor/method`
- Custom error codes in reserved ranges
- Extension discovery via capability negotiation
- Fallback to standard methods when extensions unavailable