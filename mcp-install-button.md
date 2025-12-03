# Building a One-Click Install Button for MCP Servers in VS Code Extensions

This guide demonstrates how to build a one-click install button for MCP (Model Context Protocol) servers in your VS Code extension, using MongoDB MCP Server as an example.

## Overview

VS Code supports installing MCP servers via a special URI protocol: `vscode:mcp/install?{config}`. By constructing this URL with the proper configuration and opening it, VS Code will prompt the user to install the MCP server with all necessary settings pre-configured.

## MCP Transport Types

MCP supports multiple transport mechanisms for client-server communication:

| Transport | Description | Use Case |
|-----------|-------------|----------|
| `stdio` | Standard input/output streams | Local servers running as child processes |
| `http` | Streamable HTTP (recommended) | Remote servers over HTTP |
| `sse` | Server-Sent Events (legacy) | Remote servers (deprecated in favor of `http`) |

> **Note:** The SSE transport has been deprecated as of MCP specification version 2025-03-26. Use HTTP Stream transport for new remote server implementations.

## The Install URL Format

The install URL follows this pattern:

```
vscode:mcp/install?{encoded-json-config}
```

### Configuration Schema

The JSON configuration varies based on transport type:

#### For stdio (Local Servers)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique identifier for the MCP server |
| `command` | string | Yes | The executable to run (e.g., `npx`, `node`, `python`) |
| `args` | string[] | No | Command line arguments passed to the executable |
| `env` | object | No | Environment variables; use `${input:id}` to reference user inputs |
| `inputs` | array | No | User prompts for sensitive data like API keys |

#### For http/sse (Remote Servers)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique identifier for the MCP server |
| `type` | string | Yes | Transport type: `"http"` or `"sse"` |
| `url` | string | Yes | The server endpoint URL |
| `headers` | object | No | Custom HTTP headers to include in requests |
| `inputs` | array | No | User prompts for sensitive data |

#### Input Prompt Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier referenced via `${input:id}` |
| `type` | string | Yes | Input type: `"promptString"` |
| `description` | string | Yes | Label shown to the user |
| `password` | boolean | No | If `true`, masks the input field |
| `default` | string | No | Default value for the input |

## Step 1: Define the MCP Server Configuration

Create a configuration object for the MCP server. Here's the MongoDB example:

```json
{
  "id": "mcp-mongodb",
  "type": "MCP_SERVER",
  "title": "MongoDB",
  "description": "A Model Context Protocol server for interacting with MongoDB databases.",
  "installConfig": {
    "name": "mongodb",
    "command": "npx",
    "args": ["-y", "mongodb-mcp-server"],
    "env": {
      "MDB_MCP_CONNECTION_STRING": "${input:mongodb-connection-string}"
    },
    "inputs": [
      {
        "id": "mongodb-connection-string",
        "type": "promptString",
        "description": "MongoDB Connection String",
        "password": true
      }
    ]
  }
}
```

The `${input:id}` syntax tells VS Code to prompt the user for this value during installation. Setting `password: true` masks the input field.

## Step 2: Generate the Install URL

Create a helper function to construct the `vscode:mcp/install` URL:

```typescript
/**
 * Helper to generate VS Code MCP install URL
 * Format: vscode:mcp/install?{json-configuration}
 */
function createMcpInstallUrl(
  name: string,
  command: string,
  args: string[],
  env?: Record<string, string>,
  inputs?: Array<{ id: string; type: string; description: string; password?: boolean }>
): string {
  const config: Record<string, unknown> = {
    name,
    command,
    args
  };

  if (env) {
    config.env = env;
  }

  if (inputs) {
    config.inputs = inputs;
  }

  // Use vscode: protocol for direct installation
  return `vscode:mcp/install?${encodeURIComponent(JSON.stringify(config))}`;
}
```

### Example: Transform Resource Data

If you're loading configurations from JSON, transform them at load time:

```typescript
interface RawResource {
  id: string;
  type: string;
  title: string;
  installConfig?: {
    name: string;
    command: string;
    args: string[];
    env?: Record<string, string>;
    inputs?: Array<{ id: string; type: string; description: string; password?: boolean }>;
  };
}

function transformResource(raw: RawResource): AIResource {
  const resource: AIResource = {
    id: raw.id,
    type: raw.type as ResourceType,
    title: raw.title,
    // ... other fields
  };

  // Generate installUrl for MCP servers
  if (raw.type === 'MCP_SERVER' && raw.installConfig) {
    resource.installUrl = createMcpInstallUrl(
      raw.installConfig.name,
      raw.installConfig.command,
      raw.installConfig.args,
      raw.installConfig.env,
      raw.installConfig.inputs
    );
  }

  return resource;
}
```

## Step 3: Create the MCP Service

Create a simple service to handle the installation by opening the URL:

```typescript
import * as vscode from 'vscode';

/**
 * Service for handling MCP server installation
 */
export class McpService {
  /**
   * Install an MCP server by opening VS Code's native MCP install dialog
   */
  async installMcpServer(installUrl: string): Promise<void> {
    await vscode.env.openExternal(vscode.Uri.parse(installUrl));
  }
}

// Export singleton instance
export const mcpService = new McpService();
```

## Step 4: Build the Webview UI

### HTML Template

Create an install button in your webview HTML:

```html
<div class="detail-actions">
    <button class="install-btn primary" id="install-btn">
        <svg><!-- terminal icon --></svg>
        Install
    </button>
</div>

<!-- Inject data for JavaScript -->
<script nonce="${nonce}">
    window.MCP_DETAIL_DATA = ${resourceJson};
    window.MCP_DETAIL_ICONS = ${iconsJson};
</script>
<script nonce="${nonce}" src="${scriptUri}"></script>
```

### JavaScript Handler

Handle the button click and communicate with the extension:

```javascript
/**
 * JavaScript for the MCP Detail panel
 *
 * Expects the following globals to be defined before this script loads:
 * - window.MCP_DETAIL_DATA: Resource data object with installUrl
 * - window.MCP_DETAIL_ICONS: Object containing SVG icon strings
 */
(function() {
    'use strict';

    // Get VS Code API
    const vscode = acquireVsCodeApi();

    // Get data from globals
    const data = window.MCP_DETAIL_DATA || {};
    const icons = window.MCP_DETAIL_ICONS || {};

    // Install button handler
    const installBtn = document.getElementById('install-btn');
    if (installBtn) {
        installBtn.addEventListener('click', function() {
            vscode.postMessage({
                command: 'installMcpServer',
                installUrl: data.installUrl
            });

            installBtn.classList.add('copied');
            installBtn.innerHTML = icons.check + 'Installing...';

            setTimeout(function() {
                installBtn.classList.remove('copied');
                installBtn.innerHTML = icons.terminal + 'Install';
            }, 2000);
        });
    }
})();
```

## Step 5: Handle Messages in the Extension

Process the webview message in your webview panel class:

```typescript
import * as vscode from 'vscode';
import { mcpService } from '../services/mcpService';
import { WebviewMessage } from '../types';

export class MCPDetailPanel {
  private readonly _panel: vscode.WebviewPanel;
  private _disposables: vscode.Disposable[] = [];

  private constructor(panel: vscode.WebviewPanel, extensionUri: vscode.Uri, resource: AIResource) {
    this._panel = panel;

    // Handle messages from the webview
    this._panel.webview.onDidReceiveMessage(
      (message: WebviewMessage) => this._handleMessage(message),
      null,
      this._disposables
    );
  }

  /**
   * Handle messages from the webview
   */
  private async _handleMessage(message: WebviewMessage): Promise<void> {
    switch (message.command) {
      case 'installMcpServer':
        if (message.installUrl) {
          await mcpService.installMcpServer(message.installUrl);
        }
        break;
      case 'openExternalUrl':
        if (message.url) {
          await vscode.env.openExternal(vscode.Uri.parse(message.url));
        }
        break;
    }
  }
}
```

## Complete Architecture

```
┌─────────────────────┐
│  resources.json     │  Define MCP server config
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  resources.ts       │  Generate vscode:mcp/install URL
│  createMcpInstallUrl│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Webview HTML/JS    │  Render install button, handle click
└──────────┬──────────┘
           │ postMessage({ command: 'installMcpServer', installUrl })
           ▼
┌─────────────────────┐
│  MCPDetailPanel.ts  │  Receive message, call service
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  mcpService.ts      │  Open vscode:mcp/install URL
│  openExternal()     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  VS Code            │  Native MCP install dialog
│  Install Dialog     │  prompts user for inputs
└─────────────────────┘
```

## More Configuration Examples

### stdio: PostgreSQL (with connection string input)

```json
{
  "installConfig": {
    "name": "postgres",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres"],
    "env": {
      "POSTGRES_CONNECTION_STRING": "${input:postgres-connection}"
    },
    "inputs": [
      {
        "id": "postgres-connection",
        "type": "promptString",
        "description": "PostgreSQL Connection String",
        "password": true
      }
    ]
  }
}
```

### stdio: Filesystem (with workspace folder variable)

```json
{
  "installConfig": {
    "name": "filesystem",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
  }
}
```

Note: `${workspaceFolder}` is a special variable that VS Code resolves to the current workspace path.

---

## Remote MCP Servers (HTTP/SSE)

For MCP servers hosted remotely, use the `http` or `sse` transport type instead of `command`/`args`.

### http: Remote API Server (Streamable HTTP)

The recommended transport for remote servers. VS Code first tries HTTP Stream and falls back to SSE if not supported.

```json
{
  "installConfig": {
    "name": "my-remote-api",
    "type": "http",
    "url": "https://api.example.com/mcp"
  }
}
```

### http: Remote Server with Authentication

```json
{
  "installConfig": {
    "name": "authenticated-api",
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${input:api-token}"
    },
    "inputs": [
      {
        "id": "api-token",
        "type": "promptString",
        "description": "API Access Token",
        "password": true
      }
    ]
  }
}
```

### http: Remote Server with Custom Headers

```json
{
  "installConfig": {
    "name": "custom-api",
    "type": "http",
    "url": "https://api.example.com/mcp/v2",
    "headers": {
      "X-API-Version": "2.0",
      "X-Client-ID": "${input:client-id}",
      "Authorization": "Bearer ${input:api-key}"
    },
    "inputs": [
      {
        "id": "client-id",
        "type": "promptString",
        "description": "Client ID",
        "password": false
      },
      {
        "id": "api-key",
        "type": "promptString",
        "description": "API Key",
        "password": true
      }
    ]
  }
}
```

### sse: Legacy SSE Server

> **Deprecated:** SSE transport is deprecated as of MCP spec 2025-03-26. Use `http` for new implementations.

```json
{
  "installConfig": {
    "name": "legacy-sse-server",
    "type": "sse",
    "url": "https://api.example.com/sse",
    "headers": {
      "VERSION": "1.2"
    }
  }
}
```

---

## Generating Install URLs for Remote Servers

Extend the URL generator to support both transport types:

```typescript
interface StdioConfig {
  name: string;
  command: string;
  args?: string[];
  env?: Record<string, string>;
  inputs?: McpInput[];
}

interface HttpConfig {
  name: string;
  type: 'http' | 'sse';
  url: string;
  headers?: Record<string, string>;
  inputs?: McpInput[];
}

interface McpInput {
  id: string;
  type: 'promptString';
  description: string;
  password?: boolean;
  default?: string;
}

type McpInstallConfig = StdioConfig | HttpConfig;

/**
 * Generate VS Code MCP install URL for any transport type
 */
function createMcpInstallUrl(config: McpInstallConfig): string {
  return `vscode:mcp/install?${encodeURIComponent(JSON.stringify(config))}`;
}

// Example: stdio server
const mongodbUrl = createMcpInstallUrl({
  name: 'mongodb',
  command: 'npx',
  args: ['-y', 'mongodb-mcp-server'],
  env: {
    MDB_MCP_CONNECTION_STRING: '${input:connection-string}'
  },
  inputs: [{
    id: 'connection-string',
    type: 'promptString',
    description: 'MongoDB Connection String',
    password: true
  }]
});

// Example: http server
const remoteApiUrl = createMcpInstallUrl({
  name: 'remote-api',
  type: 'http',
  url: 'https://api.example.com/mcp',
  headers: {
    'Authorization': 'Bearer ${input:token}'
  },
  inputs: [{
    id: 'token',
    type: 'promptString',
    description: 'API Token',
    password: true
  }]
});
```

## Key Takeaways

1. **Use `vscode:mcp/install?{config}`** - This is VS Code's native protocol for MCP installation
2. **Choose the right transport**:
   - `stdio` (command/args) - For local servers running as child processes
   - `http` (type/url) - For remote servers over HTTP (recommended)
   - `sse` (type/url) - Legacy remote transport (deprecated)
3. **URL-encode the JSON config** - Use `encodeURIComponent(JSON.stringify(config))`
4. **Use `${input:id}` for secrets** - Reference user inputs in environment variables or headers
5. **Set `password: true`** - Mask sensitive input fields
6. **Use `vscode.env.openExternal()`** - Opens the URL and triggers VS Code's install dialog
7. **Webviews can't open URLs directly** - Send a message to the extension host instead

## References

- [VS Code MCP Documentation](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP Transports](https://modelcontextprotocol.io/docs/concepts/transports)
- [MongoDB MCP Server](https://github.com/mongodb-js/mongodb-mcp-server)
- [VS Code Extension API - openExternal](https://code.visualstudio.com/api/references/vscode-api#env.openExternal)
- [VS Code Webview API](https://code.visualstudio.com/api/extension-guides/webview)
