---
title: CCA-F Domain 1 — Flashcards
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - flashcards
  - domain-1
domain: 1
domain-weight: 27
total-cards: 47
---

# CCA-F Domain 1 — Flashcards

**Agentic Architecture & Orchestration · 27% of exam**

Click any card's chevron to reveal the answer. Cards are organized by task statement.

**How to drill:**
- [ ] **Pass 1:** Cold — read the question, recall the answer, then reveal to check
- [ ] **Pass 2:** Tag missed cards (`#missed-d1`) for re-drill
- [ ] **Pass 3:** Re-drill only `#missed-d1` cards
- [ ] **Pass 4:** Random-order pass to defeat anchor effects

---

## Task 1.1 — Agentic loops

> [!question]- What two `stop_reason` values control the agentic loop, and what does each mean?
> `"tool_use"` → Claude wants to call a tool; continue the loop (execute tool, append result, send request again)
> `"end_turn"` → Claude is done; terminate the loop and return final answer

> [!question]- What's the correct loop termination check?
> Inspect `stop_reason`. Loop while `stop_reason === "tool_use"`. Terminate when `stop_reason === "end_turn"`.

> [!question]- Anti-pattern: parsing natural-language signals to determine loop termination. Why is this wrong?
> Natural language is non-deterministic. The model might say "I'm done" mid-task or fail to say it when finished. `stop_reason` is the protocol-level signal — use it.

> [!question]- Anti-pattern: setting an iteration cap as the primary stopping mechanism. Why is this wrong?
> Iteration caps are a safety backstop, not a termination signal. They mask logic bugs in the loop and produce unpredictable behavior. Use `stop_reason`; use the cap only to prevent runaway costs.

> [!question]- Anti-pattern: checking for assistant text content as a completion indicator. Why is this wrong?
> Claude may emit text alongside `tool_use` (e.g., reasoning). Presence of text doesn't mean completion. `stop_reason` is the signal.

> [!question]- What happens to tool results between iterations of the agentic loop?
> They're appended to the conversation history as a `tool_result` content block, so the model can reason about them on the next iteration.

> [!question]- Model-driven decision-making vs pre-configured decision trees: which does the agentic loop use?
> Model-driven. Claude reasons about which tool to call next based on the conversation context and tool descriptions — not a hard-coded decision tree or fixed sequence.

> [!question]- When should you NOT use an agentic loop?
> When the task has a deterministic sequence with no decision branching (e.g., "always call tool A then tool B then return"). For those, use prompt chaining or explicit orchestration.

---

## Task 1.2 — Multi-agent coordinator-subagent patterns

> [!question]- What is the hub-and-spoke architecture for multi-agent systems?
> A coordinator agent manages ALL inter-subagent communication, error handling, and information routing. Subagents never talk to each other directly — everything flows through the coordinator.

> [!question]- Do subagents inherit the coordinator's conversation history?
> No. Subagents operate with **isolated context** — they get only what the coordinator explicitly passes to them.

> [!question]- What four responsibilities does the coordinator own in a multi-agent system?
> 1. Task decomposition (breaking the query into subagent-sized work)
> 2. Delegation (selecting which subagents to invoke)
> 3. Result aggregation (combining subagent outputs)
> 4. Error handling and information routing across subagents

> [!question]- Risk of overly narrow task decomposition by the coordinator?
> Incomplete coverage. The coordinator may carve the problem into too-specific slices that miss broad-research aspects (e.g., decomposing "impact of AI on creative industries" only into "music" and "film" misses publishing, gaming, advertising).

> [!question]- Should the coordinator always route through every subagent in the pipeline?
> No. Dynamic selection: the coordinator analyzes query requirements and invokes only relevant subagents. Always-route-everywhere wastes tokens and dilutes results.

> [!question]- What's the partitioning principle for assigning work to subagents?
> Minimize duplication. Assign **distinct subtopics or source types** to each subagent (e.g., one for academic sources, one for industry reports, one for news) — not the same scope to multiple subagents.

> [!question]- What is an "iterative refinement loop" in multi-agent orchestration?
> Coordinator evaluates synthesis output for gaps → re-delegates targeted queries to search/analysis subagents → re-invokes synthesis. Continues until coverage is sufficient. Single-pass coordinator pipelines miss gaps.

> [!question]- Why route all subagent communication through the coordinator (vs direct subagent-to-subagent)?
> Observability (you can see all flows), consistent error handling, controlled information flow, and audit trail. Direct subagent communication creates uncontrolled state and untestable behaviors.

---

## Task 1.3 — Subagent invocation, context passing, spawning

> [!question]- What does it mean that subagents have "isolated context"?
> Their context starts empty (or with their system prompt only) every time they're invoked. They don't see the coordinator's history unless the coordinator explicitly passes data.

> [!question]- How do you pass context from coordinator to subagent in the Agent SDK?
> Pass it in the prompt as structured input (often XML-tagged), or via a structured state file the subagent reads (scratchpad pattern). Never assume context inheritance.

> [!question]- Can subagents spawn their own subagents?
> No. Subagents cannot spawn subagents in the SDK. Only the top-level (coordinator) agent can spawn subagents.

> [!question]- What are the built-in subagents available in the Agent SDK?
> `Explore`, `Plan`, and `general-purpose`. Plus any custom subagents you define via `AgentDefinition`.

> [!question]- What fields does `AgentDefinition` accept?
> `description`, `prompt`, `tools`, `disallowedTools`, `mcpServers`, `model`, `maxTurns`, `skills`.

> [!question]- What does `forkSession: true` (TS) / `fork_session=True` (Python) do?
> Combined with `resume: <sessionId>`, creates a new **independent branch** from the resumed session's state. Original session is unchanged; the fork explores divergent paths.

> [!question]- When should you use `forkSession` vs starting a new session?
> Fork when: same starting analysis baseline, want to explore divergent approaches (A/B paths)
> New session when: independent task, no shared context needed

> [!question]- What's the "designing coordinator prompts" pattern for subagent adaptability?
> Specify research **goals and quality criteria** in the coordinator's prompt to the subagent — not step-by-step procedural instructions. Subagents adapt better when given outcomes to achieve, not steps to execute.

---

## Task 1.4 — Multi-step workflows, enforcement, handoff

> [!question]- A scenario: agent skips `get_customer` in 12% of cases and calls `lookup_order` with just a stated name. What's the most reliable fix?
> Add a **programmatic prerequisite** that blocks `lookup_order` (and `process_refund`) until `get_customer` has returned a verified customer ID. Protocol-level enforcement beats prompt-level "please verify first."

> [!question]- Why does "enhance the system prompt to require X" fail as enforcement for critical sequences?
> Prompt-based compliance is **probabilistic**. For workflows where errors have financial/legal consequences, you need deterministic guarantees. Programmatic prerequisites (gating code) provide that.

> [!question]- Why do few-shot examples fail as enforcement for tool sequencing?
> Same reason — probabilistic. Examples shape behavior but don't guarantee it. For "must call X before Y" use protocol enforcement.

> [!question]- Why does a "routing classifier that pre-selects tool subsets" not fix tool-sequencing problems?
> It addresses tool **availability**, not tool **ordering**. The problem is "X must come before Y," which is an ordering constraint, not an availability one.

> [!question]- What goes in a structured handoff protocol for mid-process escalation?
> 1. Customer details (identity, context)
> 2. Root cause analysis (what happened, what was attempted)
> 3. Recommended actions (what the agent thinks should happen next)
> Plus any conversation summary the human needs.

---

## Task 1.5 — Agent SDK hooks

> [!question]- What are the two main hook events used for tool call interception?
> `PreToolUse` — fires before a tool call runs (use for policy enforcement, blocking)
> `PostToolUse` — fires after a tool call returns (use for data normalization, logging)

> [!question]- What's `hookSpecificOutput.permissionDecision` and what values can it take?
> The output a `PreToolUse` hook returns to control whether the tool call proceeds.
> Values: `"allow"`, `"deny"`, `"ask"` (deferral to user)

> [!question]- Scenario: refunds over $500 should require human approval, not be auto-processed. What's the fix?
> A `PreToolUse` hook on `process_refund` that inspects the arguments, and if amount > $500 returns `permissionDecision: "deny"` (or `"ask"`), redirecting to a human-escalation workflow.

> [!question]- Why use hooks vs adding the limit check in the tool itself?
> Hooks are policy/orchestration concerns; tools are capability. Hooks let you change policy without touching tool implementations, and centralize policy across tools. Also: hook denial happens before tool execution → no side effects.

> [!question]- Anti-pattern: blocking by raising an exception inside the tool body. Why is the hook approach preferred?
> Exception inside tool means partial work may already have happened. Hook denial happens **before** the tool runs — clean, observable, reversible.

> [!question]- Can hooks modify tool inputs/outputs?
> Yes — that's the data-normalization use case. `PostToolUse` hooks can transform results before they're appended to the agent's context (e.g., strip sensitive fields, normalize formats).

---

## Task 1.6 — Task decomposition for complex workflows

> [!question]- For open-ended tasks like "add comprehensive tests to a legacy codebase," what's the decomposition pattern?
> 1. **Map structure first** (what exists, what's tested already, what's not)
> 2. **Identify high-impact areas** (critical paths, complex logic, known bug sites)
> 3. **Create prioritized plan** (don't try to do everything; pick high-value chunks)
> Then execute the plan, not the original vague directive.

> [!question]- Prompt chaining vs dynamic decomposition: when to use each?
> **Prompt chaining:** task has a known sequence of focused passes (extract → classify → summarize). Deterministic flow.
> **Dynamic decomposition:** task complexity is unknown upfront and the coordinator must adapt (research, exploration, debugging). Model-driven flow.

> [!question]- What's the cost of overly narrow task decomposition by the coordinator?
> Coverage gaps. Some subtopics fall through the cracks because no subagent was assigned to them.

> [!question]- What's the cost of overly broad task decomposition by the coordinator?
> Subagents lose focus and produce shallow results across too much surface area. Each subagent should have a tight scope.

---

## Task 1.7 — Session state, resumption, forking

> [!question]- What's the difference between `continue: true` and `resume: <sessionId>`?
> `continue: true` — resume the **most recent** session on disk automatically
> `resume: <sessionId>` — resume a **specific** session by ID (you captured the ID earlier)

> [!question]- Where do you find a session's `sessionId`?
> In the initial `system` message of that session — the one with `subtype: "init"`. Capture it as soon as you start a session you might want to resume.

> [!question]- `--resume <session-name>` — what's this CLI variant?
> Named-session resumption. You can label sessions with names and resume by name instead of by raw ID. Same semantics as `resume:`; just a more memorable identifier.

> [!question]- Three options for picking up a prior session — and when to use each:
> 1. **`continue: true`** — most recent session, casual resume
> 2. **`resume: sessionId`** — specific session by ID
> 3. **`forkSession: true` + `resume:`** — branch from a baseline for divergent exploration

> [!question]- A developer worked 4 hours yesterday; today they want to continue, but pushed commits that changed files the agent was working on. Best approach?
> Start a **fresh session** with a summary of yesterday's progress (key decisions, files touched, open questions). Don't resume — stale tool results in context will mislead the agent about current file state.

> [!question]- Why is `forkSession` wrong for "files have changed since last session"?
> Forking carries the same stale state into the fork. Forking is for exploring divergent paths from a still-valid baseline — not for working around staleness.

> [!question]- When resuming a session after code modifications, what should the agent be told?
> Specific file changes for targeted re-analysis. Don't expect the agent to re-explore from scratch; inform it about what's changed since the prior session.

> [!question]- Crash recovery for long-running agent sessions — what's the pattern?
> Subagents persist structured state to a manifest file. On crash, the coordinator reads the manifest and resumes from the last known good state — not from session zero.

---

*Domain 1 weight: 27% of exam. Companion roadmap: [[cca_domain1_roadmap]]. Companion practice exam: [[cca_domain1_practice_v2]].*
