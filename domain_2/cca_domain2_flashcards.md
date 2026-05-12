---
title: CCA-F Domain 2 — Flashcards
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - flashcards
  - domain-2
domain: 2
domain-weight: 18
total-cards: 43
---

# CCA-F Domain 2 — Flashcards

**Tool Design & MCP Integration · 18% of exam**

---

## Task 2.1 — Tool interface design

> [!question]- What four elements belong in a good tool description?
> 1. **Input formats** the tool handles (data types, examples)
> 2. **Example queries** showing typical use
> 3. **Edge cases** the tool covers (or doesn't)
> 4. **Boundary explanations** — when to use this tool versus similar tools

> [!question]- The exam guide says tool descriptions are "the primary mechanism" for what?
> Tool **selection** by the model. When descriptions are weak or overlapping, the model misroutes calls. Description quality is the highest-leverage fix for tool-selection bugs.

> [!question]- Scenario: agent frequently calls `get_customer` when users ask about orders. Both tools have minimal descriptions ("Retrieves customer information" / "Retrieves order details"). What's the most effective first step?
> **Expand each tool's description** with input formats, example queries, edge cases, and explicit when-to-use boundaries differentiating from similar tools. Description quality is the root cause.

> [!question]- Why are few-shot examples NOT the right first step for ambiguous tool selection?
> They add token overhead without fixing the root cause (inadequate descriptions). Examples are a remediation when descriptions are already good but the model still mis-selects.

> [!question]- Why is a "routing classifier that parses input keywords and pre-selects the tool" wrong for ambiguous tool selection?
> Over-engineered. Bypasses the LLM's natural language understanding. The descriptions ARE the routing mechanism; fix them.

> [!question]- Why is "consolidate into a single lookup_entity tool" sometimes wrong as a first step?
> It's a valid architectural choice but requires more effort than a "first step." When the immediate problem is inadequate descriptions, the cheap fix is to fix descriptions first.

> [!question]- Tools with ambiguous/overlapping descriptions (`analyze_content` vs `analyze_document` with near-identical wording) — what's the fix?
> Rename and re-describe to eliminate overlap. E.g., rename `analyze_content` → `extract_web_results` with a web-specific description. Naming carries selection signal too.

> [!question]- Can system prompt wording override tool descriptions for routing?
> Yes — keyword-sensitive instructions in the system prompt can create unintended tool associations. Review the system prompt for keywords that might bias tool selection away from the documented description.

> [!question]- Splitting tools vs consolidating tools — what's the trade-off?
> **Split** when each variant has a distinct use case and clearer descriptions improve selection. **Consolidate** when callers genuinely need the same operation with different inputs and the surface area can be unified. Split first, consolidate after evidence.

> [!question]- What's a documented anti-pattern in tool description writing?
> Generic descriptions like "Retrieves data" or "Processes input" — they don't differentiate the tool from any other tool. Always describe **what kind of data, what input format, when to use this vs alternatives**.

---

## Task 2.2 — Structured error responses for MCP tools

> [!question]- What fields belong in a structured tool error response?
> `errorCategory`, `isRetryable` (boolean), `description`, plus partial results where applicable.

> [!question]- Three categories of tool errors — what are they and how is each handled?
> 1. **Transient** (network, rate-limit, timeout) — retry with backoff
> 2. **Business** (rule violation, e.g., refund exceeds policy) — escalate to human, do NOT retry
> 3. **Permission** (auth, scope) — fail fast, do NOT retry

> [!question]- What does `isRetryable: false` signal?
> The error is NOT a transient failure — retrying will produce the same result. The agent should escalate or fall back, not loop.

> [!question]- What's the anti-pattern of "silent error suppression"?
> Catching all errors and returning empty results or graceful defaults. Downstream code can't make informed decisions; errors disappear from observability; production incidents are hard to diagnose.

> [!question]- Why include `customer-friendly explanations` in business-rule violation errors?
> So the agent can communicate appropriately to the user. E.g., `"description": "Refund amount exceeds policy limit of $500. A human agent must approve."` lets the agent paraphrase to the user, not say "Error" or "I can't do that."

> [!question]- Should errors include partial results?
> Yes when applicable. If a search tool returned 8 of 10 expected items before failing, include the 8 in the error response so the coordinator can decide whether to proceed with partial data or retry.

> [!question]- Generic error statuses like "search unavailable" — what's wrong with them?
> They hide valuable context from the coordinator. Was it a transient network issue (retry)? An auth issue (escalate)? A rate limit (back off)? The coordinator needs structured detail to choose the right response.

> [!question]- Local recovery before escalation — what does this mean?
> The failing tool first attempts documented recovery paths (retry transient errors, refresh credentials on auth issues) before escalating. Don't escalate every error to humans — that scales badly.

---

## Task 2.3 — Tool distribution across agents

> [!question]- What's the anti-pattern of giving every agent every tool?
> Agents with tools outside their specialization tend to **misuse them**. E.g., a synthesis agent with web search will start running searches mid-synthesis instead of synthesizing, degrading both quality and predictability.

> [!question]- What's the principle of least privilege for agent tools?
> Restrict each subagent's tool set to those relevant to its role. Synthesis agent gets synthesis tools; search agent gets search tools; coordinator gets routing.

> [!question]- Scenario: synthesis agent runs unnecessary web searches mid-synthesis. Fix?
> Remove web search from the synthesis agent's tool set. Route fact-verification needs through the coordinator (which has access to a search subagent).

> [!question]- "Scoped cross-role tools" for high-frequency needs — what does this mean?
> When a subagent legitimately needs a capability outside its primary role at high frequency, give it a **constrained version** of that tool (e.g., `verify_fact` for the synthesis agent, scoped narrower than the general search tool). Complex/rare cases still route through the coordinator.

> [!question]- Why replace `fetch_url` with `load_document` in a synthesis pipeline?
> Constrained alternatives reduce misuse surface. `fetch_url` accepts any URL; `load_document` validates that the URL is in the expected document repository. Misuse becomes impossible by construction.

> [!question]- `tool_choice` options for the Claude API — what are they?
> `"auto"` — model decides (default)
> `"any"` — model must call **some** tool (but model picks which)
> `{"type": "tool", "name": "specific_tool"}` — model must call that specific tool
> `"none"` — model must not call any tool

> [!question]- Extended thinking + `tool_choice` compatibility?
> Extended thinking only works with `tool_choice: "auto"` and `"none"`. `"any"` and `{"type": "tool"}` produce errors because they prefill the assistant message, which conflicts with thinking blocks.

> [!question]- What field disables parallel tool use?
> `disable_parallel_tool_use: true` in the request. Forces the model to call tools one at a time, useful when downstream state depends on order.

---

## Task 2.4 — MCP server integration

> [!question]- Where do MCP servers get configured for Claude Code?
> `.mcp.json` (project root) — committed to the repo, shared with the team
> User-scoped: `~/.claude/.mcp.json` (or via `claude mcp add`) — personal, not shared

> [!question]- Project scope vs user scope for MCP servers — when to use each?
> **Project scope** (`.mcp.json` in repo): shared with everyone on the team; for tools the project needs (DB connectors, internal APIs)
> **User scope**: personal preference (e.g., your private notes server); not committed

> [!question]- Environment variable expansion in `.mcp.json` — what's the syntax?
> `${VAR_NAME}` — Claude Code expands these from your environment at load time. Useful for credentials: `"token": "${GITHUB_TOKEN}"`.

> [!question]- Can multiple MCP servers be active simultaneously?
> Yes. All configured servers (project + user) load together. Their tools and resources are exposed to the agent under their server prefix.

> [!question]- MCP tools vs MCP resources — what's the distinction?
> **Tools** = actions the agent invokes (verbs). Have side effects, return results.
> **Resources** = content the agent reads (nouns). Catalogs, documents, references. Don't have side effects.

> [!question]- When should something be an MCP resource rather than a tool?
> When it's a piece of content the agent might want to read (a doc, a config, a record) rather than an action. Resources let the agent enumerate-then-fetch rather than guessing tool calls.

---

## Task 2.5 — Built-in tools (Read, Write, Edit, Bash, Grep, Glob)

> [!question]- Which built-in tool finds files by **name pattern**?
> **Glob**. Pattern like `**/*.test.tsx` returns all matching file paths.

> [!question]- Which built-in tool finds files by **content** (text search)?
> **Grep**. Searches inside file contents. Use with regex.

> [!question]- When to use Glob vs Grep?
> Glob = filename pattern matching. "Find files named X."
> Grep = content search. "Find files containing string X."

> [!question]- Which built-in tool reads file contents?
> **Read**. Returns text contents for the agent to reason about.

> [!question]- Which built-in tool creates or overwrites a file?
> **Write**. Use with caution — overwrites entirely.

> [!question]- Which built-in tool modifies a file with find-and-replace?
> **Edit**. Surgical changes; preserves rest of file. Preferred over Write when modifying existing files.

> [!question]- Which built-in tool runs shell commands?
> **Bash**. Executes commands in the working directory.

> [!question]- Edit vs Write — which is safer for modifying existing code?
> **Edit**. Write replaces the entire file; one mistake clobbers unrelated code. Edit changes only what you specify.

> [!question]- Tracing a function's usage across wrapper modules — what's the multi-step pattern?
> 1. Find all **exported names** of the function (Grep on `export.*funcName`)
> 2. For each exported name, **search the codebase** for callers (Grep)
> Wrappers re-export under different names; this pattern catches them.

> [!question]- Anti-pattern: using Bash with `find` and `grep` when built-in tools fit. Why is this wrong?
> Built-in tools are optimized for the agent context — they return structured results the agent can navigate. Shelling out to `find`/`grep` produces raw text the agent has to re-parse, and bypasses safety guards.

> [!question]- Anti-pattern: using Read to search for content. What should you use instead?
> **Grep**. Read returns the entire file; Grep returns only matching lines. For "find where X is used," Grep is right.

---

*Domain 2 weight: 18% of exam. Companion roadmap: [[cca_domain2_roadmap]]. Companion practice exam: [[cca_domain2_practice_v2]].*
