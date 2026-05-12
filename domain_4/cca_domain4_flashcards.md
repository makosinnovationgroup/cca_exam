---
title: CCA-F Domain 4 — Flashcards
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - flashcards
  - domain-4
domain: 4
domain-weight: 20
total-cards: 69
---

# CCA-F Domain 4 — Flashcards

**Prompt Engineering & Structured Output · 20% of exam**

---

## Task 4.1 — Explicit criteria, clear and direct prompts

> [!question]- What four things make a prompt "clear and direct"?
> 1. **Tell the model what you want** (not what to avoid)
> 2. **Provide context** (who you are, who you're addressing, why)
> 3. **State explicit success criteria**
> 4. **Use structure** (lists, headings, ordered steps) for complex prompts

> [!question]- The "golden rule" of clear prompting?
> Show your prompt to a colleague with minimal context. If they're confused, Claude will be too.

> [!question]- Specific categorical criteria vs vague instructions — which is better for precision?
> **Specific categorical criteria.** "Flag comments only when claimed behavior contradicts actual code behavior" vs vague "check that comments are accurate" — the specific version dramatically reduces false positives.

> [!question]- Why does "be conservative" or "only report high-confidence findings" fail to improve precision?
> Both are vague. The model interprets them inconsistently. Compared to **specific categorical criteria**, vague conservatism produces unpredictable filtering.

> [!question]- Adding more prompt text — when does this fail to fix inconsistent output?
> When the problem is **structural**, not lexical. Vague output → more prompt text won't help; move constraints into schema (`tool_use`, `output_config.format`) or `tool_choice`.

> [!question]- Anti-pattern: telling the model what NOT to do (negative framing). Why is positive framing better?
> Negative framing leaves the action space undefined. "Don't be wordy" gives no target; "Use bullet points with max 8 words each" gives a target.

> [!question]- A multi-category agent has 2 categories at 38% false-positive rate, 3 categories accurate. Developer trust is eroding. What's the right interim response?
> **Temporarily disable** the 2 high-FP categories while rewriting their prompts. Keep the 3 accurate categories running. Surgical, not blanket.

> [!question]- Why is "lower confidence threshold globally" wrong for handling high-FP categories?
> Penalizes the accurate categories to dampen the bad ones. Degrades signal across the board.

> [!question]- Why is "add a global 'be more conservative' instruction" wrong for handling high-FP categories?
> Same issue. Blanket fixes degrade categories that were already calibrated correctly.

> [!question]- Why is "continue running, add confidence labels, let devs filter mentally" wrong?
> Continues shipping false positives, further eroding trust. Puts the burden on the developer the agent was supposed to help.

> [!question]- Writing review criteria for a code-review agent — what's the right level of specificity?
> Specific categorical: "Report bugs, security issues, undefined behavior. Skip minor style, local patterns, formatting." Avoid confidence-based filtering ("report only when confident") — too vague.

---

## Task 4.2 — Few-shot prompting (multishot)

> [!question]- How many few-shot examples does the exam say to use?
> **3–5 examples.** Beyond ~5, marginal value drops sharply and token cost rises.

> [!question]- How are few-shot examples formatted in prompts?
> **XML-wrapped.** Use `<example>...</example>` tags to clearly delimit each example from instructions and from each other.

> [!question]- Few-shot vs structured output — when does each fit?
> **Few-shot:** shapes **style, tone, format** (soft constraints). Use for matching a voice, format consistency on outputs that aren't strictly structured.
> **Structured output (schema):** enforces **structure** (hard constraints). Use when format must be exact (JSON shape, required fields).

> [!question]- Anti-pattern: adding more few-shot examples to fix strict format compliance issues.
> Schema beats examples for strict compliance. Examples are probabilistic guidance; schema is a contract. If the problem is "JSON keeps being malformed," use `tool_use` or `output_config.format`, not more examples.

> [!question]- Scenario: invoice extraction returns null for `total_amount` in 23% of cases when documents use non-standard layouts. Best fix?
> Few-shot examples covering **varied layouts**. Show the model what extraction looks like across the formats it's seeing. Schema alone doesn't help; the model needs to learn the layout-to-field mapping.

> [!question]- Why aren't retry loops the right fix for "null on non-standard layouts"?
> Retrying produces the same null — the data IS in the document, but the model can't locate it in unfamiliar layouts. The fix is examples showing how to locate; retry just re-asks the same broken question.

> [!question]- Why is "lower threshold to accept null" wrong?
> Hides the problem, doesn't fix it. The data exists in the source; the extraction logic is what's broken.

---

## Task 4.3 — Structured output via `tool_use` / `output_config.format`

> [!question]- What does the v0.1 exam guide identify as the primary structured-output mechanism?
> **`tool_use` with a typed schema** and `tool_choice` forcing the tool. The tool's parameters become the output structure.

> [!question]- `tool_use` vs `output_config.format` — main differences:
> | Pattern | Notes |
> |---|---|
> | `tool_use` + forced `tool_choice` | What the v0.1 exam tests; widely available; works with extended thinking; tool params ARE the output |
> | `output_config.format` | Newer beta (`structured-outputs-2025-11-13`, now GA); plain JSON output; doesn't work with citations or JSON-mode prefilling |

> [!question]- Beta header for native structured outputs (historical)?
> `structured-outputs-2025-11-13` — was required during beta. Feature is now GA; header not needed. (Exam may test the historical fact.)

> [!question]- Anti-pattern: "ask for JSON in the prompt" to get structured output.
> Probabilistic; produces malformed JSON, missing fields, hallucinated keys. Use protocol enforcement (`tool_use` or `output_config.format`), not prompt instruction.

> [!question]- Required vs nullable fields in JSON schema for structured output — when to use which?
> **Required + non-nullable** when the field must always have a real value.
> **Required + nullable** for fields that must always appear in output but can legitimately be null (extraction not present in source).
> **Optional** for fields the model can omit entirely.

> [!question]- Anti-pattern: required-and-non-nullable for fields that can legitimately be absent in source data.
> Model **fabricates** data to satisfy the schema. Fix: make those fields **nullable** so the model can return null without inventing.

> [!question]- Strict enum on a classification field — what's the failure mode?
> Model **fabricates** a category to fit when the real value is "none of the above." Fix: include `"other"` in the enum + a free-text `detail` field for the explanation.

> [!question]- "Strict mode" in tool_use — what does it guarantee?
> Eliminates **syntax errors** only: malformed JSON, missing required, type mismatches. Does NOT eliminate semantic errors (wrong values, made-up data).

> [!question]- Does strict mode replace semantic validation in your code?
> No. Strict mode = syntax. Your code = semantics (do the sums add up? do the IDs match a registry?). Both are needed for production.

> [!question]- Chain-of-thought field placement in structured output — `reasoning` before or after `answer`?
> **Reasoning BEFORE answer.** Schema order matters: the model fills fields in order, so reasoning happens first and informs the answer. Putting reasoning after = it's just decoration, doesn't influence the answer.

> [!question]- When to use CoT (chain-of-thought)?
> Complex reasoning: math, multi-step analysis, debugging, decisions with risk. Schema fields like `reasoning` before `answer`.

> [!question]- When to skip CoT?
> Simple lookups, formatting tasks, classifications with clear criteria. CoT adds latency and tokens with no quality gain.

---

## Task 4.4 — Validation, retry, feedback loops

> [!question]- When is retry the right response to a validation failure?
> When the error message gives the model **new information to correct on**. E.g., "line items sum to $147.50 but stated total is $150.00" — model can recompute. Useful info → retry useful.

> [!question]- When is retry the wrong response?
> When the field is genuinely **absent from source data**. No amount of retrying produces data that isn't there. Use null + escalate.

> [!question]- Correct retry pattern in extraction pipelines?
> Retry limit (typically 2–3) + fallback action (mark unknown, escalate to human, skip). Unbounded retry burns tokens.

> [!question]- Anti-pattern: retry without a limit.
> Burns tokens, never terminates, masks structural problems. Always cap.

> [!question]- `detected_pattern` field — what is it and why does it matter?
> A **categorical enum** on every structured finding naming which rule or code construct triggered the finding (e.g., `"implicit_type_coercion"`, `"missing_null_check"`). Lets you analyze dismissal patterns: group dismissed findings by `detected_pattern` to identify which patterns drive false positives.

> [!question]- Why is "improve the prompt" wrong when 40% of agent findings get dismissed but logs only have finding text?
> You can't fix what you can't categorize. Add `detected_pattern` first (instrumentation), then analyze which patterns drive dismissals, then improve those specific prompts.

> [!question]- Self-correction schema pattern: `calculated_total` vs `stated_total`. What does this do?
> Extract the model's computed total alongside the document's stated total in the same schema. If they differ, flag a discrepancy — without a second LLM pass. Validation becomes a schema obligation.

> [!question]- `conflict_detected` boolean in extraction schemas — when does it matter?
> When source documents have **inconsistent values** (e.g., two pages of an invoice show different totals). The model flags it via `conflict_detected: true`; coordinator decides reconciliation policy.

> [!question]- Pydantic-style validation vs JSON schema strict mode — what does Pydantic add?
> **Semantic validation**: cross-field checks, value-range constraints, business-rule enforcement. Schema strict mode handles syntax; Pydantic handles "do these values make sense together."

> [!question]- Feedback loops in retry: what info should the retry prompt include?
> The **specific reason** the prior output failed validation. Generic "try again" is useless; "the `tax_amount` field was -50 but tax cannot be negative" gives the model what it needs to correct.

---

## Task 4.5 — Message Batches API

> [!question]- Cost savings for the Batches API vs standard?
> **50%** off standard input/output token rates.

> [!question]- Maximum processing window for a batch?
> **24 hours.** Most finish in under an hour, but no SLA.

> [!question]- SLA on Batches API completion?
> **None.** Could finish in minutes, could take the full 24 hours.

> [!question]- Does Batches support multi-turn tool calling?
> **No.** Single-turn only. (Multi-turn message history in a single batch entry IS supported, but the agent can't take a tool action and continue within the same entry.)

> [!question]- Does Batches support streaming?
> **No.** Not supported for batch requests.

> [!question]- How do you correlate batch results back to inputs?
> `custom_id` field — you set it on each request, and results come back with it. Required because results aren't returned in order.

> [!question]- `custom_id` constraints?
> 1–64 chars, regex `^[a-zA-Z0-9_-]{1,64}$`. Alphanumeric + hyphens + underscores only.

> [!question]- How long are batch results available after creation?
> **29 days** after batch creation. After that, unviewable.

> [!question]- Maximum batch size?
> **100,000 requests** OR **256 MB** per batch, whichever you hit first.

> [!question]- Order of results within a batch — guaranteed?
> **Not guaranteed.** Match results to inputs by `custom_id`.

> [!question]- Failure isolation in batches?
> Failure of one request does NOT affect others. Each entry is independent.

> [!question]- Prompt caching + batches — do discounts stack?
> **Yes.** 50% batch discount + cache discount = compounded savings. Use 1-hour cache duration for batches that take >5 min.

> [!question]- Beta header for 300K-token output on batches?
> `output-300k-2026-03-24` — Opus 4.7 / Opus 4.6 / Sonnet 4.6 only. Claude API only (not Bedrock / Vertex / Foundry).

> [!question]- When does Batches API fit?
> Overnight reports, weekly audits, nightly test generation, bulk extraction, large-scale evaluations, content moderation backlogs — anything non-blocking and latency-tolerant.

> [!question]- When does Batches API NOT fit?
> Pre-merge CI checks, customer-facing UIs, anything blocking a human or system within an hour. Real-time workflows always disqualified.

> [!question]- The "blocking-or-latency" decision rule for Batches vs synchronous?
> "Will a human or downstream system be waiting on this result within the hour?" Yes → synchronous. No → Batches.

> [!question]- The chunking pattern for handling partial batch failures?
> Submit a batch of 10,000. 3% fail (300). Re-submit just those 300, possibly chunked further to isolate problematic inputs. Use `custom_id` to track which inputs failed.

> [!question]- Scenario: weekly compliance audit producing a Monday-9am report from prior-week data. Batches or sync?
> **Batches.** Non-blocking (report not needed until Monday); large dataset (cost savings matter); no real-time requirement.

> [!question]- Scenario: customer-facing chat assistant. Batches or sync?
> **Synchronous.** Real-time conversational UX. Batches' 24h SLA breaks the use case.

---

## Task 4.6 — Multi-instance & multi-pass review

> [!question]- What makes a review "independent"?
> Separate sessions / separate batch entries / separate processes. NO shared context between the review instances. Same-session "second look" is NOT independent.

> [!question]- Why does same-session review fail as a quality check?
> The model anchors on its prior output in the same context. It's predisposed to confirm its own conclusions, not challenge them.

> [!question]- Multi-instance review composition with Batches API — how?
> For high-volume runs, submit each document N times (e.g., 3) with distinct `custom_id`s like `doc_001_instance_1`, `doc_001_instance_2`, `doc_001_instance_3`. Each entry is independent. Route disagreement to human review.

> [!question]- Multi-pass review architecture for many-file PRs — what's the structure?
> Two stages:
> 1. **Per-file local passes** — one Claude instance per file with isolated context. Focused on local issues (syntax, logic, style).
> 2. **Cross-file integration pass** — one instance reviewing inter-file concerns (API contracts, data flow, dependencies). Input: file changes + summaries from local passes.

> [!question]- Why does single-pass review of 14 files fail?
> **Attention dilution.** As context fills, the model's reasoning on later files degrades materially. By file 8 of 14, quality on file 8 is worse than quality on file 1 — same file, different position.

> [!question]- Anti-pattern: "submit all 14 files in one prompt with a stricter checklist."
> Doesn't address attention dilution. Stricter prompts don't fix the position effect.

> [!question]- Anti-pattern: "3 instances each reviewing all 14 files with different prompts, then merge."
> Three copies of the same broken approach. Tripling cost without fixing dilution.

> [!question]- Anti-pattern: "auto-reject PRs over 10 files, force the developer to split."
> Punishes the developer for the agent's architectural limitation. Doesn't fix the agent.

> [!question]- Where does the integration pass get its input — full file contents or summaries?
> **Summaries** from the per-file passes (plus the file changes themselves). Smaller surface = more room for cross-file reasoning.

> [!question]- Self-reported confidence alongside findings — what's this for?
> Calibrated review routing. Each finding has a confidence score the model emits; low-confidence findings route to human review.

---

*Domain 4 weight: 20% of exam. Companion roadmap: [[cca_domain4_roadmap]]. Companion practice exam: [[cca_domain4_practice_v2]].*
