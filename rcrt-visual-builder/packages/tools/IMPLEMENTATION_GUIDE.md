# Tool System Implementation Guide

> This file previously described a `RCRTToolWrapper` class (with `createToolBreadcrumb()` and
> a registry `buildCatalogFromBreadcrumbs()` method) as an upcoming change. That plan was never
> implemented — there is no `RCRTToolWrapper`, `createToolBreadcrumb`, or `buildCatalogFromBreadcrumbs`
> anywhere in this package. The content below describes how tool loading actually works today.

## How tools actually load today

Tools are backed by breadcrumbs of two schemas:

- **`tool.code.v1`** (current, self-contained format): the breadcrumb carries its own code and
  runs inside `DenoToolRuntime`. This is the format new tools should use.
- **`tool.v1`** (legacy format): describes an `implementation` (builtin/module/http/breadcrumb/container)
  that `ToolLoader` resolves to a runnable `RCRTTool`. Still supported for backward compatibility,
  but `bootstrap-tools.ts` no longer creates these — see below.

### Loading a tool: `ToolLoader` (`src/tool-loader.ts`)

```typescript
import { ToolLoader } from '@rcrt-builder/tools';

const loader = new ToolLoader(client, 'workspace:tools');

// By breadcrumb ID — tries tool.v1 vs tool.code.v1 based on schema_name
const tool = await loader.loadToolFromBreadcrumb(breadcrumbId);

// By name — searches tool.code.v1 first, falls back to tool.v1
const tool2 = await loader.loadToolByName('file-storage');

// Discover everything available in the workspace (both schemas)
const available = await loader.discoverTools();
```

For `tool.v1` breadcrumbs with `implementation.type === 'builtin'`, `ToolLoader` dynamically
imports the named module (e.g. `@rcrt-builder/tools`) and walks `implementation.export`
(dot or bracket notation, e.g. `builtinTools['file-storage']`) to find the tool, instantiating
it if `implementation.instantiate` is true.

For `tool.code.v1` breadcrumbs, `loadSelfContainedTool()` returns a stub `RCRTTool` whose
`execute()` throws — actual execution for this format is routed through `DenoToolRuntime`,
not through the stub.

### Bootstrapping the catalog: `bootstrapTools` (`src/bootstrap-tools.ts`)

```typescript
import { bootstrapTools } from '@rcrt-builder/tools';

await bootstrapTools(client, 'workspace:tools');
```

`bootstrapTools()` no longer creates `tool.v1` breadcrumbs for the built-in tools (that step is
explicitly skipped — see the `🔧 Legacy tool.v1 bootstrap skipped` log line in the source). All
tools are expected to already exist as `tool.code.v1` breadcrumbs, created out-of-band via
`bootstrap-breadcrumbs/tools-self-contained/`. What `bootstrapTools()` actually does is call
`updateToolCatalog()`, which:

1. Searches for `tool.code.v1` breadcrumbs tagged with the workspace.
2. Fetches each one's full breadcrumb to read its metadata (`name`, `description`, `category`,
   `input_schema`, `output_schema`, `examples`, `capabilities`).
3. Builds a `tool.catalog.v1` context object from that list.
4. Searches for an existing `tool.catalog.v1` breadcrumb for the workspace and updates it if
   found, or creates a new one if not — there is no cached catalog breadcrumb ID kept between
   calls; every call re-searches.

### In-process built-ins: `builtinTools` (`src/index.ts`)

`builtinTools` is a plain object exporting ready-made `RCRTTool` implementations by key:
`agent-helper`, `file-storage`, `agent-loader`, `workflow`, `browser-context-capture`. These are
what `implementation.export` in a `tool.v1` breadcrumb points at (e.g. `builtinTools.workflow`),
and what a `tool.code.v1` breadcrumb's `export` field is expected to name if it uses the
`builtin` implementation type.

## Adding a new tool

1. Implement it as an `RCRTTool` (see `src/index.ts` for the interface) or via `createTool(...)`.
2. Either add it to `builtinTools` in `src/index.ts`, or publish it as a self-contained
   `tool.code.v1` breadcrumb (see `bootstrap-breadcrumbs/tools-self-contained/` for examples).
3. Run `bootstrapTools(client, workspace)` (or wait for tools-runner's normal startup) so the
   workspace's `tool.catalog.v1` picks it up.
