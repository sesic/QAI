# Flipbook for QA - Product and Implementation Plan

## Purpose

Flipbook for QA is a visual, recursive QA workspace. Every interaction opens a fresh page. Each page has tappable regions that drill into implementation detail, runtime behavior, risks, evidence, and suggested fixes. The same drill-in graph must serve both humans through a visual flipbook client and AI agents through structured JSON, MCP, REST, and streaming APIs.

This plan keeps the full product scope from the original proposal and makes the work executable from an empty repository. It also tightens the original plan in places where it was too speculative: performance claims become benchmark gates, product quality becomes explicit work, and the human and agent surfaces share one schema from day one.

## Product Principles

1. **One graph, many consumers.** The `FlipbookGraph` is the product contract. Visual pages, MCP resources, CI findings, and agent drill-ins all serialize from the same graph.
2. **Show motion before certainty.** A tap should produce a skeleton or partial page immediately, then stream progressively better evidence, risks, and hotspots.
3. **Visuals must carry meaning.** Flipbook pages are not decorative screenshots. Each page should make structure, causality, risk, or state easier to understand than a text report.
4. **Humans stay oriented.** Every page needs history, breadcrumb context, source references, tappable affordances, confidence, and a way to return to prior pages.
5. **Agents get stable contracts.** Agent consumers need deterministic IDs, source ranges, confidence fields, provenance, and a streaming event protocol that does not depend on UI implementation details.
6. **Self-hosted first, SaaS-ready by architecture.** The MVP must run locally against a repository, while auth, tenancy, storage, and provider boundaries must not block a hosted version later.

## Improvements Over The Existing Plan

- Add a repository bootstrap sequence because the current worktree is empty.
- Replace unverified performance assertions with benchmark tasks and release gates.
- Add explicit UX, accessibility, and design-system tasks so the product feels like a new QA paradigm rather than a JSON viewer.
- Add prompt evaluation, golden fixtures, and regression tests for Mapper, Curator, Risk, Trace, and Generator agents.
- Add privacy, secret handling, local data retention, and provider-boundary requirements for self-hosted use.
- Add failure-mode UX for slow LLMs, missing indexes, unsupported languages, unavailable trace data, and image-generation failures.
- Add a partner alpha program and metrics loop so phase exits are tied to observed review quality.
- Add compatibility work for Vite, Next.js, Babel, SWC, monorepos, and common test runners.
- Add observability, replay, and trace tooling so latency and quality regressions are debuggable.

## Technical Direction

### Default Stack

- **Language:** TypeScript-first monorepo.
- **Runtime:** Node 22 LTS for local coordinator, workers, CLI, MCP server, and GitHub Action.
- **Frontend:** Vite, React, TanStack Router, Zustand, Tailwind, motion primitives, and a template renderer.
- **Server:** Hono HTTP server with REST, SSE, and WebSocket endpoints.
- **Schemas:** Zod as source of truth, emitted to JSON Schema and OpenAPI where needed.
- **Storage:** SQLite for local metadata, symbol graph, vector metadata, page sessions, and replay fixtures.
- **Code parsing:** tree-sitter native bindings with language-specific queries.
- **Embeddings:** provider abstraction with hosted and local options.
- **LLM integration:** provider abstraction, structured tool output, streaming, prompt caching where available, deterministic test stubs.
- **Instrumentation:** shared trace transform with SWC and Babel adapters plus a small browser runtime.
- **Agent surface:** MCP server and equivalent REST/SSE API backed by the same coordinator contract.

### Rust Escape Hatches

Do not start with a Rust-first architecture. The product risk is agent quality, graph design, visual comprehension, and workflow fit. Keep Rust as a measured optimization path:

- Port vector/index hot paths when repositories over 1M vectors show p95 lookup above 25ms or cold indexing above the install-time target.
- Port trace ingestion when sustained event throughput exceeds the Node implementation's p99 latency or memory targets.
- Port transform internals only if SWC/Babel instrumentation becomes a meaningful HMR or build bottleneck in partner projects.

Each escape hatch must have a reproducible benchmark and an owner before migration begins.

## Repository Shape

Create this monorepo structure:

```text
.
├── apps/
│   ├── client/             # React flipbook UI
│   ├── coordinator/        # Local HTTP/SSE/WebSocket server entry
│   ├── cli/                # flipbook init, serve, index, replay
│   ├── demo-app/           # Dogfood app with realistic bugs and flows
│   ├── mcp-server/         # MCP transport adapter
│   └── github-action/      # PR review automation
├── packages/
│   ├── agents/
│   │   ├── llm/
│   │   ├── mapper/
│   │   ├── curator/
│   │   ├── risk/
│   │   ├── trace/
│   │   └── generator/
│   ├── coordinator/        # Orchestration library used by server, CLI, tests
│   ├── indexer/            # Parser, symbol graph, embeddings, store, watcher
│   ├── runtime/            # Browser trace runtime
│   ├── shared/             # Schemas, IDs, event protocol, utilities
│   ├── templates/          # Page templates and visual primitives
│   └── trace-plugin/       # SWC and Babel adapters
├── tools/
│   ├── bench/              # Latency, indexing, trace, and render benchmarks
│   ├── evals/              # Agent quality and prompt regression suites
│   └── fixtures/           # Repos, sessions, graph fixtures, screenshots
└── docs/
    ├── architecture.md
    ├── graph-protocol.md
    ├── self-hosting.md
    ├── security.md
    └── alpha-playbook.md
```

## Core Contracts

### FlipbookGraph

Define this before building agents:

- `GraphSession`: repository metadata, provider config, creation time, permissions, and root page ID.
- `FlipbookPage`: stable ID, parent ID, title, page kind, visual template, confidence, status, source references, agent provenance, and timestamps.
- `Hotspot`: stable ID, bounding box or semantic anchor, label, action, target query, confidence, keyboard order, and accessibility text.
- `Evidence`: file path, symbol ID, source range, runtime event, trace span, screenshot, log, test result, or external issue link.
- `Finding`: severity, category, confidence, affected evidence, impact, suggested fix, suggested tests, and status.
- `StreamEvent`: `session.started`, `page.skeleton`, `page.patch`, `hotspot.patch`, `finding.patch`, `trace.patch`, `page.ready`, `page.failed`, and `session.done`.

### Quality Requirements

- All IDs are deterministic within a repository snapshot and session.
- Every LLM-produced claim must include provenance or be clearly marked as inference.
- Every page must be serializable without the React client.
- Every visible hotspot must have an equivalent JSON action for agents.
- Every stream event must be replayable from fixtures.

## Workstreams

### 1. Product Experience

Tasks:

- Design the core navigation model: page stack, breadcrumb, back/forward, branch history, session resume, and page comparison.
- Define template taxonomy: component map, data flow, state machine, request lifecycle, dependency graph, risk board, trace timeline, test coverage map, generated image page, and source evidence page.
- Build a restrained, professional UI system for repeated review work: dense pages, clear hierarchy, accessible hotspots, low-friction navigation, and no marketing-style shell.
- Add empty, loading, streaming, degraded, and failed states for every page type.
- Add keyboard navigation for hotspots and history.
- Add visual confidence and provenance indicators without overwhelming the page.
- Add local session replay so users can reopen and share a review path.

Acceptance:

- A user can start from a root page, drill at least four levels deep, return to the root, and understand where they are.
- Every interactive visual region is accessible by mouse, touch, and keyboard.
- Page rendering has no layout shift when streamed patches arrive.

### 2. Graph, Schemas, and Streaming

Tasks:

- Implement `packages/shared/schemas` with Zod schemas for graph, page, hotspot, evidence, finding, and stream events.
- Generate JSON Schema and OpenAPI artifacts from the same source.
- Implement schema versioning and migration hooks.
- Implement replayable SSE protocol fixtures.
- Add a graph validator that rejects orphan pages, broken hotspots, invalid evidence references, and missing provenance.
- Add deterministic ID helpers for repos, symbols, pages, hotspots, findings, and trace events.

Acceptance:

- The same fixture renders in the client, validates in tests, and is exposed through the MCP server.
- A recorded stream can be replayed offline into the client and produce the same graph.

### 3. Client

Tasks:

- Scaffold `apps/client` with Vite, React, routing, Tailwind, state store, test setup, and local dev commands.
- Implement the flipbook shell: page viewport, animation layer, navigation chrome, breadcrumbs, branch history, status rail, and source drawer.
- Implement hotspot overlay for templated pages with stable dimensions and accessible controls.
- Implement streaming graph ingestion through SSE.
- Implement page virtualization or cleanup so long sessions stay responsive.
- Implement template renderer and visual primitives.
- Add screenshot and interaction tests for desktop and mobile viewports.

Acceptance:

- Client can render static fixtures before coordinator integration.
- Client can consume a live stream and patch the current page without losing interaction state.
- Desktop and mobile screenshots show correctly framed pages, no overlapping text, and usable hotspot controls.

### 4. Coordinator

Tasks:

- Scaffold `apps/coordinator` and `packages/coordinator`.
- Implement session lifecycle: create, resume, tap, drill-in, search, subscribe, export.
- Implement SSE writer with immediate skeleton events and patch events.
- Implement WebSocket endpoint for trace ingestion.
- Implement in-flight tap deduplication and cancellation.
- Implement provider configuration, prompt-cache plumbing, and deterministic test mode.
- Implement local persistence for sessions, pages, and replay logs.
- Add structured logs and per-event latency measurements.

Acceptance:

- A tap emits a skeleton frame quickly, then streams patches as work completes.
- Failed agent calls produce degraded pages instead of broken sessions.
- Replay logs can reproduce a session without providers.

### 5. Templates and Visual Generation

Tasks:

- Build initial templated pages: component map, data flow, request lifecycle, state machine, risk board, trace timeline, dependency graph, source evidence, and test suggestion page.
- Define template input schemas and renderer contracts.
- Implement Curator rules for choosing templates based on tap intent, available evidence, and graph depth.
- Implement templated generator that fills pages from structured agent output.
- Add visual regression tests for all templates.
- Later, add free-form image generation with fallback to template rendering when provider calls fail.

Acceptance:

- At least nine templates render from deterministic fixtures.
- Curator can choose a reasonable template without an LLM for the core MVP.
- Generated pages preserve all hotspot and evidence links.

### 6. Indexer and Code Intelligence

Tasks:

- Implement repository scanner with ignore rules, size limits, monorepo support, and language detection.
- Implement tree-sitter parsers for TypeScript, JavaScript, TSX, JSX, Python, and Go first.
- Add symbol extraction for imports, exports, components, functions, hooks, routes, tests, and config files.
- Build source-range and cross-reference store in SQLite.
- Implement incremental indexing with file watcher and snapshot IDs.
- Implement embedding provider abstraction with hosted and local modes.
- Implement vector search and hybrid lexical plus symbol retrieval.
- Add cold-index, warm-index, and incremental-index benchmarks.

Acceptance:

- A representative app can be indexed locally and queried by symbol, path, text, and embedding.
- Incremental changes update the index without a full rebuild.
- Unsupported files are reported clearly and do not break indexing.

### 7. Mapper Agent

Tasks:

- Convert taps into retrieval queries using hotspot context, page context, source evidence, and session history.
- Retrieve candidate symbols through hybrid search and cross-reference expansion.
- Re-rank candidates with deterministic heuristics before LLM use.
- Use LLM structured output for page intent, relevant evidence, next hotspots, and confidence.
- Add prompt and retrieval evals against fixture repositories.
- Add hallucination checks that reject missing files, invalid line ranges, and impossible symbols.

Acceptance:

- Mapper returns useful evidence for known fixture taps with measurable precision and recall.
- Mapper never emits file or range references that fail validation.
- Low-confidence results degrade into search-style pages instead of pretending certainty.

### 8. Risk Agent

Tasks:

- Build risk taxonomy: correctness, UX regression, accessibility, performance, security, data integrity, test gap, observability, and maintainability.
- Implement structured risk output with severity, confidence, impact, evidence, suggested fix, and suggested tests.
- Ground findings in code, trace, tests, and session context.
- Stream first findings early while deeper analysis continues.
- Add risk eval set from seeded bugs and real partner PRs.
- Add false-positive triage and prompt regression workflow.

Acceptance:

- On fixture PRs, the agent finds seeded high-severity issues with acceptable false positives.
- Every finding has evidence or is labeled as an inference.
- Users can mark findings as useful, wrong, duplicate, or accepted for later evals.

### 9. Trace System and Instrumentation

Tasks:

- Implement `packages/runtime` with event buffer, session ID propagation, redaction hooks, and transport.
- Implement shared transform spec for wrapping JSX handlers, state setters, route transitions, and selected exported functions.
- Implement SWC adapter.
- Implement Babel adapter.
- Add framework integration docs for Vite, Next.js, and common test environments.
- Implement trace correlation agent that links runtime events to symbols and pages.
- Add throughput and memory benchmarks.

Acceptance:

- Demo app emits trace events during interaction without breaking app behavior.
- Trace timeline pages correlate events to source evidence.
- Instrumentation can be disabled cleanly and is no-op in production by default.

### 10. MCP, REST, and CI Surfaces

Tasks:

- Implement MCP server with resources for sessions, pages, evidence, and findings.
- Implement MCP tools for tap, drill-in, search, subscribe, export, and mark-finding.
- Implement REST and SSE equivalents for non-MCP clients.
- Implement GitHub Action that indexes a PR, walks changed components, posts findings, and uploads a session artifact.
- Implement auth boundaries for local, CI token, and future SaaS use.
- Add contract tests proving MCP and REST return equivalent graph data.

Acceptance:

- An AI agent can navigate the same session as a human without using the browser.
- GitHub Action can run against a demo PR and post grounded comments.
- Stream semantics are consistent across client, MCP, REST, and replay.

### 11. Free-Form Image Generation

Tasks:

- Add image generation provider abstraction with hosted primary and fallback provider.
- Implement prompt assembly from page context, evidence, and Curator intent.
- Implement skeleton-first UX while image generation runs.
- Implement hotspot extraction from generated images with vision model validation.
- Add safety checks for source leakage and provider redaction.
- Add A/B evaluation comparing template and free-form pages.

Acceptance:

- Free-form pages stay within the 5s full-page target under configured provider conditions.
- Generated pages have usable hotspots and fallback to templated pages on failure.
- Blind evaluation shows free-form pages are preferred for the page classes where they are enabled.

### 12. Security, Privacy, and Local Control

Tasks:

- Add provider configuration for local-only, hosted embeddings, hosted LLM, and hosted image generation.
- Add secret scanning and redaction before provider calls.
- Add path allowlists, ignore patterns, max file size limits, and binary-file handling.
- Add local data retention controls for sessions, traces, screenshots, and embeddings.
- Add audit log for external provider requests.
- Document self-hosted threat model and SaaS-ready tenancy boundaries.

Acceptance:

- Users can run without sending code to hosted providers when configured.
- Provider requests are inspectable and redact known secret patterns.
- Session exports clearly identify what data they contain.

### 13. Observability, Benchmarks, and Evals

Tasks:

- Add benchmark harnesses for first frame, full page, client render, indexing, vector search, trace throughput, and replay.
- Add LLM eval harness for Mapper, Curator, Risk, Trace, and Hotspot extraction.
- Add golden fixtures from demo app, synthetic repos, and partner-approved anonymized sessions.
- Add dashboards or CLI reports for latency, token cost, cache hit rate, eval score, and finding quality.
- Add CI gates for schema validation, replay determinism, benchmark smoke tests, and eval regression budgets.

Acceptance:

- Release candidates include benchmark and eval reports.
- Latency regressions can be traced to stream, retrieval, provider, render, or trace phases.
- Prompt changes require eval diffs before merge.

## Milestones

### Milestone 0: Repo Foundation

Target: 3 to 5 days.

Tasks:

- Create pnpm workspace, root TypeScript config, linting, formatting, Vitest, Playwright, and Turborepo.
- Add initial `docs/architecture.md`, `docs/graph-protocol.md`, and `docs/self-hosting.md`.
- Add package boundaries and placeholder exports.
- Add CI workflow for typecheck, lint, test, and schema validation.
- Add deterministic fixture conventions.

Gate:

- `pnpm install`, `pnpm test`, `pnpm typecheck`, and fixture validation pass from a clean checkout.

### Milestone 1: Static Flipbook Prototype

Target: 2 weeks.

Tasks:

- Implement graph schemas, validator, and static fixtures.
- Implement React client shell, page stack, history, breadcrumbs, hotspot overlay, and source drawer.
- Implement at least nine templates from deterministic fixtures.
- Implement coordinator serving fixture sessions over REST and SSE.
- Add screenshot tests for desktop and mobile.

Gate:

- A user can navigate a hand-authored drill-in graph end to end.
- Client consumes fixture streams and replay streams.
- Accessibility smoke tests pass for keyboard hotspot navigation.

### Milestone 2: Local Repo Intelligence

Target: 4 to 5 weeks.

Tasks:

- Implement CLI `flipbook init`, `flipbook index`, `flipbook serve`, and `flipbook replay`.
- Implement repository scanner, parser, symbol extraction, store, embeddings, and hybrid search.
- Implement Mapper retrieval pipeline and structured output.
- Implement rule-based Curator and templated generator.
- Add demo app and seeded QA scenarios.
- Add indexing, retrieval, and Mapper evals.

Gate:

- A real local app can be indexed and navigated.
- Taps resolve to grounded source evidence.
- Mapper evals meet initial precision and validity thresholds.

### Milestone 3: Review-Useful Risk Analysis

Target: 6 to 8 weeks.

Tasks:

- Implement Risk agent with streaming findings and structured output.
- Implement suggested fixes and suggested tests grounded in existing project patterns.
- Implement finding triage in the client.
- Add fixture PRs with seeded issues and eval scoring.
- Add prompt caching and cost/latency reporting.
- Harden degraded states when providers are slow or unavailable.

Gate:

- Users can complete a QA review flow and get grounded findings.
- Seeded high-severity fixture bugs are detected consistently.
- False positives and unsupported claims are visible in eval reports.

### Milestone 4: Trace-Aware QA

Target: 6 to 8 weeks.

Tasks:

- Implement browser runtime, trace ingest, storage, and timeline pages.
- Implement SWC and Babel trace adapters.
- Add Vite and Next.js integration examples.
- Implement Trace agent correlation with Mapper and Risk outputs.
- Add trace replay and throughput benchmarks.

Gate:

- Demo app interactions produce trace pages tied to source evidence.
- Runtime events improve at least one Risk or Mapper eval category.
- Instrumentation overhead stays within benchmark budget.

### Milestone 5: Agent and CI Surface

Target: 4 to 5 weeks.

Tasks:

- Implement MCP server over the shared graph contract.
- Implement REST/SSE parity for non-MCP consumers.
- Implement GitHub Action for PR indexing, drill-in, and comments.
- Add contract tests across client, MCP, REST, and replay.
- Add partner integration docs.

Gate:

- An AI agent can navigate a session through MCP with no browser dependency.
- GitHub Action runs on demo PRs and posts grounded findings.
- REST, MCP, and client sessions are schema-equivalent.

### Milestone 6: Free-Form Visual Pages

Target: open-ended after core review usefulness is proven.

Tasks:

- Implement image generation provider abstraction.
- Implement free-form Curator escalation.
- Implement skeleton-first generated-page UX.
- Implement hotspot extraction and validation.
- Add provider redaction and audit trail.
- Run A/B evals against templated pages.

Gate:

- Free-form pages improve comprehension ratings for selected page classes.
- Generated pages preserve agent-readable hotspot contracts.
- Failures gracefully fall back to templates.

### Milestone 7: Alpha Hardening and SaaS Readiness

Target: starts during Milestone 3 and continues.

Tasks:

- Add auth abstraction, local project permissions, and future tenancy seams.
- Add session export/import and support bundles.
- Add setup diagnostics for Node, package manager, provider keys, and repo compatibility.
- Add product analytics for local opt-in and partner alpha.
- Add docs for installation, self-hosting, provider config, security, and troubleshooting.
- Add performance, security, accessibility, and eval release checklist.

Gate:

- Partner alpha teams can install, run, and report sessions without direct engineering support.
- The architecture can be hosted later without rewriting graph, agent, or client contracts.

## Latency Budgets

Budgets are product requirements, not assumptions:

- First visible response after tap: p95 under 800ms on supported local hardware and configured hosted providers.
- Templated page ready: p95 under 1500ms for fixture and partner baseline scenarios.
- Free-form page ready: p95 under 5000ms when image providers are healthy.
- Client page patch render: p95 under 50ms for normal page sizes.
- Vector lookup: p95 under 25ms after warm index.
- Trace ingest: no visible app interaction regression in demo app benchmarks.

Implementation requirements:

- Emit `page.skeleton` before slow provider work.
- Run retrieval, trace lookup, Curator, and Risk work concurrently where dependencies allow.
- Stream findings and patches as soon as they validate.
- Persist replay logs for every benchmark failure.
- Track latency by stream event, provider call, retrieval query, render pass, and trace ingest batch.

## Testing Strategy

- Unit tests for schemas, ID helpers, graph validation, retrieval ranking, prompt assembly, and transforms.
- Integration tests for coordinator streams, replay, indexer store, Mapper retrieval, Risk output validation, and trace ingest.
- Playwright tests for navigation, hotspots, source drawer, streaming patches, and failure states.
- Visual regression tests for each template on desktop and mobile.
- Contract tests for client, REST, MCP, and replay using the same fixtures.
- Eval tests for agent quality with fixture repos and seeded bugs.
- Bench tests for latency, index time, vector lookup, trace throughput, and client render time.

## Alpha Program

Run alpha in stages:

1. Internal demo app with seeded bugs and known expected drill paths.
2. One real internal repository with local-only mode.
3. Three partner repositories with explicit data-handling approval.
4. One CI integration where findings are compared against normal review comments.
5. One agent integration through MCP.

Track:

- Drill depth per session.
- Time to first useful finding.
- Finding usefulness and false-positive rate.
- Pages revisited, abandoned, and exported.
- User orientation failures.
- Provider latency and cost.
- Hotspot click-through rate.
- Agent navigation success rate through MCP.

## Definition of Done For A Release Candidate

- Clean install works from a fresh checkout.
- Local self-hosted flow works against the demo app.
- Client, MCP, REST, and replay all use the same graph schemas.
- Benchmarks and evals produce a release report.
- Security and provider-audit checks pass.
- Accessibility smoke tests pass.
- Docs cover installation, provider setup, local-only mode, troubleshooting, and known limits.
- At least one full QA session can be exported and replayed deterministically.

## Immediate Next Tasks

1. Create the monorepo scaffold and CI foundation.
2. Implement `packages/shared/schemas` and graph validation before any UI or agent code.
3. Build static fixtures that represent the target experience.
4. Build the client shell and template renderer against those fixtures.
5. Add the minimal coordinator that streams fixture pages.
6. Add benchmark and replay harnesses immediately, even if early thresholds are permissive.
7. Start the demo app with seeded QA scenarios so every later agent has a stable target.

