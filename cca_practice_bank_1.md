---
title: Claude Certified Architect — Foundations · Practice Bank
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
status: drilling
total-questions: 54
v1-questions: 18
new-questions: 36
scenarios-covered: 6
domains-covered: 5
target-percent: 85
---

# CCA-F Practice Bank · 54 Questions

> [!info] Linked notes
> [[cca_exam_guide]] · [[cca_docs_roadmap]] · [[cca_progress]]

> [!tip] How this works
> Every question's answer + explanation lives inside a collapsed callout. Click the callout title to expand. Cover, attempt cold, then reveal.

## How to Use This Bank

- [ ] **Round 1** — Cold attempt all 54. No notes. ~2 minutes per question.
- [ ] **Round 2** — Score using the callouts. Read every explanation, including those you got right.
- [ ] **Round 3** — One week later, re-attempt only the questions you missed in Round 1, plus the official guide's 12 sample questions.
- [ ] **Final pass** — 24-72 hours before exam, drill questions tied to your weakest task statements one more time. Then stop.

**Pass threshold:** Real exam passes at 720/1000 (72%). Aim for **85%+** on this bank to feel comfortable. Bank questions are calibrated to similar difficulty as the exam guide samples.

## Scenario Coverage

| Scenario | v1 Qs | New Qs | Total |
|----------|-------|--------|-------|
| 1 — Customer Support Resolution Agent | 4 | 6 | 10 |
| 2 — Code Generation with Claude Code | 4 | 6 | 10 |
| 3 — Multi-Agent Research System | 4 | 6 | 10 |
| 4 — Developer Productivity with Claude | 3 | 6 | 9 |
| 5 — Claude Code for Continuous Integration | 3 | 6 | 9 |
| 6 — Structured Data Extraction | 0 | 6 | 6 |
| **Total** | **18** | **36** | **54** |

> Scenario 6 was missing in v1. The exam draws 4 of 6 scenarios at random per session, so don't skip Scenario 6 questions even though they feel unfamiliar.

---

## Practice Questions

### Scenario 1: Customer Support Resolution Agent

#### Question 1

Your agent has access to 18 MCP tools spanning customer management, orders, returns, billing, account settings, and analytics. Production logs show it frequently calls the wrong tool (for example, calling `get_customer_summary` when the user asked about an order status). What is the most effective architectural change? ^q-1

A) Consolidate functionally-similar tools into fewer, broader tools with internal dispatch logic that determines the right backend call.
B) Restrict the agent to 4-5 tools relevant to its current role, routing rare cross-role needs through a supervisor agent that holds the additional tools.
C) Set `tool_choice` to `"any"` so the agent must call some tool rather than returning text.
D) Rewrite every tool description to explicitly enumerate the other tools it should not be confused with.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-3|Task 2.3 — Distribute tools appropriately across agents]]
>
> Giving an agent access to too many tools degrades selection reliability by increasing decision complexity. The exam guide explicitly uses 18 versus 4-5 as the example threshold. Scoping each agent to role-relevant tools is the documented fix. Option A hides complexity but loses specificity and testability. Option C ensures a tool is called, not which one. Option D bloats descriptions without reducing the underlying decision load.

#### Question 2

After long multi-issue conversations, your agent starts giving inconsistent refund amounts and referring to wrong order numbers. Progressive summarization has compressed numerical values and order IDs into phrases like "the recent purchase" and "approximately eighty dollars." What is the most effective fix? ^q-2

A) Disable summarization entirely and pass the full conversation history in every request.
B) Extract transactional facts (order IDs, amounts, dates, statuses) into a persistent case-facts block included in each prompt outside the summarized history.
C) Switch to a model with a larger context window so summarization is no longer necessary.
D) Instruct the agent in the system prompt to always repeat key numerical values in each response.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-1|Task 5.1 — Manage conversation context]]
>
> Progressive summarization loses precision on numerical values, dates, and customer-stated expectations. The documented fix is extracting transactional facts into a persistent structured layer that survives summarization. Option A does not scale across long sessions. Option C treats symptoms; a larger window still eventually summarizes. Option D relies on probabilistic compliance.

#### Question 3

Your agent correctly handles most refund requests, but occasionally processes refunds above your $500 business policy limit when customers frame the request emphatically. You have already updated the system prompt twice with clearer rules. What is the most reliable way to enforce the $500 cap? ^q-3

A) Add a hook that intercepts `process_refund` calls and blocks any amount exceeding $500, redirecting to human escalation.
B) Add ten to fifteen few-shot examples showing the agent refusing refunds above $500.
C) Require the agent to output its reasoning before calling `process_refund`, so a reviewer can audit decisions afterward.
D) Use `tool_choice` auto with a custom tool description emphasizing the policy limit.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-5|Task 1.5 — Agent SDK hooks for tool call interception]]
>
> When deterministic compliance is required, hooks provide guaranteed enforcement that prompt-based guidance cannot. Options B, C, and D all rely on probabilistic LLM compliance, which has a non-zero failure rate and is insufficient when financial policy is at stake. Review-after-the-fact (C) does not prevent the violation.

#### Question 4

A customer sends a single message saying: "My order #1234 arrived damaged, my subscription double-charged this month, and I need my shipping address updated." Your agent handles the first concern, acknowledges the others, and asks the customer to submit separate tickets. What approach would most effectively handle this better? ^q-4

A) Tell the agent in its system prompt to always handle only one issue per message.
B) Decompose the multi-concern request into distinct items, investigate each in parallel using shared context, then synthesize a unified resolution.
C) Immediately escalate any message containing multiple issues to a human agent.
D) Spawn three independent subagents that each handle one concern without sharing any context.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-4|Task 1.4 — Implement multi-step workflows with enforcement and handoff]]
>
> The guide explicitly calls for decomposing multi-concern requests, investigating each in parallel with shared context, then synthesizing a unified response. Option A produces worse customer experience. Option C escalates cases the agent could resolve. Option D loses the shared context needed for a coherent resolution.

#### Question 19

Your agent is processing a multi-step refund. After the first tool call (`get_customer`), your code returns Claude's response to the user without calling any further tools. The user complains the refund was never actually processed. What is wrong with your loop control? ^q-19

A) You should be checking `stop_reason` — if it equals `"tool_use"` continue executing tools and feed results back; only terminate when `stop_reason` is `"end_turn"`.
B) You should set `max_iterations` to a higher value so the loop continues longer.
C) You should parse the response text for keywords like "complete" or "done" to determine when to stop.
D) You should use a separate validator agent to determine whether more steps are needed.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-1|Task 1.1 — Design and implement agentic loops]]
>
> The agentic loop is driven by `stop_reason`: `"tool_use"` means execute the requested tool, append the result, and send back to Claude; `"end_turn"` means Claude is done. Parsing text (C) is a documented anti-pattern. Iteration caps (B) are stopgaps, not control flow. A validator agent (D) is over-engineered when the protocol already provides the signal.

#### Question 20

A customer support session spans 12 turns. The customer reports that order #5821 ($147.50) shipped damaged on Jan 14 with a $90 partial refund agreed, and separately mentions a billing dispute on subscription #B442 ($24.99/month) that double-charged on Feb 1. By turn 10 the agent is referring to "the order" and "the subscription" without specific numbers, and gets the refund amount wrong. What is the right architectural fix? ^q-20

A) Train the agent to verbally repeat all numbers in every response.
B) Increase `max_tokens` to give the agent more room to recall details.
C) Maintain a structured case-facts block (order IDs, amounts, dates, agreed actions) included verbatim in each prompt outside the conversation summary.
D) Cap conversations at six turns and force a restart with a summary handoff.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-5-1|Task 5.1 — Manage conversation context]]
>
> Progressive summarization compresses numerical values and identifiers into vague references. The documented fix is extracting facts into a persistent layer included in each prompt, separate from summarized history. Option A relies on probabilistic recall. Option B treats the symptom. Option D loses continuity.

#### Question 21

A customer types: "I want to speak to a human RIGHT NOW. I do not want to deal with the AI on this — please transfer me." Their issue (a missing item from a recent order) is straightforward and could be resolved by your refund tool. What should the agent do? ^q-21

A) Immediately escalate to a human as the customer explicitly demanded.
B) Try to resolve autonomously first because the issue is simple, then escalate only if that fails.
C) Run sentiment analysis to determine whether the demand is sincere before deciding.
D) Decompose the request and start the `lookup_order` tool flow regardless.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-5-2|Task 5.2 — Escalation and ambiguity resolution]]
>
> The guide explicitly states: honor explicit customer requests for human agents immediately without first attempting investigation. The customer has stated their preference. Option B overrides their stated wish. Option C uses sentiment for case complexity, which is documented as unreliable. Option D ignores the customer's expressed will.

#### Question 22

Your `process_refund` tool currently returns "Operation failed" on any failure. The agent then retries the operation regardless of the failure type, occasionally hammering the backend during outages and confusing customers about validation failures. What is the right shape for tool error responses? ^q-22

A) Return a uniform error string and have the agent decide whether to retry based on text parsing.
B) Return structured error metadata: `errorCategory` (transient/validation/permission/business), `isRetryable` boolean, and a human-readable description.
C) Catch all errors silently and return a successful empty result; the agent can re-query if it does not see the expected effect.
D) Throw exceptions and let the agent runtime handle them generically.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-2|Task 2.2 — Implement structured error responses for MCP tools]]
>
> Structured error metadata lets the agent make appropriate recovery decisions: retry transient errors, surface business errors to the customer, never retry validation errors. Generic strings (A) force the agent to guess. Silent suppression (C) is a documented anti-pattern. Generic exception handling (D) loses category information.

#### Question 23

Your customer service agent must verify a customer's identity via `get_customer` before processing any refund or account change. You added "always call `get_customer` first" to the system prompt, but logs show the agent occasionally skips this when the customer provides convincing context. What is the correct enforcement mechanism? ^q-23

A) Add stronger language to the system prompt: "MANDATORY: always verify identity first."
B) Add few-shot examples showing the agent always calling `get_customer` first.
C) Add a programmatic prerequisite hook that blocks `process_refund` and `account_update` calls until `get_customer` has returned a verified customer ID.
D) Move identity verification logic into the `process_refund` tool itself.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-1-4|Task 1.4 — Multi-step workflows with enforcement]]
>
> When deterministic ordering is required for critical business logic, programmatic prerequisites are the only reliable enforcement. Options A and B are probabilistic — they reduce but do not eliminate failure. Option D conflates two concerns and would still permit `account_update` without verification.

#### Question 24

Your agent has two tools: `cancel_order` ("Cancels a customer order") and `refund_order` ("Refunds a customer order"). Logs show the agent routinely confuses them — calling cancel when the customer wanted a refund on a delivered item, or refund when they wanted to stop a not-yet-shipped order. What is the most effective first fix? ^q-24

A) Combine the two tools into a single tool with an `action` parameter.
B) Rewrite each description to specify the precondition (order status), the effect, and when to choose this tool over the other.
C) Add a pre-tool routing classifier that determines the right tool from the customer message.
D) Delete one tool to force the agent to use the remaining one.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-1|Task 2.1 — Design effective tool interfaces]]
>
> Tool descriptions are the primary mechanism for selection. Differentiating each description with preconditions, effects, and discriminating criteria addresses the root cause directly. Option A may work but is more invasive than the first step warrants. Option C bypasses the model's natural routing capability. Option D is destructive.

---

### Scenario 2: Code Generation with Claude Code

#### Question 5

A new team member joins and reports that Claude Code is not applying your team's coding standards. The other team members have no issues. You inspect the senior engineer's setup and find the standards are written in `~/.claude/CLAUDE.md`. What is the fix? ^q-5

A) Ask the new team member to copy the senior engineer's `~/.claude/CLAUDE.md` into their own home directory.
B) Move the standards from `~/.claude/CLAUDE.md` to a project-level `CLAUDE.md` committed to version control.
C) Symlink the user-level `CLAUDE.md` into the project repo so every team member gets the same file.
D) Document the standards in the team wiki and ask engineers to paste them into each new Claude Code session.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-1|Task 3.1 — Configure CLAUDE.md files with appropriate hierarchy]]
>
> User-level `CLAUDE.md` applies only to that user and is not shared via version control. Standards need to live in project-level configuration that every teammate pulls automatically. Option A creates brittle, drift-prone copies. Option C is a hack that breaks across OSes. Option D defeats the purpose of automated configuration.

#### Question 6

You need to add a single null check to a utility function that is throwing an error when passed undefined input. The stack trace is clear and the fix is localized. Which approach is most appropriate? ^q-6

A) Enter plan mode so Claude can design multiple possible implementations before committing.
B) Use direct execution; the change is well-scoped and the root cause is known.
C) Use plan mode for safety; direct execution always carries more risk.
D) Use the Explore subagent to map the codebase before touching the function.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-4|Task 3.4 — Plan mode vs direct execution]]
>
> Direct execution is appropriate for simple, well-scoped changes with a clear root cause. Plan mode is designed for large-scale changes, architectural decisions, or multi-file modifications. Overusing plan mode adds friction without value. Option D is overkill for a single-file fix.

#### Question 7

Your monorepo has six packages, each with different testing frameworks, linting rules, and API conventions. Your root `CLAUDE.md` has grown to 2,000 lines and is becoming hard to maintain. What is the most scalable organization? ^q-7

A) Split conventions into topic-specific files under `.claude/rules/` and use `@import` in each package's `CLAUDE.md` to reference only the files relevant to that package.
B) Delete the root `CLAUDE.md` and rely on Claude's codebase inspection to infer conventions.
C) Move all conventions into a single `SKILL.md` file and invoke it as a skill for every task.
D) Keep everything in the root `CLAUDE.md` but reorganize with clearer section headers.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-3-1|Task 3.1 — CLAUDE.md modular organization]]
>
> `@import` plus modular rule files is the documented pattern for scaling conventions across multi-package repos. Each package's `CLAUDE.md` pulls only the relevant standards. Option B abandons explicit guidance. Option C misuses skills — skills are on-demand, while standards should be always-loaded. Option D doesn't actually reduce the monolith.

#### Question 8

You want a custom skill that analyzes codebase dependencies. The analysis produces pages of verbose output that should not clutter the main conversation. Which `SKILL.md` frontmatter option addresses this? ^q-8

A) `allowed-tools: Read, Grep`
B) `context: fork`
C) `argument-hint: <package-name>`
D) `visibility: private`

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-2|Task 3.2 — Custom slash commands and skills]]
>
> `context: fork` runs the skill in an isolated sub-agent context so its verbose output does not pollute the main conversation. The skill returns only a summary. Option A restricts tool access but doesn't isolate context. Option C prompts for arguments. Option D is not a real frontmatter field.

#### Question 25

Your codebase has API conventions that should apply to files under `src/api/`, testing conventions that apply to any file matching `**/*.test.*`, and a Terraform module style guide that applies to `terraform/**/*.tf`. Where do you put each set of conventions? ^q-25

A) All in the root `CLAUDE.md` under section headers; rely on Claude to infer relevance.
B) In `.claude/rules/*.md` files with YAML frontmatter `paths` fields containing the relevant glob patterns for each.
C) In subdirectory-level `CLAUDE.md` files at `src/api/`, every test directory, and `terraform/`.
D) In `.claude/skills/` as on-demand skills the developer must invoke per file.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-3|Task 3.3 — Path-specific rules for conditional convention loading]]
>
> `.claude/rules/` with YAML frontmatter paths supports glob patterns including patterns like `**/*.test.*` that span the codebase. Subdirectory `CLAUDE.md` (C) cannot easily handle globs that span directories. Section headers in a monolith (A) rely on inference. Skills (D) are on-demand, not always-applied.

#### Question 26

Your team is migrating from REST to GraphQL across roughly 70 files. The migration involves choosing between two server frameworks (each with different deployment implications), restructuring shared types, and updating all client call sites. Should you use plan mode or direct execution? ^q-26

A) Direct execution with comprehensive upfront instructions describing the desired end state.
B) Plan mode for the architectural decisions and dependency mapping; switch to direct execution for implementation once the plan is approved.
C) Direct execution; switch to plan mode only if you encounter unexpected complexity.
D) Plan mode for everything from start to finish — never use direct execution for migrations.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-4|Task 3.4 — Plan mode vs direct execution]]
>
> Plan mode is designed for large-scale changes, multiple valid approaches, and architectural decisions — all present here. Use plan mode for investigation and design, then direct execution for the planned implementation. Option A risks costly rework when dependencies are discovered late. Option C ignores stated complexity. Option D is overkill in the implementation phase.

#### Question 27

Your team has two new conventions: (1) every Pull Request description should follow a specific multi-section template, and (2) when running database migrations the engineer should follow a five-step checklist that involves dry-run, peer review, and rollback verification. The PR template is needed every time anyone writes a PR description; the migration checklist is invoked maybe twice a month. Where should each live? ^q-27

A) Both in `CLAUDE.md`.
B) Both as skills in `.claude/skills/`.
C) PR template in `CLAUDE.md` (always-loaded universal standard); migration checklist as a skill in `.claude/skills/` (on-demand task workflow).
D) PR template as a slash command; migration checklist in `CLAUDE.md`.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-3-2|Task 3.2 — Choose between skills and CLAUDE.md]]
>
> `CLAUDE.md` is for always-loaded universal standards — applicable every session. Skills are for on-demand task-specific workflows — invoked only when relevant. The PR template applies every PR. The migration checklist is rare. Option A pollutes context with rarely-needed content. Option B forces explicit invocation of standards that should be ambient.

#### Question 28

You ask Claude to implement a caching layer for an unfamiliar third-party API. Claude produces working code, but on review the cache invalidation logic is naive and will not handle the API's eventual-consistency model. You hadn't mentioned the consistency model in your prompt. What technique would have prevented this? ^q-28

A) Prompt Claude to "think harder" before implementing.
B) The interview pattern: ask Claude to ask clarifying questions before implementing, surfacing considerations like cache invalidation strategy and failure modes.
C) Generate three implementations and pick the best one.
D) Provide a completed reference implementation in the prompt.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-5|Task 3.5 — Apply iterative refinement techniques]]
>
> The interview pattern is documented for exactly this scenario — having Claude surface design considerations the developer may not have anticipated before implementing. Option A is vague. Option C is wasteful. Option D defeats the purpose of asking.

#### Question 29

You have completed a 90-minute Claude Code session analyzing your codebase's payment flow. You now want to evaluate two different refactoring strategies independently — without each one polluting the other's context. What is the right session management approach? ^q-29

A) Run `/compact` to clear context, then explore each approach sequentially.
B) Use `fork_session` twice to create two independent branches from the shared analysis baseline, exploring each refactor in its own fork.
C) Use `--resume <session-name>` to load the analysis, then alternate between strategies in the same conversation.
D) Start two completely fresh sessions and re-analyze the codebase in each.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-7|Task 1.7 — Manage session state, resumption, and forking]]
>
> `fork_session` creates independent branches from a shared baseline, perfect for divergent exploration. Option A loses the baseline analysis. Option C lets each strategy's reasoning contaminate the other. Option D wastes the analysis work.

#### Question 30

You start a Claude Code session to refactor a large legacy module. Within 15 minutes the session is producing inconsistent answers because the context fills with verbose grep results, file dumps, and tracing output from your exploration. You have barely started the actual refactor. What should you do differently? ^q-30

A) Start a new session and immediately begin the refactor without exploration.
B) Use the Explore subagent for the discovery phase; it isolates verbose output and returns clean summaries to the main session.
C) Increase the model's `max_tokens` to fit more context.
D) Save tracing output to disk and reference the file paths in the main prompt.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-4|Task 3.4 — Plan mode and the Explore subagent]]
>
> The Explore subagent is documented for this exact failure mode — it runs verbose discovery in isolation and returns clean summaries, preventing context exhaustion. Option A loses the necessary exploration. Option C does not fix attention dilution. Option D adds friction without isolation.

---

### Scenario 3: Multi-Agent Research System

#### Question 9

Your research coordinator returns complete reports, but reviewers consistently note that findings miss the latest sources on the topic. Investigation shows the coordinator calls the web search agent once, the analysis agent once, and synthesis once — in a fixed sequence — regardless of topic complexity. What is the most effective improvement? ^q-9

A) Add an iterative refinement loop where the coordinator evaluates synthesis output for gaps and re-delegates to search and analysis with targeted queries until coverage is sufficient.
B) Increase the web search agent's default result count from 10 to 50 to widen coverage.
C) Switch the synthesis agent to a larger model.
D) Run the pipeline twice in parallel and merge the outputs.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-2|Task 1.2 — Coordinator-subagent iterative refinement]]
>
> Iterative refinement — coordinator detects gaps, re-delegates targeted queries, re-synthesizes — is the documented pattern for this failure mode. Option B adds noise without addressing the missing feedback loop. Options C and D don't fix the structural lack of gap detection.

#### Question 10

Your multi-agent system needs to search three distinct subtopics to reduce latency. Currently the coordinator emits one `Task` tool call, waits for the result, then emits the next, serially. Total latency is roughly 45 seconds. What should you change? ^q-10

A) Switch to a faster model to reduce per-call latency.
B) Have the coordinator emit all three `Task` tool calls in a single response so the subagents execute in parallel.
C) Merge the three subagents into one subagent with a broader scope to reduce coordination overhead.
D) Pre-compute common search results in a cache layer.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-3|Task 1.3 — Parallel subagent spawning]]
>
> Parallel execution requires emitting multiple `Task` tool calls in a single coordinator response rather than across separate turns. Option A shaves per-call time but preserves the serial pattern. Option C loses specialization benefits. Option D solves a different problem.

#### Question 11

Your synthesis agent combines findings from the web search agent and the document analysis agent, but final reports drop source attribution. Readers cannot trace which claim came from which source. What is the root-cause fix? ^q-11

A) Instruct the synthesis agent in the system prompt to always cite sources.
B) Require upstream subagents to output structured claim-source mappings (source URL, document name, excerpt, publication date) that the synthesis agent must preserve through combination.
C) Post-process synthesis output with a separate agent that adds citations.
D) Have the synthesis agent run its own web searches to relocate the sources it needs.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-6|Task 5.6 — Preserve information provenance]]
>
> Provenance is preserved structurally — upstream agents emit claim-source mappings, downstream agents are required to carry them through. Option A is probabilistic. Option C adds an agent that does not know ground truth. Option D duplicates work, adds latency, and still isn't reliable.

#### Question 12

Your web search subagent occasionally times out on complex queries. Currently it returns an empty result set marked as successful, and the synthesis agent produces reports missing large sections with no indication anything failed. How should you fix error propagation? ^q-12

A) Have the subagent catch timeout exceptions and continue to return empty results as success.
B) Return structured error context (failure type, attempted query, partial results, suggested alternatives) so the coordinator can retry, try an alternative, or proceed with annotated gaps.
C) Terminate the entire research workflow on any subagent timeout.
D) Disable the web search agent's timeout and allow it to run indefinitely.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-3|Task 5.3 — Error propagation across multi-agent systems]]
>
> Silent suppression (A) is an anti-pattern. Full workflow termination (C) prevents recovery. Structured error context enables intelligent coordinator decisions. Option D introduces unbounded runtime risk.

#### Question 31

Your subagents are starting to communicate directly with each other to "speed things up" — the document analysis agent calls the web search agent without going through the coordinator. Two weeks in, you are seeing inconsistent error handling, duplicate work, and no observability into the workflow. What is the architectural fix? ^q-31

A) Have all subagent communication route through the coordinator (hub-and-spoke), with the coordinator handling errors, deduplication, and observability.
B) Add OpenTelemetry to the subagent-to-subagent calls.
C) Pick one subagent as the lead that other subagents report to.
D) Maintain a shared message queue that all subagents publish and subscribe to.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-1-2|Task 1.2 — Hub-and-spoke architecture]]
>
> Hub-and-spoke is the documented pattern — coordinator manages all inter-subagent communication, error handling, and information routing. Direct subagent-to-subagent links create the exact problems described. Option B treats observability symptoms only. Options C and D create alternative patterns that introduce different coordination problems.

#### Question 32

Your synthesis agent receives findings from web search and document analysis as paragraphs of prose without explicit attribution. The final report cannot trace 30% of factual claims back to any specific source. What is the right fix? ^q-32

A) Add a citation generator that runs after synthesis to add references.
B) Require upstream subagents to emit findings as structured records: claim, evidence excerpt, `source_url`, `document_name`, `publication_date` — and require the synthesis agent to preserve `source_url` and `document_name` on every claim it includes.
C) Train reviewers to accept that some claims will not have traceable sources.
D) Ask the synthesis agent in the system prompt to cite sources where possible.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-6|Task 5.6 — Claim-source mapping preservation]]
>
> Provenance must be preserved structurally — upstream emits structured mappings, downstream is required to carry them through. Option A doesn't know ground truth and will fabricate. Option C is unacceptable. Option D is probabilistic; the failure mode is exactly that "where possible" gets interpreted loosely.

#### Question 33

Your research pipeline produces a report on cybersecurity incident rates. The web search agent returned a 2024 industry report stating 40% of organizations experienced an incident, while the document analysis agent extracted 27% from a peer-reviewed 2023 study. The synthesis agent reports "around 30%" without mentioning the conflict. What is the right design? ^q-33

A) Pick the more recent number (40%) automatically since it is more current.
B) Average the two values to produce a unified estimate.
C) Have the document analysis subagent surface conflicting values explicitly with source attribution; the synthesis agent must annotate conflicts (40% per Source A 2024, 27% per Source B 2023) rather than reconciling silently.
D) Have the synthesis agent discard whichever value seems like an outlier.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-5-6|Task 5.6 — Conflict annotation in multi-source synthesis]]
>
> Conflicting credible-source values should be annotated with attribution and methodological context, not silently reconciled. Temporal differences should be preserved. Options A, B, and D all collapse genuine information differences into a single misleading number.

#### Question 34

Your synthesis subagent consistently produces incomplete reports, missing key findings the web search and document analysis agents had already produced. Your coordinator code spawns synthesis with this prompt: "Synthesize the findings from prior research into a coherent report." What is wrong? ^q-34

A) The synthesis agent is hitting token limits — increase `max_tokens`.
B) Subagents do not automatically inherit parent context — the coordinator must include the prior subagents' findings explicitly in the synthesis subagent's prompt.
C) The synthesis agent needs a larger model.
D) The coordinator needs to wait longer between subagent calls.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-3|Task 1.3 — Subagent context passing]]
>
> Subagent context must be explicitly provided in the prompt — they don't share memory between invocations. The synthesis agent here has no actual findings to synthesize because none were passed in. Options A, C, and D address different problems.

#### Question 35

You are three hours into a complex codebase exploration session. The agent starts referring to "the typical pattern" rather than specific class names you established earlier in the session, and answers contradict findings from the first hour. What technique would have prevented this? ^q-35

A) Running the exploration more quickly.
B) Maintaining a scratchpad file where the agent records key findings (class names, dependencies, traced flows), and referencing it in subsequent prompts to counteract context degradation.
C) Using a faster model.
D) Restarting the session every 30 minutes.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-4|Task 5.4 — Manage context in large codebase exploration]]
>
> Scratchpad files persist key findings across context boundaries and counteract the documented degradation pattern in extended sessions. Option D loses progress. Options A and C don't address context drift.

#### Question 36

Your synthesis subagent has access to the full set of tools (web search, document load, fact verify, citation lookup, image analysis, code execute). Logs show it occasionally invokes web search and document load — duplicating work the upstream agents already did, and producing inconsistent context. What is the fix? ^q-36

A) Add prompt instructions telling the synthesis agent not to use those tools.
B) Restrict the synthesis agent's tool set to only what its role requires (synthesis-relevant tools), removing search and document load. If verification is needed, route through the coordinator or provide a scoped `verify_fact` tool.
C) Increase the synthesis agent's temperature so it is less likely to repeat upstream work.
D) Have the coordinator intercept and block search and load calls from the synthesis agent.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-3|Task 2.3 — Scoped tool access by role]]
>
> Scoped tool access by role is the documented pattern. Agents with tools outside their specialization tend to misuse them. Option A is probabilistic. Option C is unrelated. Option D treats the symptom rather than the configuration root cause.

---

### Scenario 4: Developer Productivity with Claude

#### Question 13

You need to find every file in the codebase matching the pattern `**/*.test.tsx` to apply a convention change. Which built-in tool is the correct choice? ^q-13

A) Read — load each directory in turn and scan for name matches.
B) Grep — search file contents for the substring `".test.tsx"`.
C) Glob — match files by path pattern.
D) Bash — shell out to `find` or `fd`.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-2-5|Task 2.5 — Select and apply built-in tools]]
>
> Glob is purpose-built for file path pattern matching. Grep is for file content. Read is for loading file contents, not discovery. Bash would technically work but is not the purpose-built selection.

#### Question 14

You have an experimental MCP server that integrates with your personal GitHub account for private exploration. You do not want it shared with teammates when they clone the repo. Where should you configure it? ^q-14

A) `.mcp.json` at the project root.
B) `~/.claude.json` in your user home directory.
C) An environment variable exported from your shell profile.
D) `.claude/mcp-personal.json` (a new subdirectory).

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-4|Task 2.4 — MCP server scoping]]
>
> User-scoped MCP servers go in `~/.claude.json`. Project-scoped shared servers go in `.mcp.json`. Environment variables are for credentials within `.mcp.json`, not server configuration. Option D is fabricated.

#### Question 15

You try to use Edit to change a line in a large file, but the operation fails because the target text appears in multiple locations. What is the documented fallback? ^q-15

A) Retry the Edit with a longer anchor string to make the match unique, then use Read plus Write as a fallback when uniqueness cannot be achieved.
B) Ignore the failure and assume the edit succeeded.
C) Delete the file and ask Claude to regenerate it from scratch.
D) Use Bash and `sed` to perform the substitution.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-2-5|Task 2.5 — Built-in tool selection and fallback]]
>
> When Edit fails due to non-unique text matches, the documented fallback is Read plus Write — load the full file, modify in-memory, write it back. Extending the anchor first is the cheapest attempt. Option C is destructive. Option D is not the documented path.

#### Question 37

Your team needs every developer to have access to the same Jira MCP server with team-specific authentication, but you cannot commit the API token to version control. What is the correct configuration? ^q-37

A) Commit the token as part of `.mcp.json` and rotate it regularly.
B) Configure the MCP server in `.mcp.json` using environment variable expansion (e.g., `${JIRA_TOKEN}`); each developer sets the variable in their local shell from a shared secrets manager.
C) Have each developer add the Jira MCP server to their personal `~/.claude.json` with their own token.
D) Use a third-party secret-distribution MCP server that proxies the credentials.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-4|Task 2.4 — MCP server scoping with credentials]]
>
> `.mcp.json` with environment variable expansion is the documented pattern: server config is shared via VCS, secrets are sourced from each developer's environment. Option A leaks credentials. Option C breaks the shared-team-tooling requirement. Option D is over-engineered.

#### Question 38

Your team needs an MCP integration with Jira. Your engineers are excited to build a custom MCP server. The popular open-source `@atlassian/jira-mcp` server already exists, covers 90% of your needs, and has community maintainers. What should you do? ^q-38

A) Build the custom server because it gives you full control.
B) Use the community MCP server for the standard 90%; build a small custom server only for the team-specific 10% the community server doesn't cover.
C) Fork the community server and rewrite it with your team's preferred patterns.
D) Use only the community server even when it doesn't cover all requirements; work around the gaps in prompts.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-4|Task 2.4 — Community vs custom MCP servers]]
>
> The guide recommends choosing existing community MCP servers over custom for standard integrations, reserving custom for team-specific workflows. Option A duplicates community work. Option C creates fork maintenance burden. Option D leaves real needs unmet.

#### Question 39

You need to understand how authentication flows through a 200,000-line monorepo you have never seen before. What is the documented incremental pattern? ^q-39

A) Read every file in the `auth/` directory upfront to build complete context.
B) Use Grep to find entry points (login route handlers, JWT verification calls), then Read to follow specific imports and trace flows from there.
C) Run a static analysis tool externally and feed Claude the output.
D) Ask Claude to summarize each file in the repository one at a time.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-5|Task 2.5 — Incremental codebase building]]
>
> Building codebase understanding incrementally — Grep for entry points, Read to follow imports — preserves context and avoids attention dilution. Option A floods context with irrelevant code. Option C bypasses agent exploration. Option D is wasteful.

#### Question 40

Your codebase has a single MCP tool called `analyze_document`. Logs show the agent uses it for everything: extracting bullet points, summarizing executive summaries, verifying claims against the document, answering arbitrary questions. Outputs are inconsistent in shape and quality. What is the documented fix? ^q-40

A) Add a longer description to `analyze_document` explaining all its modes.
B) Add an `action` parameter to `analyze_document` that switches behavior based on the value.
C) Split into purpose-specific tools: `extract_data_points`, `summarize_content`, `verify_claim_against_source` — each with defined input/output contracts.
D) Replace the tool with a more general LLM-call tool.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-2-1|Task 2.1 — Splitting generic tools]]
>
> Splitting generic tools into purpose-specific ones with defined contracts is the documented pattern. Each tool gets a focused description and predictable output. Option A doesn't fix structural ambiguity. Option B retains contract ambiguity. Option D goes the wrong direction.

#### Question 41

A user asks Claude Code to "add comprehensive tests to a legacy codebase." Should you decompose this with prompt chaining (fixed sequence) or dynamic decomposition? ^q-41

A) Prompt chaining: write a fixed sequence — analyze, then identify gaps, then write tests for each gap.
B) Dynamic decomposition: first map structure, identify high-impact areas, then create a prioritized plan that adapts as test dependencies and integration points are discovered.
C) Single-pass execution: have Claude write tests while exploring.
D) Parallel decomposition: fan out to multiple agents to write tests for separate files simultaneously.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-6|Task 1.6 — Task decomposition strategies]]
>
> Open-ended tasks where the right subtasks emerge from exploration are documented as dynamic decomposition territory. Option A imposes premature structure. Option C overloads context. Option D may produce inconsistent test patterns without first establishing priorities.

#### Question 42

Your agent needs visibility into what is available in a 5,000-document knowledge base. Currently it issues exploratory tool calls like "search for X," "search for Y" — burning many turns and tokens. The MCP server hosting the knowledge base is yours. What should you add? ^q-42

A) A bigger search index.
B) An MCP resource exposing a content catalog (document index with titles, summaries, IDs) the agent can read once to know what exists.
C) More search tools with different ranking algorithms.
D) Increase the agent's `max_tokens` to allow more exploratory searches.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-2-4|Task 2.4 — MCP resources for content catalogs]]
>
> MCP resources are documented for exactly this purpose — exposing content catalogs (issue summaries, doc hierarchies, schemas) to reduce exploratory tool calls. The agent can read the catalog once and target searches more precisely. Options A, C, D increase capacity without addressing discovery cost.

---

### Scenario 5: Claude Code for Continuous Integration

#### Question 16

A single Claude Code session generates a complex refactor and then reviews its own output for issues. The review consistently approves code that later fails in production. What is the most effective fix? ^q-16

A) Instruct Claude during the review step to think harder and be more skeptical.
B) Run the review in a second independent Claude instance without the generator's reasoning context.
C) Extend `max_tokens` on the review step to allow longer analysis.
D) Enable extended thinking during the review step.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-4-6|Task 4.6 — Multi-instance and multi-pass review]]
>
> A model retains reasoning context from generation in the same session, making it unlikely to question its own prior decisions. Independent review instances catch subtle issues the generator rationalized away. Options A, C, D don't address the shared-context root cause.

#### Question 17

Your CI pipeline runs Claude Code for PR review and posts comments as inline annotations. The post-processing step frequently fails to parse Claude's output because the format varies across runs. What is the best fix? ^q-17

A) Write a more permissive parser using regex with many fallback patterns.
B) Use `--output-format json` together with `--json-schema` to enforce a structured schema on the output.
C) Append "RESPOND IN VALID JSON" at the end of every prompt.
D) Run Claude Code twice per PR and diff the outputs for consistency.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-6|Task 3.6 — Integrate Claude Code into CI/CD pipelines]]
>
> The documented CLI flags enforce structured output at the protocol level, eliminating format drift. Option A delays the root issue. Option C is probabilistic. Option D doubles cost without guaranteeing parseable output.

#### Question 18

You have a nightly job that analyzes roughly 2,000 PRs for technical debt signals. The report is read the following morning. Your manager wants to cut the API bill. What is the appropriate approach? ^q-18

A) Use the Message Batches API with a submission window that guarantees completion before morning.
B) Use the real-time API but cache responses aggressively across PRs.
C) Split the 2,000 PRs into 20 batches of 100 and submit sequentially to the real-time API.
D) Reduce the analysis scope to fewer PRs to cut cost directly.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-4-5|Task 4.5 — Design efficient batch processing strategies]]
>
> Overnight, latency-tolerant workloads are the canonical fit for the Message Batches API — 50% cost savings with up to a 24-hour window. A next-morning deadline aligns with batch completion. Option B doesn't help because each PR is unique. Option C retains full per-request cost. Option D is a scope cut, not an optimization.

#### Question 43

Your CI code review prompt currently reads: "Review this PR for issues. Be conservative — only report high-confidence findings." Reviews flag noise across the board: trivial style preferences, false-positive null-check warnings, and occasional real bugs. What is the most effective change? ^q-43

A) Add "be more conservative" to the prompt.
B) Replace vague guidance with explicit categorical criteria: which issues to report (bugs, security vulnerabilities, missing error handling) versus which to skip (minor style, local patterns), with concrete code examples for each severity level.
C) Lower the model temperature to make outputs more conservative.
D) Filter the output post-hoc using regex.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-4-1|Task 4.1 — Design prompts with explicit criteria]]
>
> General instructions like "be conservative" are documented as ineffective for precision improvement. Specific categorical criteria with concrete examples define which findings count. Option A doubles down on a failed pattern. Options C and D treat symptoms.

#### Question 44

Your code review reports a potential null pointer on every nullable type access, even where the value is provably non-null based on a prior check in the same function. After three prompt revisions explaining the rule, false positives persist. What technique is most effective? ^q-44

A) Add 2-4 few-shot examples showing acceptable patterns (with the prior null check) versus genuine issues (without the check), so the model can generalize the underlying rule.
B) Remove the null-check rule from the review entirely.
C) Train a separate classifier on labeled examples.
D) Add "do not report null pointer issues" to the prompt.

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-4-2|Task 4.2 — Few-shot for false positive reduction]]
>
> Few-shot examples are the documented technique for reducing false positives while preserving the ability to catch real issues — the model generalizes from the contrast. Option B loses real findings. Option C is over-engineered. Option D throws out the rule entirely.

#### Question 45

Your CI test-generation step produces tests that don't follow your team's testing conventions: it uses a different mocking library than your codebase, doesn't follow the AAA (Arrange-Act-Assert) pattern your team prefers, and creates fixtures inline instead of using your shared factory. What is the documented way to fix this? ^q-45

A) Manually edit every generated test file before committing.
B) Document testing conventions, mock patterns, fixture factories, and AAA structure in `CLAUDE.md` so CI-invoked Claude Code picks them up.
C) Train a custom model on your team's existing tests.
D) Have a senior engineer review every CI-generated test.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-6|Task 3.6 — CLAUDE.md for CI context]]
>
> `CLAUDE.md` is the documented mechanism for providing project context to CI-invoked Claude Code: testing standards, fixture conventions, review criteria. Option A is a workaround. Option C is overkill. Option D doesn't scale.

#### Question 46

You use the Message Batches API for nightly code review. Your SLA requires findings to be available within 30 hours of commit. Batches process within 24 hours but offer no latency guarantee. How do you guarantee the 30-hour SLA? ^q-46

A) Submit each commit individually as it arrives.
B) Submit batches at 4-hour intervals — even if a batch takes the full 24 hours, total time from earliest commit in the batch (4 hours old at submission) to result is 28 hours, within SLA.
C) Reduce the batch size to a single document so processing is fast.
D) Switch to the synchronous API.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-4-5|Task 4.5 — Batch SLA window calculation]]
>
> SLA-bounded use of the batch API requires calculating submission frequency such that the worst-case sum (window duration plus 24-hour processing) stays within the SLA. The 4-hour window plus 24-hour processing equals a 28-hour worst case. Option A defeats the cost-saving purpose. Option C does too. Option D forfeits the savings.

#### Question 47

A pull request modifies 22 files spanning the auth module, the API gateway, and shared types. A single-pass review produces uneven depth (detailed comments on some files, superficial on others) and contradictory findings (flagging a pattern as bad in one file, approving identical code in another). What is the right restructuring? ^q-47

A) One review pass over the entire PR with extended thinking enabled.
B) Per-file local-analysis passes for each of the 22 files, plus a separate cross-file integration pass examining the data flow between auth, gateway, and shared types.
C) Reject the PR and require it to be split into smaller PRs before review.
D) Three independent passes over the full PR, only flagging issues that appear in 2 of 3 runs.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-1-6|Task 1.6 — Per-file vs cross-file passes]]
>
> Decomposing into per-file local passes plus a cross-file integration pass is documented to prevent attention dilution and contradictory findings. Option A doesn't address attention dilution. Option C shifts the problem. Option D suppresses real findings that emerge in single passes.

#### Question 48

Your CI re-runs Claude Code review on every push to a PR. Developers complain that the same findings appear in every iteration as new comments, even after they have addressed earlier rounds. What is the fix? ^q-48

A) Switch to running review only on the final push to main.
B) Include prior review findings in the context of subsequent runs, instructing Claude to report only new findings or still-unaddressed ones from prior reviews — not previously-resolved issues.
C) Delete prior comments before each new run.
D) Have humans manually deduplicate after each run.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-3-6|Task 3.6 — Include prior findings to avoid duplicates]]
>
> Including prior findings as context and asking the agent to report only new or still-unaddressed items is documented for exactly this iterative-review pattern. Option A loses valuable feedback during development. Option C destroys history. Option D doesn't scale.

---

### Scenario 6: Structured Data Extraction

#### Question 49

You're extracting fields from contracts including a `termination_clause_section_number` field. About 30% of contracts don't have a numbered termination clause. Your schema marks the field as required, and the model fabricates plausible-looking section numbers when the value isn't present. What is the right schema design? ^q-49

A) Make `termination_clause_section_number` nullable so the model can return null when the field is absent.
B) Lower the temperature so the model is less creative.
C) Add a post-extraction validator that compares against a list of known section numbers.
D) Instruct the model in the prompt: "Never invent values."

> [!success]- Reveal answer
> **Answer: A** · [[cca_exam_guide#^task-4-3|Task 4.3 — Nullable fields prevent fabrication]]
>
> Required fields create structural pressure for the model to satisfy the schema — including by fabricating. Nullable fields permit honest absence. Option B doesn't address the structural incentive. Option C catches symptoms but doesn't prevent fabrication. Option D is probabilistic.

#### Question 50

Your extraction pipeline returns a structured invoice with fields `{line_items[], total, subtotal, tax}`. Validation fails because line items don't sum to the subtotal in some cases. What is the right retry pattern? ^q-50

A) Retry with the same prompt and hope for a better result.
B) Retry with the original document, the failed extraction, and the specific validation error ("line items sum to $87.50 but subtotal is $90.00 — please re-extract"). The model can use targeted feedback to self-correct.
C) Drop the validation and accept whatever the model produces.
D) Add a separate calculator agent that recomputes values from line items.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-4-4|Task 4.4 — Retry with error feedback]]
>
> Retry-with-error-feedback is the documented pattern: send the original input, the failed output, and the specific validation error. The model uses targeted feedback for self-correction. Option A wastes calls. Option C abandons quality. Option D may overcorrect.

#### Question 51

Your pipeline keeps failing on a `manufacturer_address` field for certain product spec sheets. After five retries with error feedback, the field is still null. What is the most likely explanation, and how should the pipeline respond? ^q-51

A) The model needs more retries — set retry count to ten.
B) The model needs a higher temperature for more creative answers.
C) Likely the address simply isn't in the source document. The pipeline should detect absent-information failures (versus format/structure errors) and route to human review or accept null without further retries.
D) The model is broken for this field name — rename it to `manufacturer_addr`.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-4-4|Task 4.4 — Limits of retry]]
>
> The guide explicitly states: retries are ineffective when required information is absent from the source. Pipelines should distinguish absent-info failures from format/structural errors. Options A and B compound the problem. Option D mistakes the cause.

#### Question 52

Your extraction pipeline reports 96% accuracy on a labeled validation set. The compliance team plans to reduce human review to 10% of high-confidence outputs. What ongoing measurement should you put in place? ^q-52

A) Re-evaluate accuracy quarterly using the same validation set.
B) Implement stratified random sampling of high-confidence outputs across document types and field types, reviewing the sample to detect novel error patterns and per-segment accuracy drift.
C) Trust the 96% number indefinitely.
D) Have the model self-report confidence and use that as the only ongoing signal.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-5|Task 5.5 — Stratified sampling for confidence calibration]]
>
> Stratified sampling across segments is documented to detect novel error patterns and prevent aggregate metrics from masking poor performance on specific types. Option A misses pattern drift. Option C is dangerous. Option D doesn't validate calibration.

#### Question 53

Your extraction pipeline has multiple tools: `extract_metadata`, `enrich_with_external_data`, `validate_compliance`. You need to guarantee that `extract_metadata` is called first on every document, before any enrichment. What is the right `tool_choice` configuration? ^q-53

A) `tool_choice: "auto"` — let the model decide.
B) `tool_choice: "any"` — guarantee some tool is called, model picks.
C) `tool_choice: {"type": "tool", "name": "extract_metadata"}` for the first turn, then process subsequent steps in follow-up turns.
D) Combine all three tools into a single mega-tool.

> [!success]- Reveal answer
> **Answer: C** · [[cca_exam_guide#^task-4-3|Task 4.3 — tool_choice forced selection]]
>
> Forced tool selection is documented for ensuring a specific tool is called first, with subsequent steps handled in follow-up turns. Option A allows the model to skip `extract_metadata`. Option B doesn't guarantee which tool. Option D loses schema separation.

#### Question 54

Your extraction pipeline outputs a single overall confidence score per document. You want to route only low-confidence extractions to human review. Currently, half the documents flagged for review turn out to be correct, while many actual errors slip through with high overall confidence. What design improvement would help? ^q-54

A) Drop confidence-based routing entirely; review every document.
B) Have the model output field-level confidence scores; calibrate per-field thresholds using a labeled validation set; route extractions where ANY required field falls below its threshold.
C) Reduce the threshold for the overall score to flag more documents.
D) Increase the model temperature so confidence scores spread more.

> [!success]- Reveal answer
> **Answer: B** · [[cca_exam_guide#^task-5-5|Task 5.5 — Field-level confidence calibration]]
>
> Field-level confidence with per-field calibration is documented for routing review attention precisely. Errors typically concentrate in specific fields (low confidence on those) while overall scores average them out. Option A defeats the optimization. Option C produces more false-positive reviews. Option D doesn't address calibration.

---

## Diagnostic Matrix

After scoring, mark every question you missed. If two or more questions tied to the same task statement come back wrong, that task is a study target. Update [[cca_progress]] when you finish a round.

| Task | What it tests | # Qs | Question Numbers |
|------|---------------|------|------------------|
| 1.1 | Design and implement agentic loops | 1 | [[#^q-19\|19]] |
| 1.2 | Coordinator-subagent iterative refinement | 2 | [[#^q-9\|9]], [[#^q-31\|31]] |
| 1.3 | Parallel subagent spawning | 2 | [[#^q-10\|10]], [[#^q-34\|34]] |
| 1.4 | Multi-step workflows with enforcement and handoff | 2 | [[#^q-4\|4]], [[#^q-23\|23]] |
| 1.5 | Agent SDK hooks for tool call interception | 1 | [[#^q-3\|3]] |
| 1.6 | Task decomposition strategies | 2 | [[#^q-41\|41]], [[#^q-47\|47]] |
| 1.7 | Manage session state, resumption, and forking | 1 | [[#^q-29\|29]] |
| 2.1 | Design effective tool interfaces | 2 | [[#^q-24\|24]], [[#^q-40\|40]] |
| 2.2 | Implement structured error responses for MCP tools | 1 | [[#^q-22\|22]] |
| 2.3 | Distribute tools appropriately across agents | 2 | [[#^q-1\|1]], [[#^q-36\|36]] |
| 2.4 | MCP server scoping | 4 | [[#^q-14\|14]], [[#^q-37\|37]], [[#^q-38\|38]], [[#^q-42\|42]] |
| 2.5 | Select and apply built-in tools | 3 | [[#^q-13\|13]], [[#^q-15\|15]], [[#^q-39\|39]] |
| 3.1 | Configure CLAUDE.md files with appropriate hierarchy | 2 | [[#^q-5\|5]], [[#^q-7\|7]] |
| 3.2 | Custom slash commands and skills | 2 | [[#^q-8\|8]], [[#^q-27\|27]] |
| 3.3 | Path-specific rules for conditional convention loading | 1 | [[#^q-25\|25]] |
| 3.4 | Plan mode vs direct execution | 3 | [[#^q-6\|6]], [[#^q-26\|26]], [[#^q-30\|30]] |
| 3.5 | Apply iterative refinement techniques | 1 | [[#^q-28\|28]] |
| 3.6 | Integrate Claude Code into CI/CD pipelines | 3 | [[#^q-17\|17]], [[#^q-45\|45]], [[#^q-48\|48]] |
| 4.1 | Design prompts with explicit criteria | 1 | [[#^q-43\|43]] |
| 4.2 | Few-shot for false positive reduction | 1 | [[#^q-44\|44]] |
| 4.3 | Enforce structured output using tool use and JSON schemas | 2 | [[#^q-49\|49]], [[#^q-53\|53]] |
| 4.4 | Validation, retry, and feedback loops | 2 | [[#^q-50\|50]], [[#^q-51\|51]] |
| 4.5 | Design efficient batch processing strategies | 2 | [[#^q-18\|18]], [[#^q-46\|46]] |
| 4.6 | Multi-instance and multi-pass review | 1 | [[#^q-16\|16]] |
| 5.1 | Manage conversation context | 2 | [[#^q-2\|2]], [[#^q-20\|20]] |
| 5.2 | Escalation and ambiguity resolution | 1 | [[#^q-21\|21]] |
| 5.3 | Error propagation across multi-agent systems | 1 | [[#^q-12\|12]] |
| 5.4 | Manage context in large codebase exploration | 1 | [[#^q-35\|35]] |
| 5.5 | Human review workflows and confidence calibration | 2 | [[#^q-52\|52]], [[#^q-54\|54]] |
| 5.6 | Preserve information provenance | 3 | [[#^q-11\|11]], [[#^q-32\|32]], [[#^q-33\|33]] |

> [!warning] Weak-spot reinforcement
> Tasks 2.4 (4 qs), 3.4 (3 qs), 5.6 (3 qs), 1.4 (2 qs), 3.2 (2 qs), 5.1 (2 qs) are weighted because they were the v1 miss pattern. If you're still missing these in round 2, re-read the linked exam guide sections and rebuild the relevant exam-guide exercises.

---

## Recovery Map by Domain

If you miss multiple questions tied to one of these task statements, here's where to focus your remediation:

- **Tasks 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7** — Re-read the Claude Agent SDK reference on docs.claude.com (agentic loop, hooks, `Task` tool, `AgentDefinition`, `fork_session`). Build Exercise 1 and Exercise 4 from the official exam guide.
- **Tasks 2.1, 2.2, 2.3, 2.4, 2.5** — Re-read MCP docs (modelcontextprotocol.io plus the Anthropic MCP integration page). For 2.4 specifically, configure both a project `.mcp.json` and a personal `~/.claude.json` server in a real Claude Code session and verify both load.
- **Tasks 3.1, 3.2, 3.3, 3.4, 3.5, 3.6** — Re-take the Claude Code 101 and Claude Code in Action courses paying attention to configuration semantics. Build Exercise 2 from the exam guide. For 3.2 specifically, create a skill with all three frontmatter options (`context: fork`, `allowed-tools`, `argument-hint`) and verify each behavior.
- **Tasks 4.1, 4.2, 4.3, 4.4, 4.5, 4.6** — Re-read the Claude API tool use guide and Message Batches API docs. Build Exercise 3 from the exam guide end-to-end including the validation-retry loop and human review routing.
- **Tasks 5.1, 5.2, 5.3, 5.4, 5.5, 5.6** — These are cross-cutting patterns. Re-read the Context Management and Reliability section of the exam guide directly (pages 21-25). For 5.6 specifically, build Exercise 4 with explicit claim-source mappings preserved through synthesis.

---

*54 questions · All 6 scenarios · All 30 task statements covered. Update [[cca_progress]] after each round.*
