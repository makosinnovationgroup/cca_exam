---
title: Claude Certified Architect — Foundations · Progress Tracker
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - tracker
status: in-progress
created: 2026-05-08
exam-target-date: 
overall-readiness: not-ready
last-practice-score: 12/18
last-practice-date: 2026-05-08
last-practice-percent: 67
target-score-percent: 85
weak-spots: ["1.4", "2.4", "3.2", "3.4", "5.1", "5.6"]
---

# CCA-F Progress Tracker

> [!info] Linked notes
> [[cca_exam_guide]] · [[cca_docs_roadmap]]

## Status Dashboard

- **Exam target date:** _set when you book it_
- **Last practice score:** 12 / 18 = 67% (v1 bank, cold attempt)
- **Target score:** 85%+ before sitting the real exam
- **Current weak spots:** Tasks 1.4, 2.4, 3.2, 3.4, 5.1, 5.6
- **Overall readiness:** 🟡 Not ready · update to 🟢 Ready when all gates below are green

### Readiness Gates

- [x] All 8 Anthropic Academy courses completed
- [ ] All Priority 1–5 docs in [[cca_docs_roadmap]] read
- [ ] Exercises 1, 2, 3, and 4 from the exam guide built end-to-end
- [ ] Practice Bank v2 attempted twice with 85%+ on round 2
- [ ] Official 60-question Skilljar practice exam completed with 85%+
- [ ] All 30 task statements rated 4 or 5 on confidence
- [ ] No more than 2 weak spots remaining

---

## Domain Confidence

Update after each major study block. Rating: **1 = unfamiliar · 2 = recognize · 3 = can answer MCQ · 4 = could explain · 5 = could teach**.

| Domain | Weight | Initial | Current | Notes |
|--------|--------|---------|---------|-------|
| 1 — Agentic Architecture & Orchestration | 27% | 3 | 3 | |
| 2 — Tool Design & MCP Integration | 18% | 4 | 4 | Strong from AzureConduit work |
| 3 — Claude Code Configuration & Workflows | 20% | 2 | 2 | |
| 4 — Prompt Engineering & Structured Output | 20% | 3 | 3 | |
| 5 — Context Management & Reliability | 15% | 2 | 2 | |

confidence-d1:: 3
confidence-d2:: 4
confidence-d3:: 2
confidence-d4:: 3
confidence-d5:: 2

---

## Task Statement Mastery

One row per task statement. Tick the checkbox when you've studied AND drilled it. Update confidence as you go.

### Domain 1 — Agentic Architecture & Orchestration (27%)

- [ ] **1.1** Design and implement agentic loops · `stop_reason` control flow · [[cca_exam_guide#^task-1-1]] · confidence: _3_
- [ ] **1.2** Coordinator-subagent orchestration · hub-and-spoke · iterative refinement · [[cca_exam_guide#^task-1-2]] · confidence: _3_
- [ ] **1.3** Subagent invocation · `Task` tool · explicit context passing · parallel spawning · [[cca_exam_guide#^task-1-3]] · confidence: _3_
- [ ] **1.4** ⚠️ **Multi-step workflows with enforcement** · prerequisite hooks · multi-concern decomposition · [[cca_exam_guide#^task-1-4]] · confidence: _2_
- [ ] **1.5** Agent SDK hooks · `PostToolUse` · pre-call interception · [[cca_exam_guide#^task-1-5]] · confidence: _3_
- [ ] **1.6** Task decomposition · prompt chaining vs dynamic · per-file vs cross-file passes · [[cca_exam_guide#^task-1-6]] · confidence: _3_
- [ ] **1.7** Session state · `--resume` · `fork_session` · resume vs fresh-with-summary · [[cca_exam_guide#^task-1-7]] · confidence: _3_

### Domain 2 — Tool Design & MCP Integration (18%)

- [ ] **2.1** Tool interface design · differentiating descriptions · splitting generic tools · [[cca_exam_guide#^task-2-1]] · confidence: _4_
- [ ] **2.2** Structured error responses · `errorCategory` · `isRetryable` · access vs empty results · [[cca_exam_guide#^task-2-2]] · confidence: _3_
- [ ] **2.3** Tool distribution · 4-5 per agent · `tool_choice` (auto/any/forced) · [[cca_exam_guide#^task-2-3]] · confidence: _4_
- [ ] **2.4** ⚠️ **MCP server scoping** · `.mcp.json` (project) vs `~/.claude.json` (user) · env var expansion · [[cca_exam_guide#^task-2-4]] · confidence: _2_
- [ ] **2.5** Built-in tools · Grep/Glob/Read/Write/Edit · Read+Write fallback for Edit · [[cca_exam_guide#^task-2-5]] · confidence: _4_

### Domain 3 — Claude Code Configuration & Workflows (20%)

- [ ] **3.1** `CLAUDE.md` hierarchy · user/project/directory · `@import` · `.claude/rules/` · [[cca_exam_guide#^task-3-1]] · confidence: _3_
- [ ] **3.2** ⚠️ **Custom commands and skills** · `SKILL.md` frontmatter · `context: fork` · `allowed-tools` · `argument-hint` · [[cca_exam_guide#^task-3-2]] · confidence: _2_
- [ ] **3.3** Path-specific rules · YAML frontmatter `paths:` glob patterns · [[cca_exam_guide#^task-3-3]] · confidence: _2_
- [ ] **3.4** ⚠️ **Plan mode vs direct execution** · architectural complexity test · Explore subagent · [[cca_exam_guide#^task-3-4]] · confidence: _2_
- [ ] **3.5** Iterative refinement · I/O examples · test-driven · interview pattern · [[cca_exam_guide#^task-3-5]] · confidence: _3_
- [ ] **3.6** CI/CD integration · `-p` flag · `--output-format json` · `--json-schema` · [[cca_exam_guide#^task-3-6]] · confidence: _2_

### Domain 4 — Prompt Engineering & Structured Output (20%)

- [ ] **4.1** Explicit criteria · categorical not vague · severity with examples · [[cca_exam_guide#^task-4-1]] · confidence: _3_
- [ ] **4.2** Few-shot prompting · 2-4 targeted examples · false positive reduction · [[cca_exam_guide#^task-4-2]] · confidence: _3_
- [ ] **4.3** Structured output · `tool_use` with JSON schemas · nullable to prevent fabrication · [[cca_exam_guide#^task-4-3]] · confidence: _3_
- [ ] **4.4** Validation-retry loops · error feedback · retry limits when info absent · [[cca_exam_guide#^task-4-4]] · confidence: _3_
- [ ] **4.5** Batch processing · 50% / 24-hour · `custom_id` · NOT for blocking workflows · [[cca_exam_guide#^task-4-5]] · confidence: _2_
- [ ] **4.6** Multi-instance review · independent reviewer pattern · per-file + cross-file · [[cca_exam_guide#^task-4-6]] · confidence: _3_

### Domain 5 — Context Management & Reliability (15%)

- [ ] **5.1** ⚠️ **Manage conversation context** · case-facts blocks · trim verbose tool outputs · lost-in-the-middle · [[cca_exam_guide#^task-5-1]] · confidence: _2_
- [ ] **5.2** Escalation patterns · explicit request → immediate · sentiment is unreliable · [[cca_exam_guide#^task-5-2]] · confidence: _3_
- [ ] **5.3** Error propagation · structured context · access vs empty results · [[cca_exam_guide#^task-5-3]] · confidence: _3_
- [ ] **5.4** Large codebase context · scratchpad files · subagent delegation · `/compact` · [[cca_exam_guide#^task-5-4]] · confidence: _2_
- [ ] **5.5** Human review · stratified sampling · field-level confidence calibration · [[cca_exam_guide#^task-5-5]] · confidence: _2_
- [ ] **5.6** ⚠️ **Provenance preservation** · claim-source mappings · conflict annotation · temporal data · [[cca_exam_guide#^task-5-6]] · confidence: _2_

> [!warning] Weak-spot tasks (⚠️)
> The 6 tasks marked ⚠️ are your current focus areas based on the v1 practice round. When all 6 reach confidence ≥ 4, remove the warning markers.

---

## Resource Completion

### Anthropic Academy Courses

- [x] Building with the Claude API
- [x] Introduction to Model Context Protocol
- [x] Model Context Protocol: Advanced Topics
- [x] Introduction to Subagents
- [x] Introduction to Agent Skills
- [x] Claude Code 101
- [x] Claude Code in Action
- [x] AI Capabilities and Limitations

### Docs Reading (from [[cca_docs_roadmap]])

- [ ] Priority 1 — Claude Agent SDK pages
- [ ] Priority 2 — Tool Use & MCP pages
- [ ] Priority 3 — Claude Code Configuration pages
- [ ] Priority 4 — API & Structured Output pages
- [ ] Priority 5 — Context Management pages

### Hands-On Exercises (from exam guide pages 33-36)

- [ ] **Exercise 1** — Multi-tool agent with escalation logic (`PreToolUse` hook for $500 cap, multi-concern handling)
- [ ] **Exercise 2** — Claude Code team workflow (CLAUDE.md hierarchy, `.claude/rules/` glob patterns, skill with all 3 frontmatter fields, MCP servers)
- [ ] **Exercise 3** — Structured extraction pipeline (JSON schema, validation-retry loop, batch processing, human review routing)
- [ ] **Exercise 4** — Multi-agent research pipeline (parallel `Task` calls, claim-source mappings, error propagation, conflict annotation)

### Practice Question Banks

- [x] Exam guide sample questions (12) — _attempted_
- [ ] Exam guide sample questions (12) — _re-attempted with 100%_
- [x] Practice Bank v1 (18) — _12/18 cold_
- [ ] Practice Bank v1 (18) — _re-attempted_
- [ ] Practice Bank v2 (54) — _round 1 cold_
- [ ] Practice Bank v2 (54) — _round 2 (misses only)_
- [ ] Rick Hightower 60-question article (Towards AI) — free
- [ ] Tutorials Dojo CCA-F practice exams — paid, ~$15
- [ ] Udemy 360-question set (60 per scenario) — paid
- [ ] **Official Skilljar 60-question practice exam** — save for the final week

---

## Practice Score Log

Add a row after every practice attempt. Goal: trend up to 85%+.

| Date | Source | Raw | % | Time | Notes |
|------|--------|-----|---|------|-------|
| 2026-05-08 | Practice Bank v1 (cold) | 12/18 | 67% | _untimed_ | Misses: Q2, 4, 6, 8, 11, 14 — Tasks 5.1, 1.4, 3.4, 3.2, 5.6, 2.4 |
| | | | | | |
| | | | | | |
| | | | | | |

---

## Miss Log

Every question you miss goes here. Pattern recognition is the point — if a task statement shows up 3+ times, it's a study target.

| Date | Q# | Source | Task | Reason | Resolved |
|------|----|----|------|--------|----------|
| 2026-05-08 | Q2 | v1 | 5.1 | Didn't reach for case-facts pattern | [ ] |
| 2026-05-08 | Q4 | v1 | 1.4 | Defaulted to "ask user" instead of decomposing | [ ] |
| 2026-05-08 | Q6 | v1 | 3.4 | Picked plan mode for a single null check | [ ] |
| 2026-05-08 | Q8 | v1 | 3.2 | Didn't recognize `context: fork` frontmatter | [ ] |
| 2026-05-08 | Q11 | v1 | 5.6 | Didn't think to make claim-source mappings structural | [ ] |
| 2026-05-08 | Q14 | v1 | 2.4 | Forgot `~/.claude.json` was the user-level path | [ ] |

**Reason categories:** _knowledge gap_ · _careless_ · _ambiguous Q_ · _option seemed plausible_ · _confused similar concepts_

---

## Study Session Log

Quick log per session. Brief is fine — the goal is to see momentum.

### 2026-05-08

- Completed all 8 Skilljar courses
- First cold attempt of v1 practice bank: 12/18
- Diagnosed 6 weak task statements (1.4, 2.4, 3.2, 3.4, 5.1, 5.6)
- Built v2 bank with 54 questions for next round

### Template for new entries

```
### YYYY-MM-DD

- What I read/built/practiced
- Score if applicable
- Insights or stuck points
- Next session plan
```

---

## Final Readiness Checklist

Don't book the exam until everything below is checked.

### Knowledge

- [ ] Every task statement above rated 4 or 5
- [ ] Can recite from memory: `stop_reason` values and what they mean
- [ ] Can recite from memory: `.mcp.json` vs `~/.claude.json` scope rules
- [ ] Can recite from memory: Message Batches API tradeoffs (50%, 24h, no SLA, no multi-turn tools)
- [ ] Can recite from memory: when to use plan mode vs direct execution
- [ ] Can explain hooks vs prompt-based enforcement decision in one sentence
- [ ] Can explain claim-source mapping pattern in one paragraph

### Practice

- [ ] Practice Bank v2 round 2: 85%+
- [ ] Official Skilljar practice exam: 85%+ (saved for final week)
- [ ] At least one paid third-party set completed (Tutorials Dojo or Udemy)
- [ ] All 12 official sample questions: 100%

### Logistics

- [ ] Anthropic Partner Network access confirmed (or alternative arranged via Alphabyte)
- [ ] Exam date booked
- [ ] Quiet space and reliable internet for exam day confirmed
- [ ] Government ID ready (if proctored requires it)

### Day-Before

- [ ] Light review only — no new material
- [ ] Skim the [[cca_exam_guide]] domain weighting table
- [ ] Skim the Technologies and Concepts appendix
- [ ] Sleep 7+ hours

---

## Quick Links

- [[cca_exam_guide]] — full exam guide with all 30 task statements
- [[cca_docs_roadmap]] — docs reading roadmap with hands-on
- v1 study plan docx — original prep plan
- v2 practice bank docx — 54 questions with diagnostic matrix

---

*Update this file after every study session. Even a single checked box counts.*
