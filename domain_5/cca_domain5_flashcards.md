---
title: CCA-F Domain 5 — Flashcards
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - flashcards
  - domain-5
domain: 5
domain-weight: 15
total-cards: 74
---

# CCA-F Domain 5 — Flashcards

**Context Management & Reliability · 15% of exam**

This is the domain with the most patterns and anti-patterns. Drill heavy.

---

## Task 5.1 — Conversation context management

> [!question]- What is the "context rot" phenomenon?
> As tokens grow, accuracy and recall degrade. More context is NOT automatically better — what's in context matters more than how much space is available.

> [!question]- Is more context always better?
> No. Quality of what's in context matters more than quantity. Long context with noise performs worse than short context with signal.

> [!question]- The "lost in the middle" effect — what is it?
> Models reliably process information at the **beginning** and **end** of long inputs but may omit or under-weight content from **middle sections**.

> [!question]- Three mitigations for the lost-in-the-middle effect (exam-tested):
> 1. **Front-load critical findings** at the beginning of aggregated input
> 2. **Explicit section headers** to organize the middle structurally
> 3. **Restate key task instructions at the end** so they're freshest in working memory

> [!question]- Anti-pattern: "put everything in chronological order" to fix lost-in-the-middle.
> Chronological order doesn't protect against the position effect; **importance order** does.

> [!question]- "Case facts" pattern — what is it?
> Structured JSON block injected into **every prompt**. Survives compaction. Not subject to interpretation. Use for facts that must always be available (customer tier, account ID, current state).

> [!question]- Case facts vs conversation summary — what's the difference?
> **Conversation summary:** prose, subject to compaction, interpretive
> **Case facts:** structured JSON, injected every turn, survives compaction, not interpretive

> [!question]- "Progressive summarization risks" — what specifically degrades?
> Numerical values, percentages, dates, customer-stated expectations — all get **condensed into vague summaries** that lose precision. The bug: a customer says "I want a refund of exactly $147.50 by Friday" gets summarized as "customer wants refund soon."

> [!question]- For long-running agentic workflows, what's the primary context-management strategy?
> **Server-side compaction.** Specialized needs (clearing tool results, thinking blocks) get **context editing**. Multi-session agents use the **memory tool**.

> [!question]- For multi-issue support sessions (multiple order IDs, amounts, statuses), how should structured data be persisted?
> Extract issue data into a **separate context layer** (case-facts block or scratchpad) — not buried in the conversation prose where summarization corrupts it.

> [!question]- "Trimming verbose tool outputs" — what does this do?
> Keep only return-relevant fields before they accumulate in context. E.g., from a customer lookup, keep `id`, `tier`, `account_status` — discard the 50-field full record.

> [!question]- Extended thinking + tool use — what's special about the tokens?
> Thinking blocks accompanying tool calls must be **returned unmodified** with the tool result. Don't strip or summarize them between turns.

---

## Task 5.1 (cont) — Compaction, context editing, memory tool

> [!question]- Beta header for context editing?
> `context-management-2025-06-27` with the `clear_tool_uses_20250919` tool.

> [!question]- What does context editing clear?
> Tool results (verbose outputs accumulating in context) and optionally thinking blocks. Surgical, not summarization.

> [!question]- Beta string for compaction?
> `compact-2026-01-12` with the `compact_20260112` tool.

> [!question]- Default trigger for compaction?
> **150,000 input tokens.** Configurable.

> [!question]- Compaction vs context editing — when to use each?
> **Compaction:** broad summarization when context is filling up; primary strategy for long workflows
> **Context editing:** surgical removal of specific content (verbose tool results); preserves the rest

> [!question]- Memory tool commands?
> `view`, `create`, `str_replace`, `insert`, `delete`, `rename`. Operates on the `/memories` directory.

> [!question]- The "ALWAYS VIEW MEMORY DIRECTORY" protocol — what is it?
> When the memory tool is enabled, the agent must `view /memories` first to see what's there before deciding whether to read or write. Otherwise the agent acts on stale assumptions.

> [!question]- Auto memory vs memory tool vs case facts — distinction?
> | Pattern | Mechanism | When |
> |---|---|---|
> | Auto memory | Claude Code feature, automatic | Recently-touched files, recent commands |
> | Memory tool | Explicit `/memories` directory | Agent-managed persistent notes |
> | Case facts | App-injected JSON every turn | Critical facts that must survive compaction |

> [!question]- Scenario: customer tier (Free/Pro/Enterprise) must be available every turn even after compaction. Where does it go?
> **Case facts** (app-injected JSON), not memory tool, not auto memory. Case facts inject every turn deterministically.

---

## Task 5.2 — Escalation triggers

> [!question]- Three legitimate escalation triggers (the only ones that count):
> 1. **Customer explicitly requests a human**
> 2. **Policy exception or policy gap** (policy doesn't address the case)
> 3. **Inability to make meaningful progress** (multiple dead-end retries, agent stuck)

> [!question]- Anti-pattern trigger: customer is frustrated. Why isn't this a trigger?
> Frustration ≠ desire for human. Many frustrated users are best helped by the agent continuing. Sentiment is unreliable. Wait for explicit request.

> [!question]- Anti-pattern trigger: case is complex. Why isn't this a trigger?
> Complexity ≠ escalation. If agent has a clear path, complexity is fine. Escalate when path is blocked, not when path is just non-trivial.

> [!question]- Anti-pattern trigger: tool returned an error. Why isn't this a trigger?
> Try documented recovery first (retry transient, refresh credentials). Single error ≠ blocked path.

> [!question]- Anti-pattern trigger: agent feels uncertain about its confidence. Why isn't this a trigger?
> Not by itself. Field-level confidence routes specific fields to review (Task 5.5) — but "I'm uncertain" generically isn't an escalation signal.

> [!question]- Scenario: customer asks for competitor price-matching; policy only addresses own-site adjustments. Escalate?
> **Yes.** Policy is silent/ambiguous on this specific request — a policy gap. Don't extrapolate; escalate.

> [!question]- Scenario: `lookup_customer` returns 3 records for "John Smith." What's the right action?
> **Request a disambiguating identifier** (email, account number, ZIP). Don't heuristically pick "most recent" or "highest-spending" — that's silent reconciliation.

> [!question]- "Sentiment-based escalation" — why is this a known anti-pattern?
> Sentiment is unreliable: false positives on frustrated venting (user doesn't want a human, just to vent), false negatives on calm requests (user explicitly asks for human in even tone).

> [!question]- Structured handoff to a human — what fields should it include?
> Customer details (identity, context), root cause analysis (what was attempted), recommended actions (agent's diagnosis of next step). Not just "agent escalated."

---

## Task 5.3 — Error propagation across agents

> [!question]- Structured error response — required fields?
> `errorCategory`, `isRetryable` (bool), `description`, partial results (where applicable).

> [!question]- Three error categories?
> **Transient** (network, rate-limit) — retry
> **Business** (policy violation, e.g., refund > $500) — escalate, don't retry
> **Permission** (auth, scope) — fail fast, don't retry

> [!question]- Anti-pattern: silent error suppression — what's wrong?
> Errors disappear from logs; downstream code can't make informed decisions; production incidents are hard to diagnose.

> [!question]- Anti-pattern: terminate entire workflow on single subagent failure — what's wrong?
> One subagent's transient error doesn't mean the whole orchestration must fail. Coordinator should handle locally (retry, partial results, alternate subagent) before failing the whole flow.

> [!question]- "Local recovery before escalation" — what's this?
> Failing tool first attempts documented recovery (retry transient, refresh auth) before escalating. Don't escalate every error to humans — that scales badly.

> [!question]- Generic error statuses like "search unavailable" — what's wrong with them?
> Hide context the coordinator needs. Was it transient (retry)? Auth (escalate)? Rate limit (back off)? Structure the error so the coordinator can decide.

> [!question]- Synthesis output with coverage annotations — what is this?
> Structured synthesis output that explicitly **distinguishes well-supported findings from coverage-gap areas** (where sources were unavailable or thin). Preserves which subagent searches returned empty so gaps are auditable.

> [!question]- Anti-pattern: uniform prose synthesis with no coverage signaling. What goes wrong?
> Readers (and downstream agents) cite the report as confident evidence for areas that actually had thin source backing. The gap is invisible.

> [!question]- Anti-pattern: re-run synthesis with "confident findings only" to handle thin coverage. Why wrong?
> Drops thin-coverage areas entirely — stakeholders no longer know the topic was undersupported, can't ask follow-ups or commission more research. Gap becomes invisible *and* unrecoverable.

> [!question]- Anti-pattern: have a human reviewer insert disclaimers where coverage was weak.
> Doesn't scale. Doesn't propagate to downstream agents. Relies on reviewer noticing what's missing in prose — exactly the failure mode coverage annotations fix.

---

## Task 5.4 — Large codebase exploration

> [!question]- What's the scratchpad pattern?
> Write key findings to external files (e.g., `.claude/notes/<topic>.md`) so they survive `/compact` and don't bloat active context. Re-read on demand.

> [!question]- Compaction vs scratchpads — what's the trade-off?
> Compaction is **lossy summarization** (in-context). Scratchpads are **lossless external state**. For high-value detail, scratchpad.

> [!question]- Why use subagents during large codebase exploration?
> Context isolation. Verbose exploration stays inside the subagent's context; only summary returns. Main session's context stays clean.

> [!question]- "Summarize key findings before spawning sub-agents for the next phase" — what's the pattern?
> Phase-by-phase exploration. After each phase, **write a summary to scratchpad / inject as initial context for the next phase**. Don't carry the full prior context forward.

> [!question]- Spawning subagents for specific questions like "find all test files" or "trace refund flow dependencies" — what's the principle?
> The main agent preserves **high-level coordination**; subagents handle **verbose investigations**. Main agent stays focused; subagents do the dirty work.

> [!question]- Five signs of context degradation in an extended session?
> 1. **Inconsistent answers** to the same question across turns
> 2. References to **"typical patterns"** instead of specific classes/functions discovered earlier
> 3. **Generic best-practices** replacing project-specific guidance
> 4. **Refusing to commit** to specifics ("it depends")
> 5. **Forgetting decisions** — re-suggesting approaches already rejected

> [!question]- When the agent says "typical patterns" instead of citing specific classes from your codebase — what does this signal?
> Context degradation. Specific detail has been lost; the agent is filling in with generic knowledge.

> [!question]- Anti-pattern: "just compact more often" when context degrades in a long codebase session.
> Compaction is summarization — what's already been lost is the specific detail that further summarization will further degrade. Fix is architectural: scratchpads + subagent isolation.

> [!question]- Anti-pattern: "switch to a model with larger context window."
> Context windows always have a limit. The fix isn't more capacity; it's choosing what to keep in context (case facts, scratchpads, subagent isolation).

---

## Task 5.5 — Human review and confidence calibration

> [!question]- Field-level vs record-level confidence — which is the right unit and why?
> **Field-level.** Record-level ("80% confident in this whole invoice") doesn't tell reviewers which fields to check. Field-level ("95% on `vendor_name`, 60% on `line_items`, 30% on `tax_amount`") routes review effort precisely.

> [!question]- The documented routing rule for human review (limited reviewer capacity):
> | State | Route to |
> |---|---|
> | High field-level confidence on all fields | Auto-pass + stratified sample |
> | **Low confidence on critical fields** | Human review (prioritized) |
> | **Ambiguous or contradictory source documents** | Human review (prioritized) |
> | High confidence + source clear | Auto-pass |

> [!question]- Stratified sampling — what's it for in review workflows?
> Measuring ongoing accuracy on the **auto-pass population**. Stratify by document type, vendor, field — not random sampling. Random sampling gives an overall number; stratified sampling tells you WHERE accuracy is dropping.

> [!question]- Anti-pattern: random sampling for review.
> Gives one aggregate accuracy number but doesn't surface where accuracy is dropping (which document types, which fields, which vendors). Stratified beats random for measurement.

> [!question]- Anti-pattern: same-session "self-review" by Claude.
> Not independent. Same context, anchored on prior output. Same-session review is not a quality check.

> [!question]- "Calibrated review routing" — what does the model emit alongside findings?
> Confidence scores per finding (or per field). The routing layer uses these to decide auto-pass vs human review.

> [!question]- When the source documents are ambiguous or contradictory — auto-pass or human review?
> **Human review.** Even if model confidence is moderate, ambiguous sources mean any single answer is unreliable. Human judgment is needed.

> [!question]- Reviewer capacity is always limited — what's the prioritization principle?
> Route the **lowest-confidence and most-ambiguous** cases to humans. Don't review everything; don't review randomly. Use the model's confidence signal to spend reviewer time where it has the most leverage.

---

## Task 5.6 — Information provenance and uncertainty

> [!question]- The claim-source mapping pattern — what fields?
> Every claim in synthesis output carries: `source_url`, `document_name`, `excerpt` (the source text supporting the claim), and `publication_date` / `data_collection_date`.

> [!question]- Anti-pattern: "Sources cited" section at the end of synthesis (footnotes-style).
> Source mapping isn't queryable. Readers can't tell which source supports which claim. Each **claim** needs its source attached, structurally.

> [!question]- Anti-pattern: "cite sources where possible" prompt instruction.
> Probabilistic; model cites sometimes, not always. Structural enforcement: schema requires `source_url` and `document_name` per claim, not as a suggestion.

> [!question]- Anti-pattern: parenthetical mentions of sources in prose.
> Doesn't give queryable structure. Downstream agents can't extract claim → source mappings reliably.

> [!question]- Why are dates (publication / collection) first-class structured fields in synthesis?
> Apparent conflicts between sources are often **temporal differences misread as contradictions** — Source A from 2019, Source B from 2023. Without dates, "40% unemployment" vs "25% unemployment" looks like a conflict. With dates, it's evolution over time.

> [!question]- Temporal evolution vs true contradiction — how do you tell?
> Check dates first. If dates differ materially, annotate as **temporal evolution**. If dates are the same time period and values conflict, it's a true contradiction — apply conflict-annotation pattern.

> [!question]- The conflict-annotation pattern — what to do with genuinely conflicting credible sources?
> **Annotate the conflict with source attribution.** Don't silently reconcile (averaging, picking most recent, picking most confident). Surface both values with their sources; let the coordinator decide.

> [!question]- Anti-pattern: silently averaging two conflicting values from credible sources.
> Reader has no idea there was disagreement. Important methodological context is lost. Annotate the conflict; let the decision be conscious.

> [!question]- Anti-pattern: synthesis agent picks "the more recent source" automatically.
> Recency ≠ correctness. The older source might be the more authoritative one. Silent reconciliation strips information; annotation preserves it.

> [!question]- Anti-pattern: prose summary in synthesis ("washes out evidence"). What's the structural fix?
> Make synthesis **schema-defined output**: claim objects with required evidence fields. Schema forces what's preserved. Model can't wash out evidence the schema requires.

> [!question]- Rendering different content types in synthesis — what's the pattern?
> Match output format to content: **financial data → tables**, **news → prose**, **technical findings → structured lists**. Don't convert everything to a single uniform format — kills the signal each content type carries.

> [!question]- Schema example for a synthesis claim with full provenance:
> ```
> {
>   "claim": "Unemployment was 40% in the affected region",
>   "source_url": "...",
>   "document_name": "...",
>   "publication_date": "2020-03-15",
>   "data_collection_date": "2019-Q4",
>   "excerpt": "...",
>   "conflict_detected": false
> }
> ```

> [!question]- Coverage annotations on synthesis output — what does the structure look like?
> Sections distinguish **well-supported findings (3+ sources)** from **gap-flagged areas (sources unavailable or contradictory)**. Preserve which subagent searches returned empty so gaps are visible.

---

## Cross-cutting (any task)

> [!question]- The general principle behind "structural surfacing of uncertainty"?
> When uncertainty exists, the system must surface it **structurally** (schema fields, separate sections, explicit flags) — not paper over it with prose. Prose loses; schema preserves.

> [!question]- The general principle behind "annotation over reconciliation"?
> When there's a conflict, an ambiguity, or a gap — **annotate it** so the next layer (coordinator, human, downstream agent) can decide. Don't silently reconcile; that strips information.

> [!question]- The general principle of "field-level over record-level" — what's it about?
> Granular signals route action precisely. Record-level confidence is too coarse to route review effort. Field-level confidence routes reviewers to specific fields. Generalizes beyond confidence: any per-item annotation beats aggregate.

> [!question]- Pattern: "prose loses, schema preserves." Where does this apply?
> Source attribution (claim-source mappings), uncertainty (coverage annotations), conflicts (conflict_detected booleans), confidence (field-level scores), validation (calculated_total vs stated_total). Any time you'd be tempted to "tell the model to do X" — make X a schema obligation instead.

---

*Domain 5 weight: 15% of exam. Companion roadmap: [[cca_domain5_roadmap]]. Companion practice exam: [[cca_domain5_practice_v2]].*
