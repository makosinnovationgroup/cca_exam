---
title: Claude Certified Architect — Foundations · Docs Reading Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
status: in-progress
created: 2026-05-08
last-updated: 2026-05-08
exam-domains: 5
estimated-hours: 10-14
---

# Claude Certified Architect — Foundations · Docs Roadmap

A focused `docs.claude.com` roadmap organized by exam domain, weighted toward weak spots from practice round 1 (Tasks 1.4, 2.4, 3.2, 3.4, 5.1, 5.6).

**Total time:** ~10–14 hours of reading plus hands-on.
**Note:** Some URLs may resolve to slightly different paths; in-page navigation will get you there.

> [!tip] How to read this roadmap
> Each priority section now uses a three-bucket lens: **Read in full** (exam-relevant — work through these), **Skim** (useful context, don't dwell), **Skip** (out of scope per the official exam guide — don't get pulled in if inline links lead here). Don't try to read every page in a docs section's left nav. Be selective.

---

## Priority 1 — Claude Agent SDK · Domain 1 (27%)

### Read in full

- [ ] **Agent SDK Overview** — `docs.claude.com/en/docs/agent-sdk/overview`
  Conceptual map of the whole SDK. Establishes the vocabulary used across the rest of the docs.
- [ ] **How the agent loop works** — `docs.claude.com/en/docs/agent-sdk/agent-loop`
  Most important page for Domain 1. **Read twice.** Recite from memory: `stop_reason: "tool_use"` means execute the tool and feed back; `stop_reason: "end_turn"` means done.
- [ ] **Subagents in the SDK**
  Covers the `Task` tool, `AgentDefinition`, `allowedTools`, and explicit context passing. Where Q34 (subagents not inheriting parent context) lives.
- [ ] **Control execution with hooks**
  `PreToolUse` and `PostToolUse`. The Python SDK reference page has the best hook code example. **Direct hit on weak spots Task 1.4 and Task 1.5.**
- [ ] **Connect MCP servers**
  `.mcp.json` (project) versus `~/.claude.json` (user). **Weak spot Task 2.4.**
- [ ] **Define custom tools**
  Tool description quality and the `isError` flag.
- [ ] **Work with sessions**
  `--resume`, `fork_session`, named sessions. Touches Task 1.7.
- [ ] **Agent Skills in the SDK**
  Overlaps with Claude Code's Skills — `SKILL.md`, frontmatter fields. **Weak spot Task 3.2.**
- [ ] **Slash Commands in the SDK**
  Project vs user-scoped commands.
- [ ] **Use Claude Code features**
  How the SDK exposes CLAUDE.md, skills, hooks, slash commands, and plugins. Useful synthesis page.
- [ ] **Modifying system prompts**
  Touches prompt engineering and role-shaping.

### Skim

- [ ] **Quickstart**
  Good warm-up but partly redundant if you read Overview + Agent loop carefully. Useful for hands-on copy-paste.
- [ ] **Structured outputs in the SDK**
  The API-side tool use page is more authoritative — but worth a quick read for SDK-specific patterns.
- [ ] **Handling Permissions** + **User approvals and input**
  Touches escalation and human-in-the-loop. Skim for terminology.

### Skip — out of scope

> [!warning] Don't get pulled in
> These pages are out of scope per the exam guide's explicit out-of-scope list. If inline links point here, back out.

- Streaming Input
- Stream responses in real-time
- Hosting the Agent SDK
- Securely deploying AI agents
- File checkpointing
- Track cost and usage
- Todo Lists
- Plugins in the SDK
- Migration Guide (only if migrating from old SDK)
- Tool search
- TypeScript SDK reference / TypeScript V2 (preview) / Python SDK reference (these are **API references, not study material** — use as lookup, not reading)

### Do · ~3 hours

- [ ] Install the Python SDK: `pip install claude-agent-sdk`. Run the bug-fix quickstart from the docs end-to-end.
- [ ] Build the multi-tool agent from **Exercise 1** of the exam guide. Wire in a `PreToolUse` hook that blocks any `process_refund` call with an amount exceeding $500. Verify it actually blocks. ^task-1-4-reinforcement
- [ ] Build the multi-agent coordinator from **Exercise 4**. Spawn two subagents in **parallel** by emitting both `Task` calls in a single coordinator response. Watch the latency drop versus serial.
- [ ] Run a session, fork it with `fork_session`, and explore two divergent paths from a shared baseline.

---

## Priority 2 — Tool Use & MCP · Domain 2 (18%)

### Read in full

- [ ] **Tool use overview** — `docs.claude.com/en/docs/agents-and-tools/tool-use/overview`
  Top of the section. Establishes vocabulary.
- [ ] **How tool use works**
  Request-response shape, the tool-use turn, `stop_reason: "tool_use"`. Covers `tool_choice` (auto/any/forced).
- [ ] **Define tools**
  Tool description quality, JSON schemas, input/output contracts.
- [ ] **Handle tool calls**
  Where the loop control lives on the API side.
- [ ] **Strict tool use**
  Schema validation enforcement — directly tested in Domain 4.
- [ ] **MCP overview** — `docs.claude.com/en/docs/agents-and-tools/mcp`
  MCP servers, tools, **resources** (the catalog pattern from Q42), `isError`.
- [ ] **modelcontextprotocol.io/docs** (external)
  Worth 30 minutes. Read the **Tools** and **Resources** spec pages directly. Source-of-truth terminology that the exam mirrors.

### Skim

- [ ] **Tutorial: Build a tool-using agent**
  Hands-on warm-up. Build it if you've never wired up tool use; skip if you have.
- [ ] **Parallel tool use**
  Relevant for Domain 1.3 (parallel subagents).
- [ ] **Tool use with prompt caching**
  Just know it exists.
- [ ] **Manage tool context** + **Tool combinations**
  Adjacent context for Domain 5 — skim for terminology.

### Skip — out of scope

> [!warning] Don't get pulled in
> Built-in tool reference pages (Web search, Web fetch, Code execution, Memory tool, Bash tool, Computer use, Text editor) are **reference material**, not testable. Computer use is explicitly out of scope per the exam guide.

- Tool Runner (SDK) — implementation detail
- Server tools — peripheral
- Programmatic tool calling
- Fine-grained tool streaming (streaming is out of scope)
- Computer use tool — **explicitly out of scope**
- Web search tool / Web fetch tool / Code execution tool / Memory tool / Bash tool / Text editor tool — **reference, not study material**
- Troubleshooting (use as lookup)

### Do · ~2 hours

- [ ] Configure a project-level `.mcp.json` with environment-variable expansion (`${SOME_TOKEN}`). Verify it loads without committing the token.
- [ ] Configure a personal MCP server in `~/.claude.json`. Confirm both load simultaneously when you start Claude Code.
- [ ] Write two MCP tools with deliberately overlapping descriptions. Run a session and watch Claude pick the wrong one. Then rewrite the descriptions with clear preconditions and discriminating criteria. Watch selection improve. ^task-2-1-reinforcement

---

## Priority 3 — Claude Code Configuration · Domain 3 (20%)

### Read in full

- [ ] **How Claude remembers your project (Memory)** — `docs.claude.com/en/docs/claude-code/memory`
  **Critical.** Four-tier memory hierarchy (enterprise/project/user/local), `@import` syntax, what survives `/compact`. Q5, Q7, Q25 live here.
- [ ] **Settings** — `docs.claude.com/en/docs/claude-code/settings`
  `.claude/settings.json`, `claudeMdExcludes`, settings layering across user/project/local/managed.
- [ ] **Slash commands** — `docs.claude.com/en/docs/claude-code/slash-commands`
  Project vs user-scoped, argument hints. Q4 territory.
- [ ] **Agent Skills (Claude Code page)** — `docs.claude.com/en/docs/agents-and-tools/agent-skills/overview`
  `SKILL.md`, frontmatter fields. **Critical for weak spot Task 3.2.**
- [ ] **Subagents (Claude Code)** — `docs.claude.com/en/docs/claude-code/subagents`
  Project versus personal subagents. Different from the Agent SDK Subagents page.
- [ ] **Hooks (Claude Code)**
  Distinct from SDK hooks page in some cases — covers settings.json hook config.
- [ ] **Plan mode**
  When to use, what it locks, the Explore subagent. **Weak spot Task 3.4.**
- [ ] **Headless / non-interactive mode** + **CLI reference**
  `-p` / `--print`, `--output-format json`, `--json-schema`. Q17 tests this verbatim.

### Skim

- [ ] **Claude Code overview / quickstart**
  Orientation if you haven't used Claude Code recently.
- [ ] **MCP integration in Claude Code**
  Most of this is covered in the SDK MCP page; skim for Claude Code-specific differences.

### Skip — out of scope

> [!warning] Don't get pulled in
> Cloud provider setup (Bedrock, Vertex, Foundry) is **explicitly out of scope** per the exam guide. So is enterprise deployment, telemetry, and IDE integrations.

- VS Code extension docs
- JetBrains plugin docs
- Web / desktop chat features
- Background tasks
- Bedrock setup
- Vertex AI setup
- Azure AI Foundry setup
- Enterprise deployment / managed policy details
- Analytics API
- Telemetry / monitoring pages
- Auto memory implementation details (the page mentions auto memory; the feature itself is operational, not testable)

### Do · ~3 hours

- [ ] In a real repo, set up the full hierarchy: project `.claude/CLAUDE.md`, user `~/.claude/CLAUDE.md`. Use `@import` to pull in three different topic files. Verify load order with `/memory`.
- [ ] Create three `.claude/rules/` files with `paths` glob frontmatter — one for `**/*.test.*`, one for `terraform/**/*.tf`, one for `src/api/**/*`. Edit a matching file in each scope and verify the right rule loads.
- [ ] Create a skill in `.claude/skills/` with **all three** frontmatter fields: `context: fork`, `allowed-tools: [Read, Grep]`, `argument-hint: <package-name>`. Invoke it. Confirm verbose output stays out of your main session. ^task-3-2-reinforcement (Q8 behavior)
- [ ] Run a Claude Code task from a CI-style script: `claude -p --output-format json --json-schema schema.json "review this diff"`. Confirm parseable output.
- [ ] Try plan mode on a task you'd normally use direct execution for, and direct execution on a task that needs plan mode. Feel the friction in both directions — calibrates Task 3.4 intuition. ^task-3-4-reinforcement

---

## Priority 4 — API & Structured Output · Domain 4 (20%)

### Read in full

- [ ] **Tool use (API)** — `docs.claude.com/en/api/tool-use`
  Full tool use guide on the API side. JSON schemas, `tool_choice`, parallel tool use, strict tool use.
- [ ] **Structured outputs** — `docs.claude.com/en/api/structured-outputs`
  Best practices for nullable fields, enum patterns, validation.
- [ ] **Message Batches API** — `docs.claude.com/en/api/messages-batches`
  Read the whole page. **Memorize:** 50% cost savings, 24-hour processing window, no SLA, `custom_id` for correlation, no multi-turn tool calling. Q18 and Q46 test this.
- [ ] **Prompt engineering — Use examples (multishot prompting)**
  The few-shot prompting page. Critical for Domain 4.2.
- [ ] **Prompt engineering — Be clear and direct**
  Foundational page for Domain 4.1.
- [ ] **Prompt engineering — Chain of thought**
  Touches Domain 3.5 (interview pattern).

### Skim

- [ ] **Messages API overview**
  Just need to know the request/response shape.
- [ ] **Models page**
  Context for model selection in scenarios.
- [ ] **Errors**
  Structure of API errors — touches Domain 2.2 patterns.
- [ ] **Prompt caching**
  Per the exam guide: "beyond knowing it exists" is out of scope. Just know it exists.

### Skip — out of scope

> [!warning] Don't get pulled in
> The exam guide explicitly lists token counting, rate limits, vision, embeddings, streaming, and authentication as out of scope.

- Streaming (SSE)
- Vision / image inputs
- Files API
- Token counting algorithms
- Rate limits / quotas
- API pricing calculations
- Admin API
- Authentication / API key management / OAuth / API key rotation
- Embeddings
- Workspaces / data residency / API and data retention
- Bedrock / Vertex / Azure provider-specific API pages

### Do · ~2 hours

- [ ] Build the structured extraction pipeline from **Exercise 3** of the exam guide. Use `tool_use` with a JSON schema that has required fields, nullable fields, and an enum with `"other"` plus a detail string.
- [ ] Force a validation failure (line items don't sum to total). Implement the retry-with-error-feedback loop. Watch it succeed.
- [ ] Then try retrying when the field is genuinely absent from the source — watch it fail forever. ^task-4-4-limit
- [ ] Submit a batch of 20 documents to the Batches API. Use `custom_id` to correlate. Re-submit just the failed ones with chunking.

---

## Priority 5 — Context Management & Reliability · Domain 5 (15%)

This domain is cross-cutting — there's no dedicated docs section. Most patterns live inside the Agent SDK and Claude Code docs. **The exam guide itself (pages 21-25) is the cleanest single source for this domain.**

### Read in full

- [ ] **Context windows** — under "Context management" in the API docs
  Token budgets, the basic rules of how context fills up.
- [ ] **Compaction**
  Auto-compaction behavior, what survives, when to use `/compact`. Touches Domain 5.4.
- [ ] **Exam guide, pages 21–25** of your uploaded copy ([[cca_exam_guide]])
  Re-read. Domain 5 is more pattern than feature, and the guide states the patterns more directly than the docs.

### Skim

- [ ] **Context editing**
  Adjacent context for the case-facts pattern (Task 5.1).
- [ ] **Prompt caching**
  Just know it exists. Per the exam guide, implementation details are out of scope.

### Skip — out of scope

> [!warning] Don't get pulled in
> These are explicitly out of scope per the exam guide.

- Token counting algorithms or tokenization specifics
- Pricing implications of context length
- Detailed compaction internals (the *behavior* is testable; the *algorithm* isn't)

### Do · ~2 hours

- [ ] In Exercise 4, modify your coordinator so each subagent emits findings as structured records: `{claim, evidence, source_url, document_name, publication_date}`. Have the synthesis subagent preserve `source_url` and `document_name` on every claim it includes. ^task-5-6-reinforcement
- [ ] In Exercise 1, run a 12-turn customer support conversation. Implement a "case facts" block (order IDs, amounts, dates) injected into every prompt outside the conversation summary. Compare accuracy with and without. ^task-5-1-reinforcement
- [ ] Force a subagent timeout in Exercise 4 and verify your error propagation surfaces structured context (failure type, attempted query, partial results) rather than swallowing it.

---

## Compressed Path · 6–8 hours total

If time is limited, this hits all six weak task statements with hands-on reinforcement:

- [ ] Agent SDK Overview + Agent Loop + Hooks + Subagents (~2 hours read)
- [ ] Claude Code memory page + Skills overview (~1 hour read)
- [ ] MCP connect + MCP overview (~30 min read)
- [ ] Message Batches API + Tool use API (~30 min read)
- [ ] Build Exercise 1 with the hook (~2 hours)
- [ ] Build Exercise 4 with claim-source mappings (~2 hours)

---

## Bucket Summary by Priority

Quick reference. Stick to the "Read" column unless you have time to spare.

| Priority | Domain | Read | Skim | Skip |
|----------|--------|------|------|------|
| 1 — Agent SDK | 1 (27%) | 11 pages | 4 pages | 12+ pages |
| 2 — Tool Use & MCP | 2 (18%) | 7 pages | 4 pages | 8+ pages |
| 3 — Claude Code | 3 (20%) | 8 pages | 2 pages | 11+ pages |
| 4 — API & Structured Output | 4 (20%) | 6 pages | 4 pages | 12+ pages |
| 5 — Context Management | 5 (15%) | 3 sources | 2 pages | 3 categories |

> Roughly **35 pages to actually read**, **16 pages to skim**, ~46 pages to deliberately skip. Without this lens, the docs sections collectively contain 100+ pages and you'd burn weeks on out-of-scope material.

---

## Weak-Spot Index

Quick lookup for which sections reinforce each weak task statement:

| Task | What it tests | Reinforcement anchors |
|------|---------------|----------------------|
| 1.4 | Multi-step workflows with prerequisite enforcement | [[#^task-1-4-reinforcement]] |
| 2.1 | Tool description differentiation | [[#^task-2-1-reinforcement]] |
| 2.4 | MCP server scoping | Priority 2 — Do items |
| 3.2 | Skills frontmatter (`context: fork`, `allowed-tools`, `argument-hint`) | [[#^task-3-2-reinforcement]] |
| 3.4 | Plan mode vs direct execution | [[#^task-3-4-reinforcement]] |
| 4.4 | Retry limits when info is absent from source | [[#^task-4-4-limit]] |
| 5.1 | Context management — case facts persistence | [[#^task-5-1-reinforcement]] |
| 5.6 | Provenance — claim-source mappings through synthesis | [[#^task-5-6-reinforcement]] |

---

## Progress Log

Use this section to note what you finished, what tripped you up, and what to revisit.

### Session notes

- 

### Open questions for re-reading

- 

### Self-rated confidence (1–5) by domain

- Domain 1 (Agentic Architecture & Orchestration):
- Domain 2 (Tool Design & MCP):
- Domain 3 (Claude Code Config):
- Domain 4 (Prompt Engineering & Structured Output):
- Domain 5 (Context Management & Reliability):
