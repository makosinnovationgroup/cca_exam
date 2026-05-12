---
title: Claude Certified Architect — Foundations · Domain 4 Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - roadmap
  - domain-4
status: in-progress
domain: 4
domain-name: "Prompt Engineering & Structured Output"
domain-weight: 20
primary-scenarios: ["Structured Data Extraction", "Multi-Agent Research"]
estimated-hours: 5
coverage-level: comprehensive
---

# Domain 4 Roadmap — Prompt Engineering & Structured Output

## What this domain tests

How to **engineer prompts**, **enforce structured output**, **run validation-retry loops**, **process documents at scale via the Batches API**, and **use multi-instance review**. Tested most heavily in Scenario 6 (Structured Data Extraction) and parts of Scenario 3.

The exam rewards moving constraints out of natural language and into the protocol: `tool_use` schemas or `output_config.format` over "respond in JSON" instructions, nullable fields over "omit if missing," typed enums over free-text categories.

## Domain weight

**20% of the total exam.** Domain 4 is unusually concrete — testable facts are quotable verbatim (Batches API constants, schema field rules, retry limits).

> [!important] New API surface
> Anthropic released **native structured outputs** in beta (header `structured-outputs-2025-11-13`). This adds `output_config.format` directly on `messages.create()` for JSON output without forcing a tool call. The **old `tool_use`-based pattern still works** and is what the v0.1 exam guide tests; the new pattern is what current production code uses. Know both.

---

## Task statements (6 total)

| Task | What it tests |
|------|---------------|
| 4.1 | Explicit criteria in prompts · "be clear and direct" |
| 4.2 | Few-shot prompting · multi-shot examples |
| 4.3 | Structured output via `tool_use` / `output_config.format` |
| 4.4 | Validation-retry loops with limits |
| 4.5 | Message Batches API · cost vs latency tradeoffs |
| 4.6 | Multi-instance independent review |

---

## Read in full · ~4 hours

### Tool use (API) · ~45 min

- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/overview`
- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/how-tool-use-works`
- [ ] **Read:** `docs.claude.com/en/agents-and-tools/tool-use/implement-tool-use`

The API-side tool use is foundational for Task 4.3 (structured output via tool_use, the "classic" pattern).

**Memorize cold:**
- Two ways to get structured output:
  1. **Tool-use pattern (classic):** Define a tool whose only purpose is to receive structured output. Force the model to call it with `tool_choice: {"type": "tool", "name": "X"}`. The tool's parameters ARE the structured output. (This is what the v0.1 exam guide tests.)
  2. **Native structured outputs (newer):** Use `output_config.format` directly. See structured-outputs page below.
- `strict: true` on tool definitions enables protocol-level schema validation
- Strict mode eliminates **syntax errors** (malformed JSON, missing required fields, type mismatches) — NOT semantic errors

### Structured outputs · ~45 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/structured-outputs`
- [ ] **Re-read for memorization** (this page is heavily tested)

The native structured outputs feature (newer pattern). Read thoroughly.

**Memorize cold (Task 4.3 core facts):**

| Schema feature | Why it matters |
|----------------|----------------|
| **Required fields** | Must be present in every response |
| **Nullable fields** | Allow `null` for fields that may not exist in source — **prevents fabrication** |
| **Enum with `"other"` + detail string** | Constrains known cases; gracefully handles unknowns without forcing the model to invent a label |
| **Strict mode** | Validates against schema at the protocol level |
| **`additionalProperties: false`** | Strict validation — no extra fields allowed |

**Two complementary features (native structured outputs):**
1. **JSON outputs** (`output_config.format`): Get Claude's response in a specific JSON format
2. **Strict tool use** (`strict: true`): Guarantee schema validation on tool names and inputs

Can be used **independently or together** in the same request.

**Memorize:**
- Beta header: `structured-outputs-2025-11-13`
- Old `output_format` parameter is **deprecated**; use `output_config: {format: {...}}` on `messages.create()`
- Recommended SDK approach: `client.messages.parse()` — auto-validates response against schema
- Schemas compile to grammars; **first request: 100-300ms compile overhead; cached for 24 hours**
- Schemas cached separately from message content — caching does NOT receive PHI protections (don't put PHI in schema names/values)
- Supported on: Claude Opus 4.7, 4.6, Sonnet 4.6, 4.5, Opus 4.5, Haiku 4.5
- Complexity limits enforced (24 parameters total across all strict schemas; combined nested objects/optional params interact)
- **Incompatibilities:** doesn't work with citations (returns 400), doesn't work with message prefilling in JSON outputs mode

**The fabrication-prevention pattern (heavily tested):**
- Want to extract `shipping_address` from invoices; some don't have one
- If required + non-nullable → model fabricates one to satisfy schema
- **Correct:** make it nullable (`Optional[str]` or `"type": ["string", "null"]`). Model can return `null` legitimately when field is absent.

**The unknown-category pattern (heavily tested):**
- Want to classify `expense_type` into 5 known categories
- Some inputs will be types you didn't anticipate
- Strict enum (only 5) → model forces unknowns into closest fit (misclassification)
- **Correct:** add `"other"` to enum + require `expense_type_detail: string` when `"other"` returned. Preserves info about edge cases.

### Message Batches API · ~60 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/batch-processing`
- [ ] **Re-read for memorization** (verbatim facts are exam-critical)

Read the **entire** page. The exam tests verbatim facts about cost, timing, capabilities.

**Memorize cold (exam-day facts):**

| Fact | Value |
|------|-------|
| Cost savings vs interactive | **50%** off standard input/output token rates |
| Processing window | **24 hours** maximum (most batches finish in <1 hour) |
| SLA on completion | **None** — could finish in minutes or take the full 24 hours |
| Correlation mechanism | `custom_id` field (1-64 chars, regex `^[a-zA-Z0-9_-]{1,64}$`) — you set; results return with it |
| Multi-turn tool calling | **Not supported** — single-turn only |
| Streaming | **Not supported** for batch requests |
| Batch size limit | **256 MB** per batch; up to **100,000 requests** per batch |
| Order of results | **Not guaranteed** — match by `custom_id` |
| Failure isolation | Failure of one request does NOT affect others |
| Result retention | **29 days** after batch creation (then unviewable) |
| Cache duration | 5-min default; 1-hour extended (useful for batches that take >5min) |
| Beta header for 300K output | `output-300k-2026-03-24` (Opus 4.7, Opus 4.6, Sonnet 4.6 only; Claude API only, not Bedrock/Vertex/Foundry) |

**When the Batches API fits:**
- Processing 50,000 historical documents overnight
- Backlog clearance after a pipeline change
- Cost-sensitive batch jobs (50% savings)
- Large-scale evaluations
- Content moderation (asynchronous)

**When the Batches API does NOT fit:**
- Real-time customer-facing workflows (no SLA, could be 24 hours)
- Multi-turn conversations (single-turn only)
- Tool-using agents that need back-and-forth (single-turn only)
- Anything urgent / interactive
- Streaming UI responses

**Memorize cold — Batches vs synchronous decision rule (asked directly on sample Q11):**

| Use Batches when... | Use synchronous when... |
|---|---|
| Overnight report generation | Pre-merge CI checks (blocks PR) |
| Weekly compliance audit | Customer-facing chat / support |
| Nightly test generation | Any workflow gated by a human |
| Bulk extraction / backfill | Real-time UI / dashboard updates |
| Cost-sensitive non-blocking work | Anything with a sub-hour latency need |

**The "blocking-or-latency" decision rule:** Ask "will a human or downstream system be waiting on this result within the next hour?" If **yes → synchronous** (Batches SLA can be 24 hours; you cannot block a developer's PR or a customer's chat that long). If **no → Batches** (50% savings, no real cost beyond the wait).

Categories of legitimately-non-blocking work the exam treats as Batches-appropriate: overnight reports, weekly audits, nightly test generation, bulk evaluations, content moderation backlogs.

**The chunking pattern:**
- Submit batch of 10,000; 3% fail (300 docs)
- Re-submit just those 300, possibly chunked further to isolate problematic inputs
- Use `custom_id` to correlate failed inputs back to source documents

**Prompt caching + batches:**
- Discounts stack (50% batch + caching discount = bigger savings)
- Cache hits are best-effort for batches (30-98% typical hit rate)
- Use 1-hour cache duration with batches that take >5min

### Prompt engineering — Be clear and direct · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct`

The foundational Task 4.1 page.

**Memorize cold:**
- **Tell the model what you want**, not what to avoid
- **Provide context**: who you are, who you're addressing, why the task matters
- **State explicit criteria** for success
- **Use ordered lists, bullets, headings** to structure complex prompts
- **"The golden rule":** Show your prompt to a colleague with minimal context. If they're confused, Claude will be too.

**Memorize cold — handling high-false-positive categories (developer-trust restoration):**

When a multi-category agent has some categories generating false positives at a rate that erodes user trust, the documented pattern is **surgical, not blanket**:

- **DO:** Temporarily disable the high-false-positive categories while their prompts are being improved. Keep the accurate categories running so trust in working categories is preserved.
- **DON'T:** Apply a global "be more conservative" or "only report high-confidence findings" instruction — these degrade the categories that were working without fixing the broken ones.
- **DON'T:** Lower confidence thresholds globally — same problem; you penalize good categories to dampen bad ones.
- **DON'T:** Add human pre-review across all categories — masks the underlying issue and doesn't scale.

Why this matters for the exam: distractors often offer "global" or "blanket" fixes (more conservative prompts, lower thresholds) that look defensible. The correct answer isolates the bad categories, leaves the good ones alone, and fixes the bad ones offline.

### Prompt engineering — Use examples (multishot prompting) · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting`

Task 4.2.

**Memorize cold:**
- Examples are **one of the most reliable ways** to steer output format, tone, structure
- Include **3-5 examples** for best results
- Wrap examples in `<example>` tags (multiple in `<examples>` parent tag) so Claude distinguishes from instructions
- Examples should be:
  - **Relevant** — mirror actual use case closely
  - **Diverse** — cover edge cases; vary enough that Claude doesn't pick unintended patterns
  - **Structured** — XML-tagged

### Prompt engineering — Chain of thought (CoT) · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought`

Touches Task 4.6 + iterative refinement.

**Memorize cold:**
- CoT prompting reduces errors in math, logic, analysis, complex tasks
- Increases output length and latency — use judiciously
- Three levels of CoT:
  1. **Basic:** "Think step by step" (least powerful, least context cost)
  2. **Guided:** Outline specific steps to follow in thinking
  3. **Structured:** `<thinking>` and `<answer>` XML tags to separate reasoning from final answer (most powerful)
- **Always have Claude OUTPUT its thinking** — silent thinking doesn't happen

### Prompt engineering — Use XML tags · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags`

Heavily testable for Task 4.2 (multi-shot) and prompt-structure scenarios.

**Memorize cold:**
- XML tags help Claude **parse** complex prompts unambiguously
- Use distinct tags for distinct content: `<instructions>`, `<example>`, `<formatting>`, `<context>`, `<input>`, `<output>`
- Especially useful when mixing instructions, context, examples, and variable inputs
- Tags can be nested for hierarchical structure

### Prompt engineering — Chain complex prompts · ~25 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-prompts`

Prompt chaining — distinct from CoT.

**Memorize cold:**
- Prompt chaining = break complex task into smaller, sequential subtasks across separate prompts
- Best for tasks with multiple distinct steps each requiring in-depth thought
- Benefits: accuracy (each subtask gets full attention), clarity (simpler instructions), traceability (debug per step)
- Pattern: Identify subtasks → structure with XML for clear handoffs → each subtask has single clear goal → iterate
- Common pipelines: Research → Outline → Draft → Edit → Format; Extract → Transform → Analyze → Visualize
- For independent subtasks, run in parallel
- Chain with self-review: Generate content → Review → Refine → Re-review

### Claude prompting best practices (single living reference) · ~30 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-prompting-best-practices`

The Anthropic-maintained single reference for prompt engineering with latest models. Covers all techniques in one place. Useful as a check-against-cheatsheet after reading individual pages.

### Prompt engineering — Give Claude a role (system prompts) · ~20 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/system-prompts`

Role prompting via system prompts. Adjacent to Domain 1.2 (coordinator prompts) and Domain 3 (output styles).

**Memorize cold:**
- System prompts set the persona, context, and constraints that apply to the whole conversation
- More effective than embedding role in user message
- Roles should specify: who Claude is, who Claude is addressing, what quality criteria define success, what NOT to do (sparingly)

### Prompt engineering — Prefill Claude's response · ~15 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/prefill-claudes-response`

Useful technique: prefill the start of Claude's response to constrain format.

**Memorize cold:**
- Add an assistant message with partial content; Claude continues from there
- Useful for: forcing JSON output (prefill `{`), forcing specific format (prefill `<analysis>`)
- Doesn't work with native structured outputs JSON mode (incompatibility)
- Works with `tool_use` patterns

### Prompt caching · ~15 min

- [ ] **Read:** `docs.claude.com/en/docs/build-with-claude/prompt-caching`

Per the exam guide: "beyond knowing it exists" is out of scope. Know:
- Reduces input token cost on cached content (10% of base input price for reads; 25% extra for 5-min cache writes; 2x base for 1-hour cache writes)
- Add `cache_control: {"type": "ephemeral"}` to mark a cache breakpoint
- Cache hierarchy: tools → system → messages (in that order)
- 5-min default lifetime; 1-hour extended available
- Min cacheable tokens: 1024 (Opus/Sonnet); 2048 (Haiku)
- Works with batches (discounts stack)

---

## Skim · ~20 min

### Messages API overview

- [ ] **Skim:** `docs.claude.com/en/api/messages`

Request/response shape, roles, content blocks. Orientation.

### Models page

- [ ] **Skim:** `docs.claude.com/en/docs/about-claude/models-overview`

Context for model selection in scenarios. Opus vs Sonnet vs Haiku — when each fits.

### Errors

- [ ] **Skim:** `docs.claude.com/en/api/errors`

Structure of API errors. Touches Domain 2.2.

### Long context tips

- [ ] **Skim:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/long-context-tips`

Adjacent to Domain 5. Tips for >100K-token prompts.

### Extended thinking tips

- [ ] **Skim:** `docs.claude.com/en/docs/build-with-claude/prompt-engineering/extended-thinking-tips`

For tasks using extended thinking.

---

## Skip — out of scope

> [!warning] Don't get pulled in
> The exam guide explicitly lists these as out of scope:

- **Streaming (SSE)** — `docs.claude.com/en/docs/build-with-claude/streaming`
- **Vision / image inputs** — `docs.claude.com/en/docs/build-with-claude/vision`
- **Files API** — peripheral
- **Token counting** algorithms
- **Rate limits / quotas**
- **API pricing** calculations (knowing Batches is 50% cheaper IS in scope; exact dollar math IS NOT)
- **Admin API**
- **Authentication / API key management / OAuth / API key rotation**
- **Embeddings** — explicitly out of scope
- **Workspaces / data residency / API and data retention**
- **Bedrock / Vertex / Azure** provider-specific API pages
- **Citations** — peripheral
- **Search results** — peripheral
- **Multilingual support** — peripheral
- **Adaptive thinking / effort / task budgets / fast mode** — beta features, light familiarity only
- **Reduce hallucinations / Increase output consistency / Mitigate jailbreaks / Streaming refusals / Reduce prompt leak / Keep Claude in character** — useful pages but not exam-tested
- **Prompt generator / Use prompt templates / Prompt improver** — Console tools, operational

---

## Hands-on drills · ~2 hours

- [ ] **Drill 1: Structured extraction with nullable fields** — build an invoice extraction pipeline. Use `tool_use` with a JSON schema containing required fields, nullable fields (for fields that may not exist in source), and an enum with `"other"` + detail string for `expense_type`.

- [ ] **Drill 2: Force a validation failure (retry helps)** — feed an invoice where line items don't sum to total. Implement retry-with-error-feedback loop. Watch it succeed on retry.

- [ ] **Drill 3: Force a hopeless retry (retry can't help)** — feed an invoice where the field is genuinely absent from the source. Implement retry without a limit → watch it fail forever. **Then add limit (e.g., 3) + "mark unknown and escalate" fallback.**

- [ ] **Drill 4: Batches API submission** — submit batch of 20 docs to Batches API. Use `custom_id` to correlate. Re-submit just failed ones with chunking.

- [ ] **Drill 5: Multi-instance independent review** — for the same extracted invoice, run three independent extractions (separate sessions or separate batch entries). Compare. The 3% genuine ambiguity surfaces as disagreement among instances.

- [ ] **Drill 6: Few-shot vs schema comparison** — compare two approaches to the same extraction task: (a) few-shot examples in the prompt, no schema; (b) tool_use with schema, no examples. Measure consistency. The schema approach wins on consistency every time.

- [ ] **Drill 7: Native structured outputs experiment** — use `output_config.format` with `additionalProperties: false`. Compare to tool_use pattern. Note that schemas are cached for 24 hours after first compile.

- [ ] **Drill 8: tool_choice forcing** — for a structured extraction task, set `tool_choice: {"type": "tool", "name": "extract_invoice"}`. Verify the model always calls that tool first.

- [ ] **Drill 9: Strict mode test** — set `strict: true` on a tool. Feed a malformed scenario where Claude might generate invalid JSON. Verify the protocol-level validation prevents it.

- [ ] **Drill 10: XML structure test** — take a vague prompt; restructure it with `<instructions>`, `<context>`, `<example>`, `<output_format>` tags. Compare output quality.

---

## Common confusions and distractor patterns

### Distractor pattern 1: "Ask for JSON in the prompt"

Inconsistent JSON output → distractors offer "append 'respond in valid JSON'" or "use prompt caching to standardize."
**Correct:** `tool_use` with typed schema OR `output_config.format`. Protocol enforcement beats prompt instruction.

### Distractor pattern 2: Batches API misapplied

Real-time scenario + distractor proposes Batches "for cost savings." Reject criteria:
- No SLA (could be 24 hours)
- No multi-turn tool calling
- No real-time flows
- Customer-facing workflows always disqualified

### Distractor pattern 3: Retry without a limit

When validation fails:
- **Right to retry:** error message gives model new info to correct on (e.g., "line items sum to $147.50 but total is $150.00")
- **Wrong to retry:** field genuinely absent from source (no amount of retrying produces data that isn't there)

Correct pattern: retry limit (2-3) + fallback (mark unknown / escalate / skip).

### Distractor pattern 4: Required-and-non-nullable for optional fields

Model fabricating data to satisfy schema requirements:
- Distractor: "Add more prompt instructions to be honest"
- Distractor: "Use stricter schema" (this makes it WORSE)
- **Correct:** make optional fields **nullable**

### Distractor pattern 5: Strict enum forces fabrication

Model misclassifying edge cases:
- Distractor: "Add more categories"
- **Correct:** `"other"` + free-text detail field

### Distractor pattern 6: Same-session "self-review"

Independent quality check:
- Distractor: "Have Claude review its own output in the same session"
- Distractor: "Run twice and use the second result"
- **Correct:** independent review instances (separate sessions, separate batch entries)

### Distractor pattern 7: Few-shot when schema fits better

Strict format compliance scenario:
- Distractor: "Add more few-shot examples"
- **Correct:** schema (tool_use or output_config.format). Examples shape style; schema enforces structure.

### Distractor pattern 8: Single-pass review of many files

Scenario: A PR touches 10+ files; a single-instance, single-pass review missed cross-file bugs (API contract mismatches, broken data flow, race conditions). Distractors offer "longer prompt with stricter checklist," "more checks per file," or "larger context window."

**Correct architecture: two-stage multi-pass review.**
1. **Per-file local passes** — one Claude instance per file with isolated context. Each instance focuses on local issues (syntax, logic, style, single-file correctness). Independence prevents attention dilution; isolated contexts prevent cross-contamination.
2. **Cross-file integration pass** — one instance reviewing inter-file concerns (API contracts, data flow, dependencies, race conditions). Input: the file changes PLUS the structured summaries from the per-file passes. The integration pass operates on a reduced surface (summaries, not full code) so cross-file reasoning has room to work.

Why single-pass fails for many files: attention dilutes as context fills. By file 8 of 14, the model's reasoning quality on file 8 is materially worse than on file 1, even though the file is identical. The per-file split fixes this; the integration pass recovers cross-file analysis that per-file isolation would lose.

Sample Q12 in the official exam guide tests this exact pattern with a "14 files in the stock tracking module" scenario.

### Distractor pattern 9: Instrumenting false-positive analysis

Scenario: Developers dismiss 40% of agent-flagged findings, but logs only contain the finding text. There's no way to know which prompt patterns or code constructs are triggering the false positives. Distractors offer "improve the prompt," "lower thresholds," or "have humans pre-review."

**Correct: structural instrumentation with `detected_pattern` fields.**

Add a `detected_pattern` field (a categorical enum naming which rule/construct triggered the finding) to every structured finding the agent emits. Example: `detected_pattern: "implicit_type_coercion"` or `detected_pattern: "missing_null_check_on_dict_access"`. Now when developers dismiss findings, you can:
- Group dismissals by `detected_pattern` to find which patterns drive false positives
- Disable or rework the prompts associated with high-FP patterns (see Distractor pattern in "Be clear and direct" section)
- Calibrate threshold by category, not globally

Related schema-level instrumentation for **self-correcting validation**:
- Extract `calculated_total` alongside `stated_total` from invoices/receipts → flag discrepancies without a second LLM pass
- Extract a `conflict_detected` boolean when source documents have inconsistent values → coordinator decides reconciliation policy
- These patterns turn validation into a schema obligation, not a probabilistic prompt instruction

Distractors miss this because they reach for *prompt* changes when the problem is *measurement* — you can't fix what you can't categorize.

### Subtle distinction 1: Strict mode vs schema validation

- **Strict mode** (protocol): eliminates syntax errors only (malformed JSON, missing required, type mismatches)
- **Your code's validation**: checks semantic constraints (sums, dates, IDs match registry)
- Strict mode does NOT replace semantic validation. Both needed for production.

### Subtle distinction 2: tool_use vs output_config.format

| Pattern | When to use |
|---------|-------------|
| `tool_use` with forced `tool_choice` | What the v0.1 exam guide tests; widely available; works with extended thinking; tool's params ARE the output |
| `output_config.format` (native structured outputs) | Newer beta (`structured-outputs-2025-11-13`); plain JSON output without forcing a tool; doesn't work with citations or JSON-mode prefilling |
| Both together | Allowed and useful — use strict tool use + output_config.format combined |

### Subtle distinction 3: Few-shot vs structured output

- **Few-shot** for shaping style, tone, format → soft constraints
- **Structured output** for enforcing structure → hard constraints
- Mixing is fine; for strict compliance, schema beats examples

### Subtle distinction 4: CoT when to use

- **Use for:** complex reasoning (math, multi-step analysis, debugging, decisions with risk)
- **Skip for:** simple lookups, formatting, clear-criteria classification
- **Pattern:** `reasoning` field BEFORE `answer` field in schema so chain of thought informs final answer

### Subtle distinction 5: tool_choice with extended thinking

- Extended thinking only works with `tool_choice: "auto"` or `"none"`
- `"any"` and `{"type": "tool"}` both produce errors with extended thinking (they prefill assistant message)

---

## Self-assessment recall tests

### Task 4.1 · Explicit criteria

- [ ] Completed Task 4.1 self-check

1. What four things make a prompt "clear and direct"?
2. When does adding more prompt text fail to fix inconsistent output?
3. What's the "golden rule" of clear prompting?

> [!success]- Answers
> 1. Tell what you want (not what to avoid); provide context; state explicit success criteria; use structure (lists/headings)
> 2. When the problem is structural — fix is moving constraints into schema or `tool_choice`, not more words
> 3. Show your prompt to a colleague with minimal context; if they're confused, Claude will be

### Task 4.2 · Few-shot

- [ ] Completed Task 4.2 self-check

1. How many examples does the docs recommend?
2. How should examples be wrapped?
3. What three criteria define good examples?
4. When should you use few-shot vs structured output?

> [!success]- Answers
> 1. 3-5
> 2. `<example>` tags (multiple in `<examples>` parent)
> 3. Relevant, diverse, structured
> 4. Few-shot for style/format shaping; schema for strict structure enforcement

### Task 4.3 · Structured output

- [ ] Completed Task 4.3 self-check

1. How do you force the model to call a specific tool?
2. What's the documented pattern to prevent fabrication of optional fields?
3. What's the documented pattern for classifying edge cases?
4. Does `strict: true` eliminate semantic errors (e.g., math errors)?
5. What two ways exist to get structured output?

> [!success]- Answers
> 1. `tool_choice: {"type": "tool", "name": "X"}`
> 2. Nullable fields in the schema
> 3. Enum with `"other"` + required detail string when `"other"` returned
> 4. No — only **syntax** errors. Semantic validation is your responsibility.
> 5. Tool-use pattern with forced `tool_choice` (classic); `output_config.format` (native structured outputs, newer beta)

### Task 4.4 · Validation-retry loops

- [ ] Completed Task 4.4 self-check

1. When does retrying after validation failure help?
2. When does retrying NOT help — and what should you do instead?
3. What two components must a robust validation-retry loop have?

> [!success]- Answers
> 1. When the error gives the model new info to correct on
> 2. When data is genuinely absent. Instead: mark unknown / escalate / skip
> 3. Retry limit (2-3) AND fallback path when retries exhaust

### Task 4.5 · Batches API

- [ ] Completed Task 4.5 self-check

1. What's the cost saving vs the standard API?
2. What's the maximum processing window?
3. Is there an SLA on completion?
4. Does it support multi-turn tool calling?
5. How do you correlate input to output?
6. How long are results available after batch creation?
7. What's the maximum batch size (requests and bytes)?

> [!success]- Answers
> 1. 50%
> 2. 24 hours
> 3. None (most finish in <1 hour but no guarantee)
> 4. No — single-turn only
> 5. `custom_id` (1-64 chars, alphanumeric + hyphens + underscores)
> 6. 29 days after creation, then unviewable
> 7. 100,000 requests; 256 MB

### Task 4.6 · Multi-instance review

- [ ] Completed Task 4.6 self-check

1. What makes a review "independent" vs not?
2. Why does same-session review fail as a quality check?

---

## Common scenario patterns by task

**Task 4.1** — A scenario describes a vague prompt producing inconsistent output. Distractors reach for "add more words" or "longer instructions"; the correct answer is explicit success criteria and structured input/output expectations.

**Task 4.2** — A scenario describes a task needing format/tone consistency. Distractors reach for stricter rules or schema only; the correct answer balances few-shot examples (3-5, XML-wrapped) with structured output when format matters.

**Task 4.3** — A scenario describes inconsistent JSON output, fabricated fields, or misclassified edge cases. Distractors reach for prompt-level instructions; the correct answer is structural: tool_use schema, nullable fields, enum+other+detail.

**Task 4.4** — A scenario describes a validation-retry loop that fails forever. Distractors reach for "retry until success" or "fail on first error"; the correct answer is retry limit + fallback when retries can't help.

**Task 4.5** — A scenario describes large-volume processing OR a real-time customer-facing workflow. Distractors swap the two; the correct answer uses Batches API for high-volume non-urgent only, and reaches for synchronous API for real-time/multi-turn/tool-using.

**Task 4.6** — A scenario describes quality assurance or independent review. Distractors reach for same-session self-review; the correct answer is multi-instance independent review (separate sessions or separate batch entries) with disagreement surfacing genuine ambiguity.

---

## Quick-reference cheatsheet

- **Structured output:** `tool_use` with forced `tool_choice` (classic), OR `output_config.format` (newer beta). NOT "respond in JSON" in prompt.
- **`tool_choice`:** `"auto"` (default), `"any"` (force some tool), `{"type": "tool", "name": "X"}` (force specific), `"none"` (disable).
- **Extended thinking:** only `auto` or `none` work with `tool_choice`.
- **Nullable fields prevent fabrication.** Required-and-non-nullable forces invention.
- **Enum + `"other"` + detail string** for unknown handling.
- **`strict: true`** eliminates syntax errors only. You write semantic validation.
- **Schema grammar caching:** 100-300ms first compile; 24-hour cache.
- **Validation-retry:** retry helps when error gives correction info; doesn't help when source lacks data. Always have retry limit + fallback.
- **Batches API memorize:** 50% cost, 24-hour window, no SLA, `custom_id`, NO multi-turn tool calling, NO streaming, 100K requests / 256 MB max, 29-day result retention, results unordered.
- **Batches API NOT for:** real-time, customer-facing, multi-turn tool use, streaming.
- **Beta header for 300K output:** `output-300k-2026-03-24` (Opus 4.7/4.6, Sonnet 4.6, Claude API only).
- **Few-shot examples:** 3-5; wrap in `<example>` (`<examples>` parent); relevant/diverse/structured.
- **XML tags:** `<instructions>`, `<example>`, `<formatting>`, `<context>`, `<input>`, `<output>` for clarity.
- **CoT:** structured (`<thinking>` + `<answer>` XML) is most powerful. Always output thinking. Skip for simple tasks.
- **Prompt chaining** for predictable sequential subtasks; CoT for in-step reasoning. Both can combine.
- **Independent review:** separate sessions; NOT same-session self-review.
- **Multi-instance review:** disagreement among independent runs surfaces genuine ambiguity (~3% in practice).

---

## Progress tracker

- [ ] All "Read in full" pages complete
- [ ] All "Skim" pages reviewed
- [ ] All hands-on drills complete
- [ ] All self-assessment recall tests passed
- [ ] Quick-reference cheatsheet memorized

---

*Domain 4 weight: **20%** of the exam. Concrete facts (Batches API constants, schema patterns) are heavily testable verbatim.*
