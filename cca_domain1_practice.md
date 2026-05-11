---
title: Claude Certified Architect — Foundations · Domain 1 Practice Bank
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
  - domain-1
status: drilling
total-questions: 25
domain: 1
domain-weight: 27
target-percent: 85
focus: "Agentic Architecture & Orchestration"
---

# CCA-F Domain 1 Practice Bank

> [!info] Linked notes
> [[cca_exam_guide]] · [[cca_docs_roadmap]] · [[cca_progress]] · [[cca_practice_bank]]

> [!tip] Why this exists
> Domain 1 (Agentic Architecture & Orchestration) is **27% of the exam — the largest single weight**. This focused bank gives you 25 fresh questions covering all 7 task statements, organized by task rather than scenario so you can study systematically and pinpoint exact weak spots.

## How to Use This Bank

- [ ] **Round 1** — Cold attempt all 25. ~2 minutes per question. Mark answers.
- [ ] **Round 2** — Score using the callouts. Read every explanation.
- [ ] **Round 3** — Re-attempt only your misses. If you miss 2+ on the same task statement, that task is your study target — see Recovery section at the end.

**Pass threshold:** Real exam passes at 720/1000 (72%). Domain 1 alone determines a quarter of your score. Aim for **85%+** on this bank.

## Task Statement Coverage

| Task | Title | Questions in this bank |
|------|-------|------------------------|
| 1.1 | Design and implement agentic loops | 4 (Q1–Q4) |
| 1.2 | Coordinator-subagent orchestration | 4 (Q5–Q8) |
| 1.3 | Subagent invocation, context passing, spawning | 4 (Q9–Q12) |
| 1.4 ⚠️ | Multi-step workflows with enforcement and handoff | 5 (Q13–Q17) |
| 1.5 | Agent SDK hooks | 3 (Q18–Q20) |
| 1.6 | Task decomposition strategies | 3 (Q21–Q23) |
| 1.7 | Session state, resumption, forking | 2 (Q24–Q25) |

> ⚠️ Task 1.4 is a documented weak spot from earlier rounds — extra question allocated.

---

## Practice Questions

### Task 1.1 — Design and implement agentic loops

#### Question 1

You are building an agent and your loop terminates after exactly 10 iterations regardless of whether Claude has finished its work. Logs show some tasks complete after 3 iterations while others get truncated at 10 with `stop_reason: "tool_use"` still pending. What is the documented control flow? ^d1-q-1

A) Continue increasing the iteration cap (e.g., to 50) until truncation stops happening.
B) Inspect `stop_reason` after each iteration: continue when it equals `"tool_use"`, terminate when it equals `"end_turn"`. Iteration caps are safety stopgaps, not the primary control mechanism.
C) Add a separate validator agent that monitors progress and decides when to terminate.
D) Parse the response text on each iteration and terminate when keywords like "complete" or "done" appear.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-1|Task 1.1 — Design and implement agentic loops]]
>
> The agentic loop is driven by `stop_reason`. Iteration caps are stopgaps for safety, not the primary control mechanism. Option A papers over the symptom. Option C over-engineers when the protocol already provides the signal. Option D is a documented anti-pattern (parsing natural language signals to determine termination).

#### Question 2

Your agentic loop calls Claude, gets back a `tool_use` response, executes the tool, then sends the user's *original* prompt back to Claude with no mention of the tool's result. The agent acts as if no tool was ever called and asks the user to clarify their original request. What's wrong? ^d1-q-2

A) Tool results must be appended to the conversation history before the next iteration so the model can reason about them.
B) The model needs `permissionMode: "acceptEdits"` to recognize tool results.
C) Claude needs a higher `max_tokens` setting to remember tool results across iterations.
D) You should re-send the original prompt with a system message saying "the tool was called."

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-1|Task 1.1 — Tool result handling]]
>
> Tool results are appended to conversation history between iterations so the model can incorporate new information into its reasoning. Without that append, Claude has no record of what happened. Options B, C, D address unrelated mechanics — none of them is how the protocol carries tool output forward.

#### Question 3

Your code has a hardcoded `if/else` block: when user input contains "refund," call `process_refund`; when it contains "order," call `lookup_order`. The agent works for clean phrases but fails on inputs like "I want to return the item I bought yesterday." What's the documented approach? ^d1-q-3

A) Add 50 more keyword patterns to the if/else to cover edge cases.
B) Let Claude reason about which tool to call based on context — provide tools with clear descriptions and let the model select. Don't bypass model-driven decision-making with pre-configured trees.
C) Add a regex-based input classifier that runs before the agent.
D) Combine all backend operations into a single tool with internal dispatch logic that re-implements the keyword routing.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-1|Task 1.1 — Model-driven vs pre-configured decisions]]
>
> Model-driven decision-making (Claude reasons about which tool to call based on context) is the documented pattern, distinct from pre-configured decision trees or tool sequences. Option A keeps the same broken pattern with more rules. Option C bypasses the model's natural language understanding. Option D hides the routing inside a tool but doesn't fix the underlying brittleness.

#### Question 4

Which statement most accurately describes loop termination logic in the Claude Agent SDK? ^d1-q-4

A) Continue while the assistant's text content is non-empty; terminate when it returns an empty string.
B) Continue when `stop_reason` is `"tool_use"`; terminate when `stop_reason` is `"end_turn"`. Other stop reasons may exist but these are the two values that drive control flow.
C) Always run for exactly 10 iterations, then return whatever the model produced.
D) Terminate as soon as the assistant returns any text content; tool calls always happen on the first iteration only.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-1|Task 1.1 — stop_reason control flow]]
>
> `stop_reason` is the documented control signal. `"tool_use"` means continue (execute the tool, append result, send back). `"end_turn"` means done. Option A is a documented anti-pattern (checking text content as completion indicator). Option C is the iteration-cap anti-pattern. Option D fundamentally misunderstands the protocol — tool calls can occur in any iteration.

---

### Task 1.2 — Coordinator-subagent orchestration

#### Question 5

Your research coordinator always invokes web search → document analysis → synthesis, in that order, regardless of the query. A user asks "What is the capital of France?" and the system burns 30 seconds and 4 subagent calls to produce a one-sentence answer. What's the architectural improvement? ^d1-q-5

A) Cache common queries so the same question doesn't trigger the full pipeline twice.
B) The coordinator should analyze query requirements and dynamically select which subagents to invoke. Simple lookups may need only one subagent — or none.
C) Switch to a faster model for the synthesis subagent.
D) Run all three subagents in parallel to cut latency in half.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-2|Task 1.2 — Dynamic subagent selection]]
>
> Coordinators should analyze query requirements and dynamically select which subagents to invoke rather than always routing through the full pipeline. Option A is operational, not architectural. Options C and D don't address the pipeline rigidity — they just shave time off a workflow that shouldn't have run in the first place.

#### Question 6

You spawn three search subagents on the topic "modern JavaScript frameworks." All three return overlapping results dominated by React content. The synthesis agent ends up writing a React-heavy report despite the topic being broader. What's the design fix? ^d1-q-6

A) Partition research scope across subagents — assign one to React-family frameworks, one to Vue/Svelte/Solid, one to backend/full-stack frameworks. Distinct subtopics or source types per agent.
B) Increase the result count for each subagent so more topics surface naturally.
C) Have one subagent do the search and the other two summarize different sections of the result.
D) Run a fourth subagent to deduplicate findings after the fact.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-2|Task 1.2 — Partitioning research scope]]
>
> Partitioning research scope across subagents to minimize duplication is the documented pattern — distinct subtopics or source types per agent. Option B floods context with the same noise. Option C wastes parallelism. Option D treats symptoms after compute is wasted.

#### Question 7

Your coordinator gives each subagent step-by-step procedural instructions: "First fetch the latest 10 articles. Then extract claims using regex pattern X. Then format as JSON." When a subagent encounters articles in a slightly different format, the workflow breaks. What's the documented prompting approach for coordinators? ^d1-q-7

A) Add more if/else branches to the procedural instructions to cover variations.
B) Specify research goals and quality criteria in the coordinator prompt rather than step-by-step procedures, enabling subagent adaptability.
C) Use a separate validator subagent to check each procedural step before proceeding.
D) Lock the procedural format and reject any input that doesn't match it.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-2|Task 1.2 — Goals over procedures]]
>
> Designing coordinator prompts that specify research goals and quality criteria — rather than step-by-step procedural instructions — enables subagent adaptability. Option A perpetuates the brittle pattern. Option C adds more brittle steps. Option D doesn't solve inflexibility; it just rejects more inputs.

#### Question 8

Your team is debugging a production multi-agent system. Subagents call each other directly and you have no central place to observe what's happening, retry failed calls, or enforce error handling consistently. What architecture should you adopt? ^d1-q-8

A) Add tracing libraries to each subagent so logs are aggregated externally.
B) Hub-and-spoke: route all inter-subagent communication through a coordinator that handles errors, retries, and observability centrally.
C) Add a sidecar service that intercepts subagent traffic transparently.
D) Designate one subagent as the lead and have other subagents forward errors to it.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-2|Task 1.2 — Hub-and-spoke architecture]]
>
> Hub-and-spoke (coordinator manages all inter-subagent communication, error handling, and observability) is the documented pattern. Option A treats only logging. Option C adds infrastructure complexity outside the SDK. Option D doesn't establish true central authority — it just shifts the same problem to one designated subagent.

---

### Task 1.3 — Subagent invocation, context passing, spawning

#### Question 9

Your coordinator code defines two subagents in the `agents` parameter and emits a `Task` tool call. The call fails with an error indicating the Task tool isn't available. What's missing? ^d1-q-9

A) The `Task` tool must be included in the coordinator's `allowedTools` list. Subagents are invoked through the Task tool, and it must be explicitly allowed.
B) The model must be set to opus; subagents don't work on smaller models.
C) The `agents` parameter only works in the TypeScript SDK, not Python.
D) The subagent definitions must live in `.claude/agents/` filesystem files; the programmatic `agents` parameter is deprecated.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-3|Task 1.3 — allowedTools must include Task]]
>
> `allowedTools` must include `"Task"` for a coordinator to invoke subagents. The Task tool is the mechanism for spawning. Options B, C, D are fabricated — none reflects the actual SDK requirement.

#### Question 10

Your synthesis subagent has the same tool list as the main coordinator (Read, Write, Bash, web_search, Task, etc.). Logs show the synthesis agent occasionally launching new web searches, contradicting the upstream search agent's findings. What's the right scoping? ^d1-q-10

A) Use `AgentDefinition` with restricted tools per role: synthesis gets only synthesis-relevant tools (no web_search). Each subagent should have only the tools needed for its specific role.
B) Reduce the synthesis agent's `max_tokens` so search results don't fit.
C) Add an instruction in the synthesis agent's system prompt: "Do not call web_search."
D) Increase the synthesis agent's temperature so it's less likely to repeat upstream work.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-3|Task 1.3 — AgentDefinition tool restrictions]]
>
> `AgentDefinition` includes descriptions, system prompts, and tool restrictions for each subagent type. Scoped tool access prevents cross-specialization misuse. Option C is probabilistic. Options B and D address unrelated concerns — neither prevents the wrong tool call.

#### Question 11

Your coordinator hands the synthesis agent a flat paragraph: "Articles say AI adoption is up 40 percent and concerns about job displacement remain." The synthesis agent loses track of which source said what. What's the documented context-passing approach? ^d1-q-11

A) Use a longer paragraph with source names embedded in prose.
B) Use structured data formats that separate content from metadata: arrays of `{claim, source_url, document_name, publication_date}`. Preserve attribution explicitly.
C) Have the synthesis agent re-fetch the original sources to recover attribution.
D) Add a system prompt instruction: "Always remember source attribution."

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-3|Task 1.3 — Structured context passing]]
>
> Use structured data formats to separate content from metadata (source URLs, document names, page numbers) when passing context between agents to preserve attribution. Option A still mixes content and metadata. Option C duplicates work. Option D is probabilistic.

#### Question 12

Your coordinator emits one `Task` tool call, waits for the subagent's result, then emits the next `Task` call in a follow-up turn. Three subagent invocations take 60 seconds total when the underlying work is independent. What's the documented parallel pattern? ^d1-q-12

A) Emit multiple `Task` tool calls in a single coordinator response. They execute concurrently and results return for the coordinator to handle in the next turn.
B) Run three separate coordinator instances in parallel and merge their outputs at the application layer.
C) Use the streaming API so subagent invocations don't block the coordinator's loop.
D) Combine the three subagents into one with a wider scope to eliminate coordination overhead.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-3|Task 1.3 — Parallel subagent spawning]]
>
> Spawning parallel subagents requires emitting multiple Task tool calls in a single coordinator response, not across separate turns. Option B duplicates coordinator state and breaks single-source-of-truth. Option C addresses streaming, not parallelism. Option D loses specialization.

---

### Task 1.4 — Multi-step workflows with enforcement and handoff ⚠️

#### Question 13

Your refund agent must call `verify_customer_identity` before `process_refund`. Logs show 8% of refund requests skip the verification step. You've added "always verify identity first" to the system prompt three times. What's the right enforcement? ^d1-q-13

A) Add few-shot examples showing the agent always verifying first.
B) Implement a programmatic prerequisite hook that blocks `process_refund` calls until `verify_customer_identity` has returned a verified ID. Probabilistic compliance is insufficient when ordering is critical.
C) Move identity verification logic into the `process_refund` tool itself.
D) Add a sentence to the system prompt: "MANDATORY: never process refunds without verification."

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-4|Task 1.4 — Programmatic prerequisite enforcement]]
>
> When deterministic ordering is required, programmatic prerequisites (hooks blocking downstream tool calls until prior steps complete) are the only reliable enforcement. Options A and D rely on probabilistic compliance and have a non-zero failure rate that's unacceptable for financial operations. Option C couples concerns and may still permit other tools to bypass verification.

#### Question 14

When your agent escalates a complex billing dispute to a human agent, the human receives only the customer's original message: "I want my money back, this is the third time." The human spends 10 minutes catching up. What's missing in the handoff? ^d1-q-14

A) Real-time chat between the AI and the human agent.
B) A structured handoff summary including customer ID, root cause analysis, partial findings (e.g., what was already verified), recommended action, and refund amount under discussion.
C) The customer's full social media history.
D) A complete transcript of every internal subagent invocation.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-4|Task 1.4 — Structured handoff summaries]]
>
> Compile structured handoff summaries (customer ID, root cause, refund amount, recommended action) when escalating to human agents who lack access to the conversation transcript. Option A doesn't give the human asynchronous context. Option C is irrelevant. Option D is too noisy — humans need synthesized summaries, not raw transcripts.

#### Question 15

A customer message reads: "Order #5821 came damaged, my subscription auto-renewed when I had cancelled, and your tracking page shows the wrong status." Your agent handles the damaged order and tells the customer to open separate tickets for the other two issues. What's the better approach? ^d1-q-15

A) Decompose the message into three distinct concerns, investigate each in parallel using shared customer context, then synthesize a unified resolution. Don't force the customer to repeat themselves.
B) Process the first concern only and politely close the conversation.
C) Spawn three independent subagents that each handle one concern, with no shared context.
D) Escalate any message containing multiple issues directly to a human.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-4|Task 1.4 — Multi-concern decomposition with shared context]]
>
> Decomposing multi-concern requests into distinct items, investigating each in parallel using shared context, then synthesizing a unified resolution is the documented pattern. Option B punishes the customer. Option C duplicates context lookups and loses coherent resolution. Option D escalates cases the agent could resolve.

#### Question 16

Which scenario most clearly requires programmatic enforcement (hooks/prerequisites) rather than prompt-based guidance? ^d1-q-16

A) Recommending the user proofread their message before submitting.
B) Encouraging the agent to use a friendly, professional tone.
C) Blocking refund operations exceeding the company's $500 policy limit when financial loss is at stake.
D) Suggesting the agent prioritize urgent tickets over routine ones.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-1-4|Task 1.4 — When deterministic enforcement is required]]
>
> When deterministic compliance is required (financial policy, identity verification, security, regulatory), prompt instructions have a non-zero failure rate and hooks provide guarantees. Options A, B, and D are advisory and tolerate occasional drift; C requires hard enforcement and tolerates none.

#### Question 17

How would you implement a prerequisite that blocks `process_refund` until `get_customer` has returned a verified customer ID? ^d1-q-17

A) Write a `PreToolUse` hook that fires on `process_refund` calls; check the conversation state for a prior successful `get_customer` invocation; return `permissionDecision: "deny"` if the prerequisite isn't satisfied.
B) Add a system prompt rule: "Always call get_customer before process_refund."
C) Define `get_customer` and `process_refund` as a single combined tool that internally checks the prerequisite.
D) Use `tool_choice: "any"` to force the model into tool selection.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-4|Task 1.4 — Hook-based prerequisite enforcement]]
>
> A `PreToolUse` hook intercepts the call, validates the prerequisite from prior conversation/tool history, and returns deny if not met. Option B is probabilistic. Option C couples concerns and may still permit unverified flows in adjacent tools. Option D doesn't enforce ordering between specific tools.

---

### Task 1.5 — Agent SDK hooks

#### Question 18

Three different MCP tools your agent uses return timestamps in different formats: Unix epoch, ISO 8601, and a numeric status code (1 = active, 0 = inactive). The agent gets confused when reasoning across them. What's the right intervention? ^d1-q-18

A) Implement a `PostToolUse` hook that normalizes heterogeneous data formats from different MCP tools into a consistent shape before the agent processes them.
B) Tell the agent in the system prompt to handle each format individually.
C) Rewrite the upstream MCP tools to all use ISO 8601.
D) Use a separate validator agent to check timestamp formats.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-5|Task 1.5 — PostToolUse for normalization]]
>
> `PostToolUse` hooks intercept tool results for transformation before the model processes them. This is the documented pattern for normalizing heterogeneous formats. Option B is probabilistic. Option C may not be possible if the tools are external or community-maintained. Option D adds an unnecessary subagent.

#### Question 19

You're choosing between adding a `PreToolUse` hook or a few-shot example to enforce a $500 refund cap. Under what conditions does the documented guidance favor the hook? ^d1-q-19

A) Always — hooks are objectively better than prompts in every case.
B) When deterministic compliance is required and the consequences of non-compliance are material (financial loss, security, regulatory). Hooks give guaranteed enforcement; prompts have non-zero failure rate.
C) When the team can't write good prompts.
D) When the model is small enough that prompt-based reasoning is unreliable.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-5|Task 1.5 — Hooks vs prompts decision criteria]]
>
> The choice is contextual. Hooks are for deterministic guarantees when compliance is critical; prompts can be sufficient for advisory behavior. Option A overstates and misses the tradeoff (hooks add complexity not always warranted). Options C and D miscast the decision criteria — it's about consequences, not team skill or model size.

#### Question 20

Your agent occasionally tries to delete customer records during cleanup operations. Your policy: deletions require human approval. What's the cleanest implementation? ^d1-q-20

A) Add a `PreToolUse` hook on `delete_customer` that returns `permissionDecision: "deny"` with `permissionDecisionReason: "Deletion requires human approval"` and triggers an alternative escalation flow.
B) Add a system prompt: "Never call delete_customer."
C) Remove `delete_customer` from `allowedTools` entirely and rebuild the tool list.
D) Use `tool_choice: "any"` with a curated tool list that excludes deletes.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-5|Task 1.5 — PreToolUse with deny and redirect]]
>
> Hooks can intercept outgoing tool calls to enforce compliance and redirect to alternative workflows (escalation). Option B is probabilistic. Option C is too blunt — sometimes the agent legitimately needs to surface deletion requests for human review. Option D doesn't allow conditional gating; it removes the capability entirely.

---

### Task 1.6 — Task decomposition strategies

#### Question 21

A user asks: "Review this PR for security, performance, and code style issues, all three areas separately, then summarize." Three predictable, independent aspects. What decomposition pattern fits? ^d1-q-21

A) Prompt chaining — break into sequential focused passes (security → performance → style → summary). Predictable structure with well-defined steps suits chaining.
B) Dynamic decomposition based on what's discovered first.
C) Single-pass execution with all three concerns in one prompt to share context.
D) Spawn three independent agents and skip the summary step.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-6|Task 1.6 — Prompt chaining for predictable workflows]]
>
> Prompt chaining is documented for predictable multi-aspect reviews where each step is well-defined. Option B is for open-ended tasks where structure emerges from exploration. Option C overloads context and dilutes attention across all three concerns. Option D drops the synthesis step the user asked for.

#### Question 22

A user asks Claude to "improve the test coverage of this legacy codebase." The right approach depends on what gets discovered: which modules lack coverage, which have hidden integration boundaries, what fixtures already exist. What decomposition pattern fits? ^d1-q-22

A) Prompt chaining with a fixed sequence of file analyses.
B) Dynamic decomposition: first map the structure and identify high-impact areas, then create a prioritized plan that adapts as test dependencies and integration points are discovered.
C) Single-pass execution where Claude writes tests as it explores.
D) Parallel decomposition spawning one agent per file simultaneously.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-6|Task 1.6 — Dynamic decomposition]]
>
> Open-ended tasks where the right subtasks emerge from exploration are dynamic decomposition territory. Option A imposes premature structure. Option C overloads context. Option D produces inconsistent test patterns without first establishing priorities or shared fixtures.

#### Question 23

A 35-file refactor review produced uneven depth — careful analysis on some files, superficial on others — and contradictory findings (the same pattern flagged in one file but approved in another). The single-pass review is failing. Which restructuring is documented? ^d1-q-23

A) Increase model temperature to reduce contradictions.
B) Split into per-file local-analysis passes plus a separate cross-file integration pass examining data flow and shared concerns. This avoids attention dilution.
C) Reject any PR over 20 files automatically.
D) Run three independent passes and majority-vote the findings.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-6|Task 1.6 — Per-file plus cross-file decomposition]]
>
> The documented pattern is per-file local passes plus a separate cross-file integration pass to avoid attention dilution and contradictions. Option A is unrelated to attention. Option C shifts the problem onto developers. Option D suppresses real findings caught in single passes (genuine bugs sometimes only emerge in one run).

---

### Task 1.7 — Session state, resumption, and forking

#### Question 24

You're returning to a 4-hour codebase analysis session from yesterday. The team has heavily modified the analyzed files overnight (10+ files changed). Should you `--resume` or start fresh? ^d1-q-24

A) Always resume — sessions exist to be continued.
B) Start a new session with a structured summary injected into the initial context. Resuming with stale tool results is less reliable than starting fresh with a clean summary that reflects current state.
C) Resume but tell the agent to ignore everything from yesterday.
D) Run `/compact` to clean up the old session, then resume from the compacted state.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-7|Task 1.7 — Resume vs fresh start with summary]]
>
> Starting a new session with a structured summary is more reliable than resuming with stale tool results. The agent's prior context is built on file states that no longer match reality. Option A ignores staleness. Option C is contradictory — the resumed session still has the stale data in context. Option D compresses but doesn't update facts; stale tool results remain stale after compaction.

#### Question 25

You decide to resume a session because the prior analysis is mostly still valid — only 2 of 30 files have been modified. What's the right way to handle the changes? ^d1-q-25

A) Resume and pretend nothing changed; the agent will figure it out from cached file contents.
B) Inform the agent in the resumption prompt about the specific files that changed and have it re-analyze just those, rather than triggering full re-exploration or relying on stale knowledge.
C) Resume and immediately run `/compact`.
D) Resume with the same prompt as the original session.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-7|Task 1.7 — Targeted re-analysis on resumption]]
>
> When prior context is mostly valid but specific files changed, inform the resumed session about those changes for targeted re-analysis. Option A leaves stale knowledge in place. Option C compresses but doesn't update facts. Option D ignores the changes entirely.

---

## Diagnostic Matrix · Domain 1

After scoring, mark every question you missed. **Two or more misses tied to the same task statement = study target.**

| Task | What it tests | # Qs | Question Numbers |
|------|---------------|------|------------------|
| 1.1 | Design and implement agentic loops · `stop_reason` · anti-patterns | 4 | [[#^d1-q-1\|1]], [[#^d1-q-2\|2]], [[#^d1-q-3\|3]], [[#^d1-q-4\|4]] |
| 1.2 | Coordinator-subagent orchestration · hub-and-spoke · partitioning | 4 | [[#^d1-q-5\|5]], [[#^d1-q-6\|6]], [[#^d1-q-7\|7]], [[#^d1-q-8\|8]] |
| 1.3 | Subagent invocation · Task tool · context passing · parallel spawn | 4 | [[#^d1-q-9\|9]], [[#^d1-q-10\|10]], [[#^d1-q-11\|11]], [[#^d1-q-12\|12]] |
| 1.4 ⚠️ | Multi-step workflows · prerequisites · multi-concern · structured handoff | 5 | [[#^d1-q-13\|13]], [[#^d1-q-14\|14]], [[#^d1-q-15\|15]], [[#^d1-q-16\|16]], [[#^d1-q-17\|17]] |
| 1.5 | Hooks · PostToolUse normalization · PreToolUse compliance | 3 | [[#^d1-q-18\|18]], [[#^d1-q-19\|19]], [[#^d1-q-20\|20]] |
| 1.6 | Task decomposition · chaining vs dynamic · per-file vs cross-file | 3 | [[#^d1-q-21\|21]], [[#^d1-q-22\|22]], [[#^d1-q-23\|23]] |
| 1.7 | Session state · `--resume` · `fork_session` · fresh vs resume | 2 | [[#^d1-q-24\|24]], [[#^d1-q-25\|25]] |

---

## Recovery — If You Missed Multiple

### Task 1.1 (loops, `stop_reason`)

Re-read **How the agent loop works** at `docs.claude.com/en/docs/agent-sdk/agent-loop`. Memorize: `stop_reason: "tool_use"` continues, `stop_reason: "end_turn"` terminates. Build a minimal agent loop in Python that prints `stop_reason` at every iteration; watch the values change.

### Task 1.2 (coordinator-subagent)

Re-read **Subagents in the SDK** at `docs.claude.com/en/docs/agent-sdk/subagents`. Pay attention to the patterns around dynamic subagent selection and iterative refinement loops. Build Exercise 4 from the official exam guide with a coordinator that detects coverage gaps and re-delegates.

### Task 1.3 (Task tool, context, parallel)

Same Subagents page. Focus on the section about `allowedTools` requiring `"Task"`, on context not being inherited automatically, and on parallel spawning by emitting multiple `Task` calls in one response. Write code that spawns two subagents in parallel; measure latency vs sequential.

### Task 1.4 ⚠️ (workflows, prerequisites, handoff)

Re-read **Control execution with hooks** at `docs.claude.com/en/docs/agent-sdk/hooks`. Build the $500-cap hook from Exercise 1 of the exam guide — actually wire it up. Then build the multi-concern decomposition pattern: a customer message with three issues handled in a single coherent response. Drill on the exam guide's Task 1.4 skills section verbatim until you can recite the four bullet points.

### Task 1.5 (hooks)

Same hooks page. Focus on `PreToolUse` (intercepting outgoing calls) vs `PostToolUse` (transforming results before the model sees them). Implement both: a PostToolUse hook that normalizes timestamps, and a PreToolUse hook that blocks high-value operations.

### Task 1.6 (task decomposition)

This is more conceptual than feature-based. Re-read pages 5–6 of the exam guide ([[cca_exam_guide#^task-1-6]]) for the chaining-vs-dynamic distinction. Practice on a real task: take a multi-file PR and decompose it into per-file plus cross-file passes, then compare the result to a single-pass review.

### Task 1.7 (sessions)

Re-read **Work with sessions** at `docs.claude.com/en/docs/agent-sdk/sessions`. Practice three things: resume a named session, fork a session for divergent exploration, and start fresh with an injected summary when prior tool results are stale. Feel the friction of each.

---

## Update Your Progress File

When you finish a round on this bank, update [[cca_progress]] — log score in the Practice Score Log, and add any new misses to the Miss Log with question references like `[[cca_domain1_practice#^d1-q-13]]` so re-attempts are one click away.

---

*25 questions · All 7 task statements · Domain 1 weight: 27% of the exam.*
