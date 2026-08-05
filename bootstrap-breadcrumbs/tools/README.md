# Tool Definitions - Moved to tools-self-contained/

## 🎯 Current Location

Tool definitions live in `bootstrap-breadcrumbs/tools-self-contained/*.json`, one JSON file per tool. This `tools/` folder no longer holds any tool definitions — it exists only for this README.

Each tool file is a self-contained `tool.code.v1` breadcrumb: the tool's own TypeScript source lives inline in `context.code.source`, alongside its `input_schema`, `output_schema`, `permissions`, `limits`, and `examples`. There are no separate implementation folders or files to wire up.

## Structure

```
bootstrap-breadcrumbs/tools-self-contained/
├── openrouter.json
├── openrouter-models-sync.json
├── ollama.json
├── venice.json
├── calculator.json
├── random.json
├── echo.json
├── timer.json
├── scheduler.json
├── workflow.json
├── json-transform.json
├── breadcrumb-create.json
└── breadcrumb-search.json
```

## How Bootstrap Actually Works

Per `bootstrap.js`'s own top-of-file comment ("SINGLE SOURCE OF TRUTH") and its step 3 ("Loading self-contained tools"), it reads directly from this directory — it does **not** scan `rcrt-visual-builder/packages/tools/src/` for `*/definition.json` files:

```javascript
// bootstrap.js:
// Tools: bootstrap-breadcrumbs/tools-self-contained/*.json (tool.code.v1)
//
// const selfContainedToolsDir = path.join(__dirname, 'tools-self-contained');
// const toolFiles = fs.readdirSync(selfContainedToolsDir).filter(f => f.endsWith('.json'));
// ...creates tool.code.v1 breadcrumbs from each file
```

## Adding a New Tool

```bash
# 1. Create the tool definition (schema tool.code.v1)
cat > bootstrap-breadcrumbs/tools-self-contained/my-tool.json << 'EOF'
{
  "schema_name": "tool.code.v1",
  "title": "My Tool (Self-Contained)",
  "description": "What this tool does",
  "semantic_version": "1.0.0",
  "tags": ["tool", "tool:my-tool", "workspace:tools", "self-contained"],
  "llm_hints": {
    "include": ["name", "description", "input_schema", "output_schema", "examples"],
    "exclude": ["code", "permissions", "limits", "ui_schema"]
  },
  "context": {
    "name": "my-tool",
    "code": {
      "language": "typescript",
      "source": "export async function execute(input, context) {\n  return { result: '...' };\n}\n"
    },
    "input_schema": {...},
    "output_schema": {...},
    "permissions": { "net": false, "read": false, "write": false, "env": false, "run": false, "ffi": false, "hrtime": false },
    "limits": { "timeout_ms": 5000, "memory_mb": 32, "cpu_percent": 50 },
    "examples": [...]
  }
}
EOF

# 2. Bootstrap
cd bootstrap-breadcrumbs && node bootstrap.js
```

**Done!** Tool is live.

## See Full Documentation

- Tool definitions: `bootstrap-breadcrumbs/tools-self-contained/`
- Bootstrap process: `bootstrap-breadcrumbs/bootstrap.js`