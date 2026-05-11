---
title: Claude Certified Architect — Foundations · Domain 1 Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - roadmap
  - domain-1
status: in-progress
domain: 1
domain-name: "Agentic Architecture & Orchestration"
domain-weight: 27
primary-scenarios: ["Customer Support", "Multi-Agent Research", "Developer Productivity"]
estimated-hours: 6
coverage-level: comprehensive
---

# Domain 1 Roadmap — Agentic Architecture & Orchestration

## What this domain tests

How to **architect**, **orchestrate**, and **control** agent systems built on the Claude Agent SDK. Tested in Scenarios 1 (Customer Support), 3 (Multi-Agent Research), and 4 (Developer Productivity).

The exam rewards production judgment over SDK familiarity. Distractors reward over-engineering, anti-patterns reward shortcuts, and correct answers reliably pick the documented pattern even when it's not the obvious "tech-forward" choice.

## Domain weight

**27% of the total exam — the single largest domain.** Pass-rate gains here move the scaled score more than any other domain.

---

## Task statements (7 total)

| Task | What it tests |
|------|---------------|
| 1.1 | Design and implement agentic loops · `stop_reason` flow |
| 1.2 | Coordinator-subagent orchestration · hub-and-spoke |
| 1.3 | Subagent invocation · `Agent` tool · context passing |
| 1.4 | Multi-step workflows with enforcement and handoff |
| 1.5 | Agent SDK hooks · `PreToolUse` / `PostToolUse` |
| 1.6 | Task decomposition · prompt chaining vs dynamic |
| 1.7 | Session state · `--resume` · `fork_session` |

---

## Read in full · ~4.5 hours

> [!note] URL note
> The Claude Agent SDK docs moved from `docs.claude.com/en/docs/agent-sdk/...` to `docs.claude.com/en/api/agent-sdk/...` during the rename from Claude Code SDK → Claude Agent SDK (March 2026). Both URL patterns may work as aliases. The canonical current path is `/en/api/agent-sdk/`.

### Agent SDK Overview · ~20 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/overview`

The conceptual map. Establishes vocabulary used across every other page: agentic loop, agent definition, subagents, hooks, sessions, MCP servers, plugins.

**Memorize cold:**
- The SDK gives the same agentic harness that powers Claude Code, programmable in Python and TypeScript
- The SDK uses file-system-based configuration like Claude Code: `.claude/agents/`, `.claude/skills/`, `.claude/settings.json`, `.claude/commands/`, `CLAUDE.md` or `.claude/CLAUDE.md`
- **To load filesystem settings from a project, you must explicitly set `settingSources: ['project']` (TS) or `setting_sources=["project"]` (Python).** The SDK does NOT load filesystem settings by default — this is a critical gotcha
- The SDK exposes Claude Code's features: subagents, agent skills, hooks, slash commands, plugins, CLAUDE.md memory
- `query()` is the primary entry point; each call starts a fresh session unless `resume` / `continue` is passed
- Claude Opus 4.7 requires Agent SDK v0.2.111 or later

**Before reading:** Can you explain what the Claude Agent SDK is in one sentence?
**After reading you should be able to:** Name the building blocks (loop, agents, tools, hooks, sessions, MCP, skills, slash commands) and explain how `settingSources` controls filesystem-config loading.

### Quickstart · ~20 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/quickstart`

The bug-fix quickstart shows the canonical SDK setup with `permissionMode: "acceptEdits"` and `cwd:` for filesystem operations. Several testable patterns appear here: how to wire permission mode, how `cwd` interacts with file-system tools, and the minimal options shape.

**Memorize cold:**
- `permissionMode` values: `"default"` (asks), `"acceptEdits"` (auto-approves edits), `"bypassPermissions"` (no prompts — caution), `"plan"` (read-only plan mode)
- `cwd` sets the working directory for filesystem operations
- `maxTurns` caps agent iterations (safety stopgap, not control flow)

### How the agent loop works · ~45 min · **Read twice**

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/agent-loop`
- [ ] **Re-read for memorization** (the `stop_reason` values are exam-critical)

The most important page in this domain. Memorize the `stop_reason` values cold.

**Memorize cold (stop_reason values):**
- `"tool_use"` → execute the requested tool, append result to conversation, continue loop
- `"end_turn"` → loop terminates; Claude is done
- `"max_tokens"` → output limit reached (not a normal termination)
- `"stop_sequence"` → encountered a configured stop sequence
- `"pause_turn"` → server-side pause for long-running tool

**Anti-patterns called out on this page:**
- Using iteration caps as the **primary** stopping mechanism (caps are safety stopgaps)
- Parsing text content for keywords like "complete" or "done"
- Checking for empty text content as a termination signal
- Pre-configuring decision trees in code instead of providing tools and letting the model decide

**Memorize the protocol round-trip:**
1. Claude returns a message with `stop_reason: "tool_use"` and one or more `tool_use` blocks
2. Your code executes each tool, gathers results
3. You send a new message with `role: "user"` and `tool_result` blocks (matched by `tool_use_id`)
4. Claude continues, eventually returning `stop_reason: "end_turn"` with text

### Subagents in the SDK · ~60 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/subagents`

Covers `AgentDefinition`, `allowedTools`, the **`Agent` tool** for spawning, and the critical "no automatic context inheritance" rule.

> [!warning] Tool name correction
> The tool for spawning subagents is **`Agent`**, not `Task`. Pre-2026 docs and some third-party blogs still say `Task` (the legacy name). Current Anthropic docs and current SDK both use **`Agent`**. The Agent SDK documentation page explicitly states: "The Agent tool must be included in allowedTools since Claude invokes subagents through the Agent tool." Memorize **`Agent`** for the exam.

**Memorize cold:**
- The coordinator's `allowedTools` must include `"Agent"` or no subagents will spawn
- Subagents do **not** inherit conversation history automatically — context must be explicitly passed in the prompt the coordinator includes when invoking
- To spawn subagents in **parallel**, the coordinator emits **multiple `Agent` tool calls in a single response** (a single assistant message with multiple `tool_use` blocks)
- `AgentDefinition` includes: `description` (drives auto-routing), `prompt` (system prompt for subagent), `tools` (scoped subset), and optional `model` override
- Built-in general-purpose subagent is available when `Agent` is in `allowedTools` without defining custom agents
- Subagents cannot spawn further subagents (no infinite nesting). Use forking instead for related-but-isolated work
- Each subagent runs in its own fresh conversation; intermediate tool calls and results stay inside the subagent. **Only the final message returns to the parent.**

**Before reading:** What does it mean for a subagent to "have a separate context"?
**After reading you should be able to:** Explain why a synthesis subagent that "doesn't see prior findings" usually means the coordinator didn't pass them in the spawning prompt.

### Control execution with hooks · ~60 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/hooks`
- [ ] **Re-read for memorization** (this page covers two task statements)

**Memorize cold (Python SDK hook events):**
- `PreToolUse` — before a tool call executes; can `permissionDecision: "deny"`/`"allow"`/`"ask"`, return `updatedInput` to modify input, return `permissionDecisionReason` (string)
- `PostToolUse` — after a tool call; can transform result (`updatedMCPToolOutput`) before the model sees it, can add `additionalContext`
- `PostToolUseFailure` — on tool execution failure
- `UserPromptSubmit` — when user submits prompt; can inject `additionalContext`
- `Stop` — when stopping execution
- `SubagentStop` / `SubagentStart` — subagent lifecycle
- `PreCompact` — before message compaction
- `Notification` — notification events
- `PermissionRequest` — permission decision needed

**TypeScript SDK additionally has:** `SessionStart`, `SessionEnd`, `Setup`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`, `WorktreeCreate`, `WorktreeRemove`

**Hook callback signature (Python):** `HookCallback = Callable[[HookInput, str | None, HookContext], Awaitable[HookJSONOutput]]`

**Memorize the deny pattern:**
```python
return {
    "hookSpecificOutput": {
        "hookEventName": "PreToolUse",
        "permissionDecision": "deny",
        "permissionDecisionReason": "Refund exceeds $500 policy limit. Use escalate_to_human."
    }
}
```

**When is a hook required vs when does a prompt suffice:**
- **Hook required:** deterministic compliance needed; consequences of failure are material (financial, regulatory, security, legal)
- **Prompt sufficient:** advisory behavior; occasional drift is acceptable

### Connect MCP servers · ~30 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/mcp`

Crosses Domains 1 and 2. Covers `.mcp.json` (project, shared) vs `~/.claude.json` (user, personal), env var expansion for credentials, transports.

**Memorize cold:**
- `.mcp.json` at the project root → version-controlled, shared with teammates
- `~/.claude.json` in user home → personal, not shared
- Use `${TOKEN_NAME}` in `.mcp.json` for env-var-sourced credentials (never commit tokens)
- Both can be active simultaneously — tools from all servers are available
- SDK MCP tools are namespaced as `mcp__<server-name>__<tool-name>` in `allowedTools` and `matcher` patterns

### Define custom tools · ~30 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/custom-tools`

Tool description quality (Task 2.1), structured error returns (Task 2.2), input schemas.

**Memorize cold:**
- Tool descriptions are the **primary** mechanism for tool selection — minimal descriptions cause misrouting
- Differentiate similar tools with explicit preconditions, effects, and "use this when..." criteria
- Return structured error metadata: `errorCategory` (transient/validation/permission/business), `isRetryable` (boolean), `description` (human-readable)
- The `is_error: true` flag at the MCP protocol level signals failure (separate from raising a Python exception, which crashes the loop)

### Work with sessions · ~30 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/sessions`

Task 1.7. Covers `--resume`, `continue`, `fork_session`, named sessions, the resume-vs-fresh decision.

**Memorize cold (the three options):**
- `continue: true` — resume the **most recent** session on disk automatically
- `resume: sessionId` — resume a **specific** session by ID (captured from a prior `system` message with `subtype: "init"`)
- `forkSession: true` (TS) / `fork_session=True` (Python) — combined with `resume`, creates a new session ID that **branches** from the resumed state. The original session is unchanged.

**Memorize the decision criteria:**
- Same line of work, continuing, no major file changes → `resume`
- Same starting point, want to explore two divergent paths → `forkSession: true`
- Files have changed substantially since last session → **fresh start with injected summary**, not resume
- A fork cannot spawn further forks (one level deep only)

**Forking is a session-history branch, not a filesystem branch.** If a forked agent edits files, those edits are visible to any session in the same directory. Use **file checkpointing** for filesystem branching.

### Agent Skills in the SDK · ~45 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/skills`

The SDK side of skills — overlaps significantly with the Claude Code `code.claude.com/docs/en/skills` page. The Claude Code page is the **authoritative frontmatter reference** (covered in the Domain 3 roadmap).

**Memorize cold (skill frontmatter fields):**

| Field | Use |
|-------|-----|
| `name` | Display name |
| `description` | What the skill does (drives auto-invocation) |
| `when_to_use` | Trigger phrases |
| `argument-hint` | Prompt text for argument autocomplete |
| `arguments` | Named positional arguments |
| `disable-model-invocation` | If `true`, skill can only be invoked manually |
| `user-invocable` | If `false`, hidden from `/` menu |
| `allowed-tools` | Pre-approved tools the skill can use |
| `model` | Override model for skill execution |
| `effort` | Effort level override |
| **`context`** | **Set to `fork` to run in a forked subagent context** |
| **`agent`** | **Which subagent type to use when `context: fork` is set** |
| `hooks` | Skill-scoped hooks |
| `paths` | Glob patterns to limit activation by file path |
| `shell` | `bash` or `powershell` |

**The `context: fork` pattern (heavily tested):**
- Skill runs in isolated subagent context — verbose output stays out of main conversation
- Skill content becomes the prompt that drives the subagent
- Only a summary returns to the main session
- Pair with `agent: Explore` (or `Plan`, or `general-purpose`, or any custom subagent name) to specify which subagent type
- Use for analysis skills that produce pages of output

### Slash Commands in the SDK · ~20 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/slash-commands`

Project vs user-scoped commands. Slash commands and skills are merged — a file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way.

**Memorize cold:**
- `.claude/commands/foo.md` → project-scoped (Git-tracked)
- `~/.claude/commands/foo.md` → user-scoped (personal)
- Argument hints prompt user for input
- Slash commands and skills consolidated — both surfaces work

### Use Claude Code features · ~30 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/claude-code-features`

Synthesis page — explains how the SDK exposes CLAUDE.md, skills, hooks, slash commands, and plugins programmatically. Helpful for seeing how the SDK and Claude Code share infrastructure.

### Modifying system prompts · ~20 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/system-prompts`

How to set or modify the system prompt programmatically. Append-mode vs replace-mode.

**Memorize cold:**
- `systemPrompt: "..."` → fully replace (minimal default, no Claude Code defaults loaded)
- `systemPrompt: { type: "preset", preset: "claude_code" }` → use Claude Code's default system prompt
- `systemPrompt: { type: "preset", preset: "claude_code", append: "..." }` → use default and append
- Output styles (Claude Code only) — replace the software-engineering portion while preserving infrastructure

### Handling Permissions · ~15 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/permissions`

Touches Domain 1.5 (hooks vs permissions) and Domain 5.2 (escalation).

**Memorize cold:**
- `permissionMode`: `"default"` (asks each time), `"acceptEdits"` (auto-approves Edit/Write), `"bypassPermissions"` (no prompts — production caution), `"plan"` (read-only plan mode)
- `canUseTool` callback gives fine-grained control over each tool call (more flexible than `permissionMode`)
- Permission prompts handled in-loop — they don't end the `query()` call

### User approvals and input · ~15 min

- [ ] **Read:** `docs.claude.com/en/api/agent-sdk/user-approvals`

**Memorize cold:**
- `AskUserQuestion` tool lets the agent ask the user multiple-choice questions during execution
- Approval flows resume the `query()` call after the user responds
- Useful for escalation patterns when sentiment alone isn't a reliable signal (Domain 5.2)

---

## Skim · ~45 min

### Structured outputs in the SDK

- [ ] **Skim:** `docs.claude.com/en/api/agent-sdk/structured-outputs`

The API-side `structured-outputs` page is more authoritative — but skim for SDK-specific patterns (`output_config.format` integration). Detailed coverage lives in the Domain 4 roadmap.

### File checkpointing

- [ ] **Skim:** `docs.claude.com/en/api/agent-sdk/file-checkpointing`

The filesystem analog to session forking. Sessions persist conversation; checkpointing persists filesystem state. Not heavily tested but worth knowing exists.

### Track cost and usage

- [ ] **Skim:** `docs.claude.com/en/api/agent-sdk/cost-tracking`

Token-level cost tracking. Light familiarity helps when reading scenarios about cost-optimization.

### Tool search

- [ ] **Skim:** `docs.claude.com/en/api/agent-sdk/tool-search`

For agents with hundreds or thousands of tools. Tool definitions are deferred via `defer_loading: true` and discovered on-demand by Claude. Ties to Domain 2.3 (tool distribution).

### Plugins in the SDK

- [ ] **Skim:** `docs.claude.com/en/api/agent-sdk/plugins`

Plugin system for distributing skills, agents, commands. Skim for terminology.

---

## Skip — out of scope

> [!warning] Don't get pulled in
> Per the exam guide's explicit out-of-scope list:

- **Streaming Input** — `docs.claude.com/en/api/agent-sdk/streaming-input` — streaming is out of scope
- **Stream responses in real-time** — `docs.claude.com/en/api/agent-sdk/streaming` — streaming is out of scope
- **Hosting the Agent SDK** — `docs.claude.com/en/api/agent-sdk/hosting` — deployment infrastructure
- **Securely deploying AI agents** — `docs.claude.com/en/api/agent-sdk/secure-deployment` — deployment infrastructure
- **Todo Lists** — `docs.claude.com/en/api/agent-sdk/todo-lists` — operational feature
- **Migration Guide** — `docs.claude.com/en/api/agent-sdk/migration` — only relevant if migrating from old SDK
- **TypeScript SDK reference / TypeScript V2 / Python SDK reference** — API references, not study material (use as lookup)
- **Observability with OpenTelemetry** — out of scope

---

## Hands-on drills · ~3 hours

- [ ] **Drill 1: Hello agent** — install SDK, write minimal `query()` call, verify it returns a response with text content. Capture session ID from the first `system` message. (~30 min)

- [ ] **Drill 2: Multi-tool customer support agent with PreToolUse hook** — build an agent with tools `get_customer`, `process_refund`, `escalate_to_human`. Wire in a `PreToolUse` hook that blocks any `process_refund` with amount exceeding $500 (returns `permissionDecision: "deny"` with a reason). Verify the hook actually blocks. Test with refund $200 (allowed), $750 (denied). (~60 min)

- [ ] **Drill 3: Multi-agent research coordinator** — build a coordinator that spawns three subagents in parallel for `[topic, side_effects, alternatives]`. Each subagent returns findings as structured records: `{claim, evidence, source_url, document_name, publication_date}`. The coordinator synthesizes with all `source_url` and `document_name` preserved on every claim. (~90 min)

- [ ] **Drill 4: Session forking** — run a session asking Claude to design a REST API. Capture the session ID. Fork the session with a new prompt redesigning as GraphQL. Verify the original session is unchanged; the fork has its own ID. (~30 min)

### Failure-mode exploration

After completing the drills, deliberately break each one to internalize the failure modes:

- [ ] Remove `"Agent"` from `allowedTools` in Drill 3 → subagents fail to spawn
- [ ] Drop `source_url` requirement from the synthesis prompt in Drill 3 → attribution disappears
- [ ] Skip context passing in Drill 3 → synthesis agent produces empty findings
- [ ] Don't set `settingSources: ['project']` in Drill 3 → CLAUDE.md doesn't load

---

## Common confusions and distractor patterns

### Distractor pattern 1: "More instructions in the prompt will fix this"

Failed compliance scenarios offer "add stronger prompt language" or "add more few-shot examples." Both are probabilistic. The correct answer is a hook for deterministic enforcement.

### Distractor pattern 2: "Just iterate more / run more passes"

Missed findings or incomplete results offer "increase iterations," "run three times and majority-vote," or "increase `max_tokens`." The correct answer is structural: iterative refinement with gap detection (Task 1.2), per-file + cross-file passes (Task 1.6), or independent review instances (Task 4.6).

### Distractor pattern 3: "The subagent will figure it out"

Subagents missing context offer "use a larger model," "use a faster model," or "increase `max_tokens`." Correct: pass context explicitly. Subagents do not inherit history.

### Distractor pattern 4: Anti-patterns dressed as solutions

Parsing text content for completion, hardcoded if/else for tool selection, iteration caps as primary control flow, silent error suppression. Textbook anti-patterns the exam expects you to reject.

### Distractor pattern 5: Task vs Agent (legacy naming trap)

If an answer references `Task` in `allowedTools`, that's a trap — the current SDK uses `Agent`. If the question itself uses `Task` (older materials), the answer logic still works the same way: the spawn-subagent tool must be in `allowedTools`.

### Subtle distinction 1: Prompt chaining vs dynamic decomposition

- **Predictable multi-aspect task** (e.g., "review for security, performance, style") → prompt chaining (fixed sequence)
- **Open-ended exploration** (e.g., "improve test coverage of this legacy codebase") → dynamic decomposition (steps emerge from findings)

If you can enumerate subtasks before starting → chaining. If subtasks depend on what you discover → dynamic.

### Subtle distinction 2: Resume vs fresh start with summary vs fork

- **Same line of work, continuing, no major changes** → `resume` (or `continue: true` for most recent)
- **Same starting point, want to explore two divergent paths** → `forkSession: true` + `resume`
- **Files have changed substantially** → fresh start with injected summary

### Subtle distinction 3: Hub-and-spoke vs direct subagent-to-subagent

All inter-subagent communication routes through the coordinator (hub-and-spoke) — documented pattern.
Direct subagent-to-subagent calls create observability gaps and inconsistent error handling — anti-pattern.

### Subtle distinction 4: `settingSources` gotcha

The SDK does NOT load filesystem settings by default. To use project `CLAUDE.md`, `.claude/agents/`, `.claude/skills/`, etc., you must explicitly set `settingSources: ['project']` (TS) or `setting_sources=["project"]` (Python).

### Subtle distinction 5: Session forking vs file checkpointing

- **Forking** branches conversation history, not the filesystem
- **Checkpointing** snapshots filesystem state for rollback
- A forked agent's file edits are still real and visible to other sessions in the same directory
- For independent filesystem experimentation, combine forking + checkpointing OR use `isolation: "worktree"` on the Agent tool

---

## Self-assessment recall tests

Try to answer each from memory before checking notes. If you can't, re-read the linked task's docs page.

### Task 1.1 · Agentic loops

- [ ] Completed Task 1.1 self-check

1. What `stop_reason` value continues the loop? Terminates?
2. Why is parsing text for "done" a documented anti-pattern?
3. When should iteration caps be used, and when not?
4. After executing a tool call, what must happen to the result before the next iteration?
5. What's `pause_turn` for? `max_tokens`?

> [!success]- Answers
> 1. `"tool_use"` continues; `"end_turn"` terminates
> 2. Text-parsing is brittle, fails on phrasing variations, and bypasses the protocol-level signal
> 3. As **safety stopgaps** to prevent runaway loops, not as the primary stopping mechanism
> 4. Tool result must be appended to conversation history (as `tool_result` block matched by `tool_use_id` to the original `tool_use`) so the model can reason about it
> 5. `pause_turn` — server-side pause for long-running operations. `max_tokens` — output limit reached, not normal termination

### Task 1.2 · Coordinator orchestration

- [ ] Completed Task 1.2 self-check

1. What's hub-and-spoke and what problem does it solve?
2. When should the coordinator invoke all subagents vs select a subset dynamically?
3. What's iterative refinement, and what failure mode does it address?
4. Why is partitioning research scope preferred over running multiple subagents on the same scope?

### Task 1.3 · Subagent invocation

- [ ] Completed Task 1.3 self-check

1. What tool name must be in `allowedTools` for subagents to spawn?
2. Do subagents inherit the coordinator's conversation history? How do they get context?
3. How does parallel spawning work at the protocol level?
4. What does an `AgentDefinition` include?
5. What's the difference between defining subagents via `agents: {}` programmatic config vs filesystem `.claude/agents/<name>.md`?

> [!success]- Answers
> 1. `Agent`. Older materials still say `Task` (legacy name)
> 2. No, they don't inherit. Context must be explicitly passed in the spawning prompt
> 3. The coordinator emits multiple `Agent` tool calls in a single response (multi-block tool_use in one assistant message); they execute concurrently; results return for the coordinator to handle in the next turn
> 4. `description` (drives routing), `prompt` (subagent system prompt), `tools` (scoped subset), optional `model` override
> 5. Both work; filesystem-based requires `settingSources: ['project']` in SDK; programmatic is always available regardless of setting sources

### Task 1.4 · Multi-step workflows

- [ ] Completed Task 1.4 self-check

1. When is programmatic prerequisite enforcement required vs prompt-based ordering?
2. What does a structured handoff to a human agent contain?
3. How should a multi-concern customer message be handled?
4. How would you implement "must call `get_customer` before `process_refund`"?

### Task 1.5 · Hooks

- [ ] Completed Task 1.5 self-check

1. List as many hook event names as you can.
2. What does a hook return to block a tool call?
3. When is `PostToolUse` for data normalization the right pattern?
4. Why are hooks deterministic where prompts aren't?
5. What's the `matcher` field for in a hook config?

> [!success]- Answers
> 1. (Python) `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `SubagentStart`, `PreCompact`, `Notification`, `PermissionRequest`. (TS adds) `SessionStart`, `SessionEnd`, `Setup`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`, `WorktreeCreate`, `WorktreeRemove`
> 2. `hookSpecificOutput` with `permissionDecision: "deny"` and `permissionDecisionReason`
> 3. When tools return inconsistent formats (different date encodings, status codes, etc.) and the agent struggles to reason across them
> 4. Hooks run deterministically in code; prompts are probabilistic instructions to the model
> 5. The `matcher` is a glob pattern on tool name. `mcp__*` matches all MCP tools; `Edit|Write` matches Edit or Write; `mcp__customer-support__*` matches one server's tools

### Task 1.6 · Task decomposition

- [ ] Completed Task 1.6 self-check

1. When is prompt chaining appropriate? Dynamic decomposition?
2. What problem does per-file + cross-file decomposition solve?
3. Why isn't "run three passes and majority-vote" a good large-PR review strategy?

### Task 1.7 · Sessions

- [ ] Completed Task 1.7 self-check

1. What's the difference between `resume`, `continue: true`, and `forkSession: true`?
2. When should you start fresh with an injected summary rather than resume?
3. What does a fork inherit from its baseline?
4. Why can't a fork spawn further forks?
5. Does a fork branch the filesystem too?

> [!success]- Answers
> 1. `resume: sessionId` resumes a specific session; `continue: true` resumes most-recent; `forkSession: true` (with resume) creates an independent branch from the resumed state
> 2. When prior tool results are stale relative to current file state — a fresh session with summary is more reliable than carrying stale context forward
> 3. Conversation history, system prompt, tool definitions, prompt cache
> 4. To prevent infinite nesting / runaway branching
> 5. No — forking branches **conversation history only**, not filesystem. Use file checkpointing or `isolation: "worktree"` for filesystem branching

---

## Common scenario patterns by task

What kinds of scenarios test each task statement on the exam:

**Task 1.1** — A scenario describes an agent that won't stop iterating, or stops too early. Distractor answers reach for iteration caps or text-parsing; the correct answer reaches for `stop_reason` and protocol-level signals.

**Task 1.2** — A scenario describes work that's missing perspectives or has gaps. Distractor answers reach for "run more times" or "use a bigger model"; the correct answer involves iterative refinement or hub-and-spoke partitioning.

**Task 1.3** — A scenario describes a subagent that "doesn't see" context or produces empty results. Distractors reach for bigger context windows; the correct answer is explicit context passing in the prompt.

**Task 1.4** — A scenario describes a compliance failure (refund over cap, skipped verification, missing prerequisite). Distractors reach for stronger prompt language; the correct answer is a `PreToolUse` hook for deterministic enforcement.

**Task 1.5** — A scenario describes inconsistent tool output formats or post-tool data normalization needs. The correct answer pairs a hook event to the failure mode: `PreToolUse` for blocking, `PostToolUse` for transformation, `PostToolUseFailure` for failure handling.

**Task 1.6** — A scenario describes a complex multi-step task. Distractor answers reach for "one big prompt with many instructions"; the correct answer is chaining (when steps are predictable) or dynamic decomposition (when steps emerge from findings).

**Task 1.7** — A scenario describes a long-running task being resumed, or two divergent approaches being explored. Distractors swap `resume` and `fork`; the correct answer uses `forkSession` for branching, `resume` for continuation, fresh-with-summary when state has drifted.

---

## Quick-reference cheatsheet

- **Loop control:** `stop_reason: "tool_use"` continues; `"end_turn"` stops. No text parsing. Iteration caps are stopgaps.
- **Subagent spawning:** `"Agent"` in `allowedTools` (NOT `"Task"`). Pass context in prompt — no auto-inheritance. Multiple `Agent` calls in one response = parallel.
- **Subagents are isolated:** intermediate work stays inside; only final message returns.
- **`AgentDefinition`** fields: `description`, `prompt`, `tools`, optional `model`.
- **Hooks vs prompts:** Hooks for deterministic compliance (financial, security, regulatory). Prompts for advisory behavior.
- **Hook events:** PreToolUse (block/modify before), PostToolUse (transform after), PostToolUseFailure (handle failure). Deny pattern: `permissionDecision: "deny"` + `permissionDecisionReason`.
- **Decomposition:** Predictable steps → chaining. Steps depend on findings → dynamic. Many files with attention dilution → per-file + cross-file pass.
- **Sessions:** `resume` for continuation. `continue: true` for most-recent. `forkSession: true` for divergent exploration. Fresh start with summary when tool results are stale.
- **Forking ≠ filesystem branching.** Use file checkpointing for that.
- **`settingSources: ['project']`** required in SDK to load `CLAUDE.md`, `.claude/agents/`, `.claude/skills/`, etc.
- **`permissionMode`** values: `"default"`, `"acceptEdits"`, `"bypassPermissions"`, `"plan"`.
- **Skills frontmatter:** `context: fork` for isolation; `agent: <type>` for which subagent; `allowed-tools` for tool restriction; `argument-hint` for autocomplete; `disable-model-invocation: true` for manual-only.

---

## Progress tracker

- [ ] All "Read in full" pages complete
- [ ] All "Skim" pages reviewed
- [ ] All hands-on drills complete
- [ ] All failure-mode explorations complete
- [ ] All self-assessment recall tests passed
- [ ] Quick-reference cheatsheet memorized

---

*Domain 1 weight: **27%** of the exam.*
