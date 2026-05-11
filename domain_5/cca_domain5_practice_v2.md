---
title: Claude Certified Architect — Foundations · Domain 5 Practice Exam
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
  - domain-5
status: drilling
total-questions: 30
domain: 5
domain-weight: 15
target-percent: 80
difficulty: harder-than-real-exam
---

# CCA-F Domain 5 — Practice Exam

30 questions covering Context Management & Reliability (15% of the real exam). Calibrated **harder than the real exam** — distractors include realistic-but-wrong patterns (sentiment-based escalation, record-level confidence, prose summarization of evidence) and fabricated-but-plausible SDK fields and beta header strings.

## How to use

- [ ] **Round 1** — cold attempt all 30 without checking answers
- [ ] **Round 2** — score using the collapsed callouts
- [ ] **Round 3** — re-attempt any miss after re-reading the relevant roadmap section

**Aim for 80%+ on Round 1.**

## Task statement coverage

| Task | Questions | Focus |
|------|-----------|-------|
| 5.1 | Q1-Q6 | Case facts (app-injected) vs memory tool vs auto memory |
| 5.2 | Q7-Q10 | Escalation triggers and structured handoff |
| 5.3 | Q11-Q15 | Error propagation across agent boundaries |
| 5.4 | Q16-Q21 | Scratchpads, large codebases, context isolation |
| 5.5 | Q22-Q25 | Human review, stratified sampling, field-level confidence |
| 5.6 | Q26-Q30 | Provenance, claim-source mappings, conflict annotation |

---

## Practice questions

### Task 5.1 — Case facts and persistence

#### Question 1

A customer support agent needs to know the customer's tier (Free/Pro/Enterprise) for every turn, even after conversation compaction. Where should this fact be stored? ^d5-q-1

A. In the agent's session memory, written by the memory tool the first time it's queried.
B. As **case facts** — an app-injected JSON payload that's appended to the system prompt or first user message at every turn. The app controls what's injected; it's deterministic and survives compaction by being re-injected each turn.
C. In `~/.claude/MEMORY.md` so it auto-loads at session start.
D. In a `tool_result` block that the agent caches for re-use.

> [!success]- Reveal answer
> **Correct: B**
>
> Case facts are the documented pattern for **app-controlled persistent facts**. The application injects them deterministically on every turn — independent of what the model remembers — so they're guaranteed available and never lost to compaction.
>
> **Why others are wrong:**
> - A: The memory tool is model-managed — the model decides when to write/read. For facts the application owns (like customer tier), case-facts injection is more reliable than hoping the model writes to memory.
> - C: `MEMORY.md` is Claude Code's auto-memory; it doesn't apply to API-driven agents and isn't customer-specific.
> - D: `tool_result` blocks live in conversation history and may be compacted away.

#### Question 2

The **memory tool** (introduced for the Claude API) is best characterized as: ^d5-q-2

A. A read-only cache of conversation history.
B. A persistent store written by the application; the model reads but doesn't write.
C. A built-in tool Claude Code uses for `MEMORY.md` management.
D. A model-managed protocol with `view`, `create`, `str_replace`, `insert`, `delete`, `rename` commands on a `/memories` filesystem. The model decides what to persist. The application provides the storage backend and is the source of truth; the application must implement the commands to a real filesystem or equivalent.

> [!success]- Reveal answer
> **Correct: D**
>
> The memory tool is the documented API-level mechanism. The model decides what to write and read. The application is responsible for implementing the actual storage operations behind the tool interface.
>
> **Why others are wrong:**
> - A: It's read-write, not read-only.
> - B: The model writes via the tool; the application provides storage but doesn't write content.
> - C: The Claude Code `MEMORY.md` mechanism is separate from the API's memory tool.

#### Question 3

A best-practice convention for using the memory tool is: ^d5-q-3

A. Always call `view /memories` at the start of every session/task to load relevant context before reasoning.
B. Only use the memory tool when the user explicitly asks the agent to "remember" something.
C. Use the memory tool to store conversation history; rely on it instead of multi-turn context.
D. Use the memory tool for short-term storage within a single turn; clear it between turns.

> [!success]- Reveal answer
> **Correct: A**
>
> The documented convention: call `view /memories` at session start. The model loads any persisted notes and uses them to inform reasoning. Without this, the model doesn't know what's stored.
>
> **Why others are wrong:**
> - B: Memory tool use is broader than explicit-remember requests; it's an ambient persistence mechanism.
> - C: Conversation history shouldn't be duplicated in memory; the conversation context is its own mechanism.
> - D: Memory persists *across* sessions; that's its purpose.

#### Question 4

A team's customer support agent occasionally forgets the customer's account ID partway through a conversation. The case-fact injection is set up correctly, but compaction is removing the relevant tool call later. What's the **best** intervention? ^d5-q-4

A. Increase the model's context window.
B. Disable compaction with a config flag.
C. Inject the account ID via **case facts** every turn (continuously re-injected by the application). This bypasses compaction entirely — the fact is always present in the prompt regardless of conversation history state.
D. Write the account ID to memory tool storage at session start.

> [!success]- Reveal answer
> **Correct: C**
>
> Case facts are deterministically injected by the application on every turn. They survive compaction trivially — the application re-injects them.
>
> **Why others are wrong:**
> - A: Larger context delays compaction but doesn't prevent it; eventually the issue returns.
> - B: Disabling compaction isn't generally available — and even if it were, it'd cause the conversation to error when context is exceeded.
> - D: Memory tool writes persist but the model has to *read* memory at the right moment. Case facts inject without requiring the model to remember to look.

#### Question 5

What's the **difference** between case facts and the memory tool? ^d5-q-5

A. Case facts are read-only; memory tool is read-write.
B. **Case facts** are app-injected (the application controls what gets injected, when, every turn); the **memory tool** is model-managed (the model decides what to write and read via tool calls). They solve different problems: case facts for app-owned deterministic context, memory for model-curated persistent knowledge.
C. Case facts are for production; memory tool is for development.
D. They're the same mechanism with different names.

> [!success]- Reveal answer
> **Correct: B**
>
> Two distinct mechanisms with different ownership semantics. Case facts: app-owned, deterministic. Memory tool: model-owned, model-curated.
>
> **Why others are wrong:**
> - A: Case facts aren't "read-only" in the agent's view — the agent just reads them as part of the prompt. The application writes them on every turn.
> - C: Both are production mechanisms.
> - D: They're documented as distinct.

#### Question 6

In a long-running customer support session, the conversation has compacted twice. The customer mentions "the discount code from last time." Where does the discount code most likely now live? ^d5-q-6

A. In the conversation history, perfectly preserved.
B. In auto memory at `~/.claude/MEMORY.md`.
C. The model has lost it; the agent should ask the customer to restate it.
D. If the application is correctly persisting it via case facts or memory tool: it's still available. Otherwise, the model may have lost it to compaction. The exam tests whether the **architecture** persists important facts deterministically — not whether models "remember" things across compactions on their own.

> [!success]- Reveal answer
> **Correct: D**
>
> Compaction summarizes; specific facts can be lost. Correct architecture persists important facts via app-controlled mechanisms (case facts, memory tool) rather than hoping the model retains them.
>
> **Why others are wrong:**
> - A: Compaction modifies conversation history; perfect preservation isn't guaranteed.
> - B: Auto memory is Claude Code-specific; this is an API-side scenario.
> - C: Asking the customer to restate is a fallback for missing data, not a solution to architectural compaction issues.

### Task 5.2 — Escalation

#### Question 7

An escalation hook should be triggered when: ^d5-q-7

A. The user **explicitly** requests a human (e.g., "I want to talk to a person") OR when the agent's logic detects an out-of-policy condition (e.g., refund cap exceeded, suspicious account activity, complex compliance issue).
B. The user expresses any negative sentiment (frustrated, annoyed, angry).
C. The conversation has continued for more than 10 turns.
D. The agent's confidence score on the next response is below a threshold.

> [!success]- Reveal answer
> **Correct: A**
>
> Documented criteria: explicit user request OR out-of-policy/material-risk conditions. Both are deterministic and clearly justified.
>
> **Why others are wrong:**
> - B: Sentiment-based escalation is documented as an anti-pattern. Many frustrated users are best helped by the agent continuing — escalating on sentiment alone wastes human-reviewer capacity.
> - C: Turn-count-based escalation is arbitrary; some queries legitimately take many turns.
> - D: "Confidence score" isn't an exposed signal in the API; relying on it isn't documented.

#### Question 8

A structured handoff payload to a human reviewer should include **all of the following except**: ^d5-q-8

A. The customer's original message and key prior turns from the conversation.
B. A list of actions the agent already took (tools called, with arguments and key results).
C. The model's internal logit distributions and per-token probabilities for the most recent response.
D. The explicit reason for escalation (e.g., "amount exceeds $500 policy cap; customer is calm but persistent").

> [!success]- Reveal answer
> **Correct: C**
>
> Logit distributions and per-token probabilities are internal model details, not actionable context for a human support agent. The handoff should give the human everything they need to continue the case — not LLM internals.
>
> **Why others are wrong:**
> - A: Customer context is essential.
> - B: Actions taken inform the human what's been tried.
> - D: Escalation reason guides the human to focus on the right issue.

#### Question 9

A "soft" handoff (model staying available while a human takes lead) vs a "hard" handoff (model exits the conversation entirely) differ in what way? ^d5-q-9

A. Hard handoff is for VIP customers; soft for everyone else.
B. Hard handoff requires the agent to clear the conversation and start a fresh thread; soft handoff retains conversation continuity. Use hard handoff when continuing in the same context would create confusion or compliance risk (e.g., the agent shouldn't see what the human types to the customer); soft handoff when the human can supplement the agent's work.
C. Hard handoff is faster.
D. They're the same with different names.

> [!success]- Reveal answer
> **Correct: B**
>
> Documented distinction. Hard handoff for cases where continued agent participation could confuse responsibilities, expose customer information to the wrong actor, or create audit trail issues. Soft handoff for collaborative scenarios.
>
> **Why others are wrong:**
> - A: Hard/soft isn't customer-tier-based.
> - C: Latency isn't the criterion.
> - D: They're distinct documented patterns.

#### Question 10

A customer expresses frustration ("This is ridiculous, why is this taking so long?"). The agent should: ^d5-q-10

A. Escalate to a human immediately to defuse the situation.
B. Apologize and try again with a more capable model.
C. End the conversation politely.
D. Acknowledge the frustration empathetically, continue working on resolving the issue, and only escalate if (a) the customer explicitly requests a human, or (b) the agent reaches a real escalation trigger (policy cap, irresolvable issue, etc.). Sentiment is a signal to acknowledge, not to escalate.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented decision rule for escalation is explicit-request or out-of-policy — not sentiment. Frustrated customers often need the agent to keep working effectively, not to throw the case to a human queue.
>
> **Why others are wrong:**
> - A: Sentiment-based escalation is an anti-pattern.
> - B: Switching models mid-conversation doesn't address frustration.
> - C: Ending the conversation abandons the customer and worsens the experience.

### Task 5.3 — Error propagation

#### Question 11

A subagent fails partway through its work (an MCP tool returns 500). What should the subagent return to the coordinator? ^d5-q-11

A. A structured failure payload: `{success: false, errorCategory: "transient", isRetryable: true, description: "...", partial_results: [...]}`. The coordinator can then decide whether to retry the subagent, route to a fallback, or escalate.
B. Throw an exception; the coordinator's try/catch handles it.
C. Return `null` to signal complete failure.
D. Continue silently and produce a partial result, hoping the coordinator notices the gap.

> [!success]- Reveal answer
> **Correct: A**
>
> The documented pattern: subagents return **structured failure** with category, retryability, description, and any partial results captured. The coordinator reasons over this to decide next steps.
>
> **Why others are wrong:**
> - B: Throwing crashes the coordinator loop unless every coordinator-to-subagent call is wrapped in try/catch — which is the chain-topology anti-pattern.
> - C: `null` loses category and retryability information.
> - D: Silent partial results break the coordinator's expectation that subagent results are complete or explicitly failed.

#### Question 12

A coordinator receives `{success: false, errorCategory: "permission", isRetryable: false}` from a subagent. What's the documented response? ^d5-q-12

A. Retry the subagent immediately.
B. Try a different model.
C. Don't retry (permission failures aren't retryable as-is); either fall back to an alternative subagent or escalate to a human with context about what failed and why.
D. Mark the entire pipeline failed and exit.

> [!success]- Reveal answer
> **Correct: C**
>
> Permission errors mean the subagent lacks authorization for the operation. Retrying doesn't grant authorization. The coordinator should fall back (try alternative subagent) or escalate to a human who can grant access or handle the case manually.
>
> **Why others are wrong:**
> - A: `isRetryable: false` explicitly signals don't retry.
> - B: Model isn't the issue; authorization is.
> - D: Exiting the whole pipeline is destructive; the documented pattern is graceful degradation (fallback) or escalation.

#### Question 13

A subagent's `partial_results` field on failure should include: ^d5-q-13

A. The full stack trace from the failure point.
B. Any work the subagent completed before failing (e.g., 3 out of 5 documents processed) so the coordinator can decide to use what's been done, retry only the unfinished portion, or discard everything.
C. A copy of the subagent's full conversation history.
D. The complete error category taxonomy for reference.

> [!success]- Reveal answer
> **Correct: B**
>
> Partial results enable graceful degradation. The coordinator can salvage completed work and only retry the unfinished portion, avoiding wasted re-computation.
>
> **Why others are wrong:**
> - A: Stack traces are debugging information, not coordinator decision input.
> - C: Full conversation history is observability data, not actionable for the coordinator.
> - D: Taxonomy reference is documentation, not per-call data.

#### Question 14

A coordinator should distinguish between **partial success** and **complete failure** in error propagation because: ^d5-q-14

A. They have different cost implications.
B. They affect user-facing messaging.
C. They route differently through the team's monitoring system.
D. Partial success means *some* work is salvageable; complete failure means everything done is invalid. The coordinator's next decision (use-partial, retry-incomplete-portion, escalate, fully-restart) depends on this distinction. Treating them the same loses information and may cause expensive re-work.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented rationale: different next actions. Partial success lets you use what you have and retry only the remainder; complete failure means starting over (or escalating).
>
> **Why others are wrong:**
> - A, B, C: All are real consequences but they're secondary to the routing-decision impact, which is the documented core reason.

#### Question 15

In the structured error payload, the `errorCategory` field aligns with which four documented categories? ^d5-q-15

A. `transient` (retryable network/upstream issues), `validation` (input shape problems), `permission` (authorization denial), `business` (policy rule).
B. `recoverable`, `unrecoverable`, `internal`, `external`.
C. `network`, `auth`, `validation`, `unknown`.
D. `system`, `user`, `data`, `policy`.

> [!success]- Reveal answer
> **Correct: A**
>
> Documented four-category taxonomy. Each maps to a different coordinator response: transient → retry with backoff; validation → fix input; permission → escalate or fall back; business → don't retry, communicate policy.
>
> **Why others are wrong:** B, C, D are fabricated alternative taxonomies.

### Task 5.4 — Scratchpads and large codebase strategies

#### Question 16

A team's agent is working through a 40-file refactor. The conversation compacts mid-task and the agent loses track of which files have been refactored, what conventions were established, and which are still pending. What's the **documented intervention**? ^d5-q-16

A. Increase the model's context window.
B. Disable compaction.
C. Use a **scratchpad** — write progress notes to a file in `.claude/notes/<topic>.md` (or equivalent persistent location) that survives compaction. The agent reads the scratchpad at the start of each turn (or after compaction) to recover state. Scratchpads are lossless: they store specifics that prose compaction would summarize away.
D. Switch to plan mode mid-task.

> [!success]- Reveal answer
> **Correct: C**
>
> Scratchpads are the documented pattern. Persistent notes survive compaction by living outside the context window; the agent reads them as needed.
>
> **Why others are wrong:**
> - A: Larger context delays compaction but doesn't eliminate it.
> - B: Disabling compaction isn't a documented option.
> - D: Plan mode is for designing changes before execution; it doesn't solve compaction during execution.

#### Question 17

What information is **best** kept in a scratchpad rather than relying on conversation history? ^d5-q-17

A. The full text of the customer's original message.
B. **Specifics that prose compaction would lossily summarize**: file paths refactored, exact convention decisions, rejected approaches, line-numbered observations from the codebase exploration, key test results.
C. The agent's reasoning chain-of-thought from each turn.
D. The full token-level model invocation log.

> [!success]- Reveal answer
> **Correct: B**
>
> Scratchpads capture specifics that survive verbatim — paths, decisions, observations — that prose compaction would lose by paraphrasing.
>
> **Why others are wrong:**
> - A: Customer messages should stay in conversation history; that's their natural location.
> - C: Reasoning chains are voluminous and most chain-of-thought is intermediate; final decisions belong in the scratchpad.
> - D: Invocation logs are observability data, not agent working memory.

#### Question 18

When should an agent run `/compact` proactively? ^d5-q-18

A. After every turn to keep context lean.
B. Never; let auto-compaction handle it.
C. When the agent's response latency increases noticeably.
D. At natural breakpoints (around 60% context utilization, after completing a logical chunk of work) so compaction happens on stable ground rather than mid-task. Auto-compaction kicks in around 64-75% utilization, which can interrupt mid-task work.

> [!success]- Reveal answer
> **Correct: D**
>
> Proactive `/compact` at logical breakpoints avoids mid-task interruption. The documented threshold is around 60% to stay ahead of the 64-75% auto-compact range.
>
> **Why others are wrong:**
> - A: Per-turn compaction is overkill and wastes context summarizing work that hasn't accumulated yet.
> - B: Letting auto-compaction handle it can interrupt mid-task; proactive is documented as better.
> - C: Latency isn't the trigger; context utilization is.

#### Question 19

A team is debugging a complex issue. The investigation has 8 distinct subtasks (read logs, query database, inspect config, etc.). Each subtask produces 500-2000 tokens of tool output, but only 50 tokens of *conclusion* are needed for the next subtask. What's the **best** architecture? ^d5-q-19

A. Run each subtask in a **subagent** (separate context). The coordinator receives only the 50-token conclusion from each, keeping its own context lean. Subagent context isolation prevents the 500-2000-token-per-subtask bloat from accumulating in the main session.
B. Run all subtasks sequentially in the main session and rely on compaction.
C. Use plan mode and execute everything as one big batch.
D. Increase the model's context window so all 8 subtasks fit comfortably.

> [!success]- Reveal answer
> **Correct: A**
>
> Subagent context isolation is the documented pattern for high-volume-input/low-volume-conclusion subtasks. Each subagent's verbose tool output stays in its own context; only the summary returns.
>
> **Why others are wrong:**
> - B: Running in main session accumulates 4,000-16,000 tokens of tool output across 8 subtasks; compaction then loses the specifics summarized away.
> - C: Plan mode doesn't change the context-accumulation pattern.
> - D: Context size is a stopgap, not a solution; subagent isolation is the documented architecture.

#### Question 20

The **context editing** beta feature (`context-management-2025-06-27`) with strategy `clear_tool_uses_20250919` does what? ^d5-q-20

A. Manually trims conversation history on user request.
B. Automatically compacts conversation summaries when context fills.
C. Removes tool-use blocks from conversation history while preserving the textual narrative. Old tool calls and their results are dropped when context pressure rises, keeping the conversation thread coherent while shedding the verbose tool history.
D. Edits the system prompt automatically based on observed behavior.

> [!success]- Reveal answer
> **Correct: C**
>
> Context editing strategically removes tool-use history (the largest single category of context bloat in long agentic sessions) while keeping the conversation narrative intact.
>
> **Why others are wrong:**
> - A: It's automatic, not user-triggered.
> - B: That's a compaction-style summarization. Context editing is different — it surgically removes tool history rather than summarizing.
> - D: System prompt isn't modified.

#### Question 21

The **compaction** beta feature (header `compact-2026-01-12`, strategy `compact_20260112`) and **context editing** differ in approach how? ^d5-q-21

A. Both produce a summary; they're aliases.
B. **Compaction** produces a *summary* of conversation history (lossy: specifics paraphrased away); **context editing** *removes* specific blocks (typically tool-use history) without summarizing the rest. Different trade-offs: compaction preserves narrative continuity but loses specifics; context editing preserves specifics in retained blocks but removes blocks entirely.
C. Compaction is server-side; context editing is client-side.
D. Compaction is for Claude Code; context editing is for the API.

> [!success]- Reveal answer
> **Correct: B**
>
> Two different strategies for managing context pressure. Compaction summarizes (lossy paraphrase). Context editing removes blocks (lossless for what remains, complete removal for what's removed). They can be combined.
>
> **Why others are wrong:**
> - A: They're distinct mechanisms.
> - C: Both run in the API.
> - D: Both apply to the API.

### Task 5.5 — Human review and stratified sampling

#### Question 22

A team has an extraction pipeline processing 50,000 documents/night. They want to manually review a sample for quality. Which sampling approach is **documented**? ^d5-q-22

A. Random sampling of 1% (500 documents).
B. Sample only the documents the model flagged as low-confidence.
C. Sample only the documents that downstream consumers complained about.
D. **Stratified sampling**: ensure the review includes representative samples from each meaningful stratum — by document type, by extraction complexity, by model confidence levels (including high-confidence cases to catch overconfidence), by date range, by source. Random sampling can under-represent rare-but-important categories.

> [!success]- Reveal answer
> **Correct: D**
>
> Stratified sampling is the documented pattern. It ensures rare-but-important strata aren't missed by random sampling. High-confidence samples are particularly important — they catch the overconfidence failure mode that random sampling under-represents.
>
> **Why others are wrong:**
> - A: Random sampling at low rates can systematically miss rare strata.
> - B: Sampling only low-confidence cases hides high-confidence-but-wrong cases (overconfidence is a real failure mode).
> - C: Reactive sampling (only complained-about cases) is biased toward the kinds of errors downstream consumers happen to notice.

#### Question 23

For a high-volume extraction pipeline, the team wants per-record quality signals to drive review priority. Which is **most aligned** with documented practice? ^d5-q-23

A. Produce **field-level confidence** estimates (e.g., per-field "high/medium/low" or per-field reasoning) and route records to review based on the lowest-confidence field. Record-level confidence aggregates obscure where the actual uncertainty is.
B. Produce one overall record-level confidence score; route records with score < 0.8 to review.
C. Skip confidence entirely; review all records.
D. Run extraction twice and route disagreements to review.

> [!success]- Reveal answer
> **Correct: A**
>
> Field-level confidence is the documented preference. Record-level aggregates lose granularity: a record may be 90% confident overall but have one field at 30% — the reviewer needs to know that. Per-field signals direct the reviewer to the actual uncertainty.
>
> **Why others are wrong:**
> - B: Record-level aggregates hide field-level uncertainty.
> - C: Reviewing all records doesn't scale to 50K/night.
> - D: Multi-instance disagreement (Task 4.6) is also a valid pattern but field-level confidence is the specific Task 5.5 documented signal.

#### Question 24

A human-review pipeline includes 200 records flagged for review out of 50,000 processed. The reviewers want to know which fields specifically need their attention on each record. The pipeline should: ^d5-q-24

A. Show reviewers the full extracted record and let them read through it.
B. Highlight the lowest-confidence field on each record.
C. **Surface field-level details**: for each flagged record, show which fields triggered the review (e.g., "Date confidence: low — source has two date candidates"), the source excerpt the model used, and the model's reasoning where applicable. This lets the reviewer focus on the actual decision points.
D. Show only the record's overall confidence score.

> [!success]- Reveal answer
> **Correct: C**
>
> Surfacing field-level details (with source excerpts and reasoning) lets reviewers focus and decide efficiently. Otherwise reviewers re-do work the model already did.
>
> **Why others are wrong:**
> - A: Full records without highlighting fields requires reviewers to re-do the entire extraction analysis themselves.
> - B: Highlighting only the lowest-confidence field misses cases where multiple fields are uncertain.
> - D: Overall scores don't tell reviewers what to look at.

#### Question 25

A team's pipeline has been running for 6 months. They want to **continuously improve** the extraction by feeding reviewer corrections back into the system. The documented pattern is: ^d5-q-25

A. Retrain the model with the corrections.
B. **Periodically update prompts, few-shot examples, and (where appropriate) the schema based on aggregated reviewer-correction patterns.** This is the documented continuous-improvement loop: corrections aren't training data, they're product-team signals to refine the prompting strategy.
C. Manually correct each individual record before downstream use, but make no changes to the pipeline.
D. Build a separate ML classifier that learns from corrections to override the model.

> [!success]- Reveal answer
> **Correct: B**
>
> Continuous improvement via prompt and schema iteration is the documented pattern. Reviewer corrections surface systemic gaps in prompting; updating the prompt is the lever.
>
> **Why others are wrong:**
> - A: Retraining isn't an option for users of the API; the model is what it is.
> - C: One-off corrections without pipeline updates means the same errors recur indefinitely.
> - D: Building a parallel classifier is significant engineering investment; the lower-effort, documented path is prompt iteration.

### Task 5.6 — Provenance and claim-source

#### Question 26

A research synthesis pipeline aggregates findings from 6 sources. Stakeholders complain that the synthesis "loses provenance" — readers can't tell which source supports which claim. What's the **structural fix**? ^d5-q-26

A. Add a "Sources cited:" section at the end of the synthesis.
B. Use a more capable model to retain source attribution naturally.
C. Add prompt instructions: "Cite sources where possible."
D. Restructure the synthesis schema so **every claim carries structured provenance fields**: `source_url`, `document_name`, `publication_date`, and an `excerpt` showing the source text the claim is grounded in. Provenance becomes part of the data model, not a post-hoc citation.

> [!success]- Reveal answer
> **Correct: D**
>
> Structural provenance — making source attribution part of the schema for every claim — is the documented pattern. It's deterministic: no claim can be emitted without its provenance fields populated.
>
> **Why others are wrong:**
> - A: A "Sources cited" section doesn't link individual claims to specific sources.
> - B: Model capability doesn't change the probabilistic nature of prompt-based attribution.
> - C: "Cite where possible" is probabilistic — claims will sometimes lack attribution.

#### Question 27

Two sources disagree on a fact. The synthesis should: ^d5-q-27

A. **Surface the disagreement explicitly** with attribution: "Source A reports X; Source B reports Y. The sources conflict on this point." Don't silently merge or pick one. Disagreement is information for the reader.
B. Use the more recent source's claim.
C. Use the source with higher domain authority.
D. Omit the conflicting claim from the synthesis entirely.

> [!success]- Reveal answer
> **Correct: A**
>
> Documented pattern: surface conflicts with attribution. The reader needs to know the sources disagree; that's often as important as the underlying facts.
>
> **Why others are wrong:**
> - B: Recency isn't always a reliable signal — older sources may be more authoritative.
> - C: Authority assessment is itself a judgment that the synthesis shouldn't make silently. Surface the conflict; let the reader judge.
> - D: Omission hides relevant disagreement from the reader.

#### Question 28

A `claim-source mapping` should be preserved through which pipeline stage? ^d5-q-28

A. Source ingestion only — once synthesis happens, claim-source links can be dropped.
B. Through ingestion and analysis, then dropped at synthesis.
C. **Through every stage**: ingestion → analysis → synthesis → final output. The mapping is part of the data structure; lossy stages (especially synthesis) must propagate it. If it's lost at synthesis, recovering it post-hoc is unreliable.
D. Only at the final output stage; intermediate stages can drop it.

> [!success]- Reveal answer
> **Correct: C**
>
> End-to-end provenance preservation is the documented pattern. The mapping must survive every transformation; dropping it at any stage breaks the chain.
>
> **Why others are wrong:**
> - A, B: Dropping at synthesis is exactly the failure mode the structural pattern prevents.
> - D: Only-at-final means intermediate stages can lose accuracy, requiring post-hoc reconstruction.

#### Question 29

A synthesis pipeline produces:

> "Q3 revenue grew 12% YoY in the North American market."

Which provenance representation is **most aligned** with documented guidance? ^d5-q-29

A. Add a footnote linking to the source.
B. Attach a structured object: `{claim: "Q3 revenue grew 12% YoY in NA", sources: [{source_url, document_name, publication_date, excerpt: "...12% year-over-year growth in North America..."}], confidence: "high"}`. The provenance is data, not annotation.
C. Mention the source name in parentheses after the claim.
D. Maintain a separate "Sources" file with all source references but no per-claim mapping.

> [!success]- Reveal answer
> **Correct: B**
>
> Provenance as structured data (objects on each claim) survives downstream transformations. Prose annotation (footnotes, parentheticals) is lossy if anyone reformats or paraphrases later.
>
> **Why others are wrong:**
> - A: Footnotes can be lost in reformatting; the link isn't queryable.
> - C: Parenthetical mentions don't give queryable structure.
> - D: A separate sources file without per-claim mapping has the same problem as a "Sources cited" section: readers can't tell which source supports which claim.

#### Question 30

A team is concerned that prose summarization in their synthesis pipeline is washing out specific evidence. They consider three interventions. Which is **most aligned** with documented guidance? ^d5-q-30

A. Use a larger model that produces better summaries.
B. Add prompt instructions to "include specifics."
C. Reduce the synthesis length so less paraphrasing happens.
D. **Make synthesis structural rather than prose**: instead of producing a prose summary, require the synthesis to be a schema-defined output with claim objects, each carrying source excerpts and provenance fields. The model fills the schema; the schema enforces what's preserved. Specifics survive because the schema requires them.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented fix for prose lossiness is structural: define the synthesis as a schema with required evidence fields, not as prose. The model can no longer wash out evidence because the schema requires it.
>
> **Why others are wrong:**
> - A: Larger models produce better prose but prose is fundamentally lossy.
> - B: Prompt instructions are probabilistic; the loss recurs.
> - C: Shorter synthesis loses more, not less.

---

## Answer key summary

| Q | A | Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | B | 6 | D | 11 | A | 16 | C | 21 | B | 26 | D |
| 2 | D | 7 | A | 12 | C | 17 | B | 22 | D | 27 | A |
| 3 | A | 8 | C | 13 | B | 18 | D | 23 | A | 28 | C |
| 4 | C | 9 | B | 14 | D | 19 | A | 24 | C | 29 | B |
| 5 | B | 10 | D | 15 | A | 20 | C | 25 | B | 30 | D |

## Stats summary

- **Answer distribution:** A=7 (23%) · B=8 (27%) · C=7 (23%) · D=8 (27%)
- **Hardest questions** (subtle distinctions, two-look-right options): Q2 (memory tool protocol details), Q6 (architecture vs model "memory"), Q14 (partial-vs-complete distinction rationale), Q21 (compaction vs context editing trade-offs), Q23 (field-level vs record-level confidence), Q30 (structural vs prose synthesis)
- **Trap questions:** Q7 and Q10 (sentiment-based escalation anti-pattern), Q22 (random vs stratified sampling), Q23 (record-level confidence is the wrong unit)
- **Anti-pattern questions:** Q11-Q12 (silent failures, retrying non-retryable), Q26-Q30 (prose-based provenance and "cite where possible")

---

*Domain 5 weight: **15%** of the exam. Companion roadmap: [[cca_domain5_roadmap.md]]. Companion exercises: [[cca_domain5_exercises.md]].*
