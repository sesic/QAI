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

## Scope Guardrail

This plan does not reduce the original scope. The full destination remains:

- Visual recursive QA workspace.
- Source-grounded PR review.
- Static code intelligence.
- Risk analysis.
- Runtime trace enrichment.
- Human web client.
- MCP and REST agent surfaces.
- CI integration.
- Free-form generated visual pages.
- Self-hosted MVP with SaaS-ready boundaries.

The change is sequencing and definition: prove a narrow, valuable loop first, then layer the full platform on top of the same graph and contracts.

## First Winning Use Case

Start with this wedge:

**AI-assisted PR review for React and Next.js applications, where every finding is visually explorable, source-grounded, and exportable as an agent-readable graph.**

Why this wedge:

- Frontend PRs have visible behavior, component hierarchy, state changes, accessibility risk, data loading risk, and test gaps that fit the visual drill-in paradigm.
- React and Next.js give a large initial market without needing every language or framework on day one.
- Senior engineers can judge whether findings are real, which creates useful alpha feedback and eval labels.
- The same work later generalizes to other frameworks, CI review, MCP agents, runtime traces, and generated visual pages.

Primary first-user persona:

- Senior or staff frontend engineer reviewing risky product PRs.
- They need to understand unfamiliar changes quickly, identify regressions before merge, and explain concerns with evidence.

Secondary personas:

- QA lead investigating regressions.
- Engineering manager reviewing release risk.
- AI coding or review agent that needs structured, navigable QA context.

First core loop:

```text
PR diff -> indexed source evidence -> visual drilldown -> grounded risk -> reviewer action -> eval feedback
```

The product is not successful until this loop works on real PRs.

## Product Positioning

The first product promise:

> Understand and review risky frontend changes faster, with every AI finding tied to source, visual context, and a drill-in path.

Initial adoption path:

1. Local CLI against a repo and branch diff.
2. Web flipbook session for human review.
3. GitHub Action that attaches findings and a replayable session artifact to a PR.
4. MCP surface for AI review agents.
5. Trace-aware and generated-visual enhancements after static review value is proven.

Primary success metrics:

- Reviewer says the session would have changed or improved their review.
- Time to first useful finding.
- Valid citation rate.
- Seeded critical bug recall.
- High-confidence false-positive rate.
- Drill depth before abandonment.
- Reopened or shared sessions.
- Accepted findings per review.

Non-goals for the first public alpha:

- Replacing human review.
- Claiming exhaustive bug detection.
- Supporting every stack equally.
- Generating beautiful images at the expense of trust.

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
- Add a product wedge and buyer path so engineering work maps to a concrete adoption loop.
- Add a claim/provenance model so trust is not left to UI copy.
- Add AI systems operations: context budgets, prompt versions, model fallbacks, cost ceilings, replay, and upgrade process.
- Add prompt-injection and repo-exfiltration defenses for hostile source input.

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
- `Claim`: model-generated statement, claim type, evidence IDs, inference category, validation status, prompt version, model version, confidence, and contradiction links.
- `Finding`: severity, category, confidence, affected evidence, impact, suggested fix, suggested tests, and status.
- `StreamEvent`: `session.started`, `page.skeleton`, `page.patch`, `hotspot.patch`, `finding.patch`, `trace.patch`, `page.ready`, `page.failed`, and `session.done`.

### Quality Requirements

- All IDs are deterministic within a repository snapshot and session.
- Every LLM-produced claim must be represented as a `Claim`, linked to evidence or clearly marked as inference.
- Every page must be serializable without the React client.
- Every visible hotspot must have an equivalent JSON action for agents.
- Every stream event must be replayable from fixtures.
- Every finding must link to one or more claims, and every claim must link to evidence, an inference category, or a validation failure.
- Claims that cite unavailable files, invalid ranges, deleted symbols, or contradicted trace evidence must be rejected or downgraded before display.

### Claim Types

Use explicit claim types so UI, evals, and agent consumers can reason about output quality:

- `observation`: directly supported by source, trace, test, screenshot, or log evidence.
- `inference`: derived from evidence but not directly proven.
- `risk`: possible negative outcome if the change ships.
- `recommendation`: suggested fix, test, instrumentation, or review action.
- `unknown`: an explicit uncertainty, missing context, or unsupported area.

### Provenance Requirements

Each generated claim must include:

- Evidence IDs or a reason evidence is unavailable.
- Source path and range when source is cited.
- Runtime event or trace span ID when runtime behavior is cited.
- Model provider, model name, prompt template version, and graph schema version.
- Validation status: `validated`, `partial`, `inferred`, `contradicted`, or `invalid`.
- User feedback status when marked useful, wrong, duplicate, accepted, or ignored.

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
- Add trust UX for every finding: "why this matters", "show source trail", "open in editor", "what would disprove this", "suggest test", and "mark wrong/useful/duplicate/accepted".
- Add PR-review mode with changed files, changed components, risk summary, review queue, unresolved findings, and exportable session link.
- Add orientation aids for recursive exploration: depth indicator, branch map, current hypothesis, page ancestry, and "return to review queue".

Acceptance:

- A user can start from a root page, drill at least four levels deep, return to the root, and understand where they are.
- Every interactive visual region is accessible by mouse, touch, and keyboard.
- Page rendering has no layout shift when streamed patches arrive.
- A reviewer can move from PR overview to a finding, inspect evidence, open source, mark a verdict, and return to the review queue.
- A user can tell which claims are direct observations, inferences, risks, recommendations, or unknowns.

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
- Implement PR review session mode: base ref, head ref, changed files, changed symbols, diff hunks, review queue, and finding verdicts.
- Implement editor deep links for common local editor URL schemes and GitHub source links for CI sessions.
- Implement session export bundles containing graph JSON, replay stream, provider audit log, and redacted evidence manifest.

Acceptance:

- A tap emits a skeleton frame quickly, then streams patches as work completes.
- Failed agent calls produce degraded pages instead of broken sessions.
- Replay logs can reproduce a session without providers.
- A PR review session can be created from a local diff or GitHub Action payload.

### 5. Templates and Visual Generation

Tasks:

- Build initial templated pages: component map, data flow, request lifecycle, state machine, risk board, trace timeline, dependency graph, source evidence, and test suggestion page.
- Define template input schemas and renderer contracts.
- Implement Curator rules for choosing templates based on tap intent, available evidence, and graph depth.
- Implement templated generator that fills pages from structured agent output.
- Add visual regression tests for all templates.
- Later, add free-form image generation with fallback to template rendering when provider calls fail.
- Add PR-oriented pages: diff impact map, changed component graph, affected route map, risk queue, claim/evidence trail, and test gap page.
- Add deterministic diagram generation from graph data before free-form image generation.

Acceptance:

- At least nine templates render from deterministic fixtures.
- Curator can choose a reasonable template without an LLM for the core MVP.
- Generated pages preserve all hotspot and evidence links.
- PR-oriented templates make the review queue navigable without requiring free-form image generation.

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
- Implement diff intelligence: changed symbol detection, dependency expansion from changed files, ownership hints, test adjacency, route/component impact, and config-change detection.
- Add framework-specific extractors for React components, hooks, Next.js routes, server actions, API routes, middleware, package scripts, and common test files.

Acceptance:

- A representative app can be indexed locally and queried by symbol, path, text, and embedding.
- Incremental changes update the index without a full rebuild.
- Unsupported files are reported clearly and do not break indexing.
- A local branch diff produces a changed-symbol graph and candidate impacted areas.

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

### 12. AI Systems and Model Operations

Tasks:

- Define per-agent context budgets for system prompt, repository summary, diff context, retrieved evidence, prior page state, trace context, and user feedback.
- Implement prompt registry with template versions, eval metadata, owner, changelog, and rollback path.
- Implement model routing by task: small/fast models for classification and extraction, stronger models for risk synthesis, vision-capable models for hotspot extraction.
- Implement provider fallback behavior for latency, rate limits, validation failures, and unavailable models.
- Implement session-level cost ceilings with visible degradation when a ceiling is reached.
- Implement token, latency, cache-hit, and validation telemetry per agent call.
- Implement deterministic replay for model outputs through recorded tool responses.
- Implement asynchronous job queue for slow Risk, Trace, and free-form image tasks while keeping the stream responsive.
- Add model upgrade workflow: run eval suite, compare cost/latency, compare claim validity, inspect diff samples, then promote.
- Add prompt-injection resistant message assembly that separates untrusted source text from system/developer instructions.

Acceptance:

- Every prompt has a version, eval report, and rollback path.
- A session can replay without contacting model providers.
- Provider outage, rate limit, or model failure produces a degraded graph state rather than a broken session.
- Cost per session is measurable and bounded by configuration.

### 13. Security, Privacy, and Local Control

Tasks:

- Add provider configuration for local-only, hosted embeddings, hosted LLM, and hosted image generation.
- Add secret scanning and redaction before provider calls.
- Add path allowlists, ignore patterns, max file size limits, and binary-file handling.
- Add local data retention controls for sessions, traces, screenshots, and embeddings.
- Add audit log for external provider requests.
- Document self-hosted threat model and SaaS-ready tenancy boundaries.
- Treat repository contents, comments, logs, traces, and generated files as hostile input.
- Add prompt-injection fixtures embedded in source comments, markdown, test names, package scripts, and logs.
- Add policy-enforced provider boundaries: never-send paths, max bytes per file, redacted snippets, and external-call manifests.
- Add sandbox rules for any command execution, test execution, package script inspection, or repo-derived tool call.
- Add local-only verification that fails if a configured hosted provider would receive source content.
- Add export redaction and preview before sharing a session bundle.

Acceptance:

- Users can run without sending code to hosted providers when configured.
- Provider requests are inspectable and redact known secret patterns.
- Session exports clearly identify what data they contain.
- Prompt-injection fixtures do not override system instructions, provider policy, or claim validation.
- Local-only mode produces an auditable proof that no source text left the machine.

### 14. Observability, Benchmarks, and Evals

Tasks:

- Add benchmark harnesses for first frame, full page, client render, indexing, vector search, trace throughput, and replay.
- Add LLM eval harness for Mapper, Curator, Risk, Trace, and Hotspot extraction.
- Add golden fixtures from demo app, synthetic repos, and partner-approved anonymized sessions.
- Add dashboards or CLI reports for latency, token cost, cache hit rate, eval score, and finding quality.
- Add CI gates for schema validation, replay determinism, benchmark smoke tests, and eval regression budgets.
- Add product-quality evals for the PR loop: valid citation rate, seeded bug recall, high-confidence false positives, time to first useful finding, and reviewer usefulness.
- Add claim-level evals: evidence validity, unsupported inference rate, contradiction rate, severity calibration, and recommendation executability.
- Add retrieval evals for changed-symbol recall, impacted-area recall, and source-range validity.
- Add UI evals for orientation failures, hotspot misclicks, drill abandonment, and source-trail completion.

Acceptance:

- Release candidates include benchmark and eval reports.
- Latency regressions can be traced to stream, retrieval, provider, render, or trace phases.
- Prompt changes require eval diffs before merge.
- Release candidates meet explicit launch gates or document approved exceptions.

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
- Implement claim objects, evidence trails, validation states, and reviewer verdict capture.
- Implement local PR review mode with changed-symbol graph and risk queue.

Gate:

- Users can complete a QA review flow and get grounded findings.
- Seeded high-severity fixture bugs are detected consistently.
- False positives and unsupported claims are visible in eval reports.
- Valid source citation rate is at least 95% on fixture PRs.
- Seeded critical bug recall is at least 80% on fixture PRs.
- High-confidence false positives are fewer than 2 per reviewed PR in the alpha eval set.
- Median time to first useful finding is under 60 seconds in the local PR review flow.

### Milestone 4: PR Workflow and CI

Target: 4 to 5 weeks.

Tasks:

- Implement GitHub Action for PR indexing, drill-in, grounded comments, and session artifact upload.
- Add GitHub source links, comment anchors, and finding update behavior.
- Add PR summary page, changed component graph, affected route map, unresolved finding queue, and exportable replay link.
- Add CI token auth boundaries and provider policy for CI environments.
- Add contract tests for local PR sessions and GitHub Action sessions.
- Add partner integration docs for adding QAI to a repository.

Gate:

- GitHub Action runs on demo PRs and posts grounded findings.
- Reviewers can move from a PR comment into the matching flipbook page and source trail.
- CI session exports replay locally without provider access.

### Milestone 5: Agent Surface

Target: 4 to 5 weeks.

Tasks:

- Implement MCP server over the shared graph contract.
- Implement REST/SSE parity for non-MCP consumers.
- Add contract tests across client, MCP, REST, and replay.
- Add partner integration docs.
- Add MCP tools for tap, drill-in, search, subscribe, export, mark-finding, and request-evidence.
- Add agent navigation evals that measure whether an external AI agent can reach relevant evidence and findings.
- Add rate limits, auth, and permissions for non-browser consumers.

Gate:

- An AI agent can navigate a session through MCP with no browser dependency.
- REST, MCP, and client sessions are schema-equivalent.
- Agent navigation success rate through MCP is at least 80% on fixture tasks.

### Milestone 6: Trace-Aware QA

Target: 6 to 8 weeks.

Tasks:

- Start with trace inputs that minimize integration friction: Playwright traces, browser console logs, network HAR files, and OpenTelemetry when already present.
- Implement browser runtime, trace ingest, storage, and timeline pages.
- Implement SWC and Babel trace adapters after replay and OpenTelemetry ingestion prove value.
- Add Vite and Next.js integration examples.
- Implement Trace agent correlation with Mapper and Risk outputs.
- Add trace replay and throughput benchmarks.

Gate:

- Demo app interactions produce trace pages tied to source evidence.
- Runtime events improve at least one Risk or Mapper eval category.
- Instrumentation overhead stays within benchmark budget.
- Teams can use trace enrichment without adopting custom instrumentation first.

### Milestone 7: Free-Form Visual Pages

Target: open-ended after core review usefulness is proven.

Tasks:

- Implement image generation provider abstraction.
- Implement free-form Curator escalation.
- Implement skeleton-first generated-page UX.
- Implement hotspot extraction and validation.
- Add provider redaction and audit trail.
- Run A/B evals against templated pages.
- Keep deterministic templates as fallback and as the agent-readable source of truth.
- Use free-form generated pages for selected page classes where they improve comprehension beyond structured diagrams.

Gate:

- Free-form pages improve comprehension ratings for selected page classes.
- Generated pages preserve agent-readable hotspot contracts.
- Failures gracefully fall back to templates.
- Generated visuals do not become the only source of truth for any finding or claim.

### Milestone 8: Alpha Hardening and SaaS Readiness

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

1. Internal demo app with seeded bugs, known expected drill paths, and PR fixtures.
2. One real internal React or Next.js repository with local-only mode.
3. Three partner frontend repositories with explicit data-handling approval.
4. One GitHub Action integration where findings are compared against normal review comments.
5. One agent integration through MCP.
6. One trace-enriched partner session using Playwright, OpenTelemetry, HAR, logs, or custom instrumentation.
7. One free-form visual evaluation on selected page classes after deterministic templates are stable.

Track:

- Drill depth per session.
- Time to first useful finding.
- Finding usefulness and false-positive rate.
- Valid source citation rate.
- Seeded critical bug recall.
- High-confidence false positives per review.
- Unsupported inference rate.
- Reviewer verdict distribution: useful, wrong, duplicate, accepted, ignored.
- Pages revisited, abandoned, and exported.
- User orientation failures.
- Provider latency and cost.
- Hotspot click-through rate.
- Agent navigation success rate through MCP.
- Session replay success rate.
- Local-only mode violations, expected to be zero.
- CI comment acceptance or resolution rate.

Alpha launch gates:

- At least 95% valid source citations on the fixture PR set.
- At least 80% seeded critical bug recall on the fixture PR set.
- Fewer than 2 high-confidence false positives per reviewed PR in the alpha eval set.
- Median time to first useful finding under 60 seconds.
- At least 70% of alpha review sessions rated "would have helped my review".
- Zero known provider-boundary violations in local-only mode.

## Definition of Done For A Release Candidate

- Clean install works from a fresh checkout.
- Local self-hosted flow works against the demo app.
- Client, MCP, REST, and replay all use the same graph schemas.
- Benchmarks and evals produce a release report.
- Security and provider-audit checks pass.
- Accessibility smoke tests pass.
- Docs cover installation, provider setup, local-only mode, troubleshooting, and known limits.
- At least one full QA session can be exported and replayed deterministically.
- PR review flow works from local diff through visual drilldown, grounded risk, reviewer verdict, and replay export.
- GitHub Action flow works on demo PRs when enabled.
- Every displayed finding is backed by claims, evidence, inference labels, and validation status.
- Prompt, model, and retrieval changes include eval diffs.
- Provider cost and latency are visible in release reports.

## Immediate Next Tasks

1. Create the monorepo scaffold and CI foundation.
2. Implement `packages/shared/schemas` and graph validation before any UI or agent code.
3. Define `Claim`, `Evidence`, `Finding`, `Hotspot`, and `StreamEvent` fixtures for the PR-review loop.
4. Build static fixtures that represent the target experience, including seeded React/Next.js PR scenarios.
5. Build the client shell, trust UX, review queue, and template renderer against those fixtures.
6. Add the minimal coordinator that streams fixture pages and replay logs.
7. Add benchmark and replay harnesses immediately, even if early thresholds are permissive.
8. Start the demo app with seeded QA scenarios so every later agent has a stable target.
9. Create the first eval suite before implementing the first LLM-backed agent.
