---
title: CCA-F Domain 3 — Flashcards
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - flashcards
  - domain-3
domain: 3
domain-weight: 20
total-cards: 42
---

# CCA-F Domain 3 — Flashcards

**Claude Code Configuration & Workflows · 20% of exam**

---

## Task 3.1 — CLAUDE.md hierarchy

> [!question]- What are the three levels of CLAUDE.md hierarchy, in load order?
> 1. **User** — `~/.claude/CLAUDE.md` (personal, applies everywhere)
> 2. **Project** — `<project-root>/CLAUDE.md` (committed, team-shared)
> 3. **Directory** — `<subdir>/CLAUDE.md` (scoped to that subtree)
> Lower levels add to (don't replace) higher levels.

> [!question]- What's CLAUDE.md for?
> Persistent project context loaded into every Claude Code session: coding conventions, project structure, testing standards, fixture conventions, review criteria, anything the agent should know without being re-told each session.

> [!question]- What's the `@import` directive in CLAUDE.md?
> Reference another markdown file's content from CLAUDE.md. Useful for splitting a long CLAUDE.md into modules (`@import ./conventions.md`).

> [!question]- User-scope vs project-scope CLAUDE.md — what goes where?
> **User-scope:** your personal preferences (shell aliases, preferred testing libraries, personal git workflow)
> **Project-scope:** team conventions (architecture, lint rules, fixture patterns, review criteria)

> [!question]- For CI-invoked Claude Code, where should testing standards / valuable test criteria / available fixtures be documented?
> In **CLAUDE.md** at the project root. CI invocations of Claude Code pick up this context automatically, so test generation respects team conventions.

> [!question]- "Auto memory" in Claude Code — what is it?
> Automatic context persistence Claude Code maintains across sessions for things like recently touched files and recent commands. Distinct from CLAUDE.md (declarative) and from the memory tool (explicit `/memories` directory).

> [!question]- Anti-pattern: putting one-shot facts into CLAUDE.md instead of just telling Claude in the session. Why is this wrong?
> CLAUDE.md is loaded **every session**, paying the token cost forever. One-shot facts should be passed in the session prompt, not added to persistent config.

---

## Task 3.2 — Custom slash commands and skills

> [!question]- Where do project-scoped custom slash commands live?
> `<project-root>/.claude/commands/<name>.md` — committed to the repo, available to everyone.

> [!question]- Where do user-scoped custom slash commands live?
> `~/.claude/commands/<name>.md` — personal, not shared.

> [!question]- What's the difference between a custom command and a custom skill?
> **Command** (`.claude/commands/<name>.md`): a slash command the user types to invoke a pre-canned prompt
> **Skill** (folder with SKILL.md): a model-invocable capability with frontmatter declaring what it does, when to use it, and what tools it can use

> [!question]- Skills frontmatter fields — which ones matter for the exam?
> `description` (what the skill does), `context: fork` (run in isolated context), `agent: Explore` (run via Explore subagent), `disable-model-invocation` (only callable manually), `argument-hint`, `paths` (scope to file paths), `allowed-tools`, `model`, `hooks`.

> [!question]- `context: fork` in a skill — what does it do?
> Runs the skill in a forked, isolated context — main session's context isn't polluted by the skill's working memory. Useful for "go investigate X" tasks.

> [!question]- `agent: Explore` in a skill — what does it do?
> Routes the skill to the Explore subagent, which is built for verbose discovery without filling the main session's context.

> [!question]- `disable-model-invocation: true` in a skill — when to use?
> When you want the skill callable only via explicit user invocation (slash command or button), not auto-selected by the model. Useful for destructive or expensive operations.

> [!question]- `allowed-tools` in skill frontmatter — what's the security implication?
> Whitelist of tools the skill can use. Skill cannot call tools outside the whitelist even if they exist in the session. Principle of least privilege for skills.

> [!question]- Scenario: build a `/review` slash command that runs your team's standard code review checklist. Where does it live?
> `<project-root>/.claude/commands/review.md` — markdown body is the prompt (the checklist). User types `/review` to invoke.

---

## Task 3.3 — Path-specific rules

> [!question]- Where do path-scoped rules live?
> `.claude/rules/<name>.md` — with YAML frontmatter declaring path scope.

> [!question]- How do you scope a rule to specific paths?
> YAML frontmatter with a `paths` field containing glob patterns. Rule applies only when the agent works within matching paths.

> [!question]- Scenario: React components in `/components/**/*.tsx` use functional components with hooks; Python in `/backend/**/*.py` uses type hints. Where do these conventions go?
> Two path-scoped rules in `.claude/rules/`:
> 1. One with `paths: ["components/**/*.tsx"]` documenting React conventions
> 2. One with `paths: ["backend/**/*.py"]` documenting Python conventions
> Each loads conditionally based on the file the agent is touching.

> [!question]- Why use path-scoped rules instead of a single CLAUDE.md with all conventions?
> Token cost. CLAUDE.md loads every session. Path-scoped rules load only when the agent is in that path — so per-language conventions don't bloat unrelated sessions.

> [!question]- Can path-scoped rules contradict CLAUDE.md?
> They add to it. For genuine overrides, the path-scoped rule should explicitly state the override. Conflicting silent contradictions are an anti-pattern.

---

## Task 3.4 — Plan mode vs direct execution

> [!question]- When to use **plan mode**?
> Tasks with **architectural implications**: microservice restructuring, library migrations affecting many files, choosing between integration approaches with downstream effects, large refactors (10+ files), tasks where the wrong approach is expensive to undo.

> [!question]- When to use **direct execution**?
> Well-understood changes with **clear scope**: single-file bug fix with a clear stack trace, adding a conditional, renaming a variable, anything where the path forward is obvious and reversal is cheap.

> [!question]- Scenario: monolithic app → microservices restructure. Plan mode or direct execution?
> **Plan mode.** Architectural implications, many files affected, multiple valid approaches with different trade-offs. You want a written plan first to evaluate before executing.

> [!question]- Scenario: single-file null check fix with a clear stack trace. Plan mode or direct execution?
> **Direct execution.** Trivial scope, clear path, easy to revert. Plan mode would add overhead for no benefit.

> [!question]- 70-file migration from CommonJS to ES modules — plan mode?
> **Yes — plan mode.** Migration affects many files, has subtle interactions (default imports, named imports, dynamic imports), and a wrong approach costs a lot to undo. Get the plan written and reviewed before touching code.

> [!question]- What's the Explore subagent for in plan mode?
> Verbose discovery phases. Explore investigates the codebase (file structure, function usage, dependencies) without filling the main session's context. Its summary returns to the main session.

> [!question]- Anti-pattern: using plan mode for trivial changes. Why is this wrong?
> Wastes tokens and adds friction. Plan mode's value is for non-obvious paths; for obvious paths it's noise.

---

## Task 3.5 — Iterative refinement

> [!question]- "Test-driven iteration" with Claude — what's the pattern?
> Write the **test suite first** (defining expected behavior), then iterate by sharing **test failures** with Claude as progressive feedback. Each iteration uses test results to guide the next change.

> [!question]- Concrete input/output examples vs prose descriptions in iteration prompts — which is more effective?
> **Concrete I/O examples.** Prose descriptions get interpreted inconsistently across iterations. Examples anchor the expected transformation precisely.

> [!question]- When to provide all issues in a single message vs fixing them sequentially?
> **All at once** when the issues **interact** (fix A might affect fix B; you want Claude to see them together).
> **Sequentially** when issues are **independent** (separate concerns; one at a time keeps each change reviewable).

> [!question]- The "interview pattern" for unfamiliar domains — what is it?
> Before implementation, ask Claude to surface design considerations: cache invalidation strategies, failure modes, edge cases, performance implications. This catches gotchas before code is written.

> [!question]- What's the documented "best way to communicate expected transformations" when prose is interpreted inconsistently?
> **Concrete input/output examples.** Show Claude an input and the expected output. Examples are unambiguous; prose isn't.

> [!question]- Anti-pattern: iterating on output by saying "make it better" / "improve this." Why is this wrong?
> "Better" is undefined. The model guesses. Each iteration drifts in an unpredictable direction. Specific criteria ("reduce duplication in lines 30-50," "add error handling for the timeout case") get specific results.

---

## Task 3.6 — CI/CD integration

> [!question]- What's the Claude Code CLI flag for non-interactive (CI-friendly) mode?
> `-p` / `--print` — runs Claude Code without the interactive REPL, prints the response and exits. Required for CI.

> [!question]- What's `--output-format json` for?
> CI-friendly structured output. Claude Code prints JSON instead of formatted markdown — parseable by CI scripts.

> [!question]- What's `--json-schema <path>` for?
> Constrain Claude Code's JSON output to a specific schema. CI can then validate without parsing free-form text.

> [!question]- Scenario: `claude "Analyze this PR for security issues"` in a CI script but the job fails. Most likely cause?
> The CLI needs `-p` / `--print` to run non-interactively. Without it, the CLI tries to start an interactive session and the CI job times out or fails.

> [!question]- How does CI-invoked Claude Code get project context (conventions, testing standards)?
> **CLAUDE.md** at the project root — loaded automatically. Same mechanism as interactive use.

> [!question]- Why document **valuable test criteria** in CLAUDE.md for CI test generation?
> Without criteria, Claude generates many low-value tests (trivial assertions, redundant cases). With criteria ("test edge cases, boundary conditions, error paths; skip trivial getter/setter tests"), test quality and signal go way up.

> [!question]- For pre-merge PR review in CI: which Batches API vs synchronous?
> **Synchronous.** Pre-merge CI blocks a PR with a 90-second feedback expectation. Batches can take 24 hours — that breaks the workflow.

> [!question]- For nightly test generation across the codebase: Batches API or synchronous?
> **Batches.** Non-blocking, latency-tolerant (results by morning standup is fine). 50% cost savings on a large workload.

---

*Domain 3 weight: 20% of exam. Companion roadmap: [[cca_domain3_roadmap]]. Companion practice exam: [[cca_domain3_practice_v2]].*
