---
title: Claude Certified Architect — Foundations · Domain 5 Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - roadmap
  - domain-5
status: in-progress
domain: 5
domain-name: "Context Management & Reliability"
domain-weight: 15
primary-scenarios: ["Customer Support", "Multi-Agent Research", "Developer Productivity", "Structured Data Extraction"]
estimated-hours: 4
coverage-level: comprehensive
---

# Domain 5 Roadmap — Context Management & Reliability

## What this domain tests

How to **persist critical context** across long conversations, **propagate errors** with structure, **route escalation** to humans, **manage context in large codebases**, design **human review workflows**, and **preserve provenance** through multi-agent synthesis. Cross-cutting — Domain 5 patterns appear in every scenario.

The exam rewards architectural fixes over prompt-level patches. "Increase context window" is almost always wrong; "case-facts injection," "scratchpads," "claim-source mappings" are the correct structural answers.

## Domain weight

**15% of the total exam — smallest single domain by weight, but cross-cutting.** Patterns appear inside scenarios driven by other domains. The exam guide itself (pages 21-25) is more authoritative than the docs for this domain.

> [!important] Context-management features expanded recently
> Three distinct context-management features now exist on the API side:
> 1. **Compaction** (server-side, beta header `compact-2026-01-12`) — recommended primary strategy
> 2. **Context editing** (beta header `context-management-2025-06-27`) — tool result clearing, thinking block clearing, client-side compaction
> 3. **Memory tool** (`memory` tool, /memories directory) — cross-session persistence; client-side storage
>
> Plus Claude Code's own auto-compact at ~64-75% utilization, the `/compact` command, scratchpads (CLAUDE.md, MEMORY.md, manual files), and subagent context isolation.

---

## Task statements (6 total)

| Task | What it tests |
|------|---------------|
| 5.1 | Manage conversation context · case-facts persistence |
| 5.2 | Escalation patterns · explicit request · sentiment limits |
| 5.3 | Error propagation · structured context over silent suppression |
| 5.4 | Large codebase context · scratchpads · `/compact` |
| 5.5 | Human review workflows · stratified sampling · field-level confidence |
| 5.6 | Provenance preservation · claim-source mappings |

---

## Read in full · ~3 hours

### Exam guide pages 21–25 · ~45 min

- [ ] **Read:** Exam guide Domain 5 section (PDF pages 21-25)
- [ ] **Re-read for memorization** (the exam guide is the primary source for this domain)

For Domain 5, the exam guide itself is more useful than the docs. Domain 5 is more pattern than feature; the guide states the patterns directly.

Pay specific attention to:
- The "case facts" pattern for customer support conversations (Task 5.1)
- The "explicit escalation request" rule; sentiment-based escalation is unreliable (Task 5.2)
- The structured error propagation pattern (Task 5.3)
- The scratchpad and `/compact` pattern for large codebases (Task 5.4)
- Stratified-sampling + field-level-confidence for human review (Task 5.5)
- Claim-source mapping pattern with conflict annotation (Task 5.6)

**After reading you should be able to:** Recite the recommended pattern for each of the six tasks from memory.

### Context windows · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/context-windows`

Token budgets, how context fills up, what gets prioritized when context approaches capacity, the "context rot" phenomenon.

**Memorize cold:**
- Context window = "working memory" for the model; **all text the model can reference**, including the response itself
- As tokens grow, **accuracy and recall degrade** — known as **"context rot"**
- More context is NOT automatically better — what's IN context matters more than how much space is available
- Claude achieves SOTA on long-context benchmarks (MRCR, GraphWalks) but gains depend on what's there, not just how much
- For long-running conversations and agentic workflows: **server-side compaction is the recommended primary strategy**
- For more specialized needs: context editing offers tool result clearing, thinking block clearing
- For multi-session agents: design state artifacts so context recovery is fast on new session start (memory tool's multi-session pattern)
- Extended thinking + tool use has special token-handling: thinking blocks accompanying tool calls must be returned unmodified with the tool result

### Compaction · ~45 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/compaction`
- [ ] **Re-read for memorization** (compaction is the primary context-management strategy)

Server-side compaction. The recommended strategy for managing context in long-running conversations.

**Memorize cold:**
- Beta header: `compact-2026-01-12`
- Enable via `context_management.edits` with strategy `compact_20260112`
- Process:
  1. API detects when input tokens exceed configured trigger threshold
  2. Generates summary of current conversation
  3. Creates a compaction block containing the summary
  4. Continues the response with compacted context
  5. On subsequent requests, append response to messages — API automatically drops message blocks prior to the compaction block
- When using server-side tools (web search, web fetch), compaction trigger is checked at start of each sampling iteration; can occur multiple times within a single request
- The `/v1/messages/count_tokens` endpoint applies existing compaction blocks but doesn't trigger new compactions; useful for checking effective token count after prior compactions

**Compaction in Claude Code specifically:**
- Auto-compact triggers around **64-75% context utilization** in current versions (was 95%+ historically)
- Run `/compact` manually at **~60%** for proactive control
- Skill bodies are **re-injected** after compaction (large skills truncated; oldest invoked skills dropped if budget exceeded)
- Auto memory (MEMORY.md) preserved (not subject to compaction)
- Static CLAUDE.md (root + user) preserved
- Path-scoped rules and nested CLAUDE.md summarized away; reload on next matching file read

### Context editing · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/context-editing`

For specialized scenarios where you need more fine-grained control than compaction provides.

**Memorize cold:**
- Beta header: `context-management-2025-06-27`
- Three strategies:
  1. **Tool result clearing** (`clear_tool_uses_20250919`) — best for agentic workflows with heavy tool use; old tool results no longer needed once Claude processed them
  2. **Thinking block clearing** — for managing thinking blocks in extended thinking; options to preserve recent thinking for continuity
  3. **Client-side SDK compaction** — SDK-based summary alternative; server-side preferred
- Server-side compaction is generally preferred over client-side
- These strategies are useful when you need to selectively clear specific content rather than full conversation compaction

### Memory tool · ~45 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/memory-tool`

The memory tool — distinct from Claude Code's auto memory. Enables cross-session knowledge retention.

**Memorize cold:**
- Tool name: `memory`
- Memory directory: `/memories`
- Commands: `view`, `create`, `update`, `delete` (CRUD over files in /memories)
- **Operates client-side** — you control where/how data is stored through your own infrastructure
- ZDR-eligible (data not retained after API response)
- Pattern: Claude automatically checks memory directory before starting tasks
- Use cases:
  - Maintain project context across multiple agent executions
  - Learn from past interactions, decisions, feedback
  - Build knowledge over time without keeping everything in context
- Pairs with **compaction** (compaction handles within-session; memory tool handles across-session)

**Memory protocol pattern (canonical from docs):**
```
ALWAYS VIEW YOUR MEMORY DIRECTORY BEFORE DOING ANYTHING ELSE
MEMORY PROTOCOL:
1. Use the `view` command of your `memory` tool to check for earlier progress.
2. ... (work on the task) ...
- As you make progress, record status/progress/thoughts in your memory.
ASSUME INTERRUPTION: Context might reset at any moment.
```

**Distinct from Claude Code's auto memory:**
- **Auto memory (Claude Code):** `MEMORY.md` file written by Claude as it works; loads first 200 lines/25 KB at session start
- **Memory tool (API):** Claude actively uses `view`/`create`/`update`/`delete` commands to manage `/memories` directory across sessions
- Both serve cross-session continuity but at different layers

### Claude Code context window page · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/context-window`

Same page covered in Domain 3, but worth re-reading from Domain 5 lens — what survives compaction, the subagent isolation pattern, scratchpads.

**Memorize the scratchpad pattern:**
- For large codebase exploration: write key findings to scratchpad files
- Scratchpads can be: `.claude/notes/<topic>.md`, custom scratchpad files, or use the memory tool
- Survive `/compact` (auto memory does; manual scratchpad files are external to the conversation)
- Re-read on demand without holding full content in active context
- Subagents (Explore, custom) provide context isolation — verbose exploration stays inside subagent context, only summary returns

---

## Skim · ~15 min

### Prompt caching

- [ ] **Skim:** `docs.claude.com/en/docs/build-with-claude/prompt-caching`

Just know it exists. Per the exam guide, implementation details are out of scope. Relevant only as cost-reduction mechanism for repeated prompt prefixes.

### Context engineering cookbook (external recipe)

- [ ] **Skim:** `docs.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools`

A practical walkthrough comparing compaction, context editing, and memory tool. Useful for solidifying the mental model:
- **Compaction:** compresses the whole window when it grows too large
- **Clearing:** drops stale re-fetchable data inside the window
- **Memory:** moves information OUT of the window so it survives across sessions

Each addresses a different bottleneck.

### Automatic context compaction (cookbook)

- [ ] **Skim:** `docs.claude.com/cookbook/tool-use-automatic-context-compaction`

Practical recipe showing context compaction in agentic customer service workflow. Concrete example of the case-facts + compaction pattern.

---

## Skip — out of scope

> [!warning] Don't get pulled in
> Explicitly out of scope per the exam guide:

- **Token counting algorithms** or tokenization specifics
- **Pricing implications** of context length
- **Detailed compaction internals** (the *behavior* is testable; the *algorithm* is not)
- **Streaming** (out of scope across the board)
- **Embedding models** (out of scope)

---

## Hands-on drills · ~2 hours

- [ ] **Drill 1: Case-facts persistence** — in a multi-turn customer support agent, run a 12-turn conversation. Implement a "case facts" block (order IDs, amounts, dates) injected into every prompt outside the conversation summary. Compare accuracy with vs without across 12 turns.

- [ ] **Drill 2: Structured error propagation** — in a multi-agent setup, force a subagent timeout. Verify error propagation surfaces structured context (`failureType`, `attemptedQuery`, `partialResults`) rather than swallowing it. Have the coordinator decide retry/escalate/continue based on the structure.

- [ ] **Drill 3: Provenance preservation** — modify a research coordinator so each subagent emits findings as structured records: `{claim, evidence, source_url, document_name, publication_date}`. Have the synthesis subagent preserve `source_url` and `document_name` on every claim it includes.

- [ ] **Drill 4: Provenance break experiment** — weaken the synthesis prompt to just "cite sources where possible." Watch attribution disappear. Restore strict instruction. Watch it return.

- [ ] **Drill 5: Conflict annotation** — feed two subagents the same question with conflicting source documents. Confirm synthesis annotates conflicts with attribution ("Reuters reports 40%; Bloomberg reports 25%") rather than silently reconciling.

- [ ] **Drill 6: Scratchpad pattern** — for a large codebase exploration in Claude Code, write key findings to `.claude/notes/<topic>.md`. Continue the session, let context auto-compact, then ask Claude to re-read the scratchpad. Confirm it resumes with earlier findings.

- [ ] **Drill 7: Memory tool usage** — implement the memory tool in a multi-session agent. Verify it views memory directory at session start, records progress as it works, and resumes correctly when context is reset.

- [ ] **Drill 8: Sentiment-vs-explicit escalation** — design two test cases: (a) frustrated but not escalating customer ("This is so annoying"); (b) explicit escalation ("I want to speak to a human"). Confirm agent escalates on (b) but not (a).

- [ ] **Drill 9: Stratified sampling for review** — for a batch of 1000 extracted invoices with field-level confidence scores, design a sampling strategy that prioritizes low-confidence fields for human review. Verify it's not just "every Nth record."

- [ ] **Drill 10: `/context` exploration** — in Claude Code, work through a long session. Run `/context` periodically to see live breakdown. Notice which categories grow fastest.

---

## Common confusions and distractor patterns

### Distractor pattern 1: "Use a longer context window"

Context loss after many turns:
- "Switch to a model with larger context window"
- "Increase `max_tokens`"
- **Correct:** architectural fix — case facts injection (5.1), scratchpads (5.4), claim-source structure (5.6), memory tool, subagent isolation. Context windows always have a limit; the fix is what you choose to keep in context.

### Distractor pattern 2: Sentiment-based escalation

When/how to escalate to human:
- "Analyze sentiment and escalate on frustration threshold"
- "Use sentiment subagent"
- **Correct:** escalate on **explicit request**. Sentiment is unreliable (false positives on frustrated venting, false negatives on calm requests).

### Distractor pattern 3: Silent error suppression

Errors disappearing from production logs:
- "Catch all errors and return graceful default responses"
- "Skip failed operations silently"
- **Correct:** structured error propagation — `errorCategory`, `isRetryable`, `description`, partial results — pushed up to a layer that can decide.

### Distractor pattern 4: "Cite sources where possible"

Lost attribution in synthesis:
- "Add 'cite sources where possible' to synthesis prompt"
- "Use few-shot examples of cited synthesis"
- **Correct:** structural claim-source mappings in data passed between agents. Synthesis prompt should require `source_url` and `document_name` on every claim as part of the output **schema**, not a suggestion.

### Distractor pattern 5: Silent reconciliation of conflicts

Subagents producing conflicting values (40% vs 25%):
- "Have synthesis agent average them"
- "Pick the more recent source"
- "Use sentiment to pick the most confident"
- **Correct:** annotate the conflict with attribution. Don't silently reconcile.

### Distractor pattern 6: Same-session "review"

Production quality assurance:
- "Have Claude review its own output in the same session"
- "Use a second-pass refinement in same conversation"
- **Correct:** stratified sampling for human review + field-level confidence (5.5). Same-session review isn't independent.

### Distractor pattern 7: Record-level vs field-level confidence

Routing for human review:
- "Use record-level confidence score" → distractor; "80% confident in this whole invoice" doesn't tell reviewers which fields to check
- **Correct:** field-level confidence — "95% on vendor_name, 60% on line_items, 30% on tax_amount" — routes reviewers to specific fields

### Distractor pattern 8: "Just compact more often"

Long codebase session degrading:
- "Run /compact every 10 turns"
- "Increase compaction frequency"
- **Correct:** scratchpads + subagent isolation. Compaction is summarization (lossy); scratchpads are external files (lossless).

### Subtle distinction 1: Case facts vs conversation summary

- **Conversation summary** — natural prose; subject to compaction; interpretive
- **Case facts** — structured JSON block injected into every prompt; survives compaction; not subject to interpretation
- Use case facts for must-not-drift data (order IDs, amounts, dates, identifiers)
- Use conversation summary for context ("customer is frustrated, has been waiting, primary concern is...")

### Subtle distinction 2: Compaction vs scratchpad vs memory tool

| Mechanism | Scope | Lossy? | Survives across sessions? |
|-----------|-------|--------|--------------------------|
| Compaction (server-side / `/compact`) | Within-session | Lossy (summary) | No (unless re-injected) |
| Scratchpad (CLAUDE.md, MEMORY.md, custom .md) | Within-session, re-readable | Lossless | Yes (CLAUDE.md / MEMORY.md auto-load) |
| Memory tool | Multi-session, on-demand | Lossless | Yes (explicit) |
| Context editing (clear_tool_uses) | Within-session | Lossy (drops) | No |
| Subagent context isolation | Subagent's own context window | N/A (separate) | No (subagent context dies after task) |

### Subtle distinction 3: Auto memory vs Memory tool vs Case facts

- **Auto memory (Claude Code)** — Claude auto-writes notes to MEMORY.md as it works; loads first 200 lines/25 KB at session start
- **Memory tool (API)** — model actively uses `view`/`create`/`update`/`delete` commands on /memories directory
- **Case facts** — application code injects structured JSON into every prompt; not Claude-managed; deterministic

All three address persistence but at different layers.

### Subtle distinction 4: Field-level vs record-level confidence

- **Record-level:** "80% confident in this whole invoice" — useless for routing
- **Field-level:** per-field confidence scores — routes reviewers to specific fields
- For Task 5.5 human review, **field-level** is the documented pattern

### Subtle distinction 5: Provenance through synthesis

When coordinator synthesizes findings from multiple subagents:
- **Wrong:** "summarize the findings" — attribution dissolves
- **Wrong:** "include source URLs in the summary" — vague; model interprets loosely
- **Right:** every claim in synthesis output carries `source_url`, `document_name`, `publication_date` as structured fields. Synthesis = **collection of attributed claims**, not prose summary that occasionally references sources.

### Subtle distinction 6: Subagent context isolation is a Task 5.4 tool

For "exploration would flood main context" scenarios, the answer is often:
- Spawn an Explore subagent (or custom exploration subagent)
- Subagent runs in fresh context; verbose output stays inside
- Only summary returns to main context
- This is functionally equivalent to a "context-isolated scratchpad" for one task

---

## Self-assessment recall tests

### Task 5.1 · Conversation context management

- [ ] Completed Task 5.1 self-check

1. What's the documented pattern for keeping critical data consistent across long support conversations?
2. Why doesn't a conversation summary alone solve long-conversation context?
3. Where do case facts get injected — natural conversation flow or separately?
4. What's the difference between case facts and the memory tool?

> [!success]- Answers
> 1. **Case facts**: structured block (JSON) of must-remember data injected into every system prompt or user message
> 2. Summaries are lossy and interpretive — critical identifiers can drift or get paraphrased
> 3. Separately, outside the conversation summary, so they survive compaction unchanged
> 4. Case facts are application-injected (deterministic, every turn); memory tool is model-managed (Claude decides what to read/write/update in /memories)

### Task 5.2 · Escalation patterns

- [ ] Completed Task 5.2 self-check

1. Most reliable signal that a customer should be escalated?
2. Why is sentiment-based escalation unreliable?
3. Should a high-frustration message without explicit human request trigger escalation?

> [!success]- Answers
> 1. **Explicit request** ("I want to speak to a human")
> 2. False positives on frustrated venting; false negatives on calm requests
> 3. No — respect stated needs; don't override with sentiment assumptions

### Task 5.3 · Error propagation

- [ ] Completed Task 5.3 self-check

1. What's wrong with silent error suppression?
2. What should a structured error response include?
3. Who decides retry vs escalate vs fall back?

> [!success]- Answers
> 1. Downstream code can't make informed decisions; errors disappear from observability; production incidents hard to diagnose
> 2. `errorCategory`, `isRetryable`, `description`, partial results where applicable
> 3. The layer above — failing tool reports structured context; orchestrator decides what to do with it

### Task 5.4 · Large codebase context

- [ ] Completed Task 5.4 self-check

1. Documented pattern for findings that should survive compaction?
2. When should you trigger `/compact` manually in Claude Code?
3. Why isn't "use a model with bigger context window" the right answer to context pressure?
4. How do subagents help with context pressure?

> [!success]- Answers
> 1. Scratchpads (external files like `.claude/notes/<topic>.md`) OR the memory tool — both survive compaction
> 2. Around **60%** context utilization proactively (auto-compact triggers around 64-75%)
> 3. Context windows always have limits; what's IN context matters more than how much space is available ("context rot")
> 4. Subagents have separate context windows; verbose exploration stays in subagent context; only summary returns to main

### Task 5.5 · Human review workflows

- [ ] Completed Task 5.5 self-check

1. What's stratified sampling? Why is it the right approach?
2. Field-level vs record-level confidence — which is documented?
3. Why is same-session "self-review" not a valid quality check?

### Task 5.6 · Provenance preservation

- [ ] Completed Task 5.6 self-check

1. How should claim-source mappings be structured?
2. What should happen when two subagents return conflicting values for the same fact?
3. Why isn't "cite sources where possible" sufficient as a synthesis instruction?

> [!success]- Answers
> 1. Every claim is a structured record with `claim`, `evidence` (or `evidence_excerpt`), `source_url`, `document_name`, `publication_date`. NOT prose with embedded citations.
> 2. **Annotate the conflict with attribution** ("Source A reports X; Source B reports Y") rather than silently reconciling
> 3. "Where possible" is probabilistic. Structural requirements (schema demands `source_url` on every claim) are deterministic. Synthesis output = collection of attributed claims.

---

## Common scenario patterns by task

**Task 5.1** — A scenario describes a long support conversation drifting on identifiers (order IDs, amounts). Distractors reach for "use a bigger context window" or "summarize more often"; the correct answer is structured case-facts injection outside the conversation summary.

**Task 5.2** — A scenario describes when to escalate. Distractors reach for sentiment analysis; the correct answer is explicit-request detection.

**Task 5.3** — A scenario describes errors vanishing from logs or workflow continuing past silent failures. Distractors reach for graceful defaults; the correct answer is structured error propagation up to a decision-making layer.

**Task 5.4** — A scenario describes a large codebase session degrading after many file reads. Distractors reach for compaction frequency or larger models; the correct answer is scratchpads (external files) and subagent context isolation.

**Task 5.5** — A scenario describes production extraction quality assurance. Distractors reach for record-level confidence or same-session review; the correct answer is stratified sampling + field-level confidence routed to human reviewers.

**Task 5.6** — A scenario describes lost attribution in synthesis. Distractors reach for "cite sources where possible" or vague prompt instructions; the correct answer is structural claim-source mappings preserved through synthesis, with conflict annotation rather than silent reconciliation.

---

## Quick-reference cheatsheet

- **Case facts** (5.1): structured JSON block injected into every prompt; survives compaction; for must-not-drift identifiers
- **Escalation** (5.2): explicit request only; sentiment unreliable
- **Error propagation** (5.3): structured `errorCategory`, `isRetryable`, `description`, partial results; never silently suppress
- **Scratchpads** (5.4): write to files; re-read on demand; survive compaction; lossless
- **Manual `/compact`** at ~60% utilization (auto triggers around 64-75% in current versions)
- **Subagent context isolation** (5.4): verbose work in subagent stays there; only summary returns
- **Memory tool**: model uses `view`/`create`/`update`/`delete` on `/memories` for cross-session persistence
- **Auto memory** (Claude Code only): MEMORY.md auto-written; first 200 lines / 25 KB at startup
- **Compaction beta header:** `compact-2026-01-12` with strategy `compact_20260112` in `context_management.edits`
- **Context editing beta header:** `context-management-2025-06-27` with strategies `clear_tool_uses_20250919`, thinking-block clearing, client-side SDK compaction
- **Compaction trigger**: when input tokens exceed configured threshold; multiple times per request possible with server tools
- **What survives /compact in Claude Code:** auto memory, static CLAUDE.md (root + user), skill bodies (re-injected); doesn't survive: conversation history, nested CLAUDE.md, path-scoped rules (reload on demand)
- **Human review** (5.5): stratified sampling + field-level confidence; not same-session self-review; not record-level confidence
- **Provenance** (5.6): structural mappings, not "cite where possible"; every claim carries `source_url`, `document_name`, `publication_date`
- **Conflict annotation**: surface disagreements with attribution; don't silently reconcile
- **"Context rot"**: as tokens grow, accuracy/recall degrade; more context isn't automatically better
- **Three context-management features:** compaction (within-session summarization), context editing (selective clearing), memory tool (cross-session persistence)

---

## Progress tracker

- [ ] All "Read in full" pages complete
- [ ] All "Skim" pages reviewed
- [ ] All hands-on drills complete
- [ ] All self-assessment recall tests passed
- [ ] Quick-reference cheatsheet memorized

---

*Domain 5 weight: **15%** of the exam — smallest single domain, but cross-cutting. Exam guide is primary source.*
