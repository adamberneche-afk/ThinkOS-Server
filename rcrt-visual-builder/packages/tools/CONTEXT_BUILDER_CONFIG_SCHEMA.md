# Context Builder Configuration Schema

## 🎯 **Like tool.config.v1, but for Context Assembly**

Just as openrouter has `tool.config.v1` breadcrumb for API keys and models, context-builder has `context.config.v1` for assembly strategies.

## 📋 **Schema: context.config.v1**

```typescript
{
  schema_name: 'context.config.v1',
  title: 'Context Config for {agent-name}',
  tags: [
    'context:config',
    'consumer:{agent-id}',
    'ui:editable'  // Tells UI it's configurable
  ],
  context: {
    // WHO this config is for
    consumer_id: 'default-chat-assistant',
    consumer_type: 'agent',  // or 'workflow', 'dashboard', etc.
    
    // WHAT to include (configurable!)
    sources: [
      {
        schema_name: 'user.message.v1',
        method: 'recent',  // 'recent', 'vector', 'latest', 'all'
        limit: 3,          // For recent/all
        nn: 5,             // For vector
        filters: {
          tag: 'extension:chat',
          schema_name: 'user.message.v1'
        }
      },
      {
        schema_name: 'user.message.v1',
        method: 'vector',
        nn: 5,
        filters: { tag: 'extension:chat' }
      },
      // ... more sources
    ],
    
    // WHERE to write (usually fixed per agent)
    output: {
      schema_name: 'agent.context.v1',
      tags: ['agent:context', 'consumer:{agent-id}'],
      ttl_seconds: 3600
    },
    
    // HOW to format (configurable!)
    formatting: {
      max_tokens: 400000,            // Token budget (bootstrapped default, sized for gemini-2.5-flash-lite's ~1M window)
      deduplication_threshold: 0.95, // Similarity (0.90-0.99)
      include_metadata: false,       // Declared but not currently read by assembleContext() — has no effect
      enable_summarization: false    // Future: LLM summarization
    },
    
    // UI metadata
    metadata: {
      version: '1.0.0',
      created_by: 'system',
      description: 'Hybrid: recent + semantic',
      last_updated: '2025-10-06T...'
    },
    
    // UI hints
    ui_config: {
      editable: true,
      category: 'context-assembly',
      icon: '🏗️',
      description: 'Configure how context is assembled'
    }
  }
}
```

## 🎨 **What Should Be Configurable in UI**

### **1. Strategy Selection**
```
Radio buttons:
○ Recent only (chronological)
○ Semantic only (vector search)
● Hybrid (recent + semantic)  ← Default
○ Custom (advanced)
```

### **2. Message Limits** (Sliders)
```
Recent user messages: [3] (0-20)
Semantic user messages: [5] (0-20)
Recent agent responses: [2] (0-10)
Semantic agent responses: [3] (0-10)
```

### **3. Vector Search Settings**
```
Similarity threshold: [0.95] (0.80-0.99)
  Lower = more diverse results
  Higher = only very similar messages
```

### **4. Token Budget**
```
Max context tokens: [400000] (bootstrapped default; enforceTokenBudget() trims when exceeded)
```

### **5. Advanced Options** (Collapsible)
```
☐ Include tool results
☐ Include metadata (timestamps) — NOTE: not currently wired up; assembleContext() never reads formatting.include_metadata
☐ Enable summarization (future)
☐ Cache context (TTL: 3600s)
```

## 📊 **Preset Configurations**

### **Preset 1: Conversational (Default)**
```json
{
  "name": "Conversational",
  "sources": [
    { "schema_name": "user.message.v1", "method": "recent", "limit": 3 },
    { "schema_name": "user.message.v1", "method": "vector", "nn": 5 },
    { "schema_name": "agent.response.v1", "method": "recent", "limit": 2 },
    { "schema_name": "agent.response.v1", "method": "vector", "nn": 3 }
  ],
  "formatting": {
    "max_tokens": 4000,
    "deduplication_threshold": 0.95
  }
}
```

### **Preset 2: Memory-Focused**
```json
{
  "name": "Memory-Focused",
  "sources": [
    { "schema_name": "user.message.v1", "method": "vector", "nn": 10 },
    { "schema_name": "agent.response.v1", "method": "vector", "nn": 8 }
  ],
  "formatting": {
    "max_tokens": 8000,
    "deduplication_threshold": 0.90
  }
}
```

### **Preset 3: Minimal (Performance)**
```json
{
  "name": "Minimal",
  "sources": [
    { "schema_name": "user.message.v1", "method": "recent", "limit": 2 },
    { "schema_name": "agent.response.v1", "method": "recent", "limit": 1 }
  ],
  "formatting": {
    "max_tokens": 2000,
    "deduplication_threshold": 0.98
  }
}
```

### **Preset 4: Document-Focused (RAG)**
```json
{
  "name": "Document RAG",
  "sources": [
    { "schema_name": "document.v1", "method": "vector", "nn": 10 },
    { "schema_name": "code.snippet.v1", "method": "vector", "nn": 5 },
    { "schema_name": "user.message.v1", "method": "recent", "limit": 2 }
  ],
  "formatting": {
    "max_tokens": 8000,
    "deduplication_threshold": 0.90
  }
}
```

## 🔧 **Implementation Plan**

### **1. Bootstrap creates default config** ✅ (Done!)
```typescript
// On first startup, creates context.config.v1 with sensible defaults
```

### **2. Dashboard reads context.config.v1** (To Do)
```typescript
// Search for context.config.v1 breadcrumbs
// Display in "Context Builder Settings" UI
// Allow editing sources, limits, thresholds
```

### **3. Dashboard saves changes** (To Do)
```typescript
// Update context.config.v1 breadcrumb
// context-builder auto-reloads (watches for updates!)
// New strategy takes effect immediately
```

## 📱 **UI Mock (Dashboard)**

```
┌─────────────────────────────────────────────────────────┐
│ 🏗️ Context Builder Configuration                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Agent: default-chat-assistant                          │
│ Strategy: ● Hybrid  ○ Recent  ○ Semantic  ○ Custom    │
│                                                         │
│ ┌─── Message Sources ───────────────────────────────┐ │
│ │                                                     │ │
│ │ User Messages:                                      │ │
│ │   Recent: [3] messages                             │ │
│ │   Semantic: [5] messages (vector search)           │ │
│ │                                                     │ │
│ │ Agent Responses:                                    │ │
│ │   Recent: [2] messages                             │ │
│ │   Semantic: [3] messages (vector search)           │ │
│ │                                                     │ │
│ │ ☑ Include tool results (last [3])                  │ │
│ │ ☑ Include tool catalog                             │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ ┌─── Optimization ──────────────────────────────────┐ │
│ │                                                     │ │
│ │ Token Budget: [4000] (1000-16000)                  │ │
│ │ Deduplication: [0.95] (0.80-0.99)                  │ │
│ │ Context TTL: [3600]s (1 hour)                      │ │
│ │                                                     │ │
│ │ ☐ Include timestamps                               │ │
│ │ ☐ Enable summarization (experimental)              │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ ┌─── Presets ───────────────────────────────────────┐ │
│ │ [Conversational] [Memory-Focused] [Minimal] [RAG] │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ [Test Configuration] [Save] [Reset to Default]         │
└─────────────────────────────────────────────────────────┘
```

## 🎯 **Configurable Parameters**

| Parameter | Type | Range | Default | Description |
|-----------|------|-------|---------|-------------|
| **recent_user_limit** | number | 0-20 | 3 | How many recent user messages to always include |
| **vector_user_nn** | number | 0-20 | 5 | How many semantic user messages to include |
| **recent_agent_limit** | number | 0-10 | 2 | How many recent agent responses |
| **vector_agent_nn** | number | 0-10 | 3 | How many semantic agent responses |
| **tool_results_limit** | number | 0-10 | 3 | How many recent tool results |
| **max_tokens** | number | no enforced range | 400000 | Token budget (auto-trim); bootstrapped default is sized for gemini-2.5-flash-lite's ~1M context window |
| **dedup_threshold** | float | 0.80-0.99 | 0.95 | Similarity for deduplication |
| **context_ttl** | number | 300-7200 | 3600 | How long to cache context (seconds) |
| **include_timestamps** (`include_metadata`) | boolean | - | false | Declared in config but not read by `assembleContext()` — currently a no-op |
| **enable_documents** | boolean | - | false | Include document.v1 vector search |

**All stored in context.config.v1, editable via UI, changes take effect immediately!**

Want me to implement the configuration loading and UI schema?
