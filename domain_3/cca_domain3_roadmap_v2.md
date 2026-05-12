---
title: Claude Certified Architect — Foundations · Domain 3 Roadmap
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - roadmap
  - domain-3
status: in-progress
domain: 3
domain-name: "Claude Code Configuration & Workflows"
domain-weight: 20
primary-scenarios: ["Code Generation with Claude Code", "Claude Code for CI/CD"]
estimated-hours: 6
coverage-level: comprehensive
---

# Domain 3 Roadmap — Claude Code Configuration & Workflows

## What this domain tests

How to **configure** Claude Code for developers and teams, when to use **plan mode vs direct execution**, how to integrate Claude Code into **CI/CD pipelines**, and how to design **skills, slash commands, and custom subagents**. Tested in Scenarios 2 (Code Generation) and 5 (CI/CD).

The exam rewards moving facts from prompts into structured config (`CLAUDE.md`, `.claude/rules/`, frontmatter) over adding more prompt instructions.

## Domain weight

**20% of the total exam.** Claude Code documentation surface area is large but the testable patterns are well-defined.

> [!warning] Important docs URL note
> The Claude Code docs live at **`code.claude.com/docs/en/...`** (not `docs.claude.com/en/docs/claude-code/...`). The legacy URL pattern may redirect or 404. The frontmatter reference for `SKILL.md` is at `code.claude.com/docs/en/skills` (look for the frontmatter reference section).

---

## Task statements (6 total)

| Task | What it tests |
|------|---------------|
| 3.1 | `CLAUDE.md` hierarchy · `@import` · auto memory |
| 3.2 | Custom slash commands + skills · `SKILL.md` frontmatter |
| 3.3 | Path-specific rules · glob frontmatter |
| 3.4 | Plan mode vs direct execution · Explore subagent |
| 3.5 | Iterative refinement · interview pattern |
| 3.6 | CI/CD integration · `-p` flag · `--output-format json` |

---

## Read in full · ~5 hours

### How Claude Code works · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/how-claude-code-works`

The conceptual orientation — agentic loop, tools, context management, modes. Read first.

**Memorize cold:**
- Three phases: gather context, take action, verify results
- Claude Code includes built-in subagents: **Explore**, **Plan**, **general-purpose**
- Context lifecycle: CLAUDE.md + auto memory + MCP tool names + skill descriptions load at session start
- Subagents get fresh contexts; their work doesn't bloat main context (only final summary returns)
- Plan mode is mandatory in the recommended workflow

### .claude directory · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/claude-directory`

Reference page for the `.claude` directory layout. Shows where every config file goes and what it does.

**Memorize cold:**

| File/folder | Scope | Purpose |
|-------------|-------|---------|
| `.claude/CLAUDE.md` | Project | Project standards (Git-tracked) |
| `CLAUDE.md` (project root) | Project | Same as above (root-level location) |
| `~/.claude/CLAUDE.md` | User | Personal standards |
| `.claude/CLAUDE.local.md` | Local | Local overrides (not Git-tracked) |
| `.claude/settings.json` | Project | Permissions, hooks, env (shared) |
| `.claude/settings.local.json` | Local | Personal settings overrides (not Git-tracked) |
| `~/.claude/settings.json` | User | Personal settings |
| `.claude/rules/` | Project | Path-scoped rules (glob frontmatter) |
| `.claude/skills/<name>/SKILL.md` | Project | Custom skills (shared) |
| `~/.claude/skills/<name>/SKILL.md` | User | Personal skills |
| `.claude/agents/<name>.md` | Project | Custom subagents (shared) |
| `~/.claude/agents/<name>.md` | User | Personal subagents |
| `.claude/commands/<name>.md` | Project | Custom slash commands (shared, merged with skills) |
| `~/.claude/commands/<name>.md` | User | Personal slash commands |
| `MEMORY.md` (project) | Project | Auto memory (first 200 lines / 25KB load at session start) |
| `~/.claude/MEMORY.md` | User | Personal auto memory |
| `.mcp.json` (project root) | Project | MCP server config (shared) |
| `~/.claude.json` | User | Personal MCP servers + auth + preferences |

**Don't delete** `~/.claude.json`, `~/.claude/settings.json`, `~/.claude/plugins/` — they hold auth, preferences, and installed plugins.

### How Claude remembers your project (Memory) · ~45 min

- [ ] **Read:** `code.claude.com/docs/en/memory`
- [ ] **Re-read for memorization** (the four-tier hierarchy + auto memory are foundational)

Covers the CLAUDE.md hierarchy, `@import`, auto memory, what survives `/compact`.

**Memorize cold (the four-tier hierarchy, precedence highest to lowest):**

| Tier | Location | Scope | Use for |
|------|----------|-------|---------|
| Enterprise | OS-managed location | Org-wide | Compliance, mandated policies |
| Project | `.claude/CLAUDE.md` or root `CLAUDE.md` | Repo | Team standards, project conventions |
| User | `~/.claude/CLAUDE.md` | Personal | Personal coding preferences |
| Local | `.claude/CLAUDE.local.md` | Personal-per-project | Local overrides, not Git-tracked |

**Memorize `@import`:**
- Reference another file from `CLAUDE.md`: `@import path/to/other.md`
- Lets each package's `CLAUDE.md` reference only relevant standards
- Solves the "2000-line monolith CLAUDE.md" problem

**Memorize auto memory:**
- File: `MEMORY.md` (project) or `~/.claude/MEMORY.md` (user)
- First **200 lines** or **25 KB** (whichever comes first) loads at session start
- Claude saves notes automatically as it works (build commands, debugging insights, code style preferences)
- Claude decides what's worth remembering based on future usefulness
- Toggle: open `/memory` and use the auto memory toggle; or set `autoMemoryEnabled` in settings
- Requires Claude Code v2.1.59+
- **Excluded from compaction summarization** — auto memory is for cross-session persistence, conversations are within-session

**`/memory` command:** verifies which memory files are currently loaded.

**`claudeMdExcludes` setting:** glob patterns to exclude specific CLAUDE.md files at any settings layer (user, project, local, managed). Managed policy CLAUDE.md files cannot be excluded.

### Settings · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/settings`

`.claude/settings.json`, settings layering, precedence, environment variables.

**Memorize cold:**
- `.claude/settings.json` (project-scoped, shared via Git)
- `.claude/settings.local.json` (local, not Git-tracked)
- `~/.claude/settings.json` (user-scoped)
- **Precedence:** Managed > User > Project > Local (the local file overrides via merging)
- CLI flags like `--permission-mode` or `--settings` override `settings.json` for that session
- Some env vars take precedence over their equivalent setting (check env vars reference per setting)
- `$schema` line points to the official JSON schema for autocomplete/validation in editors

**Memorize permission rules:**
```json
{
  "permissions": {
    "allow": ["Bash(npm run lint)", "Bash(npm run test *)", "Read(~/.zshrc)"],
    "deny": ["Bash(curl *)", "Read(./.env)", "Read(./secrets/**)"]
  }
}
```
- Permission rules use tool-name patterns like `Bash(...)` or `Read(...)`
- Wildcards within args supported
- Useful security pattern: deny `Read(./.env)`, `Read(./.env.*)`, `Read(./secrets/**)`

### Commands · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/commands`

Built-in commands available in every session. Memorize the ones that show up in scenario questions.

**Memorize cold (high-value built-in commands):**
- `/init` — generates starter CLAUDE.md for a project
- `/memory` — refines memory, shows what's loaded, toggles auto memory
- `/mcp` — manages MCP servers
- `/agents` — manages subagents
- `/permissions` — sets approval rules
- `/plan` — switches into plan mode
- `/context` — shows live context breakdown by category with optimization suggestions
- `/compact` — replaces conversation with structured summary
- `/btw` — quick aside that shouldn't bloat history
- `/clear` — starts fresh on new task, keeps project memory
- `/resume` — returns to an earlier conversation
- `/branch` — forks an earlier conversation
- `/teleport` — pulls a web session into terminal
- `/remote-control` — continue local session from another device
- `/rewind` — rolls code and conversation back to a checkpoint
- `/diff` — shows what changed
- `/simplify` — reviews recent files, applies quality/efficiency fixes (bundled skill)
- `/review` — deeper read-only pass
- `/security-review` — security-focused review
- `/doctor` — diagnoses install/runtime issues
- `/debug` — diagnoses session issues
- `/feedback` — reports a bug with session context

### Slash commands (custom) · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/slash-commands`

Custom slash commands. Note the merge with skills.

**Memorize cold:**
- `.claude/commands/foo.md` → project-scoped `/foo` (Git-tracked)
- `~/.claude/commands/foo.md` → user-scoped `/foo` (personal)
- **Slash commands and skills are merged** — `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`
- Argument hints (`argument-hint:` frontmatter) prompt user for input
- Hooks can now be specified in slash command frontmatter (recent addition)

### Extend Claude with skills · ~60 min

- [ ] **Read:** `code.claude.com/docs/en/skills` (the **authoritative frontmatter source**)
- [ ] **Re-read for memorization** (the frontmatter reference table is foundational)

The single most important Domain 3 page. The frontmatter reference table is heavily tested.

**Memorize the full frontmatter spec:**

| Field | Use |
|-------|-----|
| `name` | Display name |
| `description` | What the skill does (drives auto-invocation; primary trigger) |
| `when_to_use` | Trigger phrases |
| `argument-hint` | Prompt text for autocomplete |
| `arguments` | Named positional arguments |
| `disable-model-invocation` | If `true`, skill is excluded from auto-invocation list — only manual `/skill-name` works |
| `user-invocable` | If `false`, hidden from `/` menu — only Claude can invoke |
| `allowed-tools` | Pre-approved tools for the skill |
| `model` | Override model for skill execution |
| `effort` | Effort level override |
| **`context`** | **`fork` to run in a forked subagent context** |
| **`agent`** | **Which subagent type (`Explore`, `Plan`, `general-purpose`, custom) when `context: fork`** |
| `hooks` | Skill-scoped hooks (recent addition — hooks in frontmatter) |
| `paths` | Glob patterns to limit activation by file path |
| `shell` | `bash` or `powershell` |
| `version` | Metadata (optional) |
| `mode` | If `true`, categorizes skill as a "mode command" (appears in special section) |

**The `context: fork` pattern (heavily tested):**
```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---
Research $ARGUMENTS thoroughly: ...
```

- Skill content becomes the **prompt** that drives the subagent
- The subagent does NOT have access to your conversation history
- Only summary returns to main session
- `context: fork` only makes sense for skills with **explicit instructions** (actionable task content). If your skill is "reference content" (conventions, style guides), forking provides no actionable prompt — it'll return without output.

**Skills vs `CLAUDE.md` decision:**
- **`CLAUDE.md`** — always-loaded, applies every session, for universal standards (coding conventions, PR templates) → "reference content"
- **Skill** — on-demand invocation, applies only when relevant, for task-specific workflows (deployment checklist) → "task content"

**Discovery:**
- At startup, only metadata (`name` and `description`) from all Skills loads into system prompt
- SKILL.md body loads only when skill is invoked
- Bundled files (scripts, references) load on-demand
- Bundle structure: `my-skill/SKILL.md`, `my-skill/scripts/`, `my-skill/references/`, `my-skill/assets/`

**Built-in / bundled skills include:** `/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`

### Create custom subagents · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/sub-agents`

Project vs personal subagents. Built-in subagents. Subagent frontmatter (extended beyond skill frontmatter).

**Memorize cold (subagent frontmatter fields):**

| Field | Use |
|-------|-----|
| `name` | Display name |
| `description` | Drives auto-routing |
| `prompt` | System prompt (or markdown body in file-based) |
| `tools` | Allowed tools |
| `disallowedTools` | Explicitly forbidden tools |
| `model` | Model override |
| `permissionMode` | Permission handling |
| `mcpServers` | MCP server allow-list |
| `hooks` | Hooks |
| `maxTurns` | Iteration cap |
| `skills` | Preloaded skills (which to surface; doesn't restrict invocation) |
| `initialPrompt` | First-turn prompt |
| `memory` | Persistent memory directory (across conversations) |
| `effort` | Effort level |
| `background` | Run in background |
| `isolation` | `"worktree"` for git-worktree isolation |
| `color` | UI color |

**Memorize built-in subagents:**
- **Explore** — codebase research, read-only tools optimized for exploration
- **Plan** — used in plan mode for research (delegates research from plan mode to avoid infinite nesting since subagents can't spawn subagents)
- **general-purpose** — complex multi-step tasks needing both exploration and modification, complex reasoning, multiple dependent steps

**Memorize cold:**
- Subagents **cannot** spawn other subagents (prevents infinite nesting)
- This is why plan mode delegates research to the Plan subagent (instead of nesting)
- `--agent <name>` flag runs the whole session as a subagent — subagent's system prompt replaces default Claude Code system prompt entirely (like `--system-prompt`)
- Set `agent` in `.claude/settings.json` to make a subagent the default for every session in a project
- For security, **plugin subagents** don't support `hooks`, `mcpServers`, or `permissionMode` fields
- Managed subagents (in managed settings directory) take precedence over project and user subagents

### Plan mode · ~45 min

- [ ] **Read:** `code.claude.com/docs/en/plan-mode`

When to enter plan mode vs direct execution. What plan mode locks. The Explore/Plan subagent delegation.

**Memorize the decision criteria:**

| Use plan mode | Use direct execution |
|---------------|----------------------|
| Large-scale changes (10+ files, refactors, migrations) | Small, well-scoped changes (single null check, one-line fix) |
| Multiple valid approaches need evaluation | Clear root cause, one obvious fix |
| Architectural decisions (framework choice, module boundaries) | Implementation phase of an already-planned change |
| Unknown territory requiring investigation | Familiar code with a known bug |
| You can't describe the diff in one sentence | You can describe the diff in one sentence |

**Memorize cold:**
- Plan mode uses **read-only tools only** — creates a plan you can approve before execution
- Pressing `Ctrl+G` opens the plan in your text editor for direct editing before Claude proceeds
- The recommended four-phase workflow: **Plan → Refine → Implement → Review**
- Plan mode delegates research to the **Plan subagent** (a built-in subagent) — keeps the main agent focused on planning rather than nesting subagents
- In Claude Code 2.1+: **no permission prompt to enter plan mode** (was friction; now seamless)
- `Auto mode` (research preview) — Claude evaluates all actions with background safety checks

### Hooks (Claude Code) · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/hooks`

Same conceptual hook events as the Agent SDK page, plus Claude Code-specific settings.json hook config.

**Memorize cold:**
- Hooks live in `.claude/settings.json` at project, user, local, or managed scope
- `matcher` field uses glob patterns for tool names — `mcp__*` (all MCP), `Edit|Write` (Edit or Write), `mcp__customer-support__*` (one server's tools)
- Hook events same as SDK (see Domain 1 roadmap)
- Hooks now also work in **skill, command, and subagent frontmatter** (recent addition)
- `once: true` config — run a hook exactly once, then done (useful for session initialization)

### Output styles · ~20 min

- [ ] **Read:** `code.claude.com/docs/en/output-styles`

Critical conceptual distinction tested in scenarios about customizing Claude Code's behavior without losing infrastructure.

**Memorize cold:**
- **Output style** replaces the software-engineering portion of the system prompt; preserves tool ecosystem, CLAUDE.md loading, sub-agents, MCP, slash commands
- **Fully custom system prompt** (SDK only) replaces entire system prompt; loses Claude Code defaults
- **`--append-system-prompt`** adds to default; preserves everything

Use output styles to turn Claude Code into "Claude Anything" (writer, tutor, domain expert) while keeping the agent infrastructure.

### Context window (Claude Code) · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/context-window`

What's in the context window in a Claude Code session. Covers compaction behavior, what survives, what reloads.

**Memorize cold (what's in context at startup):**
- CLAUDE.md (project + user + local + enterprise)
- Auto memory (MEMORY.md, first 200 lines / 25 KB)
- MCP tool names (descriptions, not bodies)
- Skill descriptions (NOT bodies — bodies load on-demand)
- Subagent descriptions
- System prompt (Claude Code's default + any --append-system-prompt or output style)

**Memorize what happens at compaction:**

| Mechanism | Behavior on /compact |
|-----------|-------------------|
| Conversation history | Summarized away |
| Path-scoped rules | Summarized away; reload on next matching file read |
| Nested CLAUDE.md files | Summarized away; reload on next matching file read |
| Skill bodies | **Re-injected** after compaction (large skills truncated to per-skill cap; oldest invoked skills dropped if total budget exceeded) |
| Auto memory | Preserved (not subject to compaction) |
| Static CLAUDE.md (root + user) | Preserved (always-loaded) |

**Memorize:**
- `/context` shows live breakdown of context usage by category with optimization suggestions
- `/memory` shows which CLAUDE.md and auto memory files loaded at startup
- Subagent context is separate — their work doesn't bloat main context
- Skills with `disable-model-invocation: true` keep their description out of context until manually invoked
- `skillOverrides` setting lets you set `disable-model-invocation` from settings for skills you didn't write

### CLI reference + Headless mode · ~30 min

- [ ] **Read:** `code.claude.com/docs/en/cli-reference`

**Memorize cold:**

| Flag | What it does |
|------|--------------|
| `-p` or `--print` | Non-interactive mode; prints output and exits (no REPL) |
| `--output-format json` | Returns structured JSON output |
| `--output-format text` | Default text output |
| `--json-schema <path>` | Validates output against a JSON schema |
| `--resume <session-id>` | Continues a named session |
| `--fork-session` | Creates a fork from a baseline session |
| `--system-prompt <text>` | Replaces system prompt entirely |
| `--append-system-prompt <text>` | Adds to default system prompt |
| `--permission-mode <mode>` | Sets permission mode (`default` / `acceptEdits` / `bypassPermissions` / `plan`) |
| `--agent <name>` | Runs whole session as the named subagent |
| `--allowedTools <list>` | Restricts available tools |
| `--settings <path>` | Override settings file for this session |
| `--no-session-persistence` | (with -p) skip writing transcripts |
| `--channels` | (advanced) plan-mode tool availability |

**The CI/CD pattern:**
```bash
claude -p \
  --output-format json \
  --json-schema review-schema.json \
  "Review this diff for security issues"
```

---

## Skim · ~30 min

### Best practices for Claude Code

- [ ] **Skim:** `code.claude.com/docs/en/best-practices`

Workflow patterns. Plan-mode workflow, when to compact vs continue, prompt structure tips.

### Permissions

- [ ] **Skim:** `code.claude.com/docs/en/permissions`

Permission rule syntax (tool-name patterns, wildcards), tool-specific patterns, managed policies. Skim for the rule syntax.

### MCP integration in Claude Code

- [ ] **Skim:** `code.claude.com/docs/en/mcp`

Most of this is covered in the agents-and-tools MCP page; skim for Claude Code-specific differences.

### Plugin system

- [ ] **Skim:** `code.claude.com/docs/en/plugins`

Plugins extend Claude Code with skills, agents, hooks, MCP servers. Out of scope for testable specifics, but light familiarity helps.

### Reduce token usage

- [ ] **Skim:** `code.claude.com/docs/en/reduce-token-usage`

Strategies for keeping context usage low. Useful Domain 5 adjacency.

### Claude Code docs map (index)

- [ ] **Skim:** `code.claude.com/docs/en/claude_code_docs_map`

The complete index of Claude Code docs pages. Useful for spotting anything not yet covered. Read once for orientation.

---

## Skip — out of scope

> [!warning] Don't get pulled in
> Per the exam guide's explicit out-of-scope list:

- **VS Code extension** — `code.claude.com/docs/en/vs-code` — IDE integration
- **JetBrains plugin** — IDE integration
- **Web / desktop chat features** — operational
- **Background tasks** — operational
- **Bedrock setup** — out of scope
- **Vertex AI setup** — out of scope
- **Azure AI Foundry setup** — out of scope
- **Enterprise deployment / managed policy details** — out of scope
- **Analytics API** — `code.claude.com/docs/en/analytics-api` — telemetry/monitoring
- **Telemetry / monitoring pages** — out of scope
- **Data and privacy** — out of scope
- **Remote Control** — operational feature
- **Teleport** — operational feature
- **Worktree isolation details** — operational

---

## Hands-on drills · ~3 hours

- [ ] **Drill 1: CLAUDE.md hierarchy** — in a real repo, set up project `.claude/CLAUDE.md`, user `~/.claude/CLAUDE.md`, and local `.claude/CLAUDE.local.md`. Use `@import` to pull in three different topic files. Verify load order with `/memory`.

- [ ] **Drill 2: Auto memory exploration** — enable auto memory, work for a bit, then check `~/.claude/MEMORY.md` to see what Claude saved. Verify the 200-line/25KB limit truncation.

- [ ] **Drill 3: `.claude/rules/` with glob patterns** — create three rule files with `paths` frontmatter — one for `**/*.test.*`, one for `terraform/**/*.tf`, one for `src/api/**/*`. Edit a matching file in each scope and verify the right rule loads.

- [ ] **Drill 4: Skill with all key frontmatter fields** — create `.claude/skills/dep-analyzer/SKILL.md` with `context: fork`, `agent: Explore`, `allowed-tools: [Read, Grep]`, `argument-hint: <package-name>`. Invoke it. Confirm verbose output stays out of your main session.

- [ ] **Drill 5: Skill with `disable-model-invocation: true`** — create a `/deploy` skill. Verify Claude doesn't auto-invoke it; only manual `/deploy` works.

- [ ] **Drill 6: CI-style non-interactive run** — `claude -p --output-format json --json-schema schema.json "review this diff"`. Confirm parseable output.

- [ ] **Drill 7: Plan mode vs direct execution** — try plan mode on a task you'd normally use direct execution for, and direct execution on a task that needs plan mode. Feel the friction in both directions.

- [ ] **Drill 8: Custom subagent for verbose discovery** — create `.claude/agents/explorer.md` that does codebase mapping. Invoke it and observe verbose output stays in subagent context, not main session.

- [ ] **Drill 9: Slash command with argument hint** — create `.claude/commands/review.md` with `argument-hint: <pr-number>`. Verify it prompts for the argument when invoked.

- [ ] **Drill 10: Output style experiment** — create a custom output style that turns Claude into a teaching tutor. Verify tools still work but the persona changes.

- [ ] **Drill 11: Settings layering test** — set the same setting at user, project, and local levels. Verify precedence: Managed > User > Project > Local.

---

## Common confusions and distractor patterns

### Distractor pattern 1: "Add it to the system prompt"

Inconsistent team standards → distractors offer "add it to the wiki and tell engineers to paste it" or "put standards in user-level config." **Correct:** move into project-level `CLAUDE.md` (or `.claude/rules/`) that travels with the repo.

### Distractor pattern 2: Plan-mode overuse / underuse

- **Trap A:** small fix; distractor proposes plan mode "for safety." Correct: direct execution.
- **Trap B:** 70-file framework migration; distractor proposes direct execution with detailed instructions. Correct: plan mode for design, then direct execution.

### Distractor pattern 3: Skill vs CLAUDE.md confusion

- Universal standards used every session → **`CLAUDE.md`** (distractor: "make it a skill" → wrong, adds friction)
- Rare multi-step task workflow → **skill** (distractor: "put it in CLAUDE.md" → wrong, pollutes context)

### Distractor pattern 4: Fabricated frontmatter fields

Watch for made-up options:
- `visibility: private` — NOT real
- `permission: restricted` — NOT real
- `auto: false` — NOT real (real one is `disable-model-invocation: true`)

Real fields: `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `model`, `effort`, `context`, `agent`, `hooks`, `paths`, `shell`, `version`, `mode`.

### Distractor pattern 5: Path-specific rules vs subdirectory CLAUDE.md

Conventions applied to files matching a glob:
- **Correct:** `.claude/rules/<name>.md` with `paths: ["**/*.test.tsx"]` frontmatter
- **Distractor:** subdirectory `CLAUDE.md` files at every test directory — can't handle globs spanning directories

### Distractor pattern 6: CI/CD parseability

CI output parsing failing because format varies:
- "Write a more permissive regex parser" → fragile
- "Run Claude twice and diff" → wasteful
- "Append 'RESPOND IN VALID JSON' to the prompt" → probabilistic
- **Correct:** `--output-format json` with `--json-schema` enforces structure at the protocol level

### Distractor pattern 7: Mixing up auto memory and conversation memory

- **CLAUDE.md** — instructions you write; always loaded at session start; for "what Claude should always know"
- **Auto memory (MEMORY.md)** — Claude writes itself based on corrections and preferences; loaded at session start; for "what Claude learned"
- Both are session-startup memory; both persist across sessions
- Conversation history (active session) is different and is subject to compaction

### Distractor pattern 8: Subagent infinite nesting

When a scenario describes a subagent that needs to delegate further work, distractors propose "have the subagent spawn its own subagent." **Wrong** — subagents cannot spawn subagents. Plan mode delegates to the Plan subagent specifically because the main agent can't be a subagent that spawns subagents during plan mode.

### Subtle distinction 1: Output style vs `--append-system-prompt`

- **`--append-system-prompt`** — adds to default Claude Code system prompt; preserves software-engineering persona
- **Output style** — replaces the software-engineering portion; preserves infrastructure (tools, MCP, sub-agents)
- **Fully custom `--system-prompt`** — replaces entire prompt; loses Claude Code defaults

### Subtle distinction 2: Project vs personal across config types

The same pattern applies to **everything** in Claude Code's config:
- `.claude/X` → project (Git-tracked, shared)
- `~/.claude/X` → user (personal)

Applies to: agents, skills, commands, settings, CLAUDE.md, hooks, MCP servers.

### Subtle distinction 3: Auto memory vs Memory tool

- **Auto memory** is a Claude Code feature — Claude writes to `MEMORY.md` automatically
- **Memory tool** is an Anthropic API feature — model uses a `memory` tool to read/write files in `/memories` directory
- Different features. The exam may test the conceptual distinction in Domain 5.

### Subtle distinction 4: Skill `context: fork` vs subagent

- **Skill with `context: fork`** — the skill *content* is the task; the subagent runs the skill's instructions
- **Subagent that uses skills** — the subagent has its own system prompt; skills are reference material it can invoke
- Both use the same underlying system but invert who controls what

---

## Self-assessment recall tests

### Task 3.1 · CLAUDE.md hierarchy

- [ ] Completed Task 3.1 self-check

1. Name the four memory tiers in precedence order (highest first).
2. Where does user-level CLAUDE.md live? Why isn't it the right place for team standards?
3. What's `@import` for?
4. What command shows which memory files are currently loaded?
5. What's auto memory? Where is it stored? How much loads at session start?

> [!success]- Answers
> 1. Enterprise → Project → User → Local
> 2. `~/.claude/CLAUDE.md`. Not shared via Git, so teammates don't get it
> 3. References external files from CLAUDE.md. Lets each package import only relevant standards
> 4. `/memory`
> 5. Auto memory is Claude saving notes itself. `MEMORY.md` (project) or `~/.claude/MEMORY.md` (user). First 200 lines or 25 KB loads at session start.

### Task 3.2 · Skills and slash commands

- [ ] Completed Task 3.2 self-check

1. What `SKILL.md` frontmatter field isolates skill execution in a subagent?
2. What field do you pair with `context: fork` to choose which subagent type runs the skill?
3. What frontmatter field restricts tools?
4. What frontmatter field prevents auto-invocation?
5. What field hides skill from the `/` menu?
6. When should you use `CLAUDE.md` vs a skill?
7. Where do project-scoped slash commands live? Personal?

> [!success]- Answers
> 1. `context: fork`
> 2. `agent: <Explore|Plan|general-purpose|custom-name>`
> 3. `allowed-tools: [Read, Grep, ...]`
> 4. `disable-model-invocation: true`
> 5. `user-invocable: false`
> 6. `CLAUDE.md` for always-loaded universal standards (reference content); skill for on-demand task workflows
> 7. `.claude/commands/<name>.md` or `.claude/skills/<name>/SKILL.md` (merged); user-level in `~/.claude/`

### Task 3.3 · Path-specific rules

- [ ] Completed Task 3.3 self-check

1. What's the documented pattern for conventions that apply to glob-matched files?
2. What frontmatter field declares the glob?
3. Why not use subdirectory CLAUDE.md for the same purpose?

### Task 3.4 · Plan mode vs direct execution

- [ ] Completed Task 3.4 self-check

1. When is plan mode the right choice?
2. When is direct execution the right choice?
3. What's the Explore subagent for?
4. What's the Plan subagent for?
5. Why can't subagents (including in plan mode) spawn other subagents?

### Task 3.5 · Iterative refinement

- [ ] Completed Task 3.5 self-check

1. What's the interview pattern?
2. When should you provide all issues to fix in one message vs sequentially?

### Task 3.6 · CI/CD integration

- [ ] Completed Task 3.6 self-check

1. What flag puts Claude Code in non-interactive mode?
2. What flags enforce structured JSON output?
3. Why should prior review findings be included in subsequent runs?

> [!success]- Answers
> 1. `-p` (or `--print`)
> 2. `--output-format json --json-schema <path>`
> 3. To avoid reporting previously-resolved findings on every PR iteration

---

## Common scenario patterns by task

**Task 3.1** — A scenario describes team standards not propagating, or one developer having different rules than another. Distractors reach for user-level config or "tell engineers to copy this"; the correct answer moves standards to project-level `.claude/CLAUDE.md` (Git-tracked).

**Task 3.2** — A scenario describes a verbose skill cluttering main conversation, or a skill being auto-invoked when it shouldn't be. The correct answer pairs a frontmatter field to the failure mode: `context: fork` for isolation, `disable-model-invocation: true` for manual-only, `allowed-tools` for restriction.

**Task 3.3** — A scenario describes conventions that apply only to certain file types (test files, Terraform, API files). Distractors propose subdir CLAUDE.md or "put everything in root CLAUDE.md"; the correct answer is `.claude/rules/` with `paths:` glob frontmatter.

**Task 3.4** — A scenario describes a too-aggressive direct-execution result, or a small task getting bogged down in plan mode. The correct answer matches scope: plan mode for large/architectural/uncertain; direct for small/scoped/familiar.

**Task 3.5** — A scenario describes iterative review or refinement workflow. The correct answer uses the interview pattern (one issue at a time, with full context) or batched feedback depending on scope.

**Task 3.6** — A scenario describes CI/CD parsing failures or non-determinism. The correct answer reaches for `-p --output-format json --json-schema <path>` and including prior findings to avoid duplicate comments.

---

## Quick-reference cheatsheet

- **Memory hierarchy:** Enterprise → Project → User → Local. Project = `.claude/CLAUDE.md`; User = `~/.claude/CLAUDE.md`.
- **`@import`** for modularization; `/memory` to verify loaded; `/context` for live breakdown.
- **Auto memory:** `MEMORY.md` (project) or `~/.claude/MEMORY.md`. First 200 lines / 25 KB load at startup. Toggle in `/memory`.
- **`SKILL.md` frontmatter:** `context: fork` for isolation; `agent: <type>` for which subagent; `allowed-tools` for restriction; `argument-hint` for autocomplete; `disable-model-invocation: true` for manual-only; `user-invocable: false` for Claude-only; `paths: [glob]` for auto-activation by file.
- **Skills vs CLAUDE.md:** universal standards → CLAUDE.md; on-demand workflows → skills.
- **Slash commands and skills are merged** — both create `/name`.
- **Path-specific rules:** `.claude/rules/<name>.md` with `paths:` glob frontmatter.
- **Plan mode:** read-only tools, creates plan, no permission prompt to enter (in 2.1+). Use for large/architectural; direct execution for small/scoped. Plan subagent delegates research.
- **Built-in subagents:** Explore, Plan, general-purpose. Subagents CANNOT spawn other subagents.
- **Subagent frontmatter** extends skill frontmatter: `tools`, `disallowedTools`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`, `memory`, `isolation: worktree`, `background`, `effort`, `color`.
- **CI/CD:** `claude -p --output-format json --json-schema <path>`. Include prior findings.
- **`/context`** shows live context breakdown; `/compact` summarizes conversation; `/init` creates starter CLAUDE.md.
- **Settings precedence:** Managed > User > Project > Local. CLI flags override.
- **Output style** replaces software-engineering persona; preserves infrastructure. `--append-system-prompt` adds. `--system-prompt` fully replaces.
- **Skills with `disable-model-invocation: true`** keep descriptions out of context until manually invoked.
- **At compaction:** conversation summarized; skills re-injected; auto memory preserved; static CLAUDE.md preserved.

---

## Progress tracker

- [ ] All "Read in full" pages complete
- [ ] All "Skim" pages reviewed
- [ ] All hands-on drills complete
- [ ] All self-assessment recall tests passed
- [ ] Quick-reference cheatsheet memorized

---

*Domain 3 weight: **20%** of the exam. URL note: Claude Code docs are at `code.claude.com`, not `docs.claude.com`.*
