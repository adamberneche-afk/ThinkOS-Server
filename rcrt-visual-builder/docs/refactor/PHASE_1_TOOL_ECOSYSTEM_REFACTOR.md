# Phase 1: Tool Ecosystem Foundation 🔧
**Status: 📋 PLANNED** (not started — see [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md))

## Overview  
**Foundation is complete and stable.** Next: **expand LangChain tools** (the heart) and add **simple enable/disable** for large catalogs. 

**Reality**: Current 12 tools are mostly test tools - **LangChain tools are the real value**. Need 50-100 real tools with easy enable/disable so 200+ tool catalogs don't overwhelm LLMs.

---

## 🎯 **Current State (PROVEN WORKING)** ✅

### **✅ Solid Foundation Achieved**
- ✅ **Stable catalog system**: 12 tools running, no version drift
- ✅ **LangChain integration**: Calculator, web_browser working 
- ✅ **Registry system**: Comprehensive duplicate prevention
- ✅ **Tool loading**: Existing tools load correctly on restart
- ✅ **All SDKs tested**: Core, Tools, Runtime, Node all verified ✅

### **🔍 Reality Check on Current Tools** 
- **Current "12 tools"**: Mostly test tools (echo, timer, random)
- **Real value**: LangChain tools (calculator, web_browser) + **expansion potential**
- **LangChain = Heart**: This will be the primary tool source, not builtins

### **🎯 Practical Goals (Not Over-Engineering)**
- **LangChain as core**: Stable foundation for 50-200+ real tools
- **Simple enable/disable**: For managing large catalogs (200+ tools overwhelming for LLMs)
- **Easy tool creation**: Base classes/SDKs for straightforward integration
- **Clean structure**: Organized but not complex

### **Current File Structure**
```
packages/tools/src/
├── index.ts          # Main interface, simple builtin tools
├── registry.ts       # Tool registration and catalog
└── langchain.ts      # LangChain bridge implementations
```

---

## 🏗️ **Pragmatic Next Steps (Build on Working Foundation)**

### **Keep Working Foundation + Strategic Additions**
**Philosophy: Build on proven stability, don't rebuild**

```
packages/tools/src/
├── index.ts              # ✅ KEEP - Main interface (working)
├── registry.ts           # ✅ ENHANCE - Add simple enable/disable  
├── langchain.ts          # ✅ EXPAND - This is the heart! Add more tools
│
├── langchain-tools/      # 🆕 NEW - Real LangChain tools (the heart)
│   ├── search/           # SerpAPI, Brave Search, DuckDuckGo, Google  
│   ├── data/             # CSV, JSON, XML processing, file operations
│   ├── web/              # Enhanced scraping, API clients
│   ├── communication/    # Slack, Discord, email (when needed)
│   └── index.ts          # Export tool collections by category
│
├── llm-tools/            # 🆕 NEW - Direct LLM access as tools
│   ├── anthropic.ts      # Claude Sonnet, Haiku (direct API)
│   ├── openai.ts         # GPT-4, GPT-4 Mini (direct API)
│   ├── openrouter.ts     # Multi-provider access (100+ models)
│   ├── ollama.ts         # Local models (privacy/speed)
│   └── index.ts          # Export LLM tool collection
│
└── utils/                # 🆕 NEW - Simple utilities (no over-engineering)
    ├── base-tool.ts      # Simple base class for easy tool creation
    ├── secrets.ts        # RCRT secrets integration helpers
    └── index.ts          # Utility exports
```

**Key Principles:**
- ✅ **Build on what works** - don't break proven registry system
- ✅ **LangChain as heart** - expand langchain.ts to be the main tool source  
- ✅ **Simple enable/disable** - basic boolean flags, not complex auto-disable
- ✅ **Easy agent creation** - simple base classes for common patterns
- ✅ **Practical organization** - group by function, not over-categorize

---

## 🚀 **Practical Implementation Plan**

### **Step 1: Add Simple Enable/Disable (Build on Working Registry)**
**Goal**: Manage 200+ tool catalogs for LLMs
```typescript
// Enhance existing registry.ts with simple flags
export interface ToolCatalogEntry {
  name: string;
  description: string; 
  category: string;
  version: string;
  inputSchema: any;
  outputSchema: any;
  capabilities: any;
  status: 'active' | 'inactive' | 'error' | 'disabled';  // 🆕 Add disabled
  enabled: boolean;  // 🆕 NEW - Simple enable/disable
  lastSeen?: string;
}

// Add simple methods to ToolRegistry
async enableTool(name: string): Promise<void>
async disableTool(name: string): Promise<void> 
async enableCategory(category: string): Promise<void>
async disableCategory(category: string): Promise<void>
```

### **Step 2: Core Dependencies (Pragmatic)**
```json
{
  "dependencies": {
    "langchain": "^0.3.0",              // 🔥 CORE - accept 150MB 
    "@langchain/anthropic": "^0.3.0",   // Claude tools
    "@langchain/openai": "^0.3.0",      // OpenAI tools
    "@anthropic-ai/sdk": "^0.24.0",     // Direct Claude API
    "openai": "^4.0.0"                  // Direct GPT API
  },
  "optionalDependencies": {
    "puppeteer": "^21.0.0"              // Only when needed
  }
}
```

### **Step 3: Add LLM Tools as Direct API Calls**
**Goal**: Make LLMs available as tools for agents to use
```typescript
// llm-tools/anthropic.ts
export const claudeSonnet = {
  name: 'claude_sonnet',
  description: 'Claude 3.5 Sonnet - Best reasoning and analysis',
  category: 'llm',
  requiredSecrets: ['ANTHROPIC_API_KEY'],
  
  async execute(input: { messages: any[], temperature?: number }) {
    const client = new Anthropic({ apiKey: await getSecret('ANTHROPIC_API_KEY') });
    const response = await client.messages.create({
      model: 'claude-3-5-sonnet-20241022', 
      messages: input.messages,
      temperature: input.temperature || 0.7,
      max_tokens: 4000
    });
    return { content: response.content, usage: response.usage };
  }
} as RCRTTool;
```

### **Step 4: Expand LangChain Tools (The Heart)**
**Goal**: Add 20-50 real LangChain tools that agents will actually use
```typescript
// langchain-tools/search/index.ts
export const searchTools = [
  'serpapi',           // ✅ Already working
  'brave_search',      // ✅ Already working  
  'duckduckgo_search', // 🆕 Add - privacy-focused search
  'google_search',     // 🆕 Add - Google Custom Search API
  'wikipedia',         // 🆕 Add - Wikipedia lookup
  'arxiv_search'       // 🆕 Add - Academic paper search
];

// langchain-tools/data/index.ts  
export const dataTools = [
  'csv_processor',     // 🆕 Read/write/transform CSV files
  'json_transformer',  // 🆕 Complex JSON manipulation
  'xml_parser',        // 🆕 XML/HTML processing
  'file_operations',   // 🆕 Read/write files, directory ops
  'data_validator'     // 🆕 Schema validation for data
];

// langchain-tools/web/index.ts
export const webTools = [
  'web_scraper',       // 🆕 Enhanced scraping with selectors
  'api_client',        // 🆕 REST API interaction tool
  'form_filler',       // 🆕 Web form automation
  'screenshot_tool',   // 🆕 Take webpage screenshots
  'link_checker'       // 🆕 Validate URLs and check status
];
```

### **Step 5: Simple Tool Management for Agents**
**Goal**: Easy enable/disable for managing large catalogs
```typescript
// Simple agent tool selection
class AgentToolManager {
  async getEnabledTools(category?: string): Promise<ToolCatalogEntry[]> {
    const catalog = await this.registry.getCatalog();
    return catalog.filter(tool => 
      tool.enabled && 
      tool.status === 'active' &&
      (!category || tool.category === category)
    );
  }
  
  async createAgentWithLimitedTools(agentConfig: any): Promise<void> {
    // Agent only sees enabled tools 
    const availableTools = await this.getEnabledTools();
    
    await this.client.createBreadcrumb({
      schema_name: 'agent.def.v1',
      title: `Agent: ${agentConfig.name}`,
      context: {
        ...agentConfig,
        available_tools: availableTools.map(t => t.name)
      }
    });
  }
}
```

### **Step 6: Simple Base Classes for Easy Tool Creation**
**Goal**: Make it easy to create new tools, not over-engineer
```typescript
// utils/base-tool.ts (SIMPLE VERSION)
export abstract class SimpleLLMTool implements RCRTTool {
  abstract name: string;
  abstract description: string; 
  abstract model: string;
  
  category = 'llm';
  version = '1.0.0';
  
  // Common LLM input schema
  get inputSchema() {
    return {
      type: 'object',
      properties: {
        messages: { type: 'array', description: 'Chat messages' },
        temperature: { type: 'number', default: 0.7, min: 0, max: 2 },
        max_tokens: { type: 'number', default: 4000 }
      },
      required: ['messages']
    };
  }
  
  get outputSchema() {
    return {
      type: 'object', 
      properties: {
        content: { type: 'string' },
        usage: { type: 'object' }
      }
    };
  }
  
  // Subclasses just implement this
  abstract execute(input: any, context: ToolExecutionContext): Promise<any>;
}

// Example: Easy tool creation
export class GPT4MiniTool extends SimpleLLMTool {
  name = 'gpt4_mini';
  description = 'GPT-4 Mini - Fast and cost-effective';
  model = 'gpt-4o-mini';
  requiredSecrets = ['OPENAI_API_KEY'];
  
  async execute(input: any) {
    // Simple implementation - no complex tracking
    const openai = new OpenAI({ apiKey: await getSecret('OPENAI_API_KEY') });
    const response = await openai.chat.completions.create({
      model: this.model,
      messages: input.messages,
      temperature: input.temperature || 0.7,
      max_tokens: input.max_tokens || 4000
    });
    return { 
      content: response.choices[0].message.content,
      usage: response.usage
    };
  }
}
```

### **Step 4: Enhanced Tool Categories**

#### **LLM Tools (Core Innovation)**
```typescript
// llm/index.ts
import { ClaudeSonnetTool } from './anthropic/claude-sonnet.js';
import { GPT4Tool } from './openai/gpt4.js';
import { GPT4MiniTool } from './openai/gpt4-mini.js';
import { OpenRouterTool } from './openrouter/openrouter.js';
import { OllamaTool } from './local/ollama.js';

export const llmTools = [
  ClaudeSonnetTool,
  GPT4Tool, 
  GPT4MiniTool,
  OpenRouterTool,
  OllamaTool
];

export const LLM_CATEGORIES = {
  ANALYSIS: ['claude_sonnet', 'gpt4'],           // Best for deep reasoning
  CODING: ['gpt4', 'claude_sonnet'],             // Best for code generation
  FAST_CHEAP: ['gpt4_mini', 'local_llama'],      // Cost-effective options
  CREATIVE: ['claude_sonnet', 'gpt4'],           // Creative writing
  LOCAL_PRIVATE: ['ollama', 'llamacpp']          // Privacy-focused
};

export function chooseLLMForTask(taskType: string, constraints?: { maxCost?: number, maxLatency?: number }): string {
  const taskMappings = {
    'deep_analysis': 'claude_sonnet',     // $0.003/1K tokens
    'code_generation': 'gpt4',           // $0.03/1K tokens  
    'simple_task': 'gpt4_mini',          // $0.0005/1K tokens
    'creative_writing': 'claude_sonnet',  // Excellent prose
    'math_reasoning': 'gpt4',            // Strong math
    'private_data': 'ollama'             // Local processing
  };
  
  let selected = taskMappings[taskType] || 'claude_sonnet';
  
  // Apply constraints
  if (constraints?.maxCost && selected === 'gpt4' && constraints.maxCost < 0.01) {
    selected = 'gpt4_mini';  // Switch to cheaper option
  }
  
  if (constraints?.maxLatency && constraints.maxLatency < 1000) {
    selected = 'ollama';  // Switch to local for speed
  }
  
  return selected;
}
```

#### **Enhanced LangChain Integration**
```typescript
// langchain/search/serpapi.ts
export class EnhancedSerpAPITool extends EnhancedBaseTool {
  name = 'serpapi_enhanced';
  description = 'Advanced web search with result caching and filtering';
  category = 'search';
  
  requiredSecrets = ['SERPAPI_API_KEY'];
  
  private cache = new Map<string, CachedResult>();
  private resultProcessor = new SearchResultProcessor();
  
  constructor() {
    super({
      rateLimit: { requests: 100, window: '1d' } // 100 searches per day
    });
  }
  
  inputSchema = {
    type: 'object',
    properties: {
      query: { type: 'string' },
      filters: {
        type: 'object',
        properties: {
          date_range: { type: 'string', enum: ['day', 'week', 'month', 'year'] },
          domain_filter: { type: 'array', items: { type: 'string' } },
          content_type: { type: 'string', enum: ['news', 'academic', 'general'] },
          language: { type: 'string', default: 'en' }
        }
      },
      result_processing: {
        type: 'object',
        properties: {
          max_results: { type: 'number', default: 10, max: 100 },
          include_snippets: { type: 'boolean', default: true },
          extract_content: { type: 'boolean', default: false },
          deduplicate: { type: 'boolean', default: true }
        }
      }
    },
    required: ['query']
  };
  
  async execute(input: SerpAPIRequest, context: ToolExecutionContext): Promise<EnhancedSearchResult> {
    // 1. Check cache first
    const cacheKey = this.generateCacheKey(input);
    if (this.cache.has(cacheKey)) {
      const cached = this.cache.get(cacheKey)!;
      if (Date.now() - cached.timestamp < 3600000) { // 1 hour cache
        return { ...cached.result, from_cache: true };
      }
    }
    
    // 2. Execute search via LangChain
    const serpApi = new SerpAPI(await this.getSecret('SERPAPI_API_KEY', context));
    const rawResults = await serpApi.call(input.query);
    
    // 3. Process and enhance results
    const processed = await this.resultProcessor.process(rawResults, input.filters);
    
    // 4. Cache results
    this.cache.set(cacheKey, {
      result: processed,
      timestamp: Date.now()
    });
    
    return processed;
  }
}
```

---

## 📋 **Migration Steps**

### **Week 1: Foundation**
1. **Create new directory structure**
2. **Move existing tools** to appropriate directories  
3. **Update package.json** with LangChain as core dependency
4. **Create base classes** (EnhancedBaseTool, BaseLLMTool)
5. **Implement enhanced registry** with enable/disable

### **Week 2: LLM Tools** 
1. **Implement BaseLLMTool** abstract class
2. **Create Claude Sonnet tool** (anthropic/claude-sonnet.ts)
3. **Create GPT-4 tool** (openai/gpt4.ts)
4. **Create OpenRouter tool** (openrouter/openrouter.ts)
5. **Add cost tracking** and usage statistics

### **Week 3: Enhanced LangChain**
1. **Restructure LangChain tools** into subcategories
2. **Add advanced search tools** with caching
3. **Implement enhanced calculator** with complex math
4. **Add web scraping tools** with content extraction
5. **Create data processing tools**

### **Week 4: Integration Tools**
1. **Create database tool category** (postgres, redis, mongodb)
2. **Implement code execution tools** with sandboxing
3. **Add GitHub integration** tool
4. **Create Slack/Discord** communication tools
5. **Build comprehensive testing suite**

---

## 🔧 **Build Configuration**

### **Enhanced tsup.config.ts**
```typescript
export default defineConfig({
  entry: {
    // Core exports
    index: 'src/index.ts',
    registry: 'src/registry.ts',
    
    // Category exports for selective loading
    builtin: 'src/builtin/index.ts',
    llm: 'src/llm/index.ts',
    langchain: 'src/langchain/index.ts', 
    database: 'src/database/index.ts',
    code: 'src/code/index.ts',
    content: 'src/content/index.ts',
    integration: 'src/integration/index.ts',
    
    // Utilities
    utils: 'src/utils/index.ts'
  },
  
  format: ['cjs', 'esm'],
  dts: true, // Generate type definitions
  splitting: false,
  sourcemap: true,
  clean: true,
  
  // External dependencies (don't bundle)
  external: [
    'dockerode',
    'puppeteer', 
    'sharp',
    'ffmpeg-static',
    'pg',
    'redis'
  ],
  
  // Bundle LangChain since it's core now
  noExternal: ['langchain', '@langchain/*']
});
```

### **Enhanced package.json exports**
```json
{
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js", 
      "types": "./dist/index.d.ts"
    },
    "./registry": {
      "import": "./dist/registry.mjs",
      "require": "./dist/registry.js"
    },
    "./builtin": {
      "import": "./dist/builtin.mjs", 
      "require": "./dist/builtin.js"
    },
    "./llm": {
      "import": "./dist/llm.mjs",
      "require": "./dist/llm.js"
    },
    "./langchain": {
      "import": "./dist/langchain.mjs",
      "require": "./dist/langchain.js"
    },
    "./database": {
      "import": "./dist/database.mjs",
      "require": "./dist/database.js"
    },
    "./code": {
      "import": "./dist/code.mjs", 
      "require": "./dist/code.js"
    },
    "./utils": {
      "import": "./dist/utils.mjs",
      "require": "./dist/utils.js"
    }
  }
}
```

---

## 🎯 **Usage Examples After Refactor**

### **Selective Tool Loading**
```typescript
// Load only needed categories
import { createToolRegistry } from '@rcrt-builder/tools/registry';
import { llmTools } from '@rcrt-builder/tools/llm';
import { searchTools } from '@rcrt-builder/tools/langchain';

const registry = await createToolRegistry(client, 'workspace:production', {
  categories: ['llm', 'search', 'builtin'], // Only load these
  enableUI: true,
  autoDisable: {
    costThreshold: 50.00,     // Auto-disable if daily cost > $50
    errorRateThreshold: 0.1   // Auto-disable if error rate > 10%
  }
});
```

### **Dynamic Tool Management**
```typescript
// Runtime tool management
await registry.disableTool('gpt4', 'Cost optimization - switching to Claude');
await registry.enableTool('claude_sonnet', 'More cost-effective for current workload');

// Category management  
await registry.disableCategory('content');  // Disable all image/video tools
await registry.enableCategory('database');   // Enable all DB tools

// Get usage report
const report = await registry.getToolUsageReport();
console.log(`Daily cost: $${report.total_cost_today}`);
```

### **Agent-Based Tool Selection**
```typescript
// Agents can now choose optimal LLM tools
class SmartSupervisorAgent {
  async chooseOptimalLLM(task: Task): Promise<string> {
    const catalog = await this.getToolCatalog();
    const llmTools = catalog.tools.filter(t => t.category === 'llm' && t.enabled);
    
    // Choose based on task requirements
    if (task.complexity === 'high' && task.budget > 0.05) {
      return 'claude_sonnet';  // Best reasoning
    } else if (task.type === 'code') {
      return 'gpt4';          // Best for code
    } else {
      return 'gpt4_mini';     // Cost-effective default
    }
  }
  
  async executeTaskWithOptimalTool(task: Task) {
    const llmTool = await this.chooseOptimalLLM(task);
    
    await this.client.createBreadcrumb({
      schema_name: 'tool.request.v1',
      context: {
        tool: llmTool,
        input: {
          messages: this.buildMessages(task),
          temperature: task.creativity || 0.7
        }
      }
    });
  }
}
```

---

## 🧪 **Testing Strategy**

### **Tool Testing Framework** 
```typescript
// utils/testing/test-helpers.ts
export class ToolTester {
  static async testTool(tool: RCRTTool, testCases: ToolTestCase[]): Promise<TestResult[]> {
    const results: TestResult[] = [];
    
    for (const testCase of testCases) {
      const result = await this.runTestCase(tool, testCase);
      results.push(result);
    }
    
    return results;
  }
  
  private static async runTestCase(tool: RCRTTool, testCase: ToolTestCase): Promise<TestResult> {
    try {
      const context = this.createMockContext(testCase.agentId);
      const startTime = Date.now();
      
      const result = await tool.execute(testCase.input, context);
      const duration = Date.now() - startTime;
      
      return {
        testName: testCase.name,
        status: 'passed',
        result,
        duration,
        cost: result.cost_tracking?.estimated_cost
      };
      
    } catch (error) {
      return {
        testName: testCase.name,
        status: 'failed',
        error: error.message,
        duration: 0
      };
    }
  }
}

// Example test suite
const claudeTests: ToolTestCase[] = [
  {
    name: 'simple_analysis',
    input: {
      messages: [{ role: 'user', content: 'Explain quantum computing in one sentence' }]
    },
    expectedFields: ['content', 'usage', 'cost_tracking']
  },
  {
    name: 'code_generation', 
    input: {
      messages: [{ role: 'user', content: 'Write a Python function to calculate fibonacci' }],
      temperature: 0.1
    },
    expectedFields: ['content']
  }
];

await ToolTester.testTool(new ClaudeSonnetTool(), claudeTests);
```

---

## 📊 **Performance and Monitoring**

### **Tool Performance Dashboard**
```typescript
// Create monitoring breadcrumbs for dashboard
{
  schema_name: 'tool.performance.v1',
  title: 'Daily Tool Performance Report',
  tags: ['workspace:tools', 'monitoring', 'performance'],
  context: {
    date: '2025-01-10',
    total_requests: 1547,
    total_cost: 23.45,
    avg_latency_ms: 1250,
    
    by_category: {
      'llm': { requests: 890, cost: 18.90, avg_latency: 1800 },
      'search': { requests: 234, cost: 3.20, avg_latency: 800 },
      'database': { requests: 423, cost: 1.35, avg_latency: 150 }
    },
    
    top_tools: [
      { name: 'claude_sonnet', requests: 456, cost: 12.34 },
      { name: 'serpapi', requests: 234, cost: 3.20 },
      { name: 'postgres', requests: 423, cost: 1.35 }
    ],
    
    alerts: [
      { tool: 'gpt4', message: 'High cost: $15.67 today' },
      { tool: 'code_executor', message: 'High error rate: 15%' }
    ]
  }
}
```

---

## ✅ **Realistic Success Criteria**

### **Phase 1 Complete When:**
1. ✅ **Foundation stable**: Registry system working (DONE!)
2. ✅ **LangChain core**: 150MB dependency accepted as heart of system
3. ✅ **Simple enable/disable**: Basic flags for managing 200+ tool catalogs
4. ✅ **LLM tools working**: Claude, GPT-4, OpenRouter available as tools
5. ✅ **Easy tool creation**: Simple base classes, no complex frameworks
6. ✅ **Clean integration**: New tools fit existing proven pattern

### **Practical Goals (No Over-Engineering):**
- **🎯 Tool count**: Expand from 12 test tools → 50-100 real tools
- **⚡ Simple registration**: New tools integrate easily with existing registry
- **🔧 Enable/disable**: Agents can manage large catalogs (200+ overwhelming LLMs)
- **💡 Easy creation**: Adding new LangChain tools = 5-10 lines of code
- **🏗️ Foundation first**: Build on proven working system

### **What We're NOT Building:**
- ❌ Complex cost tracking (LangFuse later if needed)
- ❌ Performance dashboards (breadcrumb dashboard sufficient)  
- ❌ Auto-disable logic (simple manual enable/disable enough)
- ❌ Complex base classes (keep it simple)

This creates a **stable, expandable foundation** that agents can actually use to accomplish real tasks! 🚀

**Next: Focus on agent template system** for easy agent creation with tool access.
