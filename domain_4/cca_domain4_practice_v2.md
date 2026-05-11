---
title: Claude Certified Architect — Foundations · Domain 4 Practice Exam
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
  - domain-4
status: drilling
total-questions: 30
domain: 4
domain-weight: 20
target-percent: 80
difficulty: harder-than-real-exam
---

# CCA-F Domain 4 — Practice Exam

30 questions covering Prompt Engineering & Structured Output (20% of the real exam). Calibrated **harder than the real exam** — distractors include realistic-but-wrong patterns (asking for JSON in the prompt, retrying without limits, applying Batches API to real-time scenarios) and verbatim-fact traps (incorrect retention windows, fabricated beta header strings).

## How to use

- [ ] **Round 1** — cold attempt all 30 without checking answers
- [ ] **Round 2** — score using the collapsed callouts
- [ ] **Round 3** — re-attempt any miss after re-reading the relevant roadmap section

**Aim for 80%+ on Round 1.**

## Task statement coverage

| Task | Questions | Focus |
|------|-----------|-------|
| 4.1 | Q1-Q4 | Explicit criteria, clear and direct prompts |
| 4.2 | Q5-Q8 | Multi-shot prompting, XML-wrapped examples |
| 4.3 | Q9-Q15 | Structured output via `tool_use` / `output_config.format` |
| 4.4 | Q16-Q20 | Validation-retry loops with limits and fallbacks |
| 4.5 | Q21-Q26 | Message Batches API: cost, latency, capabilities |
| 4.6 | Q27-Q30 | Multi-instance independent review |

---

## Practice questions

### Task 4.1 — Explicit criteria, clear and direct

#### Question 1

A team's prompt says "Generate a high-quality summary of the document." Output quality varies wildly. What's the **best** first intervention? ^d4-q-1

A. Replace vague terms with explicit criteria: target length, required sections, evaluation dimensions, what to exclude. "High-quality" is interpreter-dependent; concrete criteria are deterministic.
B. Add three few-shot examples of high-quality summaries.
C. Switch to a more capable model (Opus 4.7) to handle the ambiguity.
D. Add a system prompt telling the model to "think step-by-step before summarizing."

> [!success]- Reveal answer
> **Correct: A**
>
> The documented first step for vague prompts is to make criteria explicit. Replace "high-quality summary" with specifics: "Generate a 200-word summary with: (1) one-sentence thesis, (2) three key findings, (3) the document's stated limitations. Exclude opinions or speculation."
>
> **Why others are wrong:**
> - B: Few-shot examples shape style but don't fix vague criteria — the model still doesn't know what dimensions matter.
> - C: A more capable model with vague criteria produces inconsistent output just like a less capable model with vague criteria — vagueness is the bottleneck.
> - D: Step-by-step thinking improves reasoning on complex tasks; it doesn't define quality criteria the model is missing.

#### Question 2

The "golden rule" of clear and direct prompting is: ^d4-q-2

A. Always use chain-of-thought before producing the final answer.
B. Always include at least three few-shot examples.
C. Show your prompt to a colleague with minimal context; if they're confused, Claude will be too.
D. Always use XML tags to structure your prompt.

> [!success]- Reveal answer
> **Correct: C**
>
> Documented in the Anthropic prompting guide. The colleague test surfaces ambiguity — if a human reader can't tell what you're asking for, the model can't either.
>
> **Why others are wrong:**
> - A: CoT is a tactic for specific situations, not a universal rule.
> - B: Few-shot is a tactic for format-shaping, not a universal rule.
> - D: XML tags help with complex prompts but aren't the documented "golden rule."

#### Question 3

Which prompt is **most likely** to produce inconsistent output? ^d4-q-3

A. "Extract the invoice number, date, total amount, and vendor name from the document. Return them as JSON with keys 'invoice_number', 'date', 'total', and 'vendor_name'."
B. "Summarize this invoice."
C. "Classify this customer message into one of: complaint, praise, question, request, other. If 'other', also provide a one-sentence detail field."
D. "Translate this product description from English to French, preserving HTML tags and product specifications exactly."

> [!success]- Reveal answer
> **Correct: B**
>
> "Summarize this invoice" is vague on every dimension: length, format, what's important, audience, level of detail. Outputs will range from one-liners to multi-paragraph essays.
>
> **Why others are wrong:**
> - A: Specifies fields, types, and output structure.
> - C: Specifies allowed values, and what to do for "other."
> - D: Specifies source, target, and what to preserve.

#### Question 4

A team's structured-extraction prompt says: "Don't make up data. If a field isn't present, don't include it." Their output occasionally fabricates dates that don't appear in source. What's the **best** structural fix? ^d4-q-4

A. Strengthen the prompt: "ABSOLUTELY DO NOT FABRICATE DATA. NEVER. EVER."
B. Add three few-shot examples of inputs where the date is absent and the output omits it.
C. Switch to extended thinking so the model reasons more carefully about what's in the source.
D. Move the constraint from natural language into the schema: make the date field **nullable** in the schema and instruct the model to return `null` when the source lacks the field. Required-and-non-nullable schemas force fabrication to satisfy schema validity.

> [!success]- Reveal answer
> **Correct: D**
>
> Prompt-based "don't fabricate" is probabilistic. Schema-level nullability is structural — the model has a legitimate value (`null`) to return when data is absent.
>
> **Why others are wrong:**
> - A: Capitalization and forcefulness don't change probabilistic compliance.
> - B: Few-shot helps but is probabilistic. Schema-level nullability is the structural fix.
> - C: Extended thinking reasons over what's in source but still has to produce output matching the schema. If the schema is required-and-non-nullable, the model still fills it.

### Task 4.2 — Few-shot prompting

#### Question 5

The Anthropic prompting docs recommend how many examples for typical few-shot usage? ^d4-q-5

A. 3-5 examples
B. 10-15 examples
C. Exactly 1, demonstrating the canonical case
D. As many as fit in the context window

> [!success]- Reveal answer
> **Correct: A**
>
> Documented: 3-5 examples. More examples sometimes help for very complex format constraints but the documented baseline is 3-5.
>
> **Why others are wrong:**
> - B: Too many examples often hurts more than it helps — Claude over-anchors on patterns in the examples that may not generalize.
> - C: A single example is rarely enough to convey the intended pattern reliably.
> - D: Diminishing returns and context-bloat issues.

#### Question 6

A team's few-shot examples are mixed inline with the system prompt content without explicit delimiters. The model occasionally treats examples as instructions. What's the **documented fix**? ^d4-q-6

A. Number the examples explicitly: "Example 1: ... Example 2: ..."
B. Move all examples to a separate `examples` API parameter (a recent addition).
C. Wrap each example in `<example>` tags (with multiple examples in an `<examples>` parent block) so the model parses examples as illustrative rather than instructional.
D. Replace examples with explicit instructions about what to do in each scenario.

> [!success]- Reveal answer
> **Correct: C**
>
> XML tags help Claude parse the distinction between instructions and examples. `<example>...</example>` (with multiple examples in `<examples>...</examples>`) is the documented wrapping.
>
> **Why others are wrong:**
> - A: Numbering helps slightly but doesn't structurally disambiguate examples from instructions.
> - B: An `examples` API parameter is fabricated.
> - D: Converting examples to instructions defeats the point of few-shot demonstration.

#### Question 7

Which set of few-shot examples is **most aligned** with documented good practice for a customer-message classification task? ^d4-q-7

A. Five examples, all positive sentiment with similar phrasing variations.
B. Five examples covering: positive (clear), negative (clear), ambiguous mixed, sarcasm, complaint disguised as question. Each shows the expected label and reasoning.
C. Two examples, one for each major class.
D. Twenty examples, sampled randomly from production data.

> [!success]- Reveal answer
> **Correct: B**
>
> The documented criteria: examples should be **relevant** (mirror the actual task), **diverse** (cover edge cases), and **structured** (consistent format). Five examples covering distinct types — including the hard edge cases — fits all three criteria.
>
> **Why others are wrong:**
> - A: Lack of diversity — the model would learn to expect positive sentiment.
> - C: Too few to cover the variation.
> - D: Twenty examples with random sampling may include irrelevant or off-distribution cases. Curated diversity beats raw quantity.

#### Question 8

For a multi-step extraction task, few-shot examples should show what? ^d4-q-8

A. Just the input and the final output, omitting intermediate steps.
B. Just the final output formats.
C. The input and a brief description of the expected output type.
D. Both the input and the **structured output** (e.g., the JSON or XML the model should emit), so the model learns the exact format. For tasks involving reasoning, also show the intermediate reasoning steps before the final structured output.

> [!success]- Reveal answer
> **Correct: D**
>
> Few-shot examples should demonstrate the full input-to-output transformation including the structured format. For reasoning tasks, showing intermediate steps (e.g., inside `<thinking>` tags) before the final answer helps the model learn the reasoning style as well as the output format.
>
> **Why others are wrong:**
> - A: Omitting intermediate steps loses guidance on how to reason for tasks where reasoning matters.
> - B: Output-only examples lose the relationship between input and output.
> - C: Descriptions of output type are weaker than concrete examples of the output itself.

### Task 4.3 — Structured output

#### Question 9

Which approach produces the **most reliable** structured JSON output from Claude? ^d4-q-9

A. Define a tool with a JSON schema and force its use with `tool_choice: {type: "tool", name: "X"}`. The tool's `input_schema` defines the output structure; the model is constrained at the protocol level to call the tool with valid input. Alternatively, use the native structured outputs API (`output_config.format`) for direct JSON output.
B. Ask in the prompt: "Respond in valid JSON format."
C. Prefill the assistant message with `{` to force JSON-like output.
D. Set the response model temperature to 0 so the format is deterministic.

> [!success]- Reveal answer
> **Correct: A**
>
> Protocol-level structure (via `tool_use` schema or `output_config.format`) is the documented reliable approach. Both are constrained-decoding mechanisms that the model cannot break.
>
> **Why others are wrong:**
> - B: Prompt-based JSON requests are probabilistic. The model often complies but occasionally wraps in markdown, adds preamble, or malforms.
> - C: Prefilling `{` shapes the output start but doesn't guarantee valid JSON throughout.
> - D: Temperature 0 increases determinism but the model can still produce invalid JSON deterministically.

#### Question 10

A field `expense_category` is an enum: `software`, `hardware`, `services`, `office_supplies`, `travel`. The model occasionally encounters expenses that don't fit any category and force-fits them (e.g., classifying a "training course" as `services` because it's closest). What's the **structural fix**? ^d4-q-10

A. Increase the model size; bigger models classify edge cases more accurately.
B. Add specific edge cases as additional enum values: `training`, `subscriptions`, `legal`.
C. Add `"other"` to the enum and add a required `expense_category_detail` field (nullable for non-other) that captures what the "other" actually is. This handles unknown cases gracefully while preserving categorical structure.
D. Allow the field to be free-text instead of an enum.

> [!success]- Reveal answer
> **Correct: C**
>
> The documented unknown-category pattern: enum with `"other"` + a detail string. This preserves the categorical structure for known cases while giving the model a legitimate escape hatch for unknowns.
>
> **Why others are wrong:**
> - A: Model size doesn't address the categorical-mismatch problem — even a perfect classifier can't fit a case outside the enum.
> - B: Adding categories addresses *known* edge cases but doesn't generalize. New edge cases appear; the enum needs constant updating.
> - D: Free-text loses the categorical structure entirely. Downstream consumers can no longer rely on category names.

#### Question 11

What does `strict: true` on a tool definition guarantee? ^d4-q-11

A. The model will produce semantically correct output (e.g., line items sum to total).
B. The tool input is validated against the schema at the protocol level — no malformed JSON, no missing required fields, no type mismatches. The model literally cannot emit tokens that violate the schema.
C. The tool can only be called once per turn.
D. The tool's description must be 200+ characters long.

> [!success]- Reveal answer
> **Correct: B**
>
> `strict: true` enables grammar-constrained decoding against the schema. Syntax errors (malformed JSON, missing fields, type mismatches) are impossible. Semantic errors (math, dates, business rules) are NOT addressed by `strict`.
>
> **Why others are wrong:**
> - A: Strict mode is syntax-only. Semantic correctness is the application's responsibility.
> - C: Per-turn call count is unrelated.
> - D: Description length isn't constrained by `strict`.

#### Question 12

The native structured outputs feature uses which beta header? ^d4-q-12

A. `messages-2024-12-01`
B. `json-mode-stable`
C. `structured-output-v2-2025`
D. `structured-outputs-2025-11-13`

> [!success]- Reveal answer
> **Correct: D**
>
> Documented beta header for the native structured outputs feature.
>
> **Why others are wrong:** A, B, and C are fabricated. Memorize the exact beta header string.

#### Question 13

A team is using `output_config.format` with a schema. The schema's grammar takes ~250ms to compile on first request, then subsequent requests are fast. What explains this? ^d4-q-13

A. Schemas compile to grammars used for constrained decoding; the first compile incurs 100-300ms overhead. Compiled grammars are cached for 24 hours, so subsequent requests using the same schema are fast.
B. The model takes longer on the first request because it hasn't "seen" the schema before in the session.
C. The first request includes a beta-feature handshake that adds latency; subsequent requests skip it.
D. There's a connection setup overhead on the first request that gets reused.

> [!success]- Reveal answer
> **Correct: A**
>
> Documented behavior. The grammar cache prevents repeated compilation cost.
>
> **Why others are wrong:**
> - B: Models don't "remember" schemas across requests in this way.
> - C: Beta-feature headers are sent on every request; no handshake overhead.
> - D: Connection setup isn't the bottleneck for this specific feature.

#### Question 14

A team's structured output is required to include a `confidence` field with values `high`, `medium`, or `low`. They also want the model to provide a `confidence_reasoning` field but only when confidence is `medium` or `low`. How should the schema express this? ^d4-q-14

A. Make `confidence_reasoning` optional in the schema and instruct the model to omit it when confidence is high.
B. Use schema dependencies (`if`/`then`/`else` constructs in JSON Schema) to conditionally require `confidence_reasoning`.
C. Make `confidence_reasoning` required and nullable. The model returns `null` when confidence is high and a string otherwise. Pair with prompt instructions explaining when each.
D. Use two separate tools — one for high-confidence outputs and one for low/medium.

> [!success]- Reveal answer
> **Correct: C**
>
> The documented pattern: required + nullable. The model has a clear option (`null`) for the "field not applicable" case and is forced to include the field (so consumers don't have to detect absence).
>
> **Why others are wrong:**
> - A: Optional fields are sometimes omitted entirely, which complicates downstream parsing. Required + nullable is more uniform.
> - B: Conditional JSON Schema (`if`/`then`/`else`) works in spec but is less commonly supported by structured-output decoders and adds complexity. The required + nullable pattern is simpler and documented.
> - D: Two tools is over-engineering for a single optional field.

#### Question 15

A developer wants extended thinking enabled and forces a specific tool with `tool_choice: {type: "tool", name: "X"}`. The API returns an error. What's true? ^d4-q-15

A. Extended thinking works only with `claude-opus-*` models when combined with tool use.
B. Extended thinking is compatible only with `tool_choice: "auto"` or `"none"`. The `"any"` option and forced (`{type: "tool"}`) are incompatible because they prefill the assistant message in a way that conflicts with thinking blocks.
C. The error means the developer needs to set `thinking.required_tool_choice: false`.
D. Extended thinking and tool use are mutually exclusive — pick one.

> [!success]- Reveal answer
> **Correct: B**
>
> Documented incompatibility. Forced and `"any"` both prefill the assistant message with tool-call structure, which conflicts with the thinking-block prefill that extended thinking requires.
>
> **Why others are wrong:**
> - A: Extended thinking works across models that support it; not model-restricted in this way.
> - C: `required_tool_choice` is fabricated.
> - D: Extended thinking + `tool_choice: "auto"` works fine; they're not mutually exclusive in general.

### Task 4.4 — Validation-retry loops

#### Question 16

A validation-retry loop is appropriate when: ^d4-q-16

A. The model occasionally makes math errors that retry-with-error-feedback can fix.
B. The model fabricates fields that aren't in the source (retry helps the model "find" them).
C. The model returns inconsistent JSON structure (retry forces a valid structure).
D. The validation error gives the model new information that helps it correct. Math mismatches (line-items-don't-sum-to-total) provide correctable signal. Missing source data does NOT — no amount of retrying produces data that isn't in the source.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented retry-helpfulness criterion: does the error give correctable signal? Yes → retry can help. No → retry doesn't help.
>
> **Why others are wrong:**
> - A: Partially right but incomplete — D is the better generalization.
> - B: Retry doesn't help fabrication — if the data isn't in the source, the model will fabricate again.
> - C: Inconsistent JSON structure is fixed by `strict: true` schemas, not by retry.

#### Question 17

A team's retry-without-limit loop runs forever when the source document genuinely lacks a date. What's the **best** fix? ^d4-q-17

A. Add a retry limit (2-3 attempts) and a fallback path (mark unknown, escalate to human, or skip) when retries are exhausted.
B. Add a regex check after the model's response to verify the date is real.
C. Switch to a stronger model that won't fabricate dates.
D. Use the memory tool to remember past failures so the loop doesn't retry the same case.

> [!success]- Reveal answer
> **Correct: A**
>
> A bounded retry loop with a fallback is the documented pattern. The limit prevents runaway loops; the fallback handles cases where retry can't succeed.
>
> **Why others are wrong:**
> - B: Verifying the date doesn't fix the underlying loop — the model still tries.
> - C: Even stronger models can't extract data that isn't in the source.
> - D: The memory tool isn't designed for failure-tracking; it's for cross-session knowledge.

#### Question 18

A retry message sent back to the model should contain what? ^d4-q-18

A. Just "Please try again."
B. The original document, repeated for context.
C. The validation failure information **plus** any relevant context — e.g., "Line items sum to $250.00 but you reported subtotal as $300.00; please reconcile. Possibly the model misread line 3." Pass concrete error context so the model can correct on signal.
D. A full re-explanation of the schema.

> [!success]- Reveal answer
> **Correct: C**
>
> Retry effectiveness depends on the model having actionable information about what went wrong. Concrete error context (the actual mismatch, the actual numbers, the field that failed) lets the model correct meaningfully.
>
> **Why others are wrong:**
> - A: "Please try again" gives no information; retry effectiveness drops sharply.
> - B: Repeating the source document doesn't add error context.
> - D: Re-explaining the schema is redundant if the model already has it; it doesn't tell the model what went wrong.

#### Question 19

A team's extraction pipeline retries up to 5 times per failed record. After 5 retries, what should happen? ^d4-q-19

A. Try a sixth time with a stronger model.
B. The pipeline should have a documented **fallback** — typically: mark the record as needs-review, escalate to a human reviewer, or fall back to a less-strict extraction. The fallback decision should be deterministic and recorded so downstream systems know the record didn't extract cleanly.
C. Crash the pipeline and alert engineering.
D. Silently skip the record so the pipeline can continue.

> [!success]- Reveal answer
> **Correct: B**
>
> The documented pattern: bounded retry + explicit fallback. The fallback isn't just "give up" — it's a recorded outcome that downstream systems can route appropriately.
>
> **Why others are wrong:**
> - A: A sixth retry with a stronger model adds complexity for marginal benefit; the documented pattern accepts that some records need human handling.
> - C: Crashing on individual record failures fails the whole batch; failure isolation requires per-record fallbacks.
> - D: Silent skip loses observability — the team can't measure failure rate or follow up.

#### Question 20

Which type of validation error is **least helpful** to retry on? ^d4-q-20

A. Line items sum to a different total than reported (math discrepancy).
B. Date format wrong (`2026-01-15` vs `01/15/2026`) — fixable with retry plus format hint.
C. Currency missing — model didn't fill the currency field in a response where currency was present in source.
D. Field is required but source lacks the data entirely (e.g., shipping address absent on a service-only invoice).

> [!success]- Reveal answer
> **Correct: D**
>
> Missing-source-data errors are not retry-correctable. No retry can produce data that isn't in the source. The fix is to make the field nullable in the schema, not to retry.
>
> **Why others are wrong:**
> - A: Math errors are retry-correctable — feed the discrepancy back.
> - B: Format errors are retry-correctable — feed the format requirement back.
> - C: Field-omission errors when source has the data are retry-correctable — point the model at the source.

### Task 4.5 — Message Batches API

#### Question 21

What's the documented cost savings of the Message Batches API vs the synchronous Messages API? ^d4-q-21

A. 50% off standard input/output token rates.
B. 25% off all charges.
C. 75% off only on input tokens; output tokens at standard rate.
D. The same rate; Batches is convenience, not cost savings.

> [!success]- Reveal answer
> **Correct: A**
>
> Documented 50% cost savings vs interactive API.
>
> **Why others are wrong:** B, C, D incorrect on the verbatim fact.

#### Question 22

What's the **maximum processing window** for a Batches API submission? ^d4-q-22

A. 1 hour
B. 12 hours
C. 24 hours (most batches finish in <1 hour, but there's no SLA — the API may take up to 24 hours)
D. 7 days

> [!success]- Reveal answer
> **Correct: C**
>
> Documented: 24-hour maximum, no SLA on completion.
>
> **Why others are wrong:** A and B understate; D overstates.

#### Question 23

A customer-facing chatbot returns answers in 2-5 seconds. The team wants to reduce API costs by switching to the Batches API. What's the **correct** assessment? ^d4-q-23

A. Switch and save 50% on costs.
B. The Batches API is wrong for this use case. There's no SLA (could take up to 24 hours) and no streaming, which makes it unsuitable for any real-time / customer-facing workflow.
C. Switch but pair with an aggressive timeout so requests fall back to synchronous mode after 5 seconds.
D. Switch but only on weekends when load is lower.

> [!success]- Reveal answer
> **Correct: B**
>
> Documented constraint: no SLA + no streaming + no multi-turn = Batches is wrong for real-time. The exam tests this trap directly.
>
> **Why others are wrong:**
> - A: Ignores SLA and streaming constraints.
> - C: Fallback adds engineering complexity; the underlying mismatch remains.
> - D: Weekend-batch arrangements don't change the structural unsuitability.

#### Question 24

A team needs to correlate results from a Batches API submission back to their input records. Which field is **documented** for this? ^d4-q-24

A. `id` — auto-generated UUID for each request.
B. `request_id` — set by Anthropic for each request.
C. `metadata` — arbitrary JSON blob the team can pass through.
D. `custom_id` — team-set, 1-64 chars, regex `^[a-zA-Z0-9_-]{1,64}$`. Results return with the matching `custom_id`. Required for correlation since results may be unordered.

> [!success]- Reveal answer
> **Correct: D**
>
> Documented field: `custom_id`. The exact constraint (1-64 chars, alphanumeric + hyphens + underscores) is testable verbatim.
>
> **Why others are wrong:**
> - A, B: Auto-generated IDs from Anthropic exist (the batch ID, the message ID) but they're not what the team uses to correlate to their own input records.
> - C: A `metadata` field for batches with arbitrary content is fabricated.

#### Question 25

Can a Batches API request include multi-turn tool calling? ^d4-q-25

A. No. Batches API supports single-turn only. Each request is one model call with optional tool definitions, but no multi-turn tool execution within a single batch entry.
B. Yes, up to 5 turns per request.
C. Yes, with a `multi_turn: true` flag set on the request.
D. Yes, but only when the batch contains fewer than 1,000 requests.

> [!success]- Reveal answer
> **Correct: A**
>
> Documented constraint: single-turn only. If the team needs multi-turn tool use, they need the synchronous Messages API.
>
> **Why others are wrong:** B, C, D fabricated.

#### Question 26

How long are Batches API results retrievable after the batch is created? ^d4-q-26

A. 7 days
B. 14 days
C. 29 days after batch creation
D. 90 days

> [!success]- Reveal answer
> **Correct: C**
>
> Documented: 29-day result retention window.
>
> **Why others are wrong:** A, B, D incorrect on the verbatim fact.

### Task 4.6 — Multi-instance independent review

#### Question 27

A team wants to add a quality check to their structured-extraction pipeline. Which approach is **documented**? ^d4-q-27

A. Have Claude review its own output in the same session ("Here's what you produced; please verify.").
B. Run the **same** extraction through multiple **independent** instances (separate sessions, separate API calls) and route any disagreement to human review. Disagreement among independent instances surfaces genuine ambiguity in the source.
C. Run the extraction twice and use the second result.
D. Use a more capable model and trust its output.

> [!success]- Reveal answer
> **Correct: B**
>
> Multi-instance independent review is the documented pattern. Disagreement is a signal of genuine ambiguity; agreement is a signal of confidence.
>
> **Why others are wrong:**
> - A: Same-session "self-review" is anchored on the prior response — not independent.
> - C: Arbitrary; doesn't leverage disagreement as signal.
> - D: A more capable model is silently wrong on ambiguous cases just like a less capable one — no signal.

#### Question 28

Why is same-session "self-review" documented as inadequate for quality assurance? ^d4-q-28

A. The model gets bored of seeing the same output and outputs random text.
B. The session's API costs double.
C. Same-session prompts run into context limits.
D. The model is anchored on its prior response — "review your own answer" doesn't escape the original framing. Real independence requires separate sessions/instances that approach the task fresh.

> [!success]- Reveal answer
> **Correct: D**
>
> The model's prior reasoning is in context for the "review" step. The model interprets the review as a request to defend or adjust the prior answer, not to evaluate independently.
>
> **Why others are wrong:**
> - A: Models don't get bored.
> - B: Cost is a tangential issue, not the documented reason.
> - C: Context limits aren't the documented reason; anchoring is.

#### Question 29

A team runs 3 independent instances on each extraction. On 80 of 100 records, all 3 instances agree. On 20 of 100, instances disagree on at least one field. What's the **best** next step? ^d4-q-29

A. Route the 20 disagreement-records to human review with field-level disagreement details surfaced. Genuine ambiguity in the source needs a human judgment call; the disagreement itself is the signal.
B. Run 7 more instances on the disagreement-records and use majority vote.
C. Use the first instance's output for all 20 records to stay efficient.
D. Discard the 20 records and process only the 80 high-confidence ones.

> [!success]- Reveal answer
> **Correct: A**
>
> Disagreement is documented as a routing signal. Route to humans with the specific fields that disagreed surfaced; the human's decision becomes the authoritative output.
>
> **Why others are wrong:**
> - B: Majority vote on independently ambiguous cases isn't reliable — three more instances may agree by chance on a wrong interpretation. Human review is documented.
> - C: First-instance preference is arbitrary.
> - D: Discarding loses 20% of the data without informing the team about why.

#### Question 30

A team wants to use multi-instance review for a high-volume extraction pipeline (50,000 documents per night). Is the Batches API a fit? ^d4-q-30

A. No — Batches doesn't support concurrent independent runs.
B. No — multi-instance review requires real-time response, which Batches can't provide.
C. Yes — submit each document N times (e.g., 3) with distinct `custom_id`s (`doc_001_instance_1`, `doc_001_instance_2`, `doc_001_instance_3`). Each instance runs independently. Correlate by `custom_id` and route disagreement to review.
D. Yes — but only when N=2 (the API limits per-document submissions to 2).

> [!success]- Reveal answer
> **Correct: C**
>
> Batches and multi-instance review compose naturally: each document gets N independent batch entries with distinct `custom_id`s. The 50% cost savings of Batches + the disagreement signal from multi-instance review together produce a robust, cost-effective high-volume pipeline.
>
> **Why others are wrong:**
> - A: Each batch entry runs independently; that's exactly the independence multi-instance review needs.
> - B: Batches isn't real-time, but multi-instance review doesn't require real-time — it requires independence.
> - D: No per-document submission limit beyond the overall batch size (100K requests max).

---

## Answer key summary

| Q | A | Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | A | 6 | C | 11 | B | 16 | D | 21 | A | 26 | C |
| 2 | C | 7 | B | 12 | D | 17 | A | 22 | C | 27 | B |
| 3 | B | 8 | D | 13 | A | 18 | C | 23 | B | 28 | D |
| 4 | D | 9 | A | 14 | C | 19 | B | 24 | D | 29 | A |
| 5 | A | 10 | C | 15 | B | 20 | D | 25 | A | 30 | C |

## Stats summary

- **Answer distribution:** A=8 (27%) · B=7 (23%) · C=8 (27%) · D=7 (23%)
- **Hardest questions** (subtle distinctions, two-look-right options): Q3 (vagueness comparison), Q4 (prompt vs schema), Q14 (required + nullable vs optional), Q16 (when retry helps), Q23 (Batches misapplication), Q30 (Batches + multi-instance review composition)
- **Trap questions:** Q23 (looks like a cost-saving win, tests SLA trap), Q27 (looks reasonable to self-review but fails on independence), Q28 (anchoring rationale, not just "no")
- **Verbatim-fact traps:** Q5 (3-5 examples), Q12 (beta header `structured-outputs-2025-11-13`), Q21 (50%), Q22 (24 hours), Q24 (`custom_id` constraints), Q26 (29 days)

---

*Domain 4 weight: **20%** of the exam. Companion roadmap: [[cca_domain4_roadmap.md]]. Companion exercises: [[cca_domain4_exercises.md]].*
