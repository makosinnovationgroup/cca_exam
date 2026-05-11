---
title: Claude Certified Architect — Foundations · Domain 2 Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - roadmap
  - domain-2
status: in-progress
domain: 2
domain-name: "Tool Design & MCP Integration"
domain-weight: 18
primary-scenarios: ["Customer Support", "Multi-Agent Research", "Developer Productivity"]
estimated-hours: 5
coverage-level: comprehensive
---

# Domain 2 Roadmap — Tool Design & MCP Integration

## What this domain tests

How to **design tool interfaces**, **handle errors**, **distribute tools across agents**, **scope MCP servers**, and choose between **built-in tools** and **custom tools**. Tested in Scenarios 1, 3, and 4.

The exam rewards minimum-effort root-cause fixes: rewriting tool descriptions over adding routing classifiers, splitting generic tools over adding parameters, scoping tool access over telling the agent "don't misuse this."

## Domain weight

**18% of the total exam.** Smaller than Domains 1, 3, and 4 — but every Domain 2 task statement is testable with verbatim-quotable facts (`is_error` flag, env-var expansion syntax, exact tool names, `tool_choice` options). The vocabulary is unusually concrete.

---

## Task statements (5 total)

| Task | What it tests |
|------|---------------|
| 2.1 | Tool interface design · description differentiation |
| 2.2 | Structured error responses · `is_error` flag · `errorCategory` |
| 2.3 | Tool distribution · `tool_choice` · scoped access |
| 2.4 | MCP server scoping · project vs user |
| 2.5 | Built-in tools · Grep / Glob / Read / Write / Edit |

---

## Read in full · ~3.5 hours

### Tool use overview · ~20 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/overview`

Top of the tool-use docs. Establishes vocabulary: tool, tool definition, tool call, tool result, `tool_choice`, `stop_reason`. Read first; everything else builds on this.

**Memorize cold:**
- Two kinds of tools: **client tools** (Anthropic defines schema, your app executes) and **server tools** (Anthropic-hosted, e.g., web_search, code_execution)
- Server-side tools may incur additional usage-based pricing (web search charges per search)
- Tool use enables outsized capability gains on benchmarks (LAB-Bench FigQA, SWE-bench) — even basic tools surpass human expert baselines
- `strict: true` on tool definitions enables schema validation

### How tool use works · ~30 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/how-tool-use-works`

Protocol mechanics. The request-response shape. `tool_choice` options and what each guarantees.

**Memorize cold (`tool_choice` is heavily tested):**
- `tool_choice: "auto"` (default) — model decides whether to call a tool or respond in text
- `tool_choice: "any"` — model **must** call some tool, picks which one
- `tool_choice: "tool"` with `name: "X"` — model **must** call the named tool
- `tool_choice: "none"` — disables tool calling
- `disable_parallel_tool_use: true` — restricts to one tool per turn (combined with `auto`, `any`, or `tool`)

**Memorize the `tool_choice` interaction with extended thinking:**
- With extended thinking: only `tool_choice: "auto"` and `tool_choice: "none"` are supported
- `tool_choice: "any"` and `tool_choice: {"type": "tool"}` produce errors with extended thinking
- Why: `any` and forced tool_choice prefill the assistant message, which conflicts with thinking blocks

### Tutorial: Build a tool-using agent · ~30 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/build-a-tool-using-agent`

Five-ring tutorial from single tool call to production-ready agentic loop. Builds the agentic loop by hand first, then collapses to the Tool Runner SDK. **Critical pattern visibility:** `tool_use` blocks, `tool_result` blocks, `tool_use_id` matching, `stop_reason` checking, `is_error` signaling.

**Memorize cold (the protocol):**
- Each `tool_use` block has an `id`
- Each `tool_result` block matches via `tool_use_id`
- Multiple `tool_result` blocks for parallel calls must go in a **single** user message — not separate user messages (formatting them in separate messages teaches Claude to stop emitting parallel calls)
- `is_error: true` flag in `tool_result` signals failure to Claude (different from raising an exception in your code, which crashes the loop)

### Define tools · ~30 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/implement-tool-use`

Tool definitions, JSON schemas, input/output, the canonical example pattern.

**Memorize cold:**
- **Tool descriptions are the primary mechanism for tool selection.** Minimal descriptions → misrouting.
- Differentiate similar tools with:
  - **Preconditions** (when is this tool valid?)
  - **Effects** (what does it do?)
  - **Discriminating criteria** ("use this when... rather than...")
- Include example queries and edge cases
- Splitting a generic tool into purpose-specific tools (e.g., `analyze_document` → `extract_data_points`, `summarize_content`, `verify_claim_against_source`) is the documented fix for inconsistent outputs

### Handle tool calls (Implement tool use) · ~30 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/implement-tool-use` (Handle tool results section)

Where loop control lives on the API side. How tool results flow back into context. Modifying tool results to add `cache_control`.

**Memorize cold:**
- Default behavior: Claude may use multiple tools per query (parallel tool use)
- To disable: set `disable_parallel_tool_use: true` when `tool_choice` is `auto`, `any`, or `tool`
- You can modify tool results before sending back (add `cache_control`, transform output)
- The tool response method gets the tool result; you modify; you add modified version to messages
- Tool results are formatted as user-role messages with `tool_result` content blocks

### Parallel tool use · ~25 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/parallel-tool-use`

The protocol-level mechanics of parallel tool calls. Directly relevant to Domain 1.3 (parallel subagents use the same mechanism).

**Memorize cold:**
- By default, Claude may use multiple tools in one turn
- To increase parallel tool use likelihood, use stronger system prompts encouraging parallelism
- **Common bug that kills parallel tool use:** formatting tool results in separate user messages instead of one
- Correct: `[{role: "assistant", content: [tool_use_1, tool_use_2]}, {role: "user", content: [tool_result_1, tool_result_2]}]`
- Wrong: `[{role: "assistant", content: [tool_use_1, tool_use_2]}, {role: "user", content: [tool_result_1]}, {role: "user", content: [tool_result_2]}]`

### Strict tool use · ~20 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/strict-tool-use`

Schema validation enforcement at the protocol level. Critical for Domain 4.3 (structured output via tool use).

**Memorize cold:**
- Set `strict: true` on the tool definition
- Strict mode eliminates **syntax errors** in tool input (no malformed JSON, no missing required fields, no type mismatches)
- It does NOT eliminate **semantic errors** (line items not summing to total — that needs validation-retry loops)
- Strict mode is implemented via grammar-constrained decoding — the model literally cannot emit tokens that violate the schema
- 100-300ms overhead first time (grammar compilation); cached for 24 hours after

### Tool reference (Optional properties) · ~15 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/tool-reference`

Optional properties on tool definitions — `defer_loading`, `cache_control`, `allowed_callers`, `strict`. These compose.

**Memorize cold:**
- `defer_loading: true` — tool stripped from system-prompt prefix; loaded only when discovered via tool search. **Preserves prompt cache.**
- `cache_control: {"type": "ephemeral"}` — marks a cache breakpoint
- `allowed_callers: ["direct", "code_execution_20260120"]` — restricts which callers can invoke the tool
- All three properties can be set on the same tool

### MCP overview · ~45 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/mcp`

The MCP integration page on the Anthropic side. Covers MCP servers, tools, **resources**, the `is_error` flag, transports.

**Memorize cold:**
- An **MCP server** exposes tools, resources, and prompts to agents
- An **MCP tool** is an action (called by the agent) — e.g., `process_refund`
- An **MCP resource** is content exposed for read (referenced by the agent) — e.g., a 5,000-document index catalog. Lets agents see what's available without exploratory tool calls
- MCP **prompts** are template prompts the server exposes (less heavily tested)
- The `is_error: true` flag in a tool result tells the agent the call failed
- SDK MCP tools are namespaced as `mcp__<server-name>__<tool-name>`

### MCP server configuration · ~30 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/mcp`
- [ ] **Read:** `code.claude.com/docs/en/mcp` (Claude Code MCP page)

Server scoping — the heaviest-tested fact in this domain.

**Memorize cold:**

| Config location | Scope | Shared via Git? | Use for |
|-----------------|-------|-----------------|---------|
| `.mcp.json` (project root) | Project | **Yes** | Team-wide tools (Jira, Sentry, internal services) |
| `~/.claude.json` (user home) | User | **No** | Personal experimental servers, side projects |

**Credential handling:**
- Never commit tokens to `.mcp.json`
- Use environment variable expansion: `${JIRA_TOKEN}` resolves at load time
- Each developer sets the env var in their shell (from a secrets manager)
- Both `.mcp.json` and `~/.claude.json` can be active simultaneously

**Community vs custom MCP server decision:**
- Existing community server covers 90% of needs → use it, build small custom server only for the team-specific 10%
- Don't fork community servers (creates maintenance burden)
- Don't build from scratch when a maintained version exists

### modelcontextprotocol.io spec · ~30 min

- [ ] **Read:** `modelcontextprotocol.io/docs` (external authoritative source)

The MCP spec itself. Read the **Tools**, **Resources**, and **Prompts** sub-pages. Source-of-truth terminology the exam mirrors verbatim.

**Memorize cold:**
- **Tool** — agent calls it, performs an action, returns a result. Has `inputSchema` and returns `content` array
- **Resource** — agent reads it, exposes content (catalog, doc index, schema). Has `uri` and `mimeType`
- **Prompt** — template prompt the server exposes (lower priority)
- Transports: `stdio` (subprocess), `sse` (server-sent events), `http` (deprecated)

---

## Skim · ~30 min

### Tool use with prompt caching

- [ ] **Skim:** `docs.claude.com/en/agents-and-tools/tool-use/tool-use-with-prompt-caching`

How `cache_control: {"type": "ephemeral"}` placed on the last tool in the tools array caches the entire tools-prefix.

### Manage tool context

- [ ] **Skim:** `docs.claude.com/en/agents-and-tools/tool-use/manage-tool-context`

Strategies for handling tool context as it grows. Adjacent to Domain 5.

### Tool combinations

- [ ] **Skim:** `docs.claude.com/en/agents-and-tools/tool-use/tool-combinations`

How tools compose — e.g., code_execution + custom tools, web_search + memory tool. Lighter touch.

### Server tools

- [ ] **Skim:** `docs.claude.com/en/agents-and-tools/tool-use/server-tools`

Anthropic-hosted tools (Web search, Web fetch, Code execution, Memory tool, Bash, Computer use, Text editor). Know the **concept** (server tools run on Anthropic infrastructure, may incur extra cost) but don't memorize each tool's reference page.

### Troubleshooting

- [ ] **Skim:** `docs.claude.com/en/agents-and-tools/tool-use/troubleshooting`

Use as lookup. Common bugs: malformed JSON, parallel tool use breaking due to message formatting, schema validation errors.

---

## Skip — out of scope

> [!warning] Don't get pulled in
> Per the exam guide's explicit out-of-scope list:

- **Tool Runner (SDK)** — `docs.claude.com/en/agents-and-tools/tool-use/tool-runner` — implementation detail; conceptual understanding from the tutorial is enough
- **Programmatic tool calling** — `docs.claude.com/en/agents-and-tools/tool-use/programmatic-tool-calling` — peripheral
- **Fine-grained tool streaming** — `docs.claude.com/en/agents-and-tools/tool-use/fine-grained-tool-streaming` — streaming is out of scope
- **Computer use tool** — `docs.claude.com/en/agents-and-tools/tool-use/computer-use-tool` — **explicitly out of scope**
- **Web search tool / Web fetch tool / Code execution tool / Memory tool / Bash tool / Text editor tool / Advisor tool** — reference pages, not study material
- **Tool search** — `docs.claude.com/en/agents-and-tools/tool-use/tool-search-tool` — peripheral
- **Server-side tool pricing** — out of scope

---

## Hands-on drills · ~2 hours

- [ ] **Drill 1: MCP project scoping** — set up `.mcp.json` at a project root with env-var expansion (`${SOME_TOKEN}`). Confirm it loads without committing the token. Set the env var in your shell.

- [ ] **Drill 2: Personal MCP server** — add a server to `~/.claude.json`. Confirm both project and user MCP servers load simultaneously when you start Claude Code.

- [ ] **Drill 3: Tool description rewrite** — create two MCP tools with deliberately overlapping descriptions (e.g., `cancel_order` and `refund_order` both described as "handle a customer order"). Run an agent that has to choose between them. Watch it pick wrong. Rewrite descriptions with explicit preconditions, effects, and choice criteria. Watch selection improve.

- [ ] **Drill 4: Built-in tool selection** — for each scenario below, identify the right built-in tool and run it:
  - [ ] Find every `.test.tsx` file in the codebase → **Glob**
  - [ ] Find every file containing the string `"deprecated"` → **Grep**
  - [ ] Change one line in a 5000-line file with non-unique anchor text → **Read + Write** (Edit fallback)
  - [ ] Read the contents of a specific file you know the path to → **Read**

- [ ] **Drill 5: Structured error response** — modify a custom tool to return `{is_error: true}` plus structured metadata (`errorCategory`, `isRetryable`, `description`). Test agent behavior with each `errorCategory` value (transient retries, validation prompts user, permission escalates, business explains policy).

- [ ] **Drill 6: tool_choice exploration** — for the same prompt, run with `auto`, `any`, and forced (`{"type": "tool", "name": "X"}`). Observe behavior difference. Try with `disable_parallel_tool_use: true`.

- [ ] **Drill 7: MCP resource pattern** — instead of exposing 100 search tools for a doc catalog, expose one tool that lists resources + lets the agent read them. Observe the difference in tool selection accuracy.

---

## Common confusions and distractor patterns

### Distractor pattern 1: "Just rename it" vs "Just rewrite the description"

Tool confusion distractors offer:
- "Combine them into one tool with an action parameter" → over-invasive
- "Add a routing classifier" → bypasses model's natural routing
- "Delete one of them" → destructive
- **Correct:** rewrite descriptions with preconditions, effects, and choice criteria

### Distractor pattern 2: tool_choice confusion

- "Need to guarantee `extract_metadata` is called first" → forced (`{"type": "tool", "name": "extract_metadata"}`), not `"any"`
- "Need to guarantee the model calls some tool" → `"any"`, not forced
- "Want the model to use judgment" → `"auto"` (default)

### Distractor pattern 3: MCP scope confusion

- "Team-wide" / "shared with teammates" → `.mcp.json`
- "Personal" / "experimental" / "don't share" → `~/.claude.json`
- Watch for fabricated paths like `.claude/mcp-personal.json` (not real)

### Distractor pattern 4: Custom vs community MCP

When community server exists and covers most needs, distractors propose:
- Build custom for full control → loses maintenance leverage
- Fork community server → maintenance burden
- Use only community despite gaps → leaves real needs unmet
- **Correct:** community for the standard 90%, small custom for the team-specific 10%

### Distractor pattern 5: Generic tools and tool consolidation

When a single tool like `analyze_document` is used for everything:
- "Add a longer description explaining all modes" → doesn't fix ambiguity
- "Add an action parameter that switches behavior" → retains ambiguity
- **Correct:** split into purpose-specific tools with defined input/output contracts

### Distractor pattern 6: Tool overload (too many tools per agent)

When an agent has 18 tools and misroutes:
- Distractor: "Add tool routing keywords to descriptions"
- Distractor: "Add more parameters to disambiguate"
- **Correct:** restrict each agent to 4-5 role-relevant tools; route cross-role needs through a supervisor agent

### Subtle distinction 1: MCP tool vs MCP resource

- **Tool** — agent calls it, performs action, returns result
- **Resource** — agent reads it, exposes content (catalog, doc index)
- 5,000 documents → expose as a resource (catalog), not as 5,000 search tools

### Subtle distinction 2: Edit fallback

- `Edit` works on unique text anchors. If anchor isn't unique → operation fails
- Fallback: extend the anchor first (cheapest); then **Read + Write** (load full file, modify, write back)
- Not Bash + sed. Not delete-and-regenerate.

### Subtle distinction 3: Built-in tool selection

- **Grep** — content search (file contents for string/regex)
- **Glob** — path pattern matching (find files by name/path pattern)
- **Read** — load file contents
- **Write** — create or overwrite a file
- **Edit** — modify a file with unique text matching; fall back to Read+Write
- **Bash** — last resort when above don't fit

"Find every `.test.tsx` file" → file paths → **Glob**.
"Find every file with `TODO`" → file contents → **Grep**.

### Subtle distinction 4: `is_error` vs raising an exception

- `is_error: true` in `tool_result` → tells Claude the tool failed; Claude can retry/explain
- Raising a Python exception → crashes the loop
- Always return `is_error: true` with a useful error message, never raise

### Subtle distinction 5: `disable_parallel_tool_use` interactions

- With `tool_choice: "auto"` + `disable_parallel_tool_use: true` → at most one tool per turn
- With `tool_choice: "any"` + `disable_parallel_tool_use: true` → exactly one tool per turn
- With `tool_choice: {"type": "tool"}` + `disable_parallel_tool_use: true` → exactly one tool (forced)

---

## Self-assessment recall tests

### Task 2.1 · Tool interface design

- [ ] Completed Task 2.1 self-check

1. What's the primary mechanism Claude uses to select between similar tools?
2. What three things should a tool description differentiate from similar tools?
3. When should you split a generic tool into purpose-specific tools?
4. What's the documented number of tools per agent that starts hurting selection accuracy?

> [!success]- Answers
> 1. Tool **descriptions**
> 2. Preconditions, effects, choice criteria ("use this when...")
> 3. When the tool is used for varied purposes with inconsistent output quality
> 4. The exam guide uses 18 vs 4-5 — agents with too many tools (~18) misroute more often than agents with focused tool sets (4-5)

### Task 2.2 · Structured errors

- [ ] Completed Task 2.2 self-check

1. What's the `is_error` flag for?
2. What fields should a structured error metadata object include?
3. Why is silent error suppression an anti-pattern?
4. What's the difference between returning `is_error: true` vs raising an exception?

### Task 2.3 · Tool distribution

- [ ] Completed Task 2.3 self-check

1. What does `tool_choice: "any"` guarantee? What does it not guarantee?
2. How would you force `extract_metadata` to be called first?
3. What does `disable_parallel_tool_use: true` do?
4. What `tool_choice` values are incompatible with extended thinking?

> [!success]- Answers
> 1. `"any"` guarantees some tool is called. It does NOT guarantee which tool.
> 2. `tool_choice: {"type": "tool", "name": "extract_metadata"}` for the first turn
> 3. Restricts to one tool per turn (combined with `auto`, `any`, or `tool`)
> 4. `"any"` and `{"type": "tool"}` — they prefill the assistant message and conflict with thinking blocks. Only `"auto"` and `"none"` work with extended thinking.

### Task 2.4 · MCP scoping

- [ ] Completed Task 2.4 self-check

1. Where does a project-shared MCP server live? A personal one?
2. How do you handle credentials in `.mcp.json` without committing them?
3. Can both `.mcp.json` and `~/.claude.json` be active at the same time?
4. When should you build a custom MCP server vs use a community one?

> [!success]- Answers
> 1. `.mcp.json` at the project root (shared via Git); `~/.claude.json` in user home (personal)
> 2. Environment variable expansion: `${TOKEN_NAME}` references a shell env var. Each developer sets the var locally.
> 3. Yes — tools from both load simultaneously
> 4. Community server for the standard 90%; build custom only for team-specific workflows (the 10% community doesn't cover)

### Task 2.5 · Built-in tools

- [ ] Completed Task 2.5 self-check

1. Which built-in tool finds files by name pattern? By content?
2. What's the documented fallback when `Edit` fails because the anchor isn't unique?
3. When should you use `Bash` over the other built-ins?

> [!success]- Answers
> 1. Glob (name); Grep (content)
> 2. Read + Write (load full file, modify, write back)
> 3. Last resort — when reading, writing, editing, or searching with the named tools doesn't fit

---

## Common scenario patterns by task

**Task 2.1** — A scenario describes tools being confused or producing inconsistent output. Distractors reach for combining tools or adding routing layers; the correct answer is description quality (preconditions, effects, choice criteria) or splitting generic tools.

**Task 2.2** — A scenario describes errors disappearing or workflow continuing after failures. Distractors reach for graceful defaults or silent suppression; the correct answer is structured error returns with `is_error: true` and metadata.

**Task 2.3** — A scenario describes a misrouting agent with too many tools, or needing to force a specific tool. Distractors reach for keyword routing or longer descriptions; the correct answers are tool-set restriction (4-5 per agent) and `tool_choice` for forcing.

**Task 2.4** — A scenario describes a configuration that isn't shared (or vice versa), or a credential leak risk. Distractors invent fake paths or propose committing tokens; the correct answer uses `.mcp.json` for project, `~/.claude.json` for personal, env var expansion for credentials.

**Task 2.5** — A scenario describes the wrong tool being used (Bash + sed when Edit would work; Read when Glob is needed). The correct answer matches built-in tool to the operation type: Grep for content, Glob for paths, Read for known-path file loads, Edit for unique-anchor edits, Read+Write fallback, Bash as last resort.

---

## Quick-reference cheatsheet

- **Tool descriptions** are the primary selection signal. Differentiate with preconditions, effects, choice criteria.
- **Splitting > consolidating** generic tools.
- **Structured errors:** `is_error: true` + `errorCategory` (transient/validation/permission/business) + `isRetryable` (bool) + `description`.
- **`tool_choice`:** `"auto"` default; `"any"` forces some tool; `{"type": "tool", "name": "X"}` forces specific; `"none"` disables.
- **Extended thinking** only works with `tool_choice: "auto"` or `"none"`.
- **`disable_parallel_tool_use: true`** restricts to one tool per turn.
- **MCP scoping:** `.mcp.json` (project, shared, Git-tracked) vs `~/.claude.json` (user, personal).
- **MCP credentials:** env var expansion `${TOKEN}` in `.mcp.json`.
- **Community MCP > custom** when a community server covers the standard cases.
- **MCP namespacing:** `mcp__<server>__<tool>` in `allowedTools` and hook matchers.
- **MCP types:** Tool (action), Resource (content), Prompt (template).
- **Built-in tools:** Grep (content), Glob (paths), Read (load), Write (overwrite), Edit (modify unique text; fall back to Read+Write), Bash (last resort).
- **MCP resources** for content catalogs (lets agents see what's available without exploratory search).
- **Parallel tool results** go in ONE user message, not separate ones (breaks parallel tool use).
- **`is_error: true`** in tool result, NEVER raise exceptions.
- **`strict: true`** eliminates syntax errors in tool input (not semantic errors).

---

## Progress tracker

- [ ] All "Read in full" pages complete
- [ ] All "Skim" pages reviewed
- [ ] All hands-on drills complete
- [ ] All self-assessment recall tests passed
- [ ] Quick-reference cheatsheet memorized

---

*Domain 2 weight: **18%** of the exam. Vocabulary-dense but the testable facts are concrete and quotable.*
