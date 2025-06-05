# Model Context Protocol Schema 

## Core Types

```json
{
  "RequestId": "string | number",
  "ProgressToken": "string | number", 
  "Cursor": "string",
  "Role": "user | assistant",
  "LogLevel": "debug | info | notice | warning | error | critical | alert | emergency"
}
```

## Content System

```json
{
  "Annotations": {
    "audience": ["user", "assistant"],
    "priority": 0.8
  },
  "TextContent": {
    "type": "text",
    "text": "content here",
    "annotations": { "Annotations" }
  },
  "ImageContent": {
    "type": "image", 
    "data": "base64_encoded_data",
    "mimeType": "image/png",
    "annotations": { "Annotations" }
  },
  "AudioContent": {
    "type": "audio",
    "data": "base64_encoded_data", 
    "mimeType": "audio/wav",
    "annotations": { "Annotations" }
  },
  "EmbeddedResource": {
    "type": "resource",
    "resource": { "TextResource | BlobResource" },
    "annotations": { "Annotations" }
  }
}
```

## Message Flow

```json
{
  "PromptMessage": {
    "role": "user | assistant",
    "content": { "TextContent | ImageContent | AudioContent | EmbeddedResource" }
  },
  "SamplingMessage": {
    "role": "user | assistant", 
    "content": { "TextContent | ImageContent | AudioContent" }
  }
}
```

## Protocol Foundation

```json
{
  "JSONRPCRequest": {
    "jsonrpc": "2.0",
    "id": "RequestId",
    "method": "string",
    "params": {
      "_meta": {
        "progressToken": "ProgressToken"
      }
    }
  },
  "JSONRPCResponse": {
    "jsonrpc": "2.0", 
    "id": "RequestId",
    "result": {},
    "_meta": {}
  },
  "JSONRPCError": {
    "jsonrpc": "2.0",
    "id": "RequestId", 
    "error": {
      "code": -32000,
      "message": "Error description",
      "data": {}
    }
  },
  "JSONRPCNotification": {
    "jsonrpc": "2.0",
    "method": "string",
    "params": {
      "_meta": {}
    }
  }
}
```

## Capabilities

```json
{
  "ClientCapabilities": {
    "roots": {
      "listChanged": true
    },
    "sampling": {},
    "experimental": {}
  },
  "ServerCapabilities": {
    "prompts": {
      "listChanged": true
    },
    "resources": {
      "listChanged": true,
      "subscribe": true
    },
    "tools": {
      "listChanged": true  
    },
    "logging": {},
    "completions": {},
    "experimental": {}
  }
}
```

## Initialization Protocol

```json
{
  "InitializeRequest": {
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "clientInfo": {
        "name": "client-name",
        "version": "1.0.0"
      },
      "capabilities": { "ClientCapabilities" }
    }
  },
  "InitializeResult": {
    "protocolVersion": "2024-11-05",
    "serverInfo": {
      "name": "server-name", 
      "version": "1.0.0"
    },
    "capabilities": { "ServerCapabilities" },
    "instructions": "Optional LLM hints"
  },
  "InitializedNotification": {
    "method": "notifications/initialized",
    "params": {}
  }
}
```

## Resource System

```json
{
  "Resource": {
    "name": "resource-name",
    "uri": "file://path/to/resource",
    "description": "What this resource contains",
    "mimeType": "text/plain",
    "size": 1024,
    "annotations": { "Annotations" }
  },
  "ResourceTemplate": {
    "name": "template-name",
    "uriTemplate": "file://path/{id}",
    "description": "Template description", 
    "mimeType": "text/plain",
    "annotations": { "Annotations" }
  },
  "ListResourcesRequest": {
    "method": "resources/list",
    "params": {
      "cursor": "optional_cursor"
    }
  },
  "ReadResourceRequest": {
    "method": "resources/read", 
    "params": {
      "uri": "file://path/to/resource"
    }
  },
  "SubscribeRequest": {
    "method": "resources/subscribe",
    "params": {
      "uri": "file://path/to/resource"
    }
  }
}
```

## Tool System

```json
{
  "Tool": {
    "name": "tool-name",
    "description": "What this tool does",
    "inputSchema": {
      "type": "object",
      "properties": {
        "param1": { "type": "string" },
        "param2": { "type": "number" }
      },
      "required": ["param1"]
    },
    "annotations": {
      "title": "Human readable title",
      "readOnlyHint": false,
      "destructiveHint": true,
      "idempotentHint": false,
      "openWorldHint": true
    }
  },
  "CallToolRequest": {
    "method": "tools/call",
    "params": {
      "name": "tool-name",
      "arguments": {
        "param1": "value1",
        "param2": 42
      }
    }
  },
  "CallToolResult": {
    "content": [
      { "type": "text", "text": "Tool output" }
    ],
    "isError": false,
    "_meta": {}
  }
}
```

## Prompt System

```json
{
  "PromptArgument": {
    "name": "arg-name",
    "description": "Argument description",
    "required": true
  },
  "Prompt": {
    "name": "prompt-name", 
    "description": "Prompt description",
    "arguments": [{ "PromptArgument" }]
  },
  "GetPromptRequest": {
    "method": "prompts/get",
    "params": {
      "name": "prompt-name",
      "arguments": {
        "arg1": "value1",
        "arg2": "value2"
      }
    }
  },
  "GetPromptResult": {
    "description": "Prompt description",
    "messages": [
      {
        "role": "user",
        "content": { "type": "text", "text": "Prompt content" }
      }
    ]
  }
}
```

## Sampling (LLM Integration)

```json
{
  "ModelHint": {
    "name": "claude-3-5-sonnet"
  },
  "ModelPreferences": {
    "costPriority": 0.3,
    "speedPriority": 0.5,
    "intelligencePriority": 0.9,
    "hints": [{ "ModelHint" }]
  },
  "CreateMessageRequest": {
    "method": "sampling/createMessage",
    "params": {
      "messages": [{ "SamplingMessage" }],
      "maxTokens": 1000,
      "systemPrompt": "You are a helpful assistant",
      "temperature": 0.7,
      "stopSequences": ["END"],
      "modelPreferences": { "ModelPreferences" },
      "metadata": {},
      "includeContext": "thisServer"
    }
  },
  "CreateMessageResult": {
    "role": "assistant",
    "content": { "type": "text", "text": "LLM response" },
    "model": "claude-3-5-sonnet-20241022",
    "stopReason": "stop_sequence"
  }
}
```

## Completion System

```json
{
  "CompleteRequest": {
    "method": "completion/complete",
    "params": {
      "ref": {
        "type": "ref/prompt",
        "name": "prompt-name"
      },
      "argument": {
        "name": "arg-name",
        "value": "partial_value"
      }
    }
  },
  "CompleteResult": {
    "completion": {
      "values": ["completion1", "completion2"],
      "total": 50,
      "hasMore": true
    }
  }
}
```

## Root System (File Access)

```json
{
  "Root": {
    "uri": "file:///path/to/directory",
    "name": "project-root"
  },
  "ListRootsRequest": {
    "method": "roots/list",
    "params": {}
  },
  "ListRootsResult": {
    "roots": [{ "Root" }]
  }
}
```

## Notifications

```json
{
  "ResourceListChangedNotification": {
    "method": "notifications/resources/list_changed",
    "params": {}
  },
  "ResourceUpdatedNotification": {
    "method": "notifications/resources/updated", 
    "params": {
      "uri": "file://path/to/resource"
    }
  },
  "ProgressNotification": {
    "method": "notifications/progress",
    "params": {
      "progressToken": "token123",
      "progress": 50,
      "total": 100,
      "message": "Processing..."
    }
  },
  "LoggingMessageNotification": {
    "method": "notifications/message",
    "params": {
      "level": "info",
      "data": "Log message",
      "logger": "server.component"
    }
  },
  "CancelledNotification": {
    "method": "notifications/cancelled",
    "params": {
      "requestId": "req123",
      "reason": "User cancelled"
    }
  }
}
```

## Pagination Pattern

```json
{
  "PaginatedRequest": {
    "method": "resources/list",
    "params": {
      "cursor": "opaque_cursor_string"
    }
  },
  "PaginatedResult": {
    "resources": [{ "Resource" }],
    "nextCursor": "next_cursor_string",
    "_meta": {}
  }
}
```

## Protocol Rules

> **Critical implementation requirements**

1. **Initialization**: `initialize` → `initialize_result` → `notifications/initialized` (required sequence)
2. **Progress**: Optional out-of-band notifications using `progressToken`
3. **Errors**: Tool errors in `CallToolResult.isError`, protocol errors as JSON-RPC errors
4. **Pagination**: Use opaque cursors, max reasonable page sizes
5. **Cancellation**: Either side can cancel with `requestId`
6. **Notifications**: No response expected, may arrive without subscription

## Implementation Notes

| Component | Constraint | Details |
|-----------|------------|---------|
| `_meta` fields | Reserved | Protocol extensions only |
| Binary data | Base64 | Always encoded in JSON strings |
| URI schemes | `file://` | Required for roots, others server-dependent |
| JSON Schema | Object only | Tool inputs must be object type |
| Versioning | Negotiated | Client/server agree on compatible version |
| Threading | Single connection | Async via JSON-RPC batching |

## Method Reference

### Client → Server

```json
{
  "Core": ["initialize", "ping"],
  "Resources": ["resources/list", "resources/templates/list", "resources/read", "resources/subscribe", "resources/unsubscribe"],
  "Prompts": ["prompts/list", "prompts/get"],
  "Tools": ["tools/list", "tools/call"],
  "Completion": ["completion/complete"],
  "Logging": ["logging/setLevel"]
}
```

### Server → Client

```json
{
  "Core": ["ping"],
  "Sampling": ["sampling/createMessage"],
  "Roots": ["roots/list"]
}
```

---
