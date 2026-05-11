---
title: Claude Certified Architect — Foundations · Domain 3 Practice Exam
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - practice
  - domain-3
status: drilling
total-questions: 30
domain: 3
domain-weight: 20
target-percent: 80
difficulty: harder-than-real-exam
---

# CCA-F Domain 3 — Practice Exam

30 questions covering Claude Code Configuration & Workflows (20% of the real exam). Calibrated **harder than the real exam** — distractors include fabricated-but-plausible frontmatter fields, almost-correct precedence orders, and competing-but-suboptimal config approaches.

## How to use

- [ ] **Round 1** — cold attempt all 30 without checking answers
- [ ] **Round 2** — score using the collapsed callouts
- [ ] **Round 3** — re-attempt any miss after re-reading the relevant roadmap section

**Aim for 80%+ on Round 1.**

## Task statement coverage

| Task | Questions | Focus |
|------|-----------|-------|
| 3.1 | Q1-Q5 | CLAUDE.md hierarchy, `@import`, auto memory |
| 3.2 | Q6-Q12 | Skills, slash commands, frontmatter, subagents |
| 3.3 | Q13-Q16 | Path-specific rules, glob frontmatter |
| 3.4 | Q17-Q21 | Plan mode vs direct execution, Explore/Plan subagents |
| 3.5 | Q22-Q25 | Iterative refinement, interview pattern |
| 3.6 | Q26-Q30 | CI/CD integration, headless mode, schema validation |

---

## Practice questions

### Task 3.1 — CLAUDE.md hierarchy

#### Question 1

A team has team-wide coding standards, individual developer preferences, and a few local overrides per project. They want each layer to contribute its rules with proper precedence. Where should team-wide standards live? ^d3-q-1

A. `~/.claude/CLAUDE.md` so every developer's session loads them automatically.
B. `.claude/CLAUDE.local.md` in the project so they're scoped tightly.
C. A pinned message in the team's Slack so engineers can paste them when needed.
D. `.claude/CLAUDE.md` (or `CLAUDE.md` at the project root) — project-scoped, Git-tracked, and shared with the team.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented hierarchy: Enterprise > Project > User > Local. Team-wide standards belong at the **Project** tier — they travel with the repo, every team member's session loads them, and they're version-controlled.
>
> **Why others are wrong:**
> - A: User tier is personal — installing team standards there means every developer has to manually keep them in sync.
> - B: Local tier is for *personal-per-project* overrides; it's not Git-tracked, so the team would lose synchronization.
> - C: A Slack pin is not a documented mechanism and requires manual paste-in for every session.

#### Question 2

What does the `@import` directive in a `CLAUDE.md` file do? ^d3-q-2

A. Imports built-in Anthropic style guidelines and merges them with the local content.
B. Pulls in the contents of another markdown file at load time, letting `CLAUDE.md` be modular by referencing topic files (e.g., `@import .claude/standards/api-conventions.md`).
C. Defines a TypeScript-style import that's converted into a `require()` at runtime.
D. Imports auto-memory contents into the project-level CLAUDE.md.

> [!success]- Reveal answer
> **Correct: B**
>
> `@import` references another file from `CLAUDE.md`. It solves the "2000-line monolith CLAUDE.md" problem by letting each package's standards file include only relevant topic files.
>
> **Why others are wrong:**
> - A: There's no built-in Anthropic style import.
> - C: `@import` is markdown content, not TypeScript.
> - D: Auto-memory loading is separate from `@import` and doesn't go through CLAUDE.md.

#### Question 3

What's the precedence order of the CLAUDE.md hierarchy, highest to lowest? ^d3-q-3

A. Enterprise > Project > User > Local
B. Local > User > Project > Enterprise
C. Project > Local > User > Enterprise
D. User > Project > Enterprise > Local

> [!success]- Reveal answer
> **Correct: A**
>
> Documented hierarchy: Enterprise (org-mandated) > Project (team-shared) > User (personal) > Local (personal-per-project, not Git-tracked).
>
> **Why others are wrong:**
> - B: Inverted; the documented order is Enterprise > Project > User > Local.
> - C: Local doesn't outrank User in precedence.
> - D: User doesn't outrank Project. Project rules apply to everyone in the project; User rules are personal preferences.

#### Question 4

A developer has Claude Code v2.1.59+ and notices that after working in a project, a `MEMORY.md` file appears at `~/.claude/MEMORY.md` with notes Claude wrote. What's **true** about this file? ^d4-q-4 ^d3-q-4

A. It's a backup of the conversation history that's cleared each session.
B. It's manually edited only — Claude doesn't write to it.
C. It's auto memory — Claude writes to it during sessions; the first 200 lines (or 25 KB) load at the start of every subsequent session. It's distinct from `CLAUDE.md` (which the developer writes).
D. It's a temporary cache and gets cleared whenever the session ends.

> [!success]- Reveal answer
> **Correct: C**
>
> Auto memory is Claude-written based on observed patterns and corrections. The first 200 lines (or 25 KB, whichever comes first) load at session start. Distinct from CLAUDE.md, which the developer writes.
>
> **Why others are wrong:**
> - A: Conversation history isn't preserved across sessions in this file.
> - B: Auto memory is precisely Claude writing — that's the feature.
> - D: It's preserved across sessions; that's the whole point.

#### Question 5

A team wants to add an org-wide compliance policy that all sessions must follow, and they want it to take precedence over any project-level CLAUDE.md. Where does it go? ^d3-q-5

A. `~/.claude/CLAUDE.md` for every employee.
B. A pinned section at the top of every project's `.claude/CLAUDE.md`.
C. A `.claude/COMPLIANCE.md` file with a special `priority: enterprise` frontmatter.
D. The Enterprise tier — an OS-managed location for org-administered CLAUDE.md, which takes precedence over Project, User, and Local.

> [!success]- Reveal answer
> **Correct: D**
>
> The Enterprise tier is the documented mechanism for org-wide mandated policies that must override project and personal preferences.
>
> **Why others are wrong:**
> - A: User tier doesn't override Project. A determined developer could override compliance rules at the Project tier.
> - B: Project-level pinning is brittle — anyone editing the project's CLAUDE.md could remove or modify it.
> - C: `COMPLIANCE.md` and a `priority` frontmatter field are fabricated.

### Task 3.2 — Skills, slash commands, frontmatter

#### Question 6

A team's `dep-analyzer` skill produces 800 lines of grep output when run, flooding the main conversation. Which frontmatter field would **most directly** address this? ^d3-q-6

A. `verbose: false` to suppress output.
B. `context: fork` to run the skill in an isolated subagent context — only the final summary returns to the main session.
C. `disable-model-invocation: true` to keep it manual-only.
D. `allowed-tools: [Grep]` to restrict what the skill can do.

> [!success]- Reveal answer
> **Correct: B**
>
> `context: fork` is the documented field for skill isolation. The skill runs in a forked subagent context; verbose tool calls (Grep across files, Read of many files) happen inside the subagent's context, and only the final summary returns to the main conversation.
>
> **Why others are wrong:**
> - A: `verbose: false` is fabricated.
> - C: `disable-model-invocation: true` controls *invocation* (auto vs manual), not output verbosity.
> - D: Restricting tools doesn't reduce output volume — Grep on its own can produce 800 lines of matches.

#### Question 7

A skill is configured with `context: fork` and `agent: Explore`. What does this combination do? ^d3-q-7

A. The skill runs in an isolated subagent context using the built-in **Explore** subagent (which has read-only tools optimized for codebase research). Only the final summary returns to the main session.
B. The skill runs in the main context but uses the Explore subagent's tool palette.
C. The skill is forked into a worktree-isolated git branch, and the Explore subagent runs the analysis.
D. The skill runs the Explore subagent as a fallback when its `prompt` field is missing.

> [!success]- Reveal answer
> **Correct: A**
>
> `context: fork` isolates the skill's execution; `agent: <type>` chooses which subagent runs it. `Explore` is the built-in read-only research subagent.
>
> **Why others are wrong:**
> - B: `context: fork` is what creates the isolation; without it, the skill runs in main context.
> - C: Worktree isolation is a different field (`isolation: worktree` on subagent definitions), not part of `context: fork`.
> - D: The `prompt` field for skills is the skill's content; there's no "missing prompt" fallback.

#### Question 8

A team has a `deploy` skill that should only run when explicitly invoked (never auto-triggered by intent matching). Which frontmatter field enforces this? ^d8-q-8 ^d3-q-8

A. `user-invocable: true`
B. `auto-invoke: false`
C. `disable-model-invocation: true` — keeps the skill out of the auto-invocation routing list. Only manual `/deploy` triggers it.
D. `manual-only: true`

> [!success]- Reveal answer
> **Correct: C**
>
> `disable-model-invocation: true` is the documented field. Auto-routing won't pick this skill; only explicit user invocation runs it.
>
> **Why others are wrong:**
> - A: `user-invocable: true` is the default — it controls whether the skill appears in the `/` menu, not auto-invocation.
> - B: `auto-invoke: false` is fabricated.
> - D: `manual-only: true` is fabricated.

#### Question 9

In Claude Code, `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both exist. What happens? ^d3-q-9

A. The `commands/` version takes precedence and the skill is shadowed.
B. The skill takes precedence and the command is shadowed.
C. Claude Code refuses to start until one is removed.
D. Slash commands and skills are merged — they share the `/deploy` namespace and either file structure works. The two surfaces aren't independent; they're aliases of the same feature.

> [!success]- Reveal answer
> **Correct: D**
>
> Slash commands and skills are documented as merged. The two storage layouts (`.claude/commands/<name>.md` vs `.claude/skills/<name>/SKILL.md`) both create `/name` and have the same capabilities.
>
> **Why others are wrong:**
> - A, B: There's no precedence rule because they're not in competition — they're aliases.
> - C: Claude Code starts normally.

#### Question 10

A team's subagent definition includes both a `tools: [Read, Grep, Glob]` field and a `disallowedTools: [Bash]` field. Why use both? ^d3-q-10

A. `tools` is sufficient — `disallowedTools` is redundant if `tools` lists the allowed set.
B. `disallowedTools` is the documented explicit denial mechanism — preferring it makes the restriction more visible. If a future update to the subagent adds a tool to `tools`, `disallowedTools` still enforces the denial.
C. `disallowedTools` overrides `tools`, so the agent only sees what's in `disallowedTools`.
D. They're mutually exclusive; using both is a misconfiguration that causes Claude Code to error.

> [!success]- Reveal answer
> **Correct: B**
>
> Explicit denial via `disallowedTools` is documented as preferable to relying on omission from `tools`. It's defense-in-depth: even if the allowed list grows, the explicit denial persists.
>
> **Why others are wrong:**
> - A: `tools` defines the allow set, but explicit denials are clearer and survive future changes to `tools`.
> - C: `disallowedTools` doesn't override `tools` — it acts as an additional denylist.
> - D: They compose, not conflict.

#### Question 11

A developer wants a skill to prompt the user for an argument named `package-name` when invoked. Which frontmatter field provides this? ^d3-q-11

A. `argument-hint: <package-name>` — provides the prompt-text shown when the skill is invoked without an argument; `$ARGUMENTS` in the skill body substitutes the value.
B. `argument-name: package-name` — declares the named argument.
C. `inputs: [{name: "package-name", required: true}]` — fully specifies argument schema.
D. `prompt-arg: package-name` — sets the prompt argument variable.

> [!success]- Reveal answer
> **Correct: A**
>
> `argument-hint` is the documented frontmatter field for argument autocomplete and prompt UX. The hint is shown to the user; `$ARGUMENTS` in the skill content substitutes the value at invocation.
>
> **Why others are wrong:**
> - B, C, D: All fabricated. The real field is `argument-hint`.

#### Question 12

A `SKILL.md` file has `paths: ["src/api/**/*.ts"]` in its frontmatter. What does this mean? ^d3-q-12

A. The skill can only invoke tools matching those paths.
B. The skill is restricted to read/write files under those paths.
C. The skill's auto-activation is **scoped** to those file paths — the skill is more likely to be invoked when the user is working with files matching this glob.
D. The skill loads its `prompt` content from those file paths instead of from SKILL.md.

> [!success]- Reveal answer
> **Correct: C**
>
> The `paths` frontmatter scopes the skill's activation by file glob. When the user is working with matching files, the skill becomes more relevant for auto-invocation routing.
>
> **Why others are wrong:**
> - A: Tool invocations aren't restricted by `paths`.
> - B: File access restriction isn't what `paths` controls.
> - D: The skill body is the SKILL.md content; `paths` doesn't reroute the source.

### Task 3.3 — Path-specific rules

#### Question 13

A team has different conventions for `**/*.test.ts` (jest), `terraform/**/*.tf` (HCL conventions), and `src/api/**/*.ts` (REST conventions). Putting all of these in one CLAUDE.md feels overwhelming. What's the **documented** pattern? ^d3-q-13

A. Split CLAUDE.md by topic and use `@import` to pull each topic file into the main CLAUDE.md.
B. Create a separate `CLAUDE.md` in each subdirectory: `src/api/CLAUDE.md`, `terraform/CLAUDE.md`, etc.
C. Add all conventions to `~/.claude/CLAUDE.md` so each developer customizes their loadout.
D. Use `.claude/rules/<name>.md` files with `paths:` glob frontmatter. Each rule file scopes itself to matching files and loads only when those files are touched.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented path-specific rules mechanism: `.claude/rules/<name>.md` with a `paths:` frontmatter glob. The rule auto-loads when the agent reads matching files; otherwise it stays out of context, reducing bloat.
>
> **Why others are wrong:**
> - A: `@import` always loads its imported content — it doesn't scope by file path. The rules would still be in context for every operation.
> - B: Subdirectory CLAUDE.md is a real thing, but it can't handle cross-directory globs (e.g., `**/*.test.ts` which spans the whole repo).
> - C: User-level config makes the rules personal, not project-scoped.

#### Question 14

A `.claude/rules/api.md` file has `paths: ["src/api/**/*"]`. The agent reads a file at `src/api/users.ts`. What happens to the rule's content? ^d3-q-14

A. The rule's content always loads at session start regardless of which files are touched.
B. The rule's content loads into context when matching files are read, so the agent has its conventions available while working on those files.
C. The rule's content only loads if the user explicitly invokes a `/rules-api` slash command.
D. The rule's content loads at session start but is unloaded when non-matching files are read.

> [!success]- Reveal answer
> **Correct: B**
>
> Path-scoped rules load when matching files are touched. This is the whole point: conventions for files the agent isn't currently working on stay out of context to prevent bloat.
>
> **Why others are wrong:**
> - A: That would defeat the purpose of `paths`-scoping.
> - C: Path-scoped rules don't require explicit invocation.
> - D: Loading is on-demand based on file touches, not unloading.

#### Question 15

A rule file `.claude/rules/tests.md` has `paths: ["**/*.test.ts", "**/*.test.tsx", "**/*.spec.ts"]`. The agent edits `src/auth/login.ts` (a non-test file). What happens? ^d3-q-15

A. The rule doesn't load because none of the globs match `src/auth/login.ts`.
B. The rule loads anyway because it lives in `.claude/rules/`.
C. The rule loads as a tooltip in the UI.
D. The rule causes Claude Code to error because the edited file is not a test file.

> [!success]- Reveal answer
> **Correct: A**
>
> Path-scoped rules only load when the file path matches one of the globs. Editing `src/auth/login.ts` doesn't match `**/*.test.ts`, `**/*.test.tsx`, or `**/*.spec.ts`, so the rule stays unloaded.
>
> **Why others are wrong:**
> - B: Living in `.claude/rules/` is necessary but not sufficient — the glob match controls loading.
> - C: There's no UI tooltip behavior for path rules.
> - D: Non-matching files aren't errors; the rule just doesn't apply.

#### Question 16

A senior reviewer pushes back on a junior's plan to put all team conventions into the project's root `CLAUDE.md` because the conventions are different for different file types. What's the senior's **primary** rationale? ^d3-q-16

A. Root `CLAUDE.md` is loaded at every session start — putting everything there inflates context for operations that don't need those rules.
B. Root `CLAUDE.md` is hard to maintain when it grows large; team members keep stepping on each other's edits.
C. Path-specific rules in `.claude/rules/<name>.md` with `paths:` frontmatter load only when matching files are touched — keeping context lean and conventions accurate to the file at hand.
D. Root `CLAUDE.md` doesn't support imports, so it has to grow as a monolith.

> [!success]- Reveal answer
> **Correct: C**
>
> The path-specific rules pattern exists precisely for this: conventions that apply to subsets of files load only when those files are touched. This keeps context lean (no bloat for irrelevant rules) and keeps rules contextually relevant.
>
> **Why others are wrong:**
> - A: Context bloat is a real concern but it's a *symptom* — the senior's primary recommendation is using path-scoped rules to address it. A is partially correct but doesn't get to the documented mechanism.
> - B: Editing friction is a workflow concern, not the technical reason path-scoped rules exist.
> - D: Root `CLAUDE.md` does support `@import`, so the "monolith" framing is inaccurate.

### Task 3.4 — Plan mode vs direct execution

#### Question 17

A developer needs to add a null check to a single line in `src/utils/parseUser.ts`. They reach for plan mode "to be safe." What's the **most accurate** assessment? ^d3-q-17

A. Plan mode is always safer; the developer is doing the right thing.
B. Plan mode prevents file modifications, so this approach won't work for the null check.
C. Plan mode is good for any change that touches production code.
D. Plan mode is documented for large/uncertain/architectural changes; a single-line null check is small and well-scoped, so direct execution is the documented match. Plan mode adds friction without commensurate benefit.

> [!success]- Reveal answer
> **Correct: D**
>
> The documented decision criteria: large/uncertain/architectural → plan mode; small/scoped/familiar → direct execution. Reaching for plan mode for a one-line fix is mismatch in the "too much overhead" direction.
>
> **Why others are wrong:**
> - A: Plan mode isn't always safer — it adds overhead that's not warranted for small changes.
> - B: Plan mode is read-only *during planning*, but the plan can be executed afterward. The developer isn't stuck — but the overhead still isn't justified.
> - C: "Touches production code" isn't the criterion; scope and uncertainty are.

#### Question 18

A team is starting a 70-file migration from CommonJS to ES modules. Should they reach for plan mode or direct execution? ^d3-q-18

A. Direct execution — Claude Code's tooling handles file changes efficiently; specifying the migration in detail in the user message is enough.
B. Plan mode — the change is large, architectural (module system change affects imports across the codebase), and may have multiple valid approaches (which files first, whether to update package.json, naming conventions). Plan mode produces a reviewable plan before execution.
C. Plan mode for the first 10 files, then direct execution for the rest once the pattern is established.
D. Direct execution with verbose logging so the developer can review changes as they happen.

> [!success]- Reveal answer
> **Correct: B**
>
> Large, architectural, multiple-valid-approaches — exactly the plan mode use case. The plan can be reviewed and adjusted before any file is touched.
>
> **Why others are wrong:**
> - A: A 70-file migration touches many decision points the developer should see and possibly redirect before execution begins.
> - C: Mixing modes adds switching overhead. Plan mode handles all 70 files once the plan is approved.
> - D: Verbose logging shows changes after they happen, which doesn't help with architectural decisions made during planning.

#### Question 19

What's the role of the **Plan subagent** in Claude Code? ^d3-q-19

A. Plan mode delegates research and analysis to the Plan subagent. This avoids the no-nesting-subagents constraint and keeps the main agent focused on planning rather than running tools.
B. The Plan subagent reviews the plan before showing it to the user.
C. The Plan subagent is a fallback when the main agent's context fills up in plan mode.
D. The Plan subagent is invoked only when the user explicitly types `@Plan` during a plan-mode session.

> [!success]- Reveal answer
> **Correct: A**
>
> The Plan subagent exists to handle research tasks during plan mode. Subagents can't spawn subagents, so plan mode itself runs in the main agent; research that benefits from subagent-style isolation is delegated to Plan.
>
> **Why others are wrong:**
> - B: There's no documented review step.
> - C: Plan subagent isn't a context-overflow fallback.
> - D: Plan subagent is invoked by plan mode itself, not by explicit `@`-mention.

#### Question 20

Pressing `Ctrl+G` while in plan mode does what? ^d3-q-20

A. Cancels the current plan and starts over.
B. Forces plan mode to use the Explore subagent instead of Plan.
C. Opens the current plan in your default text editor, letting you edit it directly before Claude proceeds with execution.
D. Generates the plan in JSON format for CI consumption.

> [!success]- Reveal answer
> **Correct: C**
>
> `Ctrl+G` in plan mode opens the plan for direct editing in your text editor. The edited plan replaces the agent-generated one when execution proceeds.
>
> **Why others are wrong:**
> - A: There's no documented shortcut for canceling plan mode this way.
> - B: Subagent choice isn't toggleable via this shortcut.
> - D: JSON output is a CI flag (`--output-format json`), not a plan-mode shortcut.

#### Question 21

In Claude Code 2.1+, what happens when a developer enters plan mode? ^d3-q-21

A. Claude Code prompts the developer to confirm before entering plan mode each time.
B. Plan mode auto-runs the Plan subagent without the developer's input.
C. Claude Code shows a banner that all subsequent tool calls are blocked until the plan is approved.
D. Plan mode is entered seamlessly — no permission prompt. Earlier versions had a permission prompt as friction; in 2.1+, that prompt was removed.

> [!success]- Reveal answer
> **Correct: D**
>
> Claude Code 2.1+ removed the entry permission prompt. Plan mode is just a mode change; the read-only tool restriction is enforced during the mode, but entering it is friction-free.
>
> **Why others are wrong:**
> - A: That was the older behavior, replaced in 2.1+.
> - B: Plan subagent invocation happens during planning, not as automatic entry behavior.
> - C: A "tool calls blocked" banner isn't a documented feature.

### Task 3.5 — Iterative refinement

#### Question 22

A developer is using Claude Code to refactor a complex function. They want feedback before each modification rather than letting Claude Code make all changes and then reviewing. Which approach is **most aligned** with the documented iterative refinement pattern? ^d3-q-22

A. Use plan mode for every change and review the plan before approval.
B. Use the interview pattern: ask Claude Code to propose one change, review and discuss it, then proceed to the next. Repeat. This gives feedback at each step rather than as a batch.
C. Run Claude Code in headless mode with `--output-format json` so changes are machine-reviewable.
D. Use `--permission-mode default` so every tool call requires approval.

> [!success]- Reveal answer
> **Correct: B**
>
> The interview pattern is the documented iterative-refinement approach: one change at a time, with explicit discussion between changes. This lets the developer redirect early before substantial work is invested in a wrong direction.
>
> **Why others are wrong:**
> - A: Plan mode produces a complete plan before execution begins — it's batch-oriented, not iterative.
> - C: JSON output is for CI/CD parseable output, not interactive feedback.
> - D: Requiring approval for every tool call adds friction without iteration structure. The developer is approving operations, not reviewing decisions.

#### Question 23

A team's reviewer wants Claude Code to address all 12 review comments at once on a PR. Should they batch the comments or send them one at a time? ^d3-q-23

A. Send all 12 comments in one message — Claude Code can process them in parallel and respond efficiently. Iterative feedback is for designs, not batched feedback on completed work.
B. Send one comment at a time and wait for completion before the next.
C. Batch into 3-4 logical groups (security, style, correctness).
D. Use plan mode for the response.

> [!success]- Reveal answer
> **Correct: A**
>
> Batched feedback on a completed work product is appropriate when each comment is self-contained and they don't depend on each other. The interview pattern is for when each iteration's output should inform the next iteration's request — that's design and refactoring, not PR review.
>
> **Why others are wrong:**
> - B: One-at-a-time creates unnecessary friction when the comments are independent.
> - C: Grouping by category may help organize but isn't documented as the primary approach.
> - D: Plan mode is for designing changes; this is about applying a list of reviewer comments.

#### Question 24

A developer asks Claude Code: "Refactor `userService.ts`." Claude proposes a sweeping refactor touching 8 files. The developer wants more incremental control. What's the **best** intervention? ^d3-q-24

A. Add a system prompt telling Claude Code to be more incremental.
B. Decrease `maxTurns`.
C. Reframe the request as an interview: "Look at `userService.ts` and propose ONE small change you'd make to start. Tell me what and why before making it."
D. Use direct execution with `--permission-mode default` so each file requires approval.

> [!success]- Reveal answer
> **Correct: C**
>
> The interview pattern recasts the open-ended refactor request as a one-step interaction. "Propose one small change; tell me what and why" forces incremental reasoning and gives the developer a decision point.
>
> **Why others are wrong:**
> - A: System prompt instructions are probabilistic — Claude Code might still produce a sweeping plan.
> - B: `maxTurns` doesn't shape work granularity; it just caps iteration count.
> - D: Per-file approval doesn't reduce the breadth of the proposal — it adds friction without altering the underlying plan.

#### Question 25

In iterative refinement with Claude Code, when should you include prior decisions or rejected approaches in your next message? ^d3-q-25

A. Always — Claude Code's auto-memory captures these automatically, so reiterating is redundant.
B. Never — Claude Code retains conversation context within a session, so prior decisions are already available.
C. When the conversation has been compacted or you're starting a fresh session; in continuous conversation, prior decisions are in context.
D. Always include explicit context about rejected approaches, especially when the conversation has compacted. Stale context (or no context after compaction) leads Claude Code to re-propose ideas you already rejected.

> [!success]- Reveal answer
> **Correct: D**
>
> Iterative refinement benefits from explicit prior-context inclusion. If the conversation has been compacted, the rejection reasoning may have been summarized away. Even within an uncompacted session, restating decisions concisely helps Claude stay on track.
>
> **Why others are wrong:**
> - A: Auto memory captures patterns over many sessions, not specific in-session decisions. Recent rejections aren't necessarily in MEMORY.md.
> - B: Conversation context can compact mid-session at 64-75% utilization; "prior decisions are already available" isn't always true.
> - C: While correct on the surface, this answer understates the case. Even in continuous conversation, restating prior decisions concisely is beneficial — and after compaction it's essential.

### Task 3.6 — CI/CD integration

#### Question 26

A team is integrating Claude Code into their CI pipeline for code review on every PR. The CI script needs parseable output. Which CLI flag combination is **documented** for this? ^d3-q-26

A. `claude --interactive --json --validate-schema review.json`
B. `claude -p --output-format json --json-schema ./review-schema.json`
C. `claude --headless --output json --schema review.json`
D. `claude -p --json --strict-schema review-schema.json`

> [!success]- Reveal answer
> **Correct: B**
>
> Documented flags: `-p` (or `--print`) for non-interactive mode, `--output-format json` for structured output, `--json-schema <path>` for schema validation. All three are standard flag names.
>
> **Why others are wrong:**
> - A: `--interactive` is the opposite of headless mode; `--json` and `--validate-schema` are fabricated flag names.
> - C: `--headless`, `--output`, `--schema` are all incorrect flag names.
> - D: `--json` and `--strict-schema` are incorrect.

#### Question 27

In a CI/CD pipeline, why would including the previous review's findings in the next prompt **improve** quality? ^d3-q-27

A. So Claude Code can avoid re-reporting issues that were addressed in the current PR iteration, reducing comment churn.
B. So Claude Code can compare current findings to the previous run for variance analysis.
C. So Claude Code can use prior findings as few-shot examples of well-formatted output.
D. So Claude Code can rate-limit itself to avoid duplicate API calls.

> [!success]- Reveal answer
> **Correct: A**
>
> Including prior findings prevents the agent from re-reporting issues that the developer already addressed. Without this, the same issues get flagged on every PR iteration, creating reviewer fatigue.
>
> **Why others are wrong:**
> - B: Variance analysis isn't the documented purpose.
> - C: Few-shot examples of *output format* would just be schema definitions or example JSON, not prior findings.
> - D: Rate limiting is unrelated.

#### Question 28

A CI script runs:

```bash
claude -p \
  --output-format json \
  --json-schema ./review-schema.json \
  "Review this diff" > review.json
```

The output frequently has malformed JSON despite `--output-format json`. What's the **most likely** cause? ^d3-q-28

A. The `-p` flag isn't compatible with `--output-format json`; remove `-p`.
B. The `--json-schema` flag isn't being respected because the schema file has a syntax error.
C. The schema file (`./review-schema.json`) has constraints that the model can't always satisfy (e.g., a required field that requires git context not provided), and the schema validation failures result in unparseable output. Verify the schema is feasible given the input context.
D. `--output-format json` was removed in Claude Code 2.0; the new flag is `--output text`.

> [!success]- Reveal answer
> **Correct: C**
>
> Schemas constrain the generation grammar — if the schema requires impossible-to-fill fields, the constrained decoder produces malformed or incomplete output. Verify the schema is satisfiable given the inputs the model sees.
>
> **Why others are wrong:**
> - A: `-p` is compatible with `--output-format json`; that's the documented combination.
> - B: A syntactically broken schema would cause an earlier failure, not malformed JSON output.
> - D: `--output-format json` is current.

#### Question 29

A team wants Claude Code to act as a security reviewer in CI. They define a custom subagent `security-reviewer` with a specialized prompt. How do they run a session as this subagent from the CLI? ^d3-q-29

A. Add `@security-reviewer` to the user prompt.
B. Set `agent: security-reviewer` in `.claude/settings.json` for the project.
C. Pass `--system-prompt-file .claude/agents/security-reviewer.md` to Claude Code.
D. Use the `--agent security-reviewer` CLI flag. This runs the entire session as that subagent — the subagent's prompt replaces the default Claude Code system prompt for this run.

> [!success]- Reveal answer
> **Correct: D**
>
> `--agent <name>` is the documented CLI flag for running an entire session as a specified subagent.
>
> **Why others are wrong:**
> - A: `@`-mention invokes a subagent inline within a conversation; it doesn't run the whole session as the subagent.
> - B: Setting `agent` in settings.json makes a subagent the default for all sessions in the project, but the question asks about the CLI-level mechanism for a one-off run.
> - C: `--system-prompt-file` is not a documented flag. The mechanism for the whole-session subagent is `--agent`.

#### Question 30

A CI pipeline runs Claude Code with `-p --output-format json --json-schema review-schema.json`. The schema requires fields `summary`, `findings`, and `blocking`. The CI script then exits 1 if `blocking` is `true`. What's a **likely improvement** for the script's behavior over time? ^d3-q-30

A. Include a longer prompt with more domain context to improve review accuracy.
B. Persist prior findings to a file and include them in the next CI invocation's prompt, so Claude Code can avoid duplicate findings on subsequent PR iterations.
C. Cache the schema-grammar compilation between runs to reduce latency.
D. Switch from `-p` to interactive mode for human-in-the-loop review.

> [!success]- Reveal answer
> **Correct: B**
>
> Persisting prior findings and including them in the next run is the documented improvement for CI workflows. It prevents the agent from re-reporting issues addressed in previous iterations and keeps PR comments fresh.
>
> **Why others are wrong:**
> - A: Domain context is set at session start (CLAUDE.md, etc.); adding more prompt content doesn't address the duplicate-findings problem over time.
> - C: Schema grammar caching is automatic (24-hour cache); manual cache management isn't needed.
> - D: Interactive mode defeats the purpose of CI/CD automation.

---

## Answer key summary

| Q | A | Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | D | 6 | B | 11 | A | 16 | C | 21 | D | 26 | B |
| 2 | B | 7 | A | 12 | C | 17 | D | 22 | B | 27 | A |
| 3 | A | 8 | C | 13 | D | 18 | B | 23 | A | 28 | C |
| 4 | C | 9 | D | 14 | B | 19 | A | 24 | C | 29 | D |
| 5 | D | 10 | B | 15 | A | 20 | C | 25 | D | 30 | B |

## Stats summary

- **Answer distribution:** A=7 (23%) · B=8 (27%) · C=7 (23%) · D=8 (27%)
- **Hardest questions** (subtle distinctions, two-look-right options): Q8 (`disable-model-invocation` vs `user-invocable`), Q12 (`paths` interpretation), Q16 (path-scoped rules vs context bloat), Q23 (when to batch vs interview), Q25 (prior context inclusion), Q28 (schema feasibility)
- **Trap questions:** Q15 (testing whether path globs really matter), Q22 (interview pattern vs plan mode), Q29 (`--agent` vs `@`-mention)
- **Fabricated-field traps:** Q6 (`verbose: false`), Q8 (`auto-invoke: false`, `manual-only: true`), Q11 (`argument-name`, `inputs`, `prompt-arg`), Q26 (`--validate-schema`, `--strict-schema`)

---

*Domain 3 weight: **20%** of the exam. Companion roadmap: [[cca_domain3_roadmap.md]]. Companion exercises: [[cca_domain3_exercises.md]].*
