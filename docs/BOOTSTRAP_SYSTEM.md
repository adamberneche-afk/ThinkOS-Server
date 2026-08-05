# RCRT Bootstrap System

**Version:** 2.1.0 (Updated for optimized breadcrumb structure)

## Overview

The RCRT bootstrap system provides a **single source of truth** for all system initialization. All agents, tools, and templates are defined as JSON files in the `bootstrap-breadcrumbs/` directory and loaded into the system on startup.

**v2.1.0 Breaking Change:** Breadcrumb structure normalized - `description`, `semantic_version`, and `llm_hints` are now top-level fields (not in context).

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 ONE Bootstrap Process                            │
│                                                                  │
│  setup.sh                                                        │
│     │                                                            │
│     ▼                                                            │
│  bootstrap-breadcrumbs/bootstrap.js  ← THE ONLY BOOTSTRAP       │
│     │                                                            │
│     ├─→ system/*.json              (agents, configs)            │
│     ├─→ tools-self-contained/*.json (tool.code.v1 tools)        │
│     ├─→ templates/*.json           (template library)           │
│     ├─→ knowledge/*.json           (LLM semantic search)        │
│     ├─→ schemas/*.json             (llm_hints for schemas)      │
│     ├─→ themes/*.json              (theme.v1)                  │
│     ├─→ pages/*.json               (ui.page.v1 / page.layout.v1)│
│     └─→ states/*.json              (ui.state.v1)                │
│                                                                  │
│  Creates all breadcrumbs in RCRT database                       │
│     │                                                            │
│     ├─→ agent-runner auto-discovers agent.def.v1               │
│     └─→ tools-runner auto-discovers tool.code.v1                │
│                                                                  │
│  ✅ System ready!                                                │
└─────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
bootstrap-breadcrumbs/
├── bootstrap.js                    # Main bootstrap script
├── package.json                    # Dependencies
├── README.md                       # Bootstrap docs
├── system/                         # System breadcrumbs (agents, configs)
│   ├── default-chat-agent.json     # Default chat assistant
│   ├── note-tagger-agent.json      # Note tagging agent
│   ├── note-summarizer-agent.json  # Note summarizer agent
│   ├── note-insights-agent.json    # Note insights agent
│   ├── note-eli5-agent.json        # Note ELI5 agent
│   ├── context-blacklist.json      # Context assembly blacklist
│   └── bootstrap-marker.json       # Bootstrap completion marker
├── tools-self-contained/           # Tool definitions (tool.code.v1, 13 tools)
│   ├── openrouter.json
│   ├── openrouter-models-sync.json
│   ├── ollama.json
│   ├── venice.json
│   ├── calculator.json
│   ├── random.json
│   ├── echo.json
│   ├── timer.json
│   ├── scheduler.json
│   ├── workflow.json
│   ├── json-transform.json
│   ├── breadcrumb-create.json
│   └── breadcrumb-search.json
├── tools/                          # README only (tools moved to tools-self-contained/)
│   └── README.md
├── templates/                      # Templates for users
│   ├── agent-definition-template.json
│   ├── base-agent.json
│   ├── base-breadcrumb.json
│   ├── base-tool.json
│   └── llm-hints-guide.json
├── knowledge/                      # knowledge.v1 breadcrumbs for LLM semantic search
├── schemas/                        # schema.def.v1 breadcrumbs (llm_hints per schema)
├── themes/                         # theme.v1 breadcrumbs
├── pages/                          # ui.page.v1 / page.layout.v1 breadcrumbs
└── states/                         # ui.state.v1 breadcrumbs
```

## How It Works

### 1. Initialization

When you run `./setup.sh`:

1. Docker Compose builds all services
2. RCRT server starts and initializes database
3. Bootstrap script runs automatically (via Docker health check or manual trigger)
4. All JSON files are loaded as breadcrumbs
5. agent-runner discovers agents
6. tools-runner discovers tools
7. System is ready!

### 2. File Format

Every bootstrap file follows the breadcrumb schema:

```json
{
  "schema_name": "agent.def.v1",
  "title": "Default Chat Assistant",
  "tags": ["agent", "agent:default-chat-assistant", "workspace:agents"],
  "context": {
    "agent_id": "default-chat-assistant",
    "llm_config": {
      "provider": "openrouter",
      "model": "anthropic/claude-3.5-sonnet",
      "temperature": 0.7
    },
    "subscriptions": [
      {
        "schema_name": "user.message.v1",
        "any_tags": ["workspace:agents"]
      }
    ]
  }
}
```

### 3. Auto-Discovery

Services discover breadcrumbs by schema:

**agent-runner** queries:
```
GET /breadcrumbs?schema_name=agent.def.v1
```

**tools-runner** queries:
```
GET /breadcrumbs?schema_name=tool.code.v1
```

No hardcoded registration needed!

## Tool System

Tools live in `bootstrap-breadcrumbs/tools-self-contained/*.json` as self-contained `tool.code.v1` breadcrumbs. Each breadcrumb bundles the tool's own source code (executed in a Deno sandbox by tools-runner) alongside its schema and metadata — there is no separate implementation folder to wire up. The legacy `tools/` directory now contains only a `README.md` pointing here; it holds no tool definitions.

### Tool Definition Structure

Each tool is defined with:

```json
{
  "schema_name": "tool.code.v1",
  "title": "Tool Name (Self-Contained)",
  "description": "What this tool does",
  "semantic_version": "2.0.0",
  "tags": ["tool", "tool:name", "workspace:tools", "self-contained"],
  "llm_hints": {
    "include": ["name", "description", "input_schema", "output_schema", "examples"],
    "exclude": ["code", "permissions", "limits", "ui_schema"]
  },
  "context": {
    "name": "tool-name",
    "code": {
      "language": "typescript",
      "source": "export async function execute(input, context) { ... }"
    },
    "input_schema": {
      "type": "object",
      "properties": {...},
      "required": [...]
    },
    "output_schema": {
      "type": "object",
      "properties": {...}
    },
    "permissions": {
      "net": false, "read": false, "write": false,
      "env": false, "run": false, "ffi": false, "hrtime": false
    },
    "limits": {
      "timeout_ms": 5000, "memory_mb": 32, "cpu_percent": 50
    },
    "required_secrets": [],
    "ui_schema": { "configurable": false },
    "examples": [
      {
        "description": "Example usage",
        "input": {...},
        "output": {...},
        "explanation": "How to read the output"
      }
    ]
  }
}
```

`description`, `semantic_version`, and `llm_hints` are top-level fields (not nested in `context`), per the v2.1.0 breadcrumb structure normalization.

**See:** `bootstrap-breadcrumbs/templates/base-tool.json` and `bootstrap-breadcrumbs/templates/base-breadcrumb.json` for full specification

### Complete Tool List

The 13 tools currently in `tools-self-contained/`:

1. **openrouter** - Access to 100+ LLM models via unified API
2. **openrouter-models-sync** - Syncs the OpenRouter models catalog for dropdown selections
3. **ollama** - Local LLM access via Ollama (fast, free, private)
4. **venice** - Venice AI privacy-focused LLM access
5. **calculator** - Mathematical calculations (arithmetic, parentheses, math functions)
6. **random** - Random number generation
7. **echo** - Returns input unchanged (testing)
8. **timer** - Wait for a specified number of seconds
9. **scheduler** - Monitors schedule definitions and publishes tick breadcrumbs for time-based automation
10. **workflow** - Orchestrates multi-step tool operations with dependencies and variable interpolation
11. **json-transform** - Transforms JSON data using JSONPath queries and mappings
12. **breadcrumb-create** - Creates new breadcrumbs with schema, title, tags, and context
13. **breadcrumb-search** - Searches and retrieves breadcrumbs by schema, tags, or semantic query

Note: `context-builder` is **not** one of these tool breadcrumbs. It is a separate Rust microservice (`crates/rcrt-context-builder`) that assembles agent context directly; it is not defined via a `tools-self-contained/*.json` file.

### Tool Execution Model

Each tool breadcrumb embeds its own `code.source` (TypeScript), which tools-runner executes in a sandboxed Deno process governed by the breadcrumb's `permissions` and `limits`. There is no separate `builtin` / `external` / `service` implementation-type dispatch — the code and its declared permissions travel together in the same breadcrumb.

## Agent System

### Agent Definition Structure

```json
{
  "schema_name": "agent.def.v1",
  "title": "Agent Name",
  "tags": ["agent", "agent:agent-name", "workspace:agents"],
  "context": {
    "agent_id": "agent-name",
    "description": "What this agent does",
    "llm_config": {
      "provider": "openrouter",
      "model": "anthropic/claude-3.5-sonnet",
      "temperature": 0.7,
      "max_tokens": 4096
    },
    "system_prompt": "You are an AI assistant...",
    "subscriptions": [
      {
        "schema_name": "user.message.v1",
        "any_tags": ["workspace:agents"]
      },
      {
        "schema_name": "agent.context.v1",
        "all_tags": ["agent:context", "consumer:agent-name"]
      }
    ],
    "capabilities": ["chat", "tool-use", "workflow"]
  }
}
```

### Agent Subscriptions

Agents subscribe to breadcrumb updates using **selectors**:

**By Schema**:
```json
{"schema_name": "user.message.v1"}
```

**By Tags**:
```json
{
  "schema_name": "user.message.v1",
  "any_tags": ["workspace:agents"]
}
```

**By Context Match**:
```json
{
  "schema_name": "tool.response.v1",
  "context_match": [{
    "path": "$.requestedBy",
    "op": "eq",
    "value": "agent-name"
  }]
}
```

## Best Practices

### 1. Single Source of Truth
- All definitions in `bootstrap-breadcrumbs/`
- No hardcoded fallbacks in code
- JSON files are the authority

### 2. Fail Fast
- Bootstrap fails if files are invalid
- Clear error messages
- Guides to fix issues

### 3. Version Control
- JSON files in git
- Track changes to definitions
- Easy rollback

### 4. Customization
```bash
# Fork and customize
git clone your-fork
cd your-fork

# Edit definitions
vim bootstrap-breadcrumbs/system/default-chat-agent.json

# Add custom tools
cat > bootstrap-breadcrumbs/tools/custom-tool.json << 'EOF'
{...}
EOF

# Deploy
./setup.sh
```

### 5. Testing
```bash
# Validate JSON syntax
find bootstrap-breadcrumbs -name "*.json" -exec python -m json.tool {} \; > /dev/null

# Test bootstrap
docker compose down -v
./setup.sh
docker compose logs bootstrap-runner

# Verify loaded
curl http://localhost:8081/breadcrumbs?schema_name=tool.v1 | jq '. | length'
# Should return: 13
```

## Portability

Bootstrap system supports container prefixes:

```bash
# Standard deployment
./setup.sh

# With custom prefix (for forks/multi-deployment)
PROJECT_PREFIX="mycompany-" ./setup.sh
```

This creates containers like:
- `mycompany-rcrt`
- `mycompany-agent-runner`
- `mycompany-tools-runner`

Perfect for:
- Multiple deployments on same host
- Fork identification
- Organization branding

## Troubleshooting

### Bootstrap Fails

**Check logs**:
```bash
docker compose logs bootstrap-runner
```

**Common issues**:
1. Invalid JSON syntax
2. Missing required fields
3. RCRT server not ready

**Fix**:
```bash
# Validate JSON
python -m json.tool bootstrap-breadcrumbs/tools/my-tool.json

# Restart bootstrap
docker compose restart bootstrap-runner
```

### Tools Not Loading

**Verify breadcrumbs created**:
```bash
curl http://localhost:8081/breadcrumbs?schema_name=tool.v1
```

**Check tools-runner**:
```bash
docker compose logs tools-runner | grep "tools available"
# Should show: "✅ 13 tools available"
```

### Agent Not Responding

**Verify agent loaded**:
```bash
curl http://localhost:8081/breadcrumbs?schema_name=agent.def.v1
```

**Check agent-runner**:
```bash
docker compose logs agent-runner | grep "default-chat-assistant"
```

## Advanced Topics

### Dynamic Updates

Agents and tools can be updated at runtime:

```bash
# Update agent
curl -X PATCH http://localhost:8081/breadcrumbs/:agent-id \
  -H 'Content-Type: application/json' \
  -H 'If-Match: "version-etag"' \
  -d '{"context": {...}}'

# Agent-runner detects change and reloads
```

### Migration

To migrate from old system:

1. Extract definitions from code
2. Create JSON files in bootstrap-breadcrumbs/
3. Remove hardcoded registrations
4. Run bootstrap
5. Verify with queries

### Custom Bootstrap

For advanced use cases:

```javascript
// custom-bootstrap.js
import { bootstrap } from './bootstrap-breadcrumbs/bootstrap.js';

// Add custom logic
await customSetup();

// Run standard bootstrap
await bootstrap();

// Add post-bootstrap tasks
await postSetup();
```

## Summary

The bootstrap system provides:

✅ **Single Source of Truth**: All definitions in one place
✅ **Zero Duplicates**: No conflicting definitions
✅ **No Fallbacks**: Explicit and fail-fast
✅ **Auto-Discovery**: Services find breadcrumbs automatically
✅ **Version Control**: JSON files in git
✅ **Portable**: Works anywhere with container prefixes
✅ **Maintainable**: Easy to understand and modify

**One bootstrap process, all data in files, zero hardcoding!**

