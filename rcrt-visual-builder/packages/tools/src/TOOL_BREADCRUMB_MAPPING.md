# Tool Breadcrumb Mapping

## Current Builtin Tools to Breadcrumb Implementation

### 1. Random Tool — REMOVED from `builtinTools`
`random/definition.json` still points `implementation.export` at `builtinTools.random`, but
`builtinTools` in `src/index.ts` no longer has a `random` key — this tool was removed from the
built-in set. The `definition.json` is stale and would fail to load via `ToolLoader`.

### 2. Calculator Tool — REMOVED from `builtinTools`
Same situation as Random: `calculator/definition.json` points at `builtinTools.calculator`, but
`builtinTools` no longer exports a `calculator` key. The `definition.json` is stale.

### 3. Workflow Tool
Code Location: `workflow-orchestrator.ts` - `workflowOrchestrator`, exported as `builtinTools.workflow`
```javascript
export const workflowOrchestrator = new WorkflowOrchestratorTool()
// in index.ts: builtinTools = { ..., workflow: workflowOrchestrator, ... }
```

Breadcrumb Implementation (from `workflow/definition.json`):
```json
{
  "implementation": {
    "type": "builtin",
    "runtime": "nodejs",
    "module": "@rcrt-builder/tools",
    "export": "builtinTools.workflow",
    "instantiate": false
  }
}
```

### 4. OpenRouter Tool  
Code Location: `llm-tools/openrouter.ts` - Class instance created by registry
```javascript
export class OpenRouterTool extends SimpleLLMTool { ... }
```

Breadcrumb Implementation:
```json
{
  "implementation": {
    "type": "builtin",
    "runtime": "nodejs",
    "module": "@rcrt-builder/tools/llm-tools",
    "export": "OpenRouterTool",
    "instantiate": true,
    "constructor_args": []
  }
}
```

### 5. File Storage Tool
Code Location: `file-tools/file-storage.ts` - `FileStorageTool`, exported as `builtinTools['file-storage']` (already instantiated in `index.ts`)
```javascript
export class FileStorageTool implements RCRTTool { ... }
// in index.ts: builtinTools = { ..., 'file-storage': new FileStorageTool(), ... }
```

Breadcrumb Implementation (from `file-tools/definition.json`):
```json
{
  "implementation": {
    "type": "builtin",
    "runtime": "nodejs",
    "module": "@rcrt-builder/tools",
    "export": "builtinTools['file-storage']",
    "instantiate": false
  }
}
```

## Tool Runner Loading Logic

```javascript
async function loadToolFromBreadcrumb(breadcrumb) {
  const { implementation } = breadcrumb.context;
  
  if (implementation.type === 'builtin') {
    // Load from our packages
    let toolModule;
    
    switch (implementation.module) {
      case '@rcrt-builder/tools':
        toolModule = await import('./index.js');
        break;
      case '@rcrt-builder/tools/llm-tools':
        toolModule = await import('./llm-tools/index.js');
        break;
      // etc...
    }
    
    // Get the export
    const parts = implementation.export.split('.');
    let tool = toolModule;
    for (const part of parts) {
      tool = tool[part];
    }
    
    // Instantiate if needed
    if (implementation.instantiate) {
      const args = implementation.constructor_args || [];
      tool = new tool(...args);
    }
    
    return tool;
  }
  
  // Handle other implementation types...
}
```

## Migration Steps

1. Update each tool to create its breadcrumb on startup
2. Include implementation details in breadcrumb
3. Update tool runner to load from breadcrumbs
4. Remove in-memory registry

This preserves the existing code structure while making tools discoverable via breadcrumbs!
