# @rcrt-builder/tools

Universal tool integration system for RCRT. Wraps any tool (custom functions, external APIs, self-contained `tool.code.v1` breadcrumbs) to work seamlessly with the RCRT breadcrumb ecosystem.

## Features

- **Universal Interface**: Any tool can integrate via standardized schemas
- **Self-contained tools**: `tool.code.v1` breadcrumbs carry their own code, loaded and run via `ToolLoader` / `DenoToolRuntime`
- **Auto-Discovery**: Tools publish themselves (via the catalog) for agent discovery
- **Multi-Environment**: Works in Docker, Node.js, browser, and Electron

## Quick Start

### Install
```bash
pnpm add @rcrt-builder/tools @rcrt-builder/sdk
```

### Basic Usage
```typescript
import { createClient } from '@rcrt-builder/sdk';
import { bootstrapTools, builtinTools, ToolLoader } from '@rcrt-builder/tools';

const client = await createClient({ baseUrl: '/api/rcrt' });

// Bootstraps/refreshes the workspace's single tool.catalog.v1 breadcrumb
// from the tool.code.v1 breadcrumbs already present for that workspace.
await bootstrapTools(client, 'workspace:tools');

// Load a specific tool implementation by name (tries tool.code.v1, then
// falls back to legacy tool.v1) or from a known breadcrumb ID.
const loader = new ToolLoader(client, 'workspace:tools');
const tool = await loader.loadToolByName('file-storage');

// builtinTools exposes the in-process implementations directly, keyed by name
// (see "Built-in Tools" below) — e.g. builtinTools['file-storage'].execute(...)
```

### Request Tool Execution
```typescript
// From an agent or UI
await client.createBreadcrumb({
  schema_name: 'tool.request.v1',
  title: 'Search Request',
  tags: ['workspace:tools', 'tool:request'],
  context: {
    tool: 'serpapi',
    input: { query: 'electric bikes' }
  }
});

// Listen for results
client.startEventStream((evt) => {
  if (evt.schema_name === 'tool.response.v1') {
    console.log('Result:', evt.context.result);
  }
});
```

## Creating Custom Tools

### Simple Function Tool
```typescript
import { createTool } from '@rcrt-builder/tools';

const weatherTool = createTool(
  'weather',
  'Get current weather for a location',
  {
    type: 'object',
    properties: {
      location: { type: 'string', description: 'City name' }
    },
    required: ['location']
  },
  {
    type: 'object',
    properties: {
      temperature: { type: 'number' },
      condition: { type: 'string' }
    }
  },
  async (input) => {
    // Your weather API call here
    return {
      temperature: 72,
      condition: 'sunny'
    };
  }
);

// `createTool` produces a plain RCRTTool object. There is no registry to register it
// with at runtime — to make it loadable, either add it to `builtinTools` in `index.ts`
// (so `ToolLoader`'s `builtin` implementation type can find it), or publish a
// `tool.code.v1` breadcrumb so `ToolLoader.loadToolFromBreadcrumb` can load it directly.
```

### Advanced Tool Class
```typescript
import { RCRTTool } from '@rcrt-builder/tools';

class DatabaseTool implements RCRTTool {
  name = 'database_query';
  description = 'Execute SQL queries';
  category = 'database';
  
  inputSchema = {
    type: 'object',
    properties: {
      query: { type: 'string', description: 'SQL query' },
      database: { type: 'string', description: 'Database name' }
    },
    required: ['query']
  };
  
  outputSchema = {
    type: 'object',
    properties: {
      rows: { type: 'array' },
      count: { type: 'number' }
    }
  };
  
  validateInput(input: any): boolean | string {
    if (!input.query || typeof input.query !== 'string') {
      return 'Query must be a string';
    }
    if (input.query.toLowerCase().includes('drop')) {
      return 'DROP statements not allowed';
    }
    return true;
  }
  
  async execute(input: any, context: ToolExecutionContext) {
    // Your database logic here
    const rows = await executeQuery(input.query);
    return { rows, count: rows.length };
  }
  
  async cleanup() {
    // Close database connections, etc.
  }
}

// Same story here: instantiate the class, then add it to `builtinTools` or
// publish it as a `tool.code.v1` breadcrumb — there is no `registry.register()` call.
```

## Built-in Tools

`builtinTools` (exported from `src/index.ts`) currently contains:

- **agent-helper**: Provides system guidance and documentation for LLM-based agents
- **file-storage**: Stores and retrieves files as RCRT breadcrumbs (`FileStorageTool`)
- **agent-loader**: Loads agent definitions (`AgentLoaderTool`)
- **workflow**: Executes multi-step workflows with dependencies and parallel execution (`workflowOrchestrator`)
- **browser-context-capture**: Captures browser context/page state (`browserContextCaptureTool`)

## Tool Discovery

Tools are managed via a **single catalog breadcrumb** that updates when tools are added/removed:

```typescript
// Agents can discover available tools (single catalog per workspace)
const catalogs = await client.searchBreadcrumbs({
  tag: 'workspace:tools',
  schema_name: 'tool.catalog.v1'
});

const toolCatalog = catalogs[0]; // Single catalog breadcrumb
console.log('Available tools:', toolCatalog?.context.tools);
console.log('Active tools:', toolCatalog?.context.activeTools);
console.log('Last updated:', toolCatalog?.context.lastUpdated);

// Example catalog structure:
{
  "schema_name": "tool.catalog.v1",
  "title": "workspace:tools Tool Catalog",
  "context": {
    "workspace": "workspace:tools",
    "tools": [
      {
        "name": "serpapi", 
        "description": "Search the web using Google",
        "status": "active",
        "category": "search"
      }
    ],
    "totalTools": 5,
    "activeTools": 4
  }
}
```

**Key Benefits:**
- ✅ **Single Source of Truth**: One catalog breadcrumb per workspace
- ✅ **Real-time Updates**: Catalog updates when tools are added/removed
- ✅ **Efficient Discovery**: Query one breadcrumb instead of many
- ✅ **Version History**: See how tool availability changed over time

## Deployment

### Docker (secrets via RCRT)
```yaml
# docker-compose.yml
services:
  rcrt-tools:
    build: ./apps/tools-runner
    environment:
      - RCRT_BASE_URL=http://rcrt:8081
      - WORKSPACE=workspace:tools
      # API keys must be stored in RCRT Secrets.
      # Create secrets: SERPAPI_API_KEY, OPENAI_API_KEY, etc.
```

### Local Development
```bash
cd apps/tools-runner
pnpm dev
```

### Electron App
```bash
cd apps/tools-runner
pnpm build
electron .
```

## Schemas

### tool.request.v1
```json
{
  "schema_name": "tool.request.v1",
  "context": {
    "tool": "calculator",
    "input": { "expression": "2 + 2" },
    "timeout": 30000
  }
}
```

### tool.response.v1
```json
{
  "schema_name": "tool.response.v1", 
  "context": {
    "tool": "calculator",
    "requestId": "req-123",
    "result": { "result": 4, "expression": "2 + 2" },
    "executionTime": 150
  }
}
```

### tool.error.v1
```json
{
  "schema_name": "tool.error.v1",
  "context": {
    "tool": "calculator", 
    "requestId": "req-123",
    "error": "Invalid expression",
    "code": "EXECUTION_ERROR"
  }
}
```

## Architecture

The tool system follows RCRT's core principle: everything is a breadcrumb client.

1. **Tools subscribe** to `tool.request.v1` events
2. **Execute** the requested operation
3. **Publish results** as `tool.response.v1` or `tool.error.v1`
4. **Agents coordinate** tools via the same breadcrumb interface

This creates a truly composable system where:
- LLM agents can discover and use any tool
- Tools can be developed independently
- UI updates happen automatically
- Everything is auditable through breadcrumbs
