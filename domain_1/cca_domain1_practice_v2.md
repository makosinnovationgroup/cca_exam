---
title: Claude Certified Architect — Foundations · Domain 1 Practice Exam
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
  - domain-1
status: drilling
total-questions: 30
domain: 1
domain-weight: 27
target-percent: 80
difficulty: harder-than-real-exam
---

# CCA-F Domain 1 — Practice Exam

30 questions covering Agentic Architecture & Orchestration (27% of the real exam). Calibrated **harder than the real exam** — distractors are plausible production patterns, not obviously wrong choices. Multiple options will often look reasonable; the correct answer is the one matching the documented Anthropic pattern.

## How to use

- [ ] **Round 1** — cold attempt all 30 without checking answers
- [ ] **Round 2** — score using the collapsed callouts; read the distractor analysis even on questions you got right
- [ ] **Round 3** — re-attempt any miss after re-reading the relevant roadmap section

**Aim for 80%+ on Round 1.** Anything below 70% means the underlying material needs another pass through the roadmap before drilling more.

## Task statement coverage

| Task | Questions | Focus |
|------|-----------|-------|
| 1.1 | Q1-Q5 | Agentic loops, `stop_reason`, iteration limits |
| 1.2 | Q6-Q9 | Coordinator orchestration, hub-and-spoke, iterative refinement |
| 1.3 | Q10-Q14 | Subagent invocation, `Agent` tool, context passing |
| 1.4 | Q15-Q19 | Multi-step workflows, hook-enforced compliance |
| 1.5 | Q20-Q23 | Hook events, deny pattern, matcher patterns |
| 1.6 | Q24-Q27 | Prompt chaining vs dynamic decomposition |
| 1.7 | Q28-Q30 | Sessions: resume, fork, fresh-with-summary |

---

## Practice questions

### Task 1.1 — Agentic loops

#### Question 1

A developer's customer support agent occasionally fails to terminate after the customer's issue is resolved. The agent keeps calling tools redundantly even after the resolution message is generated. The developer's loop checks `response.text.includes("resolved")` to exit. What's the **best** fix?  ^d1-q-1

A. Increase the `max_tokens` limit so the agent has room to produce a clearer completion signal.
B. Replace the text-content check with a `stop_reason` check that exits when `stop_reason === "end_turn"`.
C. Lower the model's temperature so it produces more consistent completion phrasing.
D. Add a system prompt instruction telling the agent to output the exact string `LOOP_COMPLETE` when finished.

> [!success]- Reveal answer
> **Correct: B**
>
> The agentic loop documentation calls out parsing text content for completion keywords as an anti-pattern. The protocol-level signal is `stop_reason`: `"tool_use"` continues the loop, `"end_turn"` terminates it.
>
> **Why others are wrong:**
> - A: `max_tokens` controls output length, not termination logic. The bug is in the loop control, not output capacity.
> - C: Lower temperature reduces variability in phrasing but doesn't address the underlying anti-pattern of parsing text for completion.
> - D: Hardcoding a "magic string" exit signal is a documented anti-pattern — it's just text-content parsing with extra steps. The protocol already provides `stop_reason`.

#### Question 2

An agent terminates the loop immediately after the first tool call, before processing the result. The developer's loop looks like:

```typescript
while (true) {
  const response = await client.messages.create({...});
  if (response.stop_reason !== "tool_use") break;
  // execute tool, append result
}
```

What's the **most likely** cause of premature termination? ^d1-q-2

A. The tool result is being appended to the conversation but the loop is being broken before the next iteration's API call.
B. `stop_reason` is `null` on the first response, which doesn't match `"tool_use"`, so the loop exits.
C. The model is choosing to respond in text rather than calling a tool, so `stop_reason` is `"end_turn"` immediately.
D. The conversation history isn't being passed back into the next `messages.create()` call.

> [!success]- Reveal answer
> **Correct: A**
>
> Re-reading the snippet: the `break` happens when `stop_reason !== "tool_use"`. That's correct logic, but the question states the loop terminates "before processing the result" — meaning the tool execution and result-append logic is *after* the break statement, never reached. The fix is to reorder: execute tool and append result *before* checking the next iteration's stop condition, or move the break to handle `"end_turn"` only after tool processing.
>
> **Why others are wrong:**
> - B: `stop_reason` is never `null` on a successful response — it's always one of the documented values.
> - C: Possible in some scenarios but doesn't match "terminates after the first tool call" — the question states a tool call did happen.
> - D: This would cause the model to lose context but wouldn't cause termination — the loop would still iterate.

#### Question 3

What `stop_reason` value indicates that a server-hosted tool (like web_search) is taking long enough that the API is yielding control back to the client to allow it to handle the pause? ^d1-q-3

A. `"max_tokens"`
B. `"end_turn"`
C. `"stop_sequence"`
D. `"pause_turn"`

> [!success]- Reveal answer
> **Correct: D**
>
> `"pause_turn"` is the documented server-side pause indicator for long-running tools. The client can resume by sending the conversation back; the model picks up where it left off.
>
> **Why others are wrong:**
> - A: `"max_tokens"` means the output token budget was reached — not a normal termination, but not a pause for long tools.
> - B: `"end_turn"` means the model is done with the turn.
> - C: `"stop_sequence"` indicates an explicit stop sequence (configured via `stop_sequences` parameter) was hit.

#### Question 4

An engineer sets `max_turns: 5` in their Agent SDK config to guarantee their agent stops after at most 5 iterations. Their senior reviewer pushes back. What's the **best** rationale for the pushback? ^d1-q-4

A. `max_turns` doesn't exist in the Agent SDK; the correct option is `maxIterations`.
B. Setting an iteration cap forces the model to rush its reasoning and reduces final-answer accuracy.
C. Iteration caps should function as safety stopgaps to prevent runaway loops, not as the primary control mechanism. The primary control should be the agentic loop's `stop_reason` signaling.
D. The cap should be much higher (50+) to give the agent room to recover from tool failures.

> [!success]- Reveal answer
> **Correct: C**
>
> The documented pattern: iteration caps are safety stopgaps for runaway behavior, not flow control. The primary stopping mechanism is the model's `stop_reason` returning `"end_turn"` when it's actually done. A cap of 5 may be too low or too high depending on the task; the issue is treating it as the primary signal.
>
> **Why others are wrong:**
> - A: `max_turns` (and `maxTurns` in TypeScript) does exist in the SDK.
> - B: There's no documented evidence that low iteration caps cause rushed reasoning — the model returns `end_turn` when done regardless of remaining budget.
> - D: The number isn't the issue; the role of the cap is.

#### Question 5

A developer is debugging an agent that always returns `stop_reason: "max_tokens"` instead of `"end_turn"` after just two tool calls. The agent's tools each return ~500 tokens of structured data. What's the **most likely** root cause? ^d1-q-5

A. The `max_tokens` parameter is set to a value lower than the expected response size for the agent's normal flow.
B. The model is being given too many tools (>20) and the system-prompt-prefix overhead leaves no room for output.
C. The conversation history is being truncated incorrectly so the model never sees the full tool results.
D. `stop_reason: "max_tokens"` is normal for tool-heavy agents and can be ignored.

> [!success]- Reveal answer
> **Correct: A**
>
> `max_tokens` caps the *output* tokens of a single response. If the model is generating tool calls plus a final answer that together exceed the cap, it hits `max_tokens` and stops mid-generation. Two tool calls plus a final answer often exceeds a low default (1024). Raising `max_tokens` to 4096 or higher typically resolves this.
>
> **Why others are wrong:**
> - B: Too many tools affects routing accuracy but doesn't cause `max_tokens` termination unless tool *output* is the issue (it isn't — the question is about response generation hitting the cap).
> - C: Truncated history would cause logical errors or repeated work, not `max_tokens` termination.
> - D: `max_tokens` termination is a real problem — it means the response is incomplete. Ignoring it produces broken outputs.

### Task 1.2 — Coordinator orchestration

#### Question 6

A research coordinator spawns three subagents in parallel for `[market_size, competitors, regulatory_risk]`. The final synthesis consistently misses important perspectives — for example, no mention of pricing dynamics. What's the **most appropriate** fix? ^d1-q-6

A. Switch the coordinator to a more capable model (Opus 4.7) to improve synthesis quality.
B. Have each subagent produce twice as many findings to reduce the probability of missing perspectives.
C. Add a fourth subagent for `pricing_dynamics` and continue with four parallel subagents.
D. Add an iterative refinement pass: after initial synthesis, the coordinator identifies gaps (e.g., "pricing not addressed") and spawns a follow-up subagent to fill them.

> [!success]- Reveal answer
> **Correct: D**
>
> Iterative refinement with gap detection is the documented pattern for "missed perspectives" failures. The coordinator first synthesizes, then critiques its own synthesis to find gaps, then spawns targeted follow-up subagents.
>
> **Why others are wrong:**
> - A: A bigger model produces better synthesis of whatever it's given. If the input doesn't contain pricing info, no model can synthesize it.
> - B: More findings within the same scope doesn't add missing scope.
> - C: A reasonable fix for the *specific* gap "pricing dynamics," but the question is about a pattern that misses perspectives in general. Iterative refinement handles unknown future gaps; adding a fixed fourth subagent only fixes one known gap.

#### Question 7

A team builds a multi-agent system where the `triage_agent` directly invokes `billing_agent`, which directly invokes `refund_agent`, which directly invokes `notification_agent`. The team experiences inconsistent error handling and difficult debugging when one of the deep-chain agents fails. What's the **best** architectural change? ^d1-q-7

A. Refactor to a hub-and-spoke topology where a single coordinator invokes each specialist agent and handles all errors centrally.
B. Add try/catch handlers around each direct invocation and have each agent forward errors to the next agent.
C. Add an `errorBoundary` agent that wraps each deep-chain agent and converts thrown errors into structured returns.
D. Increase `maxTurns` on each agent so they have more room to recover from errors internally.

> [!success]- Reveal answer
> **Correct: A**
>
> Hub-and-spoke is the documented topology for multi-agent systems. The coordinator owns: which subagents to invoke, in what order, with what inputs; how to handle subagent errors; how to synthesize results. Direct subagent-to-subagent calls (chain topology) create observability gaps and inconsistent error handling — exactly the symptoms described.
>
> **Why others are wrong:**
> - B: Per-handoff try/catch is exactly the anti-pattern that creates inconsistent error handling.
> - C: A fabricated agent type. Even if implemented, it preserves the chain topology that's the actual problem.
> - D: `maxTurns` doesn't address communication topology issues.

#### Question 8

In a hub-and-spoke multi-agent system, when should the coordinator dynamically select which subagents to invoke (rather than invoking all of them on every query)? ^d1-q-8

A. When subagents have overlapping responsibilities and the coordinator can pick the single best fit.
B. When the team wants to reduce overall token costs and improve latency by skipping unneeded work.
C. When subagent specializations are partitioned and not every query touches every specialization (e.g., a refund question doesn't need the shipping specialist).
D. When subagents share state via the coordinator's session, allowing one subagent's output to inform whether others run.

> [!success]- Reveal answer
> **Correct: C**
>
> Dynamic subagent selection is appropriate when **specializations are partitioned** — the question domain determines which subagents are relevant. A refund question doesn't need shipping logistics; a shipping question doesn't need refund policy.
>
> **Why others are wrong:**
> - A: Overlapping responsibilities is itself a design smell. The fix is to fix the overlap (Task 2.1 — tool/agent description disambiguation), not to mask it with dynamic selection.
> - B: Cost/latency are *consequences* of correct selection but not the design rationale. Selecting subagents to skip unneeded ones is correct *because* they're not relevant, not *because* it saves money.
> - D: Subagents don't share state — each runs in isolated context. Dynamic selection happens at the coordinator level based on query analysis, not subagent output.

#### Question 9

A multi-agent research system has 5 subagents that each produce ~3,000 tokens of findings. The coordinator's synthesis prompt produces a 500-token summary. Stakeholders complain the synthesis "feels generic" and lacks the specific evidence the subagents found. What's the **best** intervention? ^d1-q-9

A. Increase the coordinator's `max_tokens` to allow a longer synthesis that retains more specifics.
B. Restructure the synthesis to require each claim to carry structured evidence fields (`source_url`, `excerpt`, `subagent_id`) rather than producing prose summarization.
C. Have the coordinator iteratively call each subagent for specific claims rather than synthesizing them in one prompt.
D. Switch the coordinator to extended thinking mode so it can reason about evidence more carefully.

> [!success]- Reveal answer
> **Correct: B**
>
> This is a **provenance/structural** problem (Task 5.6). Prose summarization is lossy by nature — specifics get paraphrased away. The structural fix is to require evidence fields in the synthesis schema so each claim carries its source data forward through synthesis.
>
> **Why others are wrong:**
> - A: A longer prose summary may capture more, but the underlying lossiness is structural. Specifics still get paraphrased.
> - C: The original parallel architecture is sound. Iterative re-calling discards the benefit of parallelism.
> - D: Extended thinking helps with reasoning, not with preserving structured evidence through aggregation. The lossiness is in the output schema, not the reasoning.

### Task 1.3 — Subagent invocation

#### Question 10

An engineer's coordinator agent has the following config:

```typescript
const result = query({
  prompt: "Research market size for organic dog food.",
  options: {
    settingSources: ['project'],
    allowedTools: ["Read", "WebSearch", "WebFetch"]
  }
});
```

The coordinator is supposed to spawn three research subagents but it just answers the question itself directly. What's the bug? ^d1-q-10

A. The coordinator's prompt is too short — it needs explicit instruction to use subagents.
B. The `settingSources: ['project']` is preventing `.claude/agents/` from loading.
C. The Anthropic API model used is too small; only Opus reliably spawns subagents.
D. `"Agent"` is missing from `allowedTools` — without it, the coordinator can't invoke any subagents.

> [!success]- Reveal answer
> **Correct: D**
>
> The `Agent` tool is the documented mechanism for spawning subagents. If it's not in `allowedTools`, the coordinator literally lacks the capability to spawn subagents and falls back to answering directly.
>
> **Why others are wrong:**
> - A: Prompt brevity may or may not influence subagent use, but the **primary** blocker is the missing tool. A longer prompt without `Agent` tool still won't spawn subagents.
> - B: Wrong direction — `settingSources: ['project']` is exactly what loads `.claude/agents/` from the filesystem. Removing it would prevent loading.
> - C: All current models can spawn subagents when the `Agent` tool is available. There's no documented model-size requirement.

#### Question 11

A subagent definition includes a `prompt` field and a `description` field. What's the difference in their roles? ^d1-q-11

A. `description` is used by the coordinator's auto-routing to decide when to invoke this subagent; `prompt` is the system prompt the subagent itself receives when invoked.
B. `description` is shown in CLI tooltips; `prompt` is the user-facing label.
C. `description` becomes the first user message to the subagent; `prompt` is the system prompt.
D. Both fields are interchangeable — they're aliases for legacy compatibility.

> [!success]- Reveal answer
> **Correct: A**
>
> `description` drives the coordinator's selection logic — when deciding *whether* to invoke a subagent, the coordinator reads descriptions. `prompt` is the system prompt the subagent runs with once invoked — it shapes the subagent's behavior, persona, and constraints.
>
> **Why others are wrong:**
> - B: There is some UI use of `description` but its primary role is routing, not display.
> - C: The subagent's first user message is the prompt the coordinator passes when invoking, not the `description` field.
> - D: They have distinct, documented roles.

#### Question 12

A coordinator should spawn three subagents in parallel — `[market_research, competitor_analysis, pricing_analysis]`. At the protocol level, how is parallel spawning expressed? ^d1-q-12

A. Set `parallel: true` on the `Agent` tool definition; the coordinator emits a single `Agent` call with a `subagents: [...]` array.
B. The coordinator emits three separate API calls in parallel (concurrent `fetch` requests), one per subagent.
C. The coordinator emits a single assistant message containing three `tool_use` blocks — one for each `Agent` invocation. The SDK executes them concurrently and returns three matching `tool_result` blocks.
D. The coordinator uses the `spawn_parallel` SDK method, which is a higher-level wrapper around the `Agent` tool.

> [!success]- Reveal answer
> **Correct: C**
>
> The documented mechanism: one assistant message with multiple `tool_use` blocks. The SDK (or your loop code) executes them concurrently and returns the tool results in a single subsequent user message containing matching `tool_result` blocks. This is the same protocol pattern used for any parallel tool use.
>
> **Why others are wrong:**
> - A: `parallel: true` and `subagents: [...]` are fabricated. No such fields exist on `AgentDefinition`.
> - B: The agentic loop is a single conversation. Three parallel API calls would be three independent conversations with no coordination.
> - D: `spawn_parallel` is fabricated. The mechanism is built into the `Agent` tool + multi-block tool_use.

#### Question 13

A team using Claude Code in plan mode notices that the Plan subagent does deep research but never spawns its own subagents to parallelize. Which statement is **correct**? ^d1-q-13

A. The Plan subagent is misconfigured — adding `Agent` to its `tools` would enable subagent spawning.
B. The Plan subagent has a hardcoded `maxSubagents: 0`; this can be overridden via `.claude/settings.json`.
C. Subagents in Claude Code can spawn subagents only when running in `permissionMode: "bypassPermissions"`.
D. Subagents cannot spawn other subagents — this is a built-in restriction. Plan mode delegates research to the Plan subagent precisely because the main agent can't be a subagent that spawns subagents during plan mode.

> [!success]- Reveal answer
> **Correct: D**
>
> Subagent infinite nesting is prevented by design — subagents cannot spawn further subagents. This is part of why plan mode works the way it does: it delegates research to the Plan subagent rather than nesting.
>
> **Why others are wrong:**
> - A: Even if `Agent` were added to Plan's tools, it wouldn't bypass the no-nesting rule.
> - B: `maxSubagents` is fabricated.
> - C: Permission mode doesn't affect subagent nesting — it controls user permission prompts.

#### Question 14

A developer wants a subagent to have access to a specific MCP server (`internal-jira`) without exposing all 12 of the project's MCP servers. The subagent definition uses the `tools` field to list allowed tools. What's the correct way to scope MCP server access? ^d1-q-14

A. Add `mcp__internal-jira__*` patterns to the subagent's `tools` field as a wildcard inclusion.
B. Set the subagent's `mcpServers` field to `["internal-jira"]` to restrict it to that server's tools only.
C. Remove the `tools` field entirely; the subagent inherits the coordinator's tool access.
D. Add an `mcp_allowlist` field with `["internal-jira"]`; this filters MCP servers at invocation time.

> [!success]- Reveal answer
> **Correct: B**
>
> The `mcpServers` field on a subagent definition restricts which MCP servers' tools the subagent can see and invoke. This is the documented mechanism.
>
> **Why others are wrong:**
> - A: While `mcp__server__*` patterns work in some contexts (hooks `matcher`, settings allowlists), scoping subagent MCP access is done via the `mcpServers` field, not via tool patterns in the `tools` field.
> - C: Removing `tools` doesn't grant inheritance; subagents don't inherit coordinator context (Task 1.3 documented rule).
> - D: `mcp_allowlist` is fabricated.

### Task 1.4 — Multi-step workflows

#### Question 15

A customer support agent is supposed to refuse refunds over $500 and escalate them to a human. In production, ~3% of high-value refund requests still slip through with the agent processing them directly, despite the system prompt explicitly stating the policy. What's the **best** fix? ^d1-q-15

A. Add a `PreToolUse` hook on the `process_refund` tool that inspects the `amount` argument and returns `permissionDecision: "deny"` with a reason when it exceeds $500.
B. Restate the policy at the end of the system prompt and add three few-shot examples of refusals.
C. Switch the model to Opus 4.7, which has stronger instruction-following than Sonnet.
D. Add a post-processing step that audits all refund actions and reverses any that exceeded $500.

> [!success]- Reveal answer
> **Correct: A**
>
> Prompts are probabilistic — even strong system prompts produce ~3% failure rates on policy enforcement. Hooks are deterministic: a `PreToolUse` hook runs in code, not in the model, so 100% of refunds get the cap check before they execute.
>
> **Why others are wrong:**
> - B: More prompt instructions reduces but doesn't eliminate the probabilistic failure mode. Material financial risks need deterministic enforcement.
> - C: Model selection affects baseline accuracy but doesn't change the probabilistic nature of prompt-based compliance.
> - D: Post-processing reversal works but it's an after-the-fact remediation. Refunds may have already been disbursed; reversing creates customer confusion. Prevention via PreToolUse is the documented pattern.

#### Question 16

An agent must call `verify_account` before `process_payment`. The agent occasionally skips verification when the customer's message is clear and uncontested. How should the prerequisite be enforced? ^d1-q-16

A. Add a sentence to the system prompt: "ALWAYS call verify_account before process_payment."
B. Combine the two tools into one `verify_and_pay` tool that does both operations atomically.
C. Add a `PreToolUse` hook on `process_payment` that inspects conversation history; if a successful `verify_account` `tool_result` doesn't appear earlier in the turn, return `permissionDecision: "deny"` with a reason.
D. Add a `disable-model-invocation: true` to `process_payment` so only the verification logic can invoke it.

> [!success]- Reveal answer
> **Correct: C**
>
> Prerequisite enforcement is a deterministic-compliance scenario, identical pattern to Question 15. The hook inspects history to verify the precondition; if absent, it blocks and forces the agent to retry the prerequisite.
>
> **Why others are wrong:**
> - A: Prompt-based ordering is probabilistic.
> - B: Combining tools obscures the verification step (no audit trail for verify vs pay) and creates an atomic operation that's harder to handle when verification fails. The documented pattern keeps them separate and enforces ordering via hook.
> - D: `disable-model-invocation: true` is a *skill* frontmatter field, not a tool field. It makes a skill manual-only; it doesn't gate tool calls.

#### Question 17

A customer's message contains three concerns: an incorrect charge, a delayed shipment, and a request to cancel future auto-renewals. The agent processes only the first concern and ends the conversation. What's the **best** architectural fix? ^d1-q-17

A. Add a system prompt instruction: "Always address every concern in the customer's message."
B. Increase `maxTurns` so the agent has room to address more concerns in subsequent iterations.
C. Add few-shot examples of multi-concern messages being handled one-by-one in order received.
D. Decompose the customer message into a prioritized list of subtasks at the start of the turn, then iterate over each subtask, dispatching to the appropriate handler (refund, shipping, subscription).

> [!success]- Reveal answer
> **Correct: D**
>
> Multi-concern messages are a structural problem — they need decomposition. The documented pattern: parse the message into a prioritized subtask list, dispatch each to a handler, then synthesize a single response. This makes the multiple concerns visible in the loop so none are dropped.
>
> **Why others are wrong:**
> - A: Prompt-only doesn't change the architecture; the agent still has to discover and track concerns implicitly.
> - B: `maxTurns` doesn't address the structural issue. Without decomposition, the agent can iterate to 50 turns and still miss concerns.
> - C: Few-shot examples shape style but the underlying issue is that concerns aren't enumerated explicitly in the agent's working set.

#### Question 18

An agent must hand off to a human when it can't resolve an issue. A "structured handoff" payload should include all of the following **except**: ^d1-q-18

A. The customer's original message and key prior turns from the conversation.
B. The conversation's full token-level model invocation history with every API call, latency, and cost.
C. A list of actions the agent already took (tools called, with arguments and results).
D. The explicit reason for escalation (e.g., "amount exceeds $500 policy cap; customer is calm but persistent").

> [!success]- Reveal answer
> **Correct: B**
>
> Token-level model invocation history is observability data for engineers, not actionable context for a human support agent. A human receiving the handoff needs to understand the customer's situation and what's been tried — not the LLM internals.
>
> **Why others are wrong:**
> - A: Customer context is essential for continuity.
> - C: Actions taken inform the human what's been attempted so they don't repeat steps or contradict prior commitments.
> - D: Escalation reason guides the human to focus on the right issue.

#### Question 19

When is a hook **required** vs when is a prompt instruction **sufficient**? ^d1-q-19

A. A hook is required when the consequences of non-compliance are material (financial, regulatory, security, or legal); prompts are sufficient for advisory behavior where occasional drift is acceptable.
B. A hook is required only when working with regulated industries (healthcare, finance, government); prompts are sufficient elsewhere.
C. A hook is required when the agent runs in production; prompts are sufficient for development and staging.
D. Hooks and prompts are interchangeable — choose whichever is easier to maintain for the team.

> [!success]- Reveal answer
> **Correct: A**
>
> The documented decision criterion: severity of non-compliance. Material consequences (financial loss, regulatory violation, security breach, legal exposure) require deterministic enforcement via hooks. Advisory behavior (formatting, tone, occasional reminders) can tolerate prompt-based probabilistic enforcement.
>
> **Why others are wrong:**
> - B: Industry isn't the criterion. A B2C marketplace's $500 refund cap needs hook enforcement just as much as a HIPAA-regulated system's PHI access.
> - C: Production vs dev isn't the criterion. The criterion is consequence severity; production just makes those consequences material.
> - D: They're not interchangeable. Hooks run in code (deterministic); prompts run in the model (probabilistic).

### Task 1.5 — Hooks

#### Question 20

Which of the following is **not** a documented Python SDK hook event name? ^d1-q-20

A. `PreToolUse`
B. `UserPromptSubmit`
C. `BeforeToolExecution`
D. `PostToolUse`

> [!success]- Reveal answer
> **Correct: C**
>
> `BeforeToolExecution` is fabricated. The documented event is `PreToolUse`. Other documented Python SDK hooks include `PostToolUse`, `PostToolUseFailure`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `SubagentStart`, `PreCompact`, `Notification`, `PermissionRequest`.
>
> **Why others are wrong:** A, B, D are all real documented hook event names.

#### Question 21

A `PreToolUse` hook needs to block a tool call. What's the correct return value? ^d1-q-21

A. Throw a `BlockedError` with a message; the SDK converts thrown exceptions into denies.
B. Return `{hookSpecificOutput: {hookEventName: "PreToolUse", permissionDecision: "deny", permissionDecisionReason: "..."}}`.
C. Return `null` to signal "no permission granted."
D. Set `result.blocked = true` on the tool input and return it modified.

> [!success]- Reveal answer
> **Correct: B**
>
> The documented hook output shape includes `hookSpecificOutput` with `hookEventName`, `permissionDecision` (one of `"allow"`, `"deny"`, `"ask"`), and `permissionDecisionReason` (a human-readable explanation that gets propagated to the agent).
>
> **Why others are wrong:**
> - A: Throwing exceptions in hooks crashes the loop. Use structured return.
> - C: `null` isn't a documented signal; behavior is undefined.
> - D: `result.blocked` is fabricated. Hooks don't mutate tool input to express denial.

#### Question 22

A team's agent uses three MCP servers that each return order data in different shapes — one returns ISO dates, one returns Unix timestamps, and one returns `MM/DD/YYYY` strings. The agent occasionally fails to compare dates correctly across orders. What's the **best** intervention? ^d1-q-22

A. Tell the agent in the system prompt that dates come in three formats and explain each.
B. Replace the three MCP servers with a single unified MCP server that normalizes output.
C. Add a `PreToolUse` hook on each MCP tool that pre-normalizes inputs before the call.
D. Add a `PostToolUse` hook that intercepts tool results and normalizes all dates to ISO 8601 before the model sees them.

> [!success]- Reveal answer
> **Correct: D**
>
> `PostToolUse` hooks can transform tool results before the model sees them. Normalizing data formats (ISO dates, units, etc.) post-tool is the documented use case for `PostToolUse` data normalization.
>
> **Why others are wrong:**
> - A: Telling the model in prose still leaves date comparison probabilistic. Normalization in code is deterministic.
> - B: Replacing third-party MCP servers may not be feasible (vendor servers, community servers). Hooks normalize at integration time without touching the servers themselves.
> - C: `PreToolUse` runs *before* the call, on inputs to the tool. Date format normalization needed here is on *outputs*. Wrong hook event.

#### Question 23

A hook is configured with `matcher: "mcp__customer-support__*"`. Which calls does it match? ^d1-q-23

A. All tools from the `customer-support` MCP server (e.g., `mcp__customer-support__get_customer`, `mcp__customer-support__process_refund`).
B. All MCP tool calls across every connected MCP server.
C. Calls to a single tool named exactly `mcp__customer-support`.
D. Any tool whose name starts with `mcp__customer-support` regardless of namespace — including misconfigured tools that accidentally use that prefix.

> [!success]- Reveal answer
> **Correct: A**
>
> The `matcher` field uses glob patterns over tool names. `mcp__customer-support__*` matches all tools whose names begin with that prefix, which by namespacing convention are all tools from the `customer-support` MCP server.
>
> **Why others are wrong:**
> - B: Would require `mcp__*`, not the specific server prefix.
> - C: No trailing `*` would match exactly that string, but the matcher includes `*`.
> - D: While technically the glob matches by name prefix, in practice tool names follow the `mcp__<server>__<tool>` convention, so "namespace" is encoded in the prefix. Plus, the answer overgeneralizes — the question asks which calls *does* match, and "any tool that accidentally uses that prefix" is theoretical noise that doesn't help with the practical answer.

### Task 1.6 — Decomposition

#### Question 24

Which of the following tasks is **best suited** for *prompt chaining* (fixed sequential subtasks) rather than dynamic decomposition? ^d1-q-24

A. Improving test coverage in a legacy codebase where it's unclear which modules need attention.
B. Resolving a production incident where the root cause is unknown and requires investigation.
C. Producing a code review that examines (1) security, (2) performance, (3) style, and (4) test coverage in that order.
D. Migrating a database schema where the migration path depends on production data shapes that haven't been characterized.

> [!success]- Reveal answer
> **Correct: C**
>
> Prompt chaining fits tasks where subtasks are **predictable and enumerable before execution**. Code review with four explicit dimensions (security, performance, style, test coverage) fits perfectly — you know the steps in advance.
>
> **Why others are wrong:**
> - A: Test coverage improvement requires investigation — steps emerge from findings. Dynamic decomposition.
> - B: Incident investigation requires hypothesis-driven exploration. Dynamic decomposition.
> - D: Schema migration where production data shapes are unknown requires investigation first. Dynamic decomposition.

#### Question 25

A senior engineer is reviewing how Claude Code should handle a 70-file refactor. They want to ensure attention isn't diluted across files. What's the **best** decomposition strategy? ^d1-q-25

A. Run Claude Code three times in parallel and merge the results to overcome single-pass attention dilution.
B. Decompose into per-file passes that examine each file independently, then a separate cross-file pass that examines relationships and inconsistencies between files.
C. Increase `maxTurns` to 200 so the agent has room to revisit files multiple times in one session.
D. Spawn 70 subagents, one per file, and have a coordinator synthesize their findings.

> [!success]- Reveal answer
> **Correct: B**
>
> Large multi-file work has a documented decomposition pattern: per-file passes (deep attention on each file's concerns) followed by a cross-file pass (relationships, shared types, calling conventions, inconsistencies). This addresses the attention-dilution problem head-on.
>
> **Why others are wrong:**
> - A: Three parallel runs without decomposition still dilute attention across the same 70 files. Majority-voting on a flawed approach.
> - C: More turns within one session doesn't increase attention quality — the model's attention budget is per-context-window, not per-turn.
> - D: 70 subagents is operationally expensive and produces 70 isolated views with no cross-file context. The per-file + cross-file pattern is what the docs describe.

#### Question 26

Which factor is **most relevant** to the decision between prompt chaining and dynamic decomposition? ^d1-q-26

A. Whether the model supports extended thinking (chaining for thinking models, dynamic for non-thinking).
B. Whether you're using Claude Code or the Agent SDK (Claude Code prefers chaining; SDK prefers dynamic).
C. The estimated cost of each approach (dynamic is always cheaper).
D. Whether the subtasks can be enumerated in advance (chaining) or whether each step's content depends on findings from previous steps (dynamic).

> [!success]- Reveal answer
> **Correct: D**
>
> The documented decision criterion: predictability of subtasks. If you can enumerate steps before starting, prompt chaining works. If steps depend on what's discovered along the way, dynamic decomposition is necessary.
>
> **Why others are wrong:**
> - A: Extended thinking is orthogonal to the decomposition strategy.
> - B: Both platforms support both patterns.
> - C: Cost is a consequence, not the criterion. Dynamic isn't always cheaper (it often does more iterations).

#### Question 27

A team needs to "improve the test coverage" of an unfamiliar codebase. They want to use Claude Code most effectively. Which decomposition strategy fits? ^d1-q-27

A. Dynamic decomposition — start with exploration to identify under-tested modules, then for each identified module, decide what kinds of tests are needed, then write them.
B. Prompt chaining — fixed sequence: (1) list files, (2) compute coverage, (3) write tests, (4) commit.
C. Single-shot — ask Claude Code "Improve test coverage" and let it figure out everything in one large prompt.
D. Parallel subagents — spawn one subagent per file with the same instruction.

> [!success]- Reveal answer
> **Correct: A**
>
> "Improve test coverage in an unfamiliar codebase" is exactly the dynamic decomposition use case: the next steps depend on what the exploration reveals. Different modules need different kinds of tests; the strategy emerges from findings.
>
> **Why others are wrong:**
> - B: Step 3 "write tests" hides where the actual decisions live (which tests, what kind). Prompt chaining presumes predictable steps; this task doesn't have them.
> - C: One shot lacks structure for a task with this much uncertainty; output quality degrades.
> - D: Parallel-per-file misses cross-file relationships (integration tests, shared fixtures, calling conventions) that often matter for coverage.

### Task 1.7 — Sessions

#### Question 28

A developer's session was 4 hours of agent work. Tomorrow they want to continue the work — but they've also pushed substantial commits since the session ended that changed several files the agent had been working on. What's the **best** approach? ^d1-q-28

A. Use `resume: <sessionId>` to pick up exactly where the previous session left off.
B. Use `continue: true` to resume the most recent session.
C. Start a fresh session and inject a summary of yesterday's progress (key decisions, files touched, open questions) — don't carry the prior tool results forward.
D. Use `forkSession: true` with `resume: <sessionId>` to branch from yesterday's state and explore.

> [!success]- Reveal answer
> **Correct: C**
>
> When files have changed substantially since the prior session, the prior tool results (file contents, search results) are now **stale**. Carrying stale context forward causes the agent to reason against outdated state and make wrong decisions. A fresh session with a clean summary is the documented pattern.
>
> **Why others are wrong:**
> - A and B: Both resume the prior session's state — the stale tool results come along, confusing the agent.
> - D: Forking carries the same stale state into the fork. Forking is for exploring divergent paths from a still-valid baseline, not for working around staleness.

#### Question 29

`forkSession: true` (used with `resume: <sessionId>`) creates a new session that branches from the resumed state. Which of the following is **true** about the fork's relationship to the original? ^d1-q-29

A. The fork and original share state — changes in either propagate to the other.
B. The fork inherits conversation history, system prompt, tool definitions, and prompt cache from the resumed state, but runs as an independent session ID with no further coupling to the original.
C. The fork is an alias of the original session; closing the fork closes the original.
D. The fork inherits only the conversation history; system prompt and tool definitions must be reconfigured.

> [!success]- Reveal answer
> **Correct: B**
>
> A forked session is fully independent after the branch point — it inherits the resumed state (conversation, system prompt, tool definitions, cache benefits) but operates as its own session. The original is unaffected by what happens in the fork.
>
> **Why others are wrong:**
> - A: There's no shared-state coupling; forks are independent.
> - C: Sessions are independent entities, not aliases.
> - D: Forks inherit the full session context, not just history.

#### Question 30

Can a forked session itself be forked (create a fork-of-a-fork)? ^d1-q-30

A. Yes, the SDK supports arbitrary fork nesting up to the model's context window limit.
B. Yes, but only if `forkSession: true` is paired with `allowNestedForks: true` in options.
C. Yes, by passing `--fork-session --recursive` on the CLI.
D. No — a fork cannot itself be forked. Forks are one level deep by design to prevent runaway branching.

> [!success]- Reveal answer
> **Correct: D**
>
> Like subagent non-nesting, fork non-nesting is a documented safety constraint. Forks are one level deep. To explore further divergence, branch from the original baseline again rather than from a fork.
>
> **Why others are wrong:**
> - A: No arbitrary nesting; the limit is structural, not memory-related.
> - B and C: `allowNestedForks` and `--recursive` are fabricated.

---

## Answer key summary

| Q | A | Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | B | 6 | D | 11 | A | 16 | C | 21 | B | 26 | D |
| 2 | A | 7 | A | 12 | C | 17 | D | 22 | D | 27 | A |
| 3 | D | 8 | C | 13 | D | 18 | B | 23 | A | 28 | C |
| 4 | C | 9 | B | 14 | B | 19 | A | 24 | C | 29 | B |
| 5 | A | 10 | D | 15 | A | 20 | C | 25 | B | 30 | D |

## Stats summary

- **Answer distribution:** A=8 (27%) · B=7 (23%) · C=7 (23%) · D=8 (27%)
- **Hardest questions** (subtle distinctions, two-look-right options): Q2, Q9, Q14, Q18, Q22, Q23, Q28
- **Trap questions** (looks like one task, tests another): Q9 (looks like 1.2, tests 5.6 provenance), Q22 (looks like 1.5, tests data normalization pattern)
- **Common-anti-pattern questions:** Q1 (text parsing), Q7 (chain topology), Q10 (missing Agent tool), Q15-Q16 (prompts for hard rules)

---

*Domain 1 weight: **27%** of the exam. Companion roadmap: [[cca_domain1_roadmap.md]]. Companion exercises: [[cca_domain1_exercises.md]].*
