---
title: Claude Certified Architect — Foundations · Docs Reading Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
status: in-progress
created: 2026-05-08
exam-domains: 5
estimated-hours: 10-14
---

# Claude Certified Architect — Foundations · Docs Roadmap

A focused `docs.claude.com` roadmap organized by exam domain, weighted toward weak spots from practice round 1 (Tasks 1.4, 2.4, 3.2, 3.4, 5.1, 5.6).

**Total time:** ~10–14 hours of reading plus hands-on.
**Note:** Some URLs may resolve to slightly different paths; in-page navigation will get you there.

---

## Priority 1 — Claude Agent SDK · Domain 1 (27%)

### Read in order

- [x] **Agent SDK Overview** — `docs.claude.com/en/docs/agent-sdk/overview`
  Conceptual map of the whole SDK. Establishes the vocabulary used across the rest of the docs.
- [x] **How the agent loop works** — `docs.claude.com/en/docs/agent-sdk/agent-loop`
  Most important page for Domain 1. **Read twice.** You should be able to recite: `stop_reason: "tool_use"` means execute the tool and feed back; `stop_reason: "end_turn"` means done.
- [ ] **Subagents in the SDK**
  Covers the `Task` tool, `AgentDefinition`, `allowedTools`, and explicit context passing. Where Q34 (subagents not inheriting parent context) lives.
- [ ] **Control execution with hooks**
  Covers `PreToolUse` and `PostToolUse`. The Python SDK reference page has the best hook code example.
  **Direct hit on weak spots Task 1.4 (prerequisite enforcement) and Task 1.5.**
- [ ] **Work with sessions**
  `--resume`, `fork_session`, named sessions. Touches Task 1.7.

### Do · ~3 hours

- [ ] Install the Python SDK: `pip install claude-agent-sdk`. Run the bug-fix quickstart from the docs end-to-end.
- [ ] Build the multi-tool agent from **Exercise 1** of the exam guide. Wire in a `PreToolUse` hook that blocks any `process_refund` call with an amount exceeding $500. Verify it actually blocks. ^task-1-4-reinforcement
- [ ] Build the multi-agent coordinator from **Exercise 4**. Spawn two subagents in **parallel** by emitting both `Task` calls in a single coordinator response. Watch the latency drop versus serial.
- [ ] Run a session, fork it with `fork_session`, and explore two divergent paths from a shared baseline.

---

## Priority 2 — Tool Use & MCP · Domain 2 (18%)

### Read

- [ ] **Tool use overview** — `docs.claude.com/en/docs/agents-and-tools/tool-use/overview`
  Covers `tool_choice` (auto/any/forced), tool definitions, and the request-response shape.
- [ ] **Connect MCP servers** (under Agent SDK)
  Covers `.mcp.json` (project) versus `~/.claude.json` (user). **Weak spot Task 2.4.**
- [ ] **MCP overview** — `docs.claude.com/en/docs/agents-and-tools/mcp`
  Covers MCP servers, tools, and resources. The "MCP resources" subsection is what Q42 tests.
- [ ] **Define custom tools** (under Agent SDK)
  Tool description quality and the `isError` flag.
- [ ] **modelcontextprotocol.io/docs**
  Outside `docs.claude.com` but worth 30 minutes. Read the **Tools** and **Resources** spec pages directly. Source-of-truth terminology.

### Do · ~2 hours

- [ ] Configure a project-level `.mcp.json` with environment-variable expansion (`${SOME_TOKEN}`). Verify it loads without committing the token.
- [ ] Configure a personal MCP server in `~/.claude.json`. Confirm both load simultaneously when you start Claude Code.
- [ ] Write two MCP tools with deliberately overlapping descriptions. Run a session and watch Claude pick the wrong one. Then rewrite the descriptions with clear preconditions and discriminating criteria. Watch selection improve. ^task-2-1-reinforcement

---

## Priority 3 — Claude Code Configuration · Domain 3 (20%)

### Read

- [ ] **How Claude remembers your project** — `docs.claude.com/en/docs/claude-code/memory`
  **Critical.** Covers the four-tier memory hierarchy (enterprise/project/user/local), `@import` syntax, what survives `/compact`. Q5, Q7, and Q25 live here.
- [ ] **Settings** — `docs.claude.com/en/docs/claude-code/settings`
  Covers `.claude/settings.json`, `claudeMdExcludes`, and how settings layer.
- [ ] **Agent Skills overview** — `docs.claude.com/en/docs/agents-and-tools/agent-skills/overview`
  `SKILL.md`, frontmatter fields. **Critical for weak spot Task 3.2.**
- [ ] **Subagents (Claude Code)** — `docs.claude.com/en/docs/claude-code/subagents`
  Project versus personal subagents. Different from the Agent SDK Subagents page.
- [ ] **Slash commands** — `docs.claude.com/en/docs/claude-code/slash-commands`
  Project versus user-scoped, argument hints.
- [ ] **Plan mode / Explore**
  Covered across the Claude Code overview pages and the auto-memory page. **Weak spot Task 3.4.**
- [ ] **Headless mode / CLI flags** — `docs.claude.com/en/docs/claude-code/cli-reference`
  The `-p` / `--print`, `--output-format json`, `--json-schema` flags. Q17 tests this verbatim.

### Do · ~3 hours

- [ ] In a real repo, set up the full hierarchy: project `.claude/CLAUDE.md`, user `~/.claude/CLAUDE.md`. Use `@import` to pull in three different topic files. Verify load order with `/memory`.
- [ ] Create three `.claude/rules/` files with `paths` glob frontmatter — one for `**/*.test.*`, one for `terraform/**/*.tf`, one for `src/api/**/*`. Edit a matching file in each scope and verify the right rule loads.
- [ ] Create a skill in `.claude/skills/` with **all three** frontmatter fields: `context: fork`, `allowed-tools: [Read, Grep]`, `argument-hint: <package-name>`. Invoke it. Confirm verbose output stays out of your main session. ^task-3-2-reinforcement (Q8 behavior)
- [ ] Run a Claude Code task from a CI-style script: `claude -p --output-format json --json-schema schema.json "review this diff"`. Confirm parseable output.
- [ ] Try plan mode on a task you'd normally use direct execution for, and direct execution on a task that needs plan mode. Feel the friction in both directions — calibrates Task 3.4 intuition. ^task-3-4-reinforcement

---

## Priority 4 — API & Structured Output · Domain 4 (20%)

### Read

- [ ] **Tool use** — `docs.claude.com/en/api/tool-use`
  Full tool use guide on the API side. JSON schemas, `tool_choice`, parallel tool use, strict tool use.
- [ ] **Structured outputs** — `docs.claude.com/en/api/structured-outputs`
  Best practices for nullable fields, enum patterns, validation.
- [ ] **Message Batches API** — `docs.claude.com/en/api/messages-batches`
  Read the whole page. **Memorize:** 50% cost savings, 24-hour processing window, no SLA, `custom_id` for correlation, no multi-turn tool calling. Q18 and Q46 test this.
- [ ] **Prompt engineering — Be clear and direct** — `docs.claude.com/en/docs/build-with-claude/prompt-engineering`
  Especially the few-shot prompting and chain-of-thought pages.

### Do · ~2 hours

- [ ] Build the structured extraction pipeline from **Exercise 3** of the exam guide. Use `tool_use` with a JSON schema that has required fields, nullable fields, and an enum with `"other"` plus a detail string.
- [ ] Force a validation failure (line items don't sum to total). Implement the retry-with-error-feedback loop. Watch it succeed.
- [ ] Then try retrying when the field is genuinely absent from the source — watch it fail forever. ^task-4-4-limit
- [ ] Submit a batch of 20 documents to the Batches API. Use `custom_id` to correlate. Re-submit just the failed ones with chunking.

---

## Priority 5 — Context Management & Reliability · Domain 5 (15%)

This domain is cross-cutting — no dedicated docs section. Most lives in patterns inside the Agent SDK and Claude Code docs. **The exam guide itself (pages 21-25) is the cleanest single source.**

### Read

- [ ] **Context windows / context management** — under Tool infrastructure in the API docs
  Compaction, context editing, prompt caching, token counting.
- [ ] **Compaction**
  Auto-memory page covers `/compact` behavior; SDK docs cover automatic compaction.
- [ ] **Exam guide, pages 21–25**
  Re-read your uploaded copy. Domain 5 is more pattern than feature, and the guide states the patterns more directly than the docs do.

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
