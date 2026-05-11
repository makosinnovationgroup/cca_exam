---
title: Claude Certified Architect — Foundations · Domain 2 Practice Exam
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
  - domain-2
status: drilling
total-questions: 30
domain: 2
domain-weight: 18
target-percent: 80
difficulty: harder-than-real-exam
---

# CCA-F Domain 2 — Practice Exam

30 questions covering Tool Design & MCP Integration (18% of the real exam). Calibrated **harder than the real exam** — distractors are plausible production patterns or fabricated-but-realistic SDK fields, not obviously wrong choices.

## How to use

- [ ] **Round 1** — cold attempt all 30 without checking answers
- [ ] **Round 2** — score using the collapsed callouts
- [ ] **Round 3** — re-attempt any miss after re-reading the relevant roadmap section

**Aim for 80%+ on Round 1.**

## Task statement coverage

| Task | Questions | Focus |
|------|-----------|-------|
| 2.1 | [[#^d2-q-1\|Q1]]-[[#^d2-q-7\|Q7]] | Tool descriptions, differentiation, splitting generic tools |
| 2.2 | [[#^d2-q-8\|Q8]]-[[#^d2-q-13\|Q13]] | Structured errors, `is_error`, error category metadata |
| 2.3 | [[#^d2-q-14\|Q14]]-[[#^d2-q-19\|Q19]] | `tool_choice`, distribution, parallel tool use |
| 2.4 | [[#^d2-q-20\|Q20]]-[[#^d2-q-25\|Q25]] | MCP server scoping, credentials, community vs custom |
| 2.5 | [[#^d2-q-26\|Q26]]-[[#^d2-q-30\|Q30]] | Built-in tools: Grep, Glob, Read, Write, Edit, Bash |

---

## Practice questions

### Task 2.1 — Tool interface design

#### Question 1

A customer support agent has 18 tools and increasingly misroutes requests — calling `lookup_billing_history` when `get_account_status` was intended. Three engineering proposals are on the table. Which is the **primary** fix? ^d2-q-1

A. Add a routing prompt prefix that lists every tool and instructs Claude which to choose based on keyword matching.
B. Reduce the agent's tool set to 4-5 role-relevant tools per agent; route cross-role needs through a supervisor agent.
C. Rewrite the tool descriptions to include preconditions, effects, and discriminating criteria ("use this when... rather than...") so the model can select on documented signal rather than overlapping intent.
D. Switch to a model with a larger context window so all 18 tool descriptions fit comfortably with conversation history.

> [!success]- Reveal answer
> **Correct: C**
>
> The documented selection mechanism is **descriptions**. With 18 tools and misrouting, the underlying issue is that descriptions don't disambiguate. Improving descriptions is the most direct fix — reducing tool count (B) helps but doesn't address the root cause if the remaining 5 still have ambiguous descriptions.
>
> **Why others are wrong:**
> - A: Adding routing prompts is a documented anti-pattern. The model already uses descriptions for routing — bolting a separate router on top fragments the signal.
> - B: Tool-count reduction is a complementary tactic but doesn't fix description quality. If the 5 retained tools are still ambiguously described, misrouting continues.
> - D: Tool descriptions are tiny relative to model context windows. Context size isn't the bottleneck here.

#### Question 2

Two tools have descriptions that begin with "Handle a customer order." Which tactic is documented as the **primary** fix for description-driven confusion? ^d2-q-2

A. Differentiate descriptions with explicit preconditions, effects, and discriminating criteria — e.g., "Use only when the order has not yet shipped" vs "Use only when the order has been shipped and the customer wants money back."
B. Combine the two tools into one tool with an `action: "cancel" | "refund"` parameter.
C. Add the longer description to whichever tool is invoked more frequently.
D. Delete one tool to eliminate ambiguity.

> [!success]- Reveal answer
> **Correct: A**
>
> Preconditions, effects, and discriminating criteria are the three documented dimensions for differentiating similar tools. The phrasing "use only when X" and "use only when Y" pushes the model to disambiguate at routing time.
>
> **Why others are wrong:**
> - B: Combining into one tool with an action parameter hides routing behind an internal switch. The exam tests **splitting** generic tools as the documented pattern; combining is the opposite direction.
> - C: Description length isn't the issue; specificity is.
> - D: Destructive and doesn't address the documented pattern.

#### Question 3

A team has a single `analyze_document` tool used for: extracting fields, summarizing, classifying, and translating. Output quality is inconsistent. Which intervention is **most aligned** with documented guidance? ^d2-q-3

A. Add an `analysis_type` parameter that switches the tool's behavior based on the requested operation.
B. Add a longer description that explains all four modes and their respective output shapes.
C. Add few-shot examples to the system prompt showing all four use cases.
D. Split `analyze_document` into four purpose-specific tools: `extract_fields`, `summarize_content`, `classify_document`, `translate_document` — each with a focused description and output schema.

> [!success]- Reveal answer
> **Correct: D**
>
> Splitting generic tools into purpose-specific tools is the documented fix when a single tool is used for too many different purposes. Each purpose-specific tool gets a focused description and output schema, which improves both routing and output quality.
>
> **Why others are wrong:**
> - A: Adding an `analysis_type` parameter retains the generic surface and pushes the ambiguity into a parameter the model still has to choose. This is the consolidation anti-pattern in disguise.
> - B: A longer description for an overloaded tool doesn't fix the overload.
> - C: Few-shot examples shape style but the underlying tool overload remains. Strict format compliance and output quality require purpose-specific schemas.

#### Question 4

A new tool `process_payment` has the description: "Process a customer payment." After deployment, the agent invokes it in unsafe contexts (e.g., when verification hasn't happened yet). Which addition to the description provides the **best** documented improvement? ^d2-q-4

A. "Use this tool when the user asks for a payment to be processed."
B. "Use only after `verify_account` has been called successfully in the same turn. Effect: charges the customer's card on file. Do not use for refunds (use `refund_order`)."
C. "Use carefully and only after verifying that the customer's identity is confirmed."
D. "Use this tool to process payments. Do not use unsafely."

> [!success]- Reveal answer
> **Correct: B**
>
> The description encodes a precondition ("after verify_account has been called"), an effect ("charges card on file"), and a discriminating criterion ("not for refunds; use refund_order"). All three documented dimensions.
>
> **Why others are wrong:**
> - A: Restates the trigger without preconditions or effects.
> - C: Adds vague guidance without specifics. "Verify identity" doesn't tell the agent which tool or signal counts as verification.
> - D: Negative-prescriptive ("don't use unsafely") is documented as less effective than positive criteria ("use only when X").

#### Question 5

What's the **most important** reason to keep tool descriptions purpose-specific rather than generic? ^d2-q-5

A. Generic descriptions slow down the model's first-token latency due to longer context.
B. Generic descriptions cost more tokens per request, increasing API costs.
C. Generic descriptions cause Claude to misroute between similar tools, since description quality is the primary mechanism for tool selection.
D. Generic descriptions break schema validation when `strict: true` is set.

> [!success]- Reveal answer
> **Correct: C**
>
> Tool description quality is the **primary** signal for routing. Generic descriptions create overlapping signals that the model can't disambiguate, causing misroutes.
>
> **Why others are wrong:**
> - A: Description length matters minimally for latency.
> - B: Cost impact is negligible.
> - D: `strict: true` validates the *input schema* (parameters), not the description. Description content has no impact on schema validation.

#### Question 6

A tool description includes example queries: "Examples: 'how much did I spend last month?' or 'show me transactions from December.'" This is **most aligned** with which documented practice? ^d2-q-6

A. Including example queries and edge cases in tool descriptions helps the model recognize when the tool fits.
B. Example queries belong in system prompt few-shot blocks, not in tool descriptions.
C. Examples reduce description quality by adding noise the model must filter.
D. Examples are useful but should be moved to a separate `examples` field on the tool definition (a recent addition to the SDK).

> [!success]- Reveal answer
> **Correct: A**
>
> Including example queries and edge cases inside the tool description is documented as a way to help the model recognize the tool's fit. The example queries serve as natural-language signposts that complement the formal description.
>
> **Why others are wrong:**
> - B: Few-shot blocks in the system prompt can complement but don't replace example queries embedded in descriptions.
> - C: Well-chosen examples sharpen signal, not noise.
> - D: An `examples` field on tool definitions is fabricated — examples go inside the `description` string.

#### Question 7

An agent has tools `search_orders` and `search_products`. Which description rewrite is **most likely** to improve routing reliability when a user asks "Find item ABC-123"? ^d2-q-7

A. Add "Use this tool when the user wants to find an order or product" to both descriptions.
B. Reduce both descriptions to a single line so the model can compare them more easily.
C. Add a routing prefix to the system prompt: "If the user says 'order' use search_orders; if they say 'product' use search_products."
D. Add discriminating criteria to each: `search_orders` says "Use when the user references a specific past purchase or order ID (typically starts with ORD-)"; `search_products` says "Use when the user is exploring the product catalog or references a product SKU (typically a non-ORD prefix like ABC-123 or PRD-)."

> [!success]- Reveal answer
> **Correct: D**
>
> Discriminating criteria with concrete signals (ID prefixes "ORD-" vs "ABC-/PRD-") give the model documented cues to route correctly. "Find item ABC-123" has the product-SKU pattern, so the description steers routing to `search_products`.
>
> **Why others are wrong:**
> - A: Adding the same generic line to both doesn't differentiate — it actively makes them more similar.
> - B: Shorter descriptions for overlapping tools removes precisely the disambiguation that's needed.
> - C: Adding a routing prefix to the system prompt is a documented anti-pattern — it bypasses the model's natural routing on descriptions.

### Task 2.2 — Structured errors

#### Question 8

A custom tool's implementation raises a Python exception when an external API returns 500. The agent's loop crashes with an unhandled exception. What's the **documented fix**? ^d2-q-8

A. Catch the exception in the tool, return a generic error string in the tool result, and continue.
B. Return a `tool_result` block with `is_error: true` and structured metadata (`errorCategory: "transient"`, `isRetryable: true`, `description: "External API returned 500..."`) so the agent can reason about the failure.
C. Catch the exception and log it; return an empty string in the tool result so the agent doesn't see the error.
D. Add a `PreToolUse` hook that wraps every tool call in try/catch and converts exceptions to denies.

> [!success]- Reveal answer
> **Correct: B**
>
> The documented pattern: tools should **not** raise exceptions (which crash the loop). Instead, return a structured `tool_result` with `is_error: true` and metadata fields the model can reason about (`errorCategory`, `isRetryable`, `description`).
>
> **Why others are wrong:**
> - A: A generic error string loses categorization — the agent can't differentiate retryable from non-retryable.
> - C: Silent failure (returning empty string) is the documented anti-pattern — the agent has no signal that anything went wrong.
> - D: PreToolUse runs before the call; it doesn't handle exceptions thrown during call execution.

#### Question 9

Which error category fits a refund request that exceeds the configured $500 cap? ^d2-q-9

A. `transient` — the agent can retry after a backoff period.
B. `validation` — the input failed schema constraints and needs correction.
C. `business` — the operation is rejected by a policy rule and cannot be retried as-is; the agent should explain the policy or escalate.
D. `permission` — the agent lacks authorization for this specific operation.

> [!success]- Reveal answer
> **Correct: C**
>
> The four documented categories: `transient` (retryable network/upstream issues), `validation` (input shape problems), `permission` (authorization denial), `business` (policy rule). A refund over $500 is a *business policy* rejection — not a permission problem (the agent is authorized to call the tool), not validation (the input is shape-valid), not transient.
>
> **Why others are wrong:**
> - A: Transient errors are retryable. A business policy violation is not.
> - B: Validation errors are about input shape (negative amount, malformed ID, wrong type). $750 is a *valid amount* — it just exceeds policy.
> - D: Permission errors mean the agent lacks authorization. Here the agent has authorization to call `process_refund` — the call's *value* exceeds policy.

#### Question 10

A team's tool returns: `{is_error: true, content: "Something went wrong."}`. The agent receives this and replies to the user "I'm sorry, something went wrong. Please try again later." regardless of what actually happened. What's the **best improvement**? ^d2-q-10

A. Add structured metadata to the error response: `{is_error: true, content: "<error description>", errorCategory: "...", isRetryable: <bool>, description: "..."}` so the agent can craft a context-appropriate response.
B. Replace the "Something went wrong" message with a longer prose explanation that the model can quote verbatim.
C. Add a system prompt instruction telling the agent to be more specific in error responses.
D. Add a `PostToolUse` hook that retries failed calls automatically before the agent sees them.

> [!success]- Reveal answer
> **Correct: A**
>
> The "Something went wrong" pattern collapses all errors into one undifferentiated message. The agent has no signal about what kind of error, whether retry helps, or what to tell the user. Structured metadata fixes this at the source.
>
> **Why others are wrong:**
> - B: A longer prose error is still unstructured — the agent has to parse semantic meaning out of natural language, which is probabilistic.
> - C: Prompt instructions can't extract information that isn't in the tool result.
> - D: Auto-retry without categorization may retry non-retryable errors (like business policy violations), creating loops.

#### Question 11

What does `is_error: true` on a `tool_result` block tell Claude? ^d2-q-11

A. The tool was misconfigured and shouldn't be called again this session.
B. The agent's loop should terminate immediately.
C. The user should be notified directly without further agent reasoning.
D. The tool call did not succeed; the content describes the failure, and Claude should reason about how to respond (retry, escalate, ask user for input, or explain).

> [!success]- Reveal answer
> **Correct: D**
>
> The `is_error: true` flag is a signal to Claude that the tool call failed. Claude then reasons about the failure context (provided in `content` and ideally structured metadata) to decide next steps.
>
> **Why others are wrong:**
> - A: `is_error: true` is a per-call signal, not a session-level "this tool is broken" flag.
> - B: The loop continues — the agent reasons about how to respond to the failure.
> - C: The agent mediates the response; the failure isn't surfaced directly to the user without the agent's reasoning layer.

#### Question 12

A developer's tool implementation has this structure:

```typescript
try {
  const result = await externalApi.call(...);
  return { content: JSON.stringify(result) };
} catch (e) {
  // What goes here?
}
```

Which catch-block implementation is **best aligned** with documented practice? ^d2-q-12

A. `throw e;` — let the loop crash so the developer can debug.
B. `return { is_error: true, content: JSON.stringify({ errorCategory: classifyError(e), isRetryable: shouldRetry(e), description: e.message }) };`
C. `return { content: "Sorry, please try again." };` — return a user-friendly message.
D. `console.error(e); return { content: "" };` — log the error and return an empty result.

> [!success]- Reveal answer
> **Correct: B**
>
> Catch the exception (don't crash the loop) and return a structured error with category and retryability metadata in JSON content, plus `is_error: true` flag.
>
> **Why others are wrong:**
> - A: Crashes the agent loop. Tools should always return; never propagate exceptions upward.
> - C: Loses error categorization — the agent treats it like a successful response with an apology.
> - D: Silent failure — the agent sees an empty result and has no way to know anything failed.

#### Question 13

A `tool_result` block returns `is_error: true` with content `{"errorCategory": "transient", "isRetryable": true, "description": "Upstream timeout"}`. What's the agent's **expected reasoning**? ^d2-q-13

A. The agent should immediately escalate to a human since the operation failed.
B. The agent should respond to the user with the description verbatim and end the conversation.
C. The agent should consider retrying (since `isRetryable: true`), possibly with a brief backoff message to the user, or fall back to an alternative if available.
D. The agent should mark the conversation as failed and not respond further.

> [!success]- Reveal answer
> **Correct: C**
>
> `transient` + `isRetryable: true` indicates a temporary issue that retry can plausibly fix. The agent should reason about whether to retry (and how to communicate the brief delay to the user) or fall back if available. The structured metadata enables exactly this reasoning.
>
> **Why others are wrong:**
> - A: Escalation is appropriate for `business` or `permission` errors, not transient ones.
> - B: Returning the error description verbatim wastes the agent's reasoning capability.
> - D: Marking failed and stopping discards the retry path that `isRetryable: true` signals is viable.

### Task 2.3 — Tool distribution and `tool_choice`

#### Question 14

A team needs to guarantee that their agent calls `extract_metadata` as the very first tool call in a turn, before doing anything else. Which configuration achieves this? ^d2-q-14

A. Set `tool_choice: {type: "tool", name: "extract_metadata"}` for the first request in the turn.
B. Set `tool_choice: "any"` to force tool use.
C. Set the model's `temperature` to 0 so the agent always picks the same first tool.
D. Add a system prompt instruction: "Always call extract_metadata first."

> [!success]- Reveal answer
> **Correct: A**
>
> `tool_choice: {type: "tool", name: "X"}` forces the model to call the specified tool. For the first request only — subsequent requests can use `tool_choice: "auto"` to let the model proceed naturally after metadata is extracted.
>
> **Why others are wrong:**
> - B: `"any"` forces some tool but doesn't specify which.
> - C: Temperature 0 increases determinism but doesn't *guarantee* a specific tool — the model's preference still depends on description + prompt analysis.
> - D: Prompt-based ordering is probabilistic. Forced `tool_choice` is the deterministic mechanism.

#### Question 15

What does `tool_choice: "any"` guarantee? ^d2-q-15

A. The model must call exactly one tool per turn.
B. The model must call a tool from a specific named subset.
C. The model is required to call the tool with the longest description.
D. The model must call some tool (rather than responding in text), but the model picks which tool.

> [!success]- Reveal answer
> **Correct: D**
>
> `"any"` forces tool calling — the model can't respond in plain text — but lets the model select which tool to call. To force a specific tool, use `{type: "tool", name: "X"}`.
>
> **Why others are wrong:**
> - A: `"any"` doesn't restrict the number of tools — the model can still call multiple in parallel unless `disable_parallel_tool_use: true` is also set.
> - B: There's no "subset" restriction with `"any"` — it's "any of the available tools."
> - C: Description length has no documented role in `tool_choice` enforcement.

#### Question 16

A developer wants extended thinking enabled (`thinking: {type: "enabled", budget_tokens: 4096}`) and also wants to force their agent to call a specific tool on the first turn. The API returns an error. What's the **most likely** cause? ^d2-q-16

A. Extended thinking and any tool use are incompatible.
B. Extended thinking is only compatible with `tool_choice: "auto"` or `"none"`. Forced (`{type: "tool"}`) and `"any"` are not supported with thinking because they prefill the assistant message in ways that conflict with thinking blocks.
C. The `budget_tokens` for thinking must equal or exceed the tool's expected output size.
D. Forced tool_choice and extended thinking work, but the API requires beta header `thinking-tools-2025-08-15`.

> [!success]- Reveal answer
> **Correct: B**
>
> Documented compatibility: extended thinking works with `tool_choice: "auto"` and `"none"` only. Forced (`{type: "tool"}`) and `"any"` are incompatible.
>
> **Why others are wrong:**
> - A: Extended thinking works with `tool_choice: "auto"` and tool use proceeds normally.
> - C: `budget_tokens` is independent of tool output size.
> - D: Fabricated beta header.

#### Question 17

A team's agent occasionally calls two tools in parallel in a single response. They want exactly one tool call per turn for an audit trail. Which configuration achieves this? ^d2-q-17

A. Set `tool_choice: "auto"` (it implies one tool per turn).
B. Set `parallel_tool_use: false` on the agent definition.
C. Set `disable_parallel_tool_use: true` along with `tool_choice` (`"auto"`, `"any"`, or forced) to restrict to one tool per turn.
D. Reduce the agent's tool set to one tool so parallelism is mechanically impossible.

> [!success]- Reveal answer
> **Correct: C**
>
> `disable_parallel_tool_use: true` is the documented mechanism. It works in combination with `tool_choice`. With `"auto"`, the model may still respond in text *or* call exactly one tool. With `"any"` or forced, exactly one tool is called.
>
> **Why others are wrong:**
> - A: `"auto"` is the default; it doesn't restrict parallelism on its own — Claude may emit multiple tool_use blocks.
> - B: `parallel_tool_use: false` is fabricated; the correct field name is `disable_parallel_tool_use`.
> - D: Restricting to one tool defeats the purpose of having tools.

#### Question 18

A coordinator emits an assistant message with two `tool_use` blocks for parallel execution. The team's loop code then posts the results back like this:

```typescript
messages.push({ role: "user", content: [tool_result_1] });
messages.push({ role: "user", content: [tool_result_2] });
```

What's the **bug**? ^d2-q-18

A. Parallel tool results must be posted in a single user message containing both `tool_result` blocks. Splitting them across two user messages teaches Claude to stop emitting parallel tool calls in subsequent turns.
B. The `tool_result` blocks need explicit `parallel: true` metadata that the example is missing.
C. The two messages need to be in opposite order — `tool_result_2` first, then `tool_result_1`.
D. Each `tool_result` needs an explicit `index` field linking it back to the corresponding `tool_use` block.

> [!success]- Reveal answer
> **Correct: A**
>
> The documented formatting rule: parallel tool results go in **one** user message with both `tool_result` blocks. Splitting into two separate user messages breaks the parallel-tool-use pattern — the model learns to stop emitting multiple tool calls because the response shape doesn't match.
>
> **Why others are wrong:**
> - B: `parallel: true` is fabricated.
> - C: Order within the single user message generally matches the order of the `tool_use` blocks in the assistant message; reversing isn't required and doesn't fix the formatting issue.
> - D: The link is via `tool_use_id` (already on the `tool_result` blocks), not an `index` field.

#### Question 19

Which is **true** about `disable_parallel_tool_use: true`? ^d2-q-19

A. It only works with `tool_choice: "auto"` — using it with `"any"` or forced is rejected by the API.
B. It overrides `tool_choice: "any"` to behave like `"auto"`.
C. It cannot be used with extended thinking.
D. It restricts the agent to at most one tool call per turn. It can be combined with `"auto"` (zero or one tool), `"any"` (exactly one tool), or `{type: "tool", name: "X"}` (exactly that tool).

> [!success]- Reveal answer
> **Correct: D**
>
> `disable_parallel_tool_use: true` works with all three modes of `tool_choice` and has slightly different effects in each: with `"auto"` it allows zero or one tool; with `"any"` exactly one tool; with forced exactly the named tool.
>
> **Why others are wrong:**
> - A: It works with all three.
> - B: It doesn't override `tool_choice` semantics.
> - C: Extended thinking restrictions are on `tool_choice` values, not on `disable_parallel_tool_use`.

### Task 2.4 — MCP server scoping

#### Question 20

A team wants their Jira MCP server to be available to all engineers when working in the project, but they don't want to commit Jira credentials to the repo. What's the **documented** approach? ^d2-q-20

A. Put the server config and the credential in `.mcp.json`, but add `.mcp.json` to `.gitignore`.
B. Put the server config in `.mcp.json` with credentials referenced via environment variable expansion (`${JIRA_TOKEN}`); each engineer sets the env var in their shell from a secrets manager.
C. Put the credentials in `~/.claude.json` per-engineer; put the server config in `.mcp.json`.
D. Put both the config and credentials in `~/.claude.json` so they stay personal.

> [!success]- Reveal answer
> **Correct: B**
>
> The documented pattern: `.mcp.json` is project-scoped (Git-tracked) with env-var-expanded credentials. The placeholder `${JIRA_TOKEN}` resolves at load time to whatever the engineer has set locally. The config is shared (no leaked credential); the credential is per-engineer.
>
> **Why others are wrong:**
> - A: `.gitignore`-ing the only project-shared config defeats the purpose of sharing it.
> - C: Separating server config and credentials across files breaks the loading mechanism — `.mcp.json` expects its credentials at load time.
> - D: Putting the server config in `~/.claude.json` makes it personal — each engineer would have to copy-paste it, defeating shared configuration.

#### Question 21

What's the difference between `.mcp.json` and `~/.claude.json`? ^d2-q-21

A. `.mcp.json` is the new format; `~/.claude.json` is deprecated.
B. `.mcp.json` is for SDK projects; `~/.claude.json` is for Claude Code.
C. `.mcp.json` lives at the project root and is project-scoped (typically Git-tracked, shared with the team); `~/.claude.json` lives in the user's home directory and is user-scoped (personal, not shared).
D. `.mcp.json` is parsed by the MCP daemon; `~/.claude.json` is parsed only by Claude Code's CLI.

> [!success]- Reveal answer
> **Correct: C**
>
> Project-scope vs user-scope. Both are active simultaneously when Claude Code starts in a project that has `.mcp.json` — the union of servers from both is available.
>
> **Why others are wrong:**
> - A: Both are current; neither is deprecated.
> - B: Both work in both contexts.
> - D: There's no separate "MCP daemon"; the same loading mechanism reads both.

#### Question 22

A developer adds an experimental MCP server to `~/.claude.json` and asks teammates to do the same. The team complains they have to manually copy the config each time. What's the **right** intervention? ^d2-q-22

A. Move the server config from `~/.claude.json` to `.mcp.json` so it's Git-tracked and shared automatically.
B. Have a senior engineer maintain a doc with the config block to copy-paste.
C. Build a setup script (`setup-mcp.sh`) that writes the config to each developer's `~/.claude.json` on first run.
D. Submit a PR to Anthropic to add a "shared experimental" config path.

> [!success]- Reveal answer
> **Correct: A**
>
> The distinction between `.mcp.json` and `~/.claude.json` is exactly this: shared vs personal. If the team wants the config shared, it belongs in `.mcp.json`. The "experimental" framing doesn't change the scoping rule — once it's something the team uses, it's project-scoped.
>
> **Why others are wrong:**
> - B: Copy-paste is the manual process the team is trying to avoid.
> - C: A setup script duplicates the role of `.mcp.json` with extra mechanics. Use the documented scoping mechanism directly.
> - D: A new config path isn't needed — the existing scoping handles this.

#### Question 23

A community MCP server covers 80% of a team's Jira needs but doesn't expose a specific custom-field workflow they rely on. What's the **best** approach? ^d2-q-23

A. Fork the community server, add the custom-field workflow, maintain the fork.
B. Replace the community server with a fully custom server that covers 100% of needs.
C. Submit the missing workflow as a PR to the community server and wait for it to merge.
D. Use the community server for the standard 80% and build a small additional custom server for the team-specific custom-field workflow. Have Claude Code load both.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented pattern: community server for the standard subset, small custom server for the team-specific gap. Multiple MCP servers can be active simultaneously; the agent has access to tools from both.
>
> **Why others are wrong:**
> - A: Forking creates a maintenance burden — the team has to track upstream changes and re-apply their patches.
> - B: Reinventing the 80% wastes engineering time and loses the community's bug fixes and improvements.
> - C: Upstream PRs may take weeks/months or be rejected. Doesn't help in the short term.

#### Question 24

In MCP, what's the documented distinction between a **tool** and a **resource**? ^d2-q-24

A. Tools are read-write; resources are read-only.
B. Tools are *called* by the agent (they perform actions and return results, with an `inputSchema`); resources are *read* by the agent (they expose content like a catalog or doc index, with a `uri` and `mimeType`).
C. Tools require authentication; resources do not.
D. Tools execute in a sandbox; resources execute on the MCP server's host.

> [!success]- Reveal answer
> **Correct: B**
>
> The protocol-level distinction: tools have an `inputSchema` and are invoked; resources have a `uri`/`mimeType` and are read. A 5,000-document catalog should be exposed as a resource, not as 5,000 individual tools.
>
> **Why others are wrong:**
> - A: A read-only tool is still a tool. A writable resource doesn't fit MCP's resource concept.
> - C: Authentication is orthogonal to the tool/resource distinction.
> - D: Execution location isn't part of the type distinction.

#### Question 25

A team has 5,000 internal wiki documents they want their agent to query. Which MCP design is **most aligned** with documented guidance? ^d2-q-25

A. Expose 5,000 search-tools, one per document, with descriptions auto-generated from titles.
B. Expose 5 search-tools, one per top-level category, and trust the agent to route queries.
C. Expose **one** `search_docs` tool plus an MCP **resource** that lists all available documents with titles and URIs, so the agent can pre-filter or directly reference docs by URI.
D. Expose 5,000 documents as 5,000 resources without any search tool — let the agent enumerate them all.

> [!success]- Reveal answer
> **Correct: C**
>
> The combination: one focused `search_docs` tool for retrieval, plus a resource that exposes the catalog. The agent can either search or browse the catalog and reference docs directly. This is the documented pattern for large doc collections.
>
> **Why others are wrong:**
> - A: 5,000 tools floods the tool list and destroys routing accuracy.
> - B: Five search tools per category creates the routing problem at smaller scale but doesn't address the underlying need for catalog visibility.
> - D: 5,000 resources without search forces enumeration, which is inefficient for large catalogs.

### Task 2.5 — Built-in tools

#### Question 26

Which tool fits "find every `.test.tsx` file in the repository"? ^d2-q-26

A. **Glob** — pattern matches file paths by name pattern.
B. **Grep** — pattern matches file contents.
C. **Read** — load file contents by path.
D. **Bash** — invoke `find . -name "*.test.tsx"`.

> [!success]- Reveal answer
> **Correct: A**
>
> Glob is the documented tool for file-name and file-path pattern matching. `Glob` with pattern `**/*.test.tsx` returns the list.
>
> **Why others are wrong:**
> - B: Grep is for file *contents*, not paths.
> - C: Read loads a specific known file — it doesn't enumerate.
> - D: Bash is a last resort when built-in tools don't fit. Glob fits here perfectly.

#### Question 27

Which tool fits "find every file containing the string `TODO: deprecate`"? ^d2-q-27

A. Glob — pattern matches names.
B. Read — load files and check contents.
C. Bash — invoke `grep -r "TODO: deprecate" .`.
D. Grep — pattern matches file contents.

> [!success]- Reveal answer
> **Correct: D**
>
> Grep is the documented tool for content search. `Grep` with pattern `TODO: deprecate` returns matching files (and optionally line numbers).
>
> **Why others are wrong:**
> - A: Glob matches names, not contents.
> - B: Reading every file in the repo to check is wildly inefficient — that's exactly what Grep optimizes.
> - C: Bash works but is a last resort; the documented preference is the built-in Grep tool when it fits.

#### Question 28

An agent tries to `Edit` a file with the anchor text `"const x = 1"`, but the anchor appears 12 times in the file. The Edit fails. What's the **documented fallback**? ^d2-q-28

A. Extend the `Edit` anchor with surrounding context until it's unique (cheapest, do this first), then **Read + Write** as a second fallback if the file's structure makes a unique anchor impractical.
B. Use `Bash` with `sed -i "s/const x = 1/.../g" file.ts` to do a global replace.
C. Delete the file and regenerate it from scratch.
D. Switch to plan mode and propose a manual diff.

> [!success]- Reveal answer
> **Correct: A**
>
> The documented fallback ladder for `Edit` failures: extend the anchor to be unique (cheapest); if that's impractical, fall back to **Read + Write** (load the file, modify in memory, write back).
>
> **Why others are wrong:**
> - B: Bash + sed isn't documented; it bypasses the tool ecosystem.
> - C: Destructive and likely loses important context.
> - D: Plan mode is for designing changes, not for executing edits.

#### Question 29

When should an agent reach for `Bash` rather than other built-in tools? ^d2-q-29

A. Whenever speed is important — Bash is faster than the built-in tools.
B. When the agent needs shell-specific features (pipes, redirections, environment variables, command chaining) that the dedicated tools don't support.
C. When the operation type doesn't fit Read/Write/Edit/Grep/Glob — Bash is the documented last resort.
D. For all file operations; Bash provides a uniform interface.

> [!success]- Reveal answer
> **Correct: C**
>
> Bash is documented as the last resort — use it when the operation doesn't fit Grep, Glob, Read, Write, or Edit. The dedicated tools are preferred when they fit because they're auditable, sandboxed, and integrate with permissions.
>
> **Why others are wrong:**
> - A: Performance isn't the criterion; fit-to-operation is.
> - B: Some shell features can be useful, but the documented preference still favors built-in tools when they fit. Bash is reserved for operations that genuinely don't fit.
> - D: Using Bash for everything defeats the audit and permission benefits of dedicated tools.

#### Question 30

A user asks the agent to "show me the README." Which tool fits? ^d2-q-30

A. **Read** with path `README.md` (or the discovered path if it's not at the root).
B. Grep with pattern `README`.
C. Glob with pattern `**/README*`.
D. Bash with `cat README.md`.

> [!success]- Reveal answer
> **Correct: A**
>
> Read is the documented tool for loading specific file contents by known path. README is typically at the project root, so `Read("README.md")` fits directly.
>
> **Why others are wrong:**
> - B: Grep searches file *contents* for a pattern; it doesn't load a file.
> - C: Glob enumerates matching paths; useful if you don't know where the README is, but a follow-up Read is still needed to display its content.
> - D: Bash works but is last resort; Read fits this exact operation type.

---

## Answer key summary

| Q | A | Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|---|---|
| [[#^d2-q-1\|1]] | C | [[#^d2-q-6\|6]] | A | [[#^d2-q-11\|11]] | D | [[#^d2-q-16\|16]] | B | [[#^d2-q-21\|21]] | C | [[#^d2-q-26\|26]] | A |
| [[#^d2-q-2\|2]] | A | [[#^d2-q-7\|7]] | D | [[#^d2-q-12\|12]] | B | [[#^d2-q-17\|17]] | C | [[#^d2-q-22\|22]] | A | [[#^d2-q-27\|27]] | D |
| [[#^d2-q-3\|3]] | D | [[#^d2-q-8\|8]] | B | [[#^d2-q-13\|13]] | C | [[#^d2-q-18\|18]] | A | [[#^d2-q-23\|23]] | D | [[#^d2-q-28\|28]] | A |
| [[#^d2-q-4\|4]] | B | [[#^d2-q-9\|9]] | C | [[#^d2-q-14\|14]] | A | [[#^d2-q-19\|19]] | D | [[#^d2-q-24\|24]] | B | [[#^d2-q-29\|29]] | C |
| [[#^d2-q-5\|5]] | C | [[#^d2-q-10\|10]] | A | [[#^d2-q-15\|15]] | D | [[#^d2-q-20\|20]] | B | [[#^d2-q-25\|25]] | C | [[#^d2-q-30\|30]] | A |

## Stats summary

- **Answer distribution:** A=9 (30%) · B=6 (20%) · C=8 (27%) · D=7 (23%)
- **Hardest questions** (subtle distinctions, two-look-right options): [[#^d2-q-4\|Q4]] (precondition wording), [[#^d2-q-9\|Q9]] (business vs permission), [[#^d2-q-16\|Q16]] (extended thinking + tool_choice), [[#^d2-q-23\|Q23]] (community + custom), [[#^d2-q-24\|Q24]] (tool vs resource at protocol level)
- **Trap questions:** [[#^d2-q-1\|Q1]] (looks like it's about routing classifiers, tests description quality), [[#^d2-q-3\|Q3]] (looks like it's about parameters, tests splitting)
- **Common-anti-pattern questions:** [[#^d2-q-2\|Q2]] (consolidating instead of splitting), [[#^d2-q-8\|Q8]] (raising exceptions), [[#^d2-q-18\|Q18]] (separate user messages breaking parallel tool use)

---

*Domain 2 weight: **18%** of the exam. Companion roadmap: [[cca_domain2_roadmap]]. Companion exercises: [[cca_domain2_exercises]].*
