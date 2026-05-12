---
title: CCA-F Flashcards — Master Index
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - flashcards
total-cards: 275
---

# CCA-F Flashcards — Master Index

275 click-to-reveal flashcards covering all testable content in the Claude Certified Architect — Foundations exam.

## Format

Each card is a native Obsidian collapsible callout:

```markdown
> [!question]- Q: What's the cost saving for Batches API vs standard?
> 50% off standard input/output token rates
```

- **Click the chevron** to reveal the answer
- **Tap "Collapse all" in Obsidian** (Cmd/Ctrl+Shift+P → "Toggle fold all") to reset between passes
- The cards work natively in any markdown viewer that supports GFM callouts

If you want **spaced-repetition with intervals** (Anki-style), install the **"Spaced Repetition"** community plugin in Obsidian. It will pick up these cards if you add `#flashcards/cca-f` tags to the file or use the plugin's question-prefix syntax. Native callouts are 90% as good without the dependency.

---

## Decks by domain

| Deck | Domain | Weight | Cards |
|------|--------|--------|-------|
| [[cca_domain1_flashcards]] | Agentic Architecture & Orchestration | 27% | 47 |
| [[cca_domain2_flashcards]] | Tool Design & MCP Integration | 18% | 43 |
| [[cca_domain3_flashcards]] | Claude Code Configuration & Workflows | 20% | 42 |
| [[cca_domain4_flashcards]] | Prompt Engineering & Structured Output | 20% | 69 |
| [[cca_domain5_flashcards]] | Context Management & Reliability | 15% | 74 |

**Total: 275 cards**

---

## Recommended drill plan

### Week 1 — First pass
- [ ] **Day 1:** Domain 1 (47 cards) — cold, mark misses
- [ ] **Day 2:** Domain 2 (43 cards) — cold, mark misses
- [ ] **Day 3:** Domain 3 (42 cards) — cold, mark misses
- [ ] **Day 4:** Domain 4 (69 cards) — cold, mark misses
- [ ] **Day 5:** Domain 5 (74 cards) — cold, mark misses
- [ ] **Day 6:** Re-drill all missed cards across all 5 domains
- [ ] **Day 7:** Rest

### Week 2 — Consolidation
- [ ] **Day 1:** Random-order pass through Domain 1 + 4 (highest weight)
- [ ] **Day 2:** Random-order pass through Domain 2 + 3
- [ ] **Day 3:** Random-order pass through Domain 5
- [ ] **Day 4:** Re-drill missed cards
- [ ] **Day 5:** Full deck cold pass — target 90%+ recall
- [ ] **Day 6:** Re-drill any remaining gaps
- [ ] **Day 7:** Final cold pass

### Week 3 — Final tuning
Drill only the cards you're still missing. By this point, the practice exams (300+ questions across all 5 domains) are your main tool.

---

## Drill mechanics in Obsidian

**To reset cards (collapse all answers):**
- Mac: `Cmd + Shift + P` → "Toggle fold all"
- Win/Linux: `Ctrl + Shift + P` → "Toggle fold all"

**To mark a missed card:**
Add a `#missed-d1` (or appropriate domain tag) above the card. Use Obsidian search `tag:#missed-d1` to re-drill only your misses.

**To check completion:**
Each deck has check-list items at the top. Tick them off as you complete passes.

---

## What each deck covers

### Domain 1 — Agentic Architecture & Orchestration (27%)
The largest weight on the exam. Cards cover: agentic loop mechanics (`stop_reason`, tool result accumulation, model-driven vs scripted), coordinator-subagent patterns (hub-and-spoke, isolated context, iterative refinement), subagent configuration (`AgentDefinition`, `forkSession`, the three built-in subagents), multi-step workflow enforcement (programmatic prerequisites vs prompt-level), Agent SDK hooks (`PreToolUse`/`PostToolUse`, `permissionDecision`), task decomposition (prompt chaining vs dynamic), session management (`continue`, `resume`, `forkSession`, when to use each, when to start fresh).

### Domain 2 — Tool Design & MCP Integration (18%)
Cards cover: tool description quality as primary routing mechanism, the four elements of good descriptions (input formats, example queries, edge cases, boundaries), splitting vs consolidating tools, structured MCP error responses (3 error categories, `isRetryable`, partial results), tool distribution across agents (least privilege, scoped cross-role tools), `tool_choice` options and extended-thinking compatibility, MCP server configuration (`.mcp.json`, project vs user scope, env vars), built-in tools (Read/Write/Edit/Bash/Grep/Glob) and when to use each.

### Domain 3 — Claude Code Configuration & Workflows (20%)
Cards cover: CLAUDE.md three-level hierarchy (user/project/directory) and `@import`, custom slash commands and skills with frontmatter (`context: fork`, `allowed-tools`, `paths`, etc.), path-scoped rules (`.claude/rules/` with glob patterns), plan mode vs direct execution decision criteria, iterative refinement patterns (test-driven iteration, interview pattern, concrete I/O examples over prose), CI/CD integration with `--print` / `--output-format json` / `--json-schema`.

### Domain 4 — Prompt Engineering & Structured Output (20%)
Cards cover: clear-and-direct prompting (the four elements, the "golden rule"), false-positive category handling (surgical disable + improve, not blanket changes), few-shot prompting (3-5 XML-wrapped examples, when examples vs schema), structured output via `tool_use` vs `output_config.format`, nullable fields to prevent fabrication, `"other"` + detail enum pattern, strict mode (syntax) vs semantic validation, CoT field placement, the Batches API in depth (50% / 24h / no SLA / `custom_id` / 29 days / 100K / 256MB / non-blocking decision rule), multi-pass review architecture (per-file + integration), `detected_pattern` instrumentation for FP analysis.

### Domain 5 — Context Management & Reliability (15%)
Heaviest on patterns and anti-patterns. Cards cover: context rot, lost-in-the-middle effect and mitigations (front-load, headers, restate at end), case facts vs conversation summary vs memory tool, compaction and context editing beta headers and triggers, the documented escalation trigger taxonomy (customer request / policy gap / no progress) and the anti-patterns (sentiment, complexity, error-once), structured error propagation (3 categories), synthesis coverage annotations and provenance (claim-source mappings with publication dates), confidence-based human-review routing (field-level vs record-level), context degradation diagnostic signals, scratchpad pattern, the cross-cutting principles ("prose loses, schema preserves" / "annotate, don't reconcile" / "field-level over record-level").

---

## Sample question cross-reference

The 12 official sample questions from the exam guide map to these cards:

| Sample Q | Card coverage |
|---|---|
| Q1 (programmatic prerequisite) | D1 Task 1.4 cards |
| Q2 (tool descriptions) | D2 Task 2.1 cards |
| Q3 (first-contact resolution) | D1 Task 1.4 + D3 Task 3.5 |
| Q4 (`/review` slash command) | D3 Task 3.2 cards |
| Q5 (monolithic restructure) | D3 Task 3.4 cards |
| Q6 (path-specific rules) | D3 Task 3.3 cards |
| Q7 (synthesis report) | D5 Task 5.5 + 5.6 cards |
| Q8 (subagent timeout) | D1 Task 1.5 + D5 Task 5.3 |
| Q9 (claim-source mapping) | D5 Task 5.6 cards |
| Q10 (CI/CD pipeline) | D3 Task 3.6 cards |
| Q11 (Batches API for cost) | D4 Task 4.5 cards (heavy) |
| Q12 (14-file PR review) | D4 Task 4.6 cards |

---

*Companion materials: [[cca_exam_guide]], [[cca_docs_roadmap]], [[cca_progress]], plus per-domain roadmaps, practice exams, and exercise files.*
