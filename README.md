# @qverisai/sdk

Official Qveris MCP Server SDK — Dynamically search and execute tools via natural language.

[![npm version](https://img.shields.io/npm/v/@qverisai/sdk.svg)](https://www.npmjs.com/package/@qverisai/sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview

This SDK provides a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that enables LLMs to discover and execute third-party tools through the Qveris API. With just two simple tools, your AI assistant can:

- **Search** for tools using natural language queries
- **Execute** any discovered tool with the appropriate parameters

## Installation

```bash
# Using npx (recommended for MCP)
npx @qverisai/sdk

# Or install globally
npm add -g @qverisai/sdk
```

## Quick Start

### 1. Get Your API Key

Visit [Qveris](https://qveris.ai) to get your API key.

### 2. Configure Your MCP Client

Add the Qveris server to your MCP client configuration:

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "qveris": {
      "command": "npx",
      "args": ["@qverisai/sdk"],
      "env": {
        "QVERIS_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

**Cursor** (Settings → MCP Servers):

```json
{
  "mcpServers": {
    "qveris": {
      "command": "npx",
      "args": ["@qverisai/sdk"],
      "env": {
        "QVERIS_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### 3. Start Using

Once configured, You could add this to system prompt:

> "You can use qveris MCP Server to dynamically search and execute tools to help the user. First think about what kind of tools might be useful to accomplish the user's task. Then use the search_tools tool with query describing the capability of the tool, not what params you want to pass to the tool later. Then call a suitable searched tool using the execute_tool tool, passing parameters to the searched tool through params_to_tool. You could reference the examples given if any for each tool. You may call make multiple tool calls in a single response."

Then your AI assistant can search for and execute tools:

> "Find me a weather tool and get the current weather in Tokyo"

The assistant will:
1. Call `search_tools` with query "weather"
2. Review the results and select an appropriate tool
3. Call `execute_tool` with the tool_id and parameters

## Available Tools

### `search_tools`

Search for available tools based on natural language queries.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | ✓ | Natural language description of the capability you need |
| `limit` | number | | Max results to return (1-100, default: 20) |
| `session_id` | string | | Session identifier for tracking (auto-generated if omitted) |

**Example:**

```json
{
  "query": "send email notification",
  "limit": 5
}
```

### `execute_tool`

Execute a discovered tool with specific parameters.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tool_id` | string | ✓ | Tool ID from search results |
| `search_id` | string | ✓ | Search ID from the search that found this tool |
| `params_to_tool` | string | ✓ | JSON string of parameters to pass to the tool |
| `session_id` | string | | Session identifier (auto-generated if omitted) |
| `max_data_size` | number | | Max response size in bytes (default: 20480) |

**Example:**

```json
{
  "tool_id": "openweathermap_current_weather",
  "search_id": "abc123",
  "params_to_tool": "{\"city\": \"London\", \"units\": \"metric\"}"
}
```

## Session Management

Providing a consistent `session_id` in a same user session in any tool call enables:
- Consistent user tracking across multiple tool calls
- Better analytics and usage patterns
- Improved tool recommendations over time

If not provided, the SDK automatically generates and maintains a session ID for the lifetime of the server process. However, this result in a much larger granularity of user sessions.

## Response Handling

### Successful Execution

```json
{
  "execution_id": "exec-123",
  "tool_id": "openweathermap_current_weather",
  "success": true,
  "result": {
    "data": {
      "temperature": 15.5,
      "humidity": 72,
      "description": "partly cloudy"
    }
  },
  "execution_time": 0.847
}
```

### Large Responses

When tool output exceeds `max_data_size`, you'll receive:

```json
{
  "result": {
    "message": "Result content is too long...",
    "truncated_content": "[[1678233600000, \"22198.56...",
    "full_content_file_url": "https://..."
  }
}
```

The `full_content_file_url` is valid for 120 minutes.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `QVERIS_API_KEY` | ✓ | Your Qveris API key |

## Requirements

- Node.js 18.0.0 or higher
- A valid Qveris API key

## Development

```bash
# Clone the repository
git clone https://github.com/qverisai/sdk.git
cd sdk

# Install dependencies
npm install

# Build
npm build

# Run locally
QVERIS_API_KEY=your-key node dist/index.js
```

## License

MIT © [QverisAI](https://github.com/qverisai)

## Support

- 🐛 [Issue Tracker](https://github.com/qverisai/sdk/issues)
- 💬 Contact: contact@qveris.ai

