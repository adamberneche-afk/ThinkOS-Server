# Documentation vs. Codebase: What We Say We're Building vs. What Exists

**Purpose:** This is a capability-level comparison, not a line-by-line doc-accuracy check (see git history for those fixes). For each capability the documentation describes, this cross-checks the actual source code to classify it as **BUILT** (real, wired-up implementation), **PARTIAL** (real code exists but key pieces are missing, unmounted, or disconnected), or **NOT BUILT** (aspirational — no corresponding code, or only a plan document).

**Methodology:** Every row below was verified by reading the actual source referenced in the Evidence column — not by trusting another doc's claim about it.

## Overall Positioning vs. Reality

The top-level docs (`README.md`, `docs/SYSTEM_ARCHITECTURE.md`) describe RCRT as a working, production-grade breadcrumb-based event system ("no mocks, no hidden fallbacks — all components are real and deployable") for coordinating LLM agents via CRUD + SSE + vector search — and that framing is largely accurate for the core Rust service, DB layer, JWT/RLS, secrets, and the browser extension/dashboard.

Separately, `rcrt-visual-builder/docs/refactor/EXECUTIVE_SUMMARY.md` claims the project is becoming "the world's first truly autonomous AI development platform" with self-bootstrapping master-supervisor agents, dynamic agent-template spawning, and auto-generated UIs. If you only read the code, the project is: a solid breadcrumb/event substrate plus two real UIs that do genuine chat/notes/3D-visualization work — but **none** of the "autonomous AI platform" machinery (master supervisor, agent templates, self-bootstrapping, UI auto-generation) exists in code at all; every one of its 5 phase docs is explicitly labeled `Status: 📋 PLANNED (not started)`.

## Capability Inventory

| # | Capability | Claiming Doc(s) | Status | Evidence |
|---|---|---|---|---|
| 1 | Breadcrumb CRUD + versioning/RLS/ACL | README, SYSTEM_ARCHITECTURE, RCRT_PRINCIPLES | **BUILT** | `crates/rcrt-server/src/main.rs` implements full CRUD, optimistic locking, JWT, and the full route table. |
| 2 | SSE + NATS event fanout | README, SYSTEM_ARCHITECTURE | **BUILT** | `/events/stream` wired in `main.rs`; extension and dashboard both consume SSE directly. |
| 3 | Vector/semantic search (pgvector + ONNX) | README, RCRT_PRINCIPLES | **BUILT** | Server search routes; extension's `semantic-search.ts`/`find-similar.ts` call them. |
| 4 | Secrets envelope encryption (AES-256-GCM + XChaCha20-Poly1305) | README | **BUILT** | Real AES-GCM DEK + XChaCha20-Poly1305 KEK wrapping in `main.rs`, with `/secrets` routes. |
| 5 | context-builder "intelligence multiplier" | SYSTEM_ARCHITECTURE | **PARTIAL** | Real Rust service exists, but `event_handler.rs` hardcodes `if schema == "user.message.v1"` — only one consumer is supported; the doc's own "Current System Gaps" section admits this. |
| 6 | Generic `context.request.v1` pattern (any agent can request context) | SYSTEM_ARCHITECTURE ("Future Enhancements") | **NOT BUILT** | No such handling exists anywhere in `event_handler.rs`. |
| 7 | Note-taking agents (tagging/summary/insights/ELI5) | SYSTEM_ARCHITECTURE, `docs/NOTE_AGENTS_SOLUTION.md` | **NOT BUILT / BROKEN** | All 4 agent definitions are still bootstrapped, and SYSTEM_ARCHITECTURE's own gap table marks all 4 as "🔴 Broken" (they bypass context-builder and get empty context). The proposed fix was never implemented. |
| 8 | Declarative workflow system (`workflow.def.v1`) | SYSTEM_ARCHITECTURE ("Future Enhancements") | **NOT BUILT** | No such schema handling anywhere; doc explicitly says "pattern designed, not implemented." (Distinct from the real imperative `workflow` *tool* below.) |
| 9 | Multi-step tool orchestration (`workflow` tool) | `bootstrap-breadcrumbs/README.md` | **BUILT** | `tools-self-contained/workflow.json` and `rcrt-visual-builder/packages/tools/src/workflow-orchestrator.ts` implement a real orchestrator (topological sort, dependency detection, variable interpolation). |
| 10 | Bootstrap system (single source of truth JSON loader) | `docs/BOOTSTRAP_SYSTEM.md`, `bootstrap-breadcrumbs/README.md` | **BUILT** | `bootstrap.js` plus populated `system/`, `tools-self-contained/`, `templates/`, `knowledge/`, `schemas/` directories. |
| 11 | Browser extension: multi-tab context tracking | SYSTEM_ARCHITECTURE, `rcrt-extension-v2/README.md` | **BUILT** | `tab-context-manager.ts` (503 lines) implements the described tagging/TTL logic. |
| 12 | Browser extension: semantic search / save-page / notes | `rcrt-extension-v2/README.md` | **BUILT** | `semantic-search.ts`, `save-page.ts`, `find-similar.ts`, `export-import.ts` all real and non-trivial. |
| 13 | Browser extension: "4 agents process notes in parallel" | `rcrt-extension-v2/README.md` | **NOT BUILT — contradicts the project's own architecture doc** | These are the same 4 broken note agents from #7; this README's feature claim directly conflicts with SYSTEM_ARCHITECTURE's "🔴 Broken" status for them. |
| 14 | Legacy `extension/` (v1) chat extension | `extension/README.md` | **BUILT but superseded** | Real implementation exists; the README itself flags it as superseded by v2. |
| 15 | Dashboard v2: 3D breadcrumb graph visualization | `rcrt-dashboard-v2/README.md` (its own roadmap marks this unchecked) | **BUILT — ahead of its own stated roadmap** | `Scene3D.tsx` (377 lines, react-three-fiber), `Canvas3D.tsx`, `Node3D.tsx` are real, despite the README's roadmap checklist showing 3D as `[ ]`. |
| 16 | Dashboard v2: self-configuration via breadcrumbs, dynamic UI rendering | `rcrt-dashboard-v2/README.md` | **BUILT** | `UIRenderer.tsx`, `DynamicPage.tsx`, `TemplateEngine.ts` present and wired up. |
| 17 | Legacy Rust dashboard (`crates/rcrt-dashboard`) | `crates/rcrt-dashboard/README.md` | **BUILT but not deployed** | Functions standalone but isn't wired into the docker-compose stack; superseded by dashboard-v2 (see the earlier doc-fix PR for the specific correction). |
| 18 | Visual Builder core packages (schemas, SDK, node-sdk, HeroUI builder) | `rcrt-visual-builder/README.md` | **BUILT** | `packages/{core,sdk,node-sdk,heroui-breadcrumbs}`, `apps/builder`, `apps/agent-runner` all populated with real source. |
| 19 | Visual Builder: React Flow drag-and-drop canvas / node palette | `rcrt-visual-builder/README.md` | **NOT BUILT — literal stub** | `FlowCanvas.tsx` and `NodePalette.tsx` are one-line `export {};` files, confirmed by direct read. |
| 20 | Visual Builder: management dashboards (DLQ Monitor, ACL Viewer, Workspace Manager) | `rcrt-visual-builder/README.md` | **NOT BUILT** | Only `AgentPanel.tsx` exists in `packages/management/src`; no DLQ/ACL/Workspace files exist anywhere. |
| 21 | Visual Builder: `SupervisorNode` (multi-agent orchestration node) | `packages/nodes/agent` | **PARTIAL — orphaned** | A real 312-line implementation exists, but it's never imported by `apps/agent-runner` or `apps/builder`, and depends on the not-built canvas to be reachable in practice. |
| 22 | "World's first autonomous AI development platform" (the Phase 1-5 vision as a whole) | `rcrt-visual-builder/docs/refactor/EXECUTIVE_SUMMARY.md` | **NOT BUILT** | All 5 `PHASE_*.md` files are headed `Status: 📋 PLANNED (not started)`; a repo-wide grep for `MasterSupervisor`, `agent.template.v1`, `AgentSpawner` returns zero hits. |
| 23 | Phase 3: Agent Template System / dynamic agent spawning | `PHASE_3_AGENT_TEMPLATE_SYSTEM.md` | **NOT BUILT** | No `agent.template.v1` schema, no `AgentSpawner` class exists anywhere. |
| 24 | Phase 4: Master Supervisor / self-bootstrapping infrastructure | `PHASE_4_SELF_BOOTSTRAPPING_INFRASTRUCTURE.md` | **NOT BUILT** | No supervisor agent definition, no knowledge-DNA breadcrumbs exist in `bootstrap-breadcrumbs/system/`. |
| 25 | Phase 5: UI Auto-Generation (agent-built interfaces) | `PHASE_5_UI_AUTO_GENERATION.md` | **NOT BUILT** | No UI-builder-agent or pattern-generator code exists anywhere; purely a design doc. |
| 26 | Hygiene/TTL auto-cleanup + Prometheus metrics | README, SYSTEM_ARCHITECTURE | **BUILT** | `crates/rcrt-server/src/hygiene.rs`, `/hygiene/run`, `/hygiene/stats`, and `/metrics` routes all present in `main.rs`. |

## Rollup

**14 BUILT · 3 PARTIAL · 9 NOT BUILT** (out of 26 capabilities catalogued)

The pattern: the breadcrumb/event substrate and both UIs (browser extension, dashboard) genuinely match their "production-grade, no mocks" framing. The 4 note-taking agents are advertised as a working feature in one doc while the project's own architecture doc lists them as broken. And the entire "autonomous AI development platform" vision — the most ambitious framing in the repo — has zero implementation behind any of its 5 phases; it's explicitly labeled as a plan, not a status report, in its own source documents.
