# Single Tool Catalog Implementation

## Problem Solved

Previously, the tool system had a design flaw where:
- ❌ Multiple `tool.catalog.v1` breadcrumbs were created (one per publishCatalog call)  
- ❌ Individual `tool.definition.v1` breadcrumbs were created for each tool
- ❌ No single source of truth for tool discovery
- ❌ Inefficient for agents to query multiple breadcrumbs

## Solution Implemented (current: `src/bootstrap-tools.ts`)

### ✅ Single Catalog Breadcrumb Approach

**Key Changes:**

1. **tool.code.v1, not tool.v1/tool.definition.v1**: `bootstrapTools()` explicitly skips creating
   `tool.v1` breadcrumbs for built-ins now (see the `🔧 Legacy tool.v1 bootstrap skipped — all
   tools are now tool.code.v1` log line). The catalog is built exclusively from `tool.code.v1`
   breadcrumbs tagged with the workspace; `tool.definition.v1` is not used at all.

2. **No cached catalog ID**: There is no `catalogBreadcrumbId` field or `initializeCatalog()`
   method. The actual function, `updateToolCatalog(client, workspace)` (private to
   `bootstrap-tools.ts`, called from `bootstrapTools()`), searches for the existing
   `tool.catalog.v1` breadcrumb for the workspace on every call and creates one only if the
   search comes back empty — there's nothing persisted in memory between calls.

3. **Update Instead of Create**: When a `tool.catalog.v1` is found, `updateToolCatalog()` updates
   it via `client.updateBreadcrumb(id, version, { context })` (optimistic concurrency via the
   breadcrumb's `version`/If-Match). If none is found, it creates one fresh — the same
   search-then-create path handles the "recreate if deleted externally" case too, since there's
   no separate recovery branch.

4. **Removed Individual Definitions**: `tool.definition.v1` breadcrumbs are not created; catalog entries are read directly off each `tool.code.v1` breadcrumb's `context` (`name`, `description`, `category`, `version`, `input_schema`, `output_schema`, `examples`, `capabilities`).

## Result

### Before:
```bash
# Multiple breadcrumbs to query
GET /breadcrumbs?schema_name=tool.catalog.v1     # Returns: [catalog1, catalog2, catalog3, ...]
GET /breadcrumbs?schema_name=tool.definition.v1  # Returns: [tool1, tool2, tool3, ...]
```

### After:
```bash
# Single catalog breadcrumb per workspace  
GET /breadcrumbs?tag=workspace:tools&schema_name=tool.catalog.v1  # Returns: [single_catalog]

# Catalog structure:
{
  "id": "catalog-uuid",
  "schema_name": "tool.catalog.v1", 
  "title": "workspace:tools Tool Catalog",
  "version": 5,  // Increments with each update
  "context": {
    "workspace": "workspace:tools",
    "tools": [
      {
        "name": "serpapi",
        "description": "Search the web using Google", 
        "status": "active",
        "category": "search",
        "inputSchema": { /* schema */ },
        "outputSchema": { /* schema */ },
        "lastSeen": "2025-01-10T..."
      },
      // ... all other tools
    ],
    "totalTools": 5,
    "activeTools": 4, 
    "lastUpdated": "2025-01-10T..."
  }
}
```

## Benefits

✅ **Efficient Discovery**: Agents query one breadcrumb instead of many  
✅ **Real-time Updates**: Catalog updates when tools are added/removed  
✅ **Version History**: See how tool availability changed over time via breadcrumb versions  
✅ **Clean Data**: No duplicate or stale tool definitions  
✅ **Event-Driven**: Catalog changes trigger SSE events for real-time UI updates  

## Agent Integration

Agents can now efficiently discover tools:

```typescript
// Simple, efficient tool discovery
const [catalog] = await client.searchBreadcrumbs({
  tag: 'workspace:tools',
  schema_name: 'tool.catalog.v1'
});

const availableTools = catalog.context.tools.filter(t => t.status === 'active');

// Use a tool
await client.createBreadcrumb({
  schema_name: 'tool.request.v1',
  context: { 
    tool: availableTools[0].name,
    input: { /* tool input */ }
  }
});
```

This implementation provides a much cleaner, more efficient, and more maintainable approach to tool catalog management in the RCRT ecosystem.
