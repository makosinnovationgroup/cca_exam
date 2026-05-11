---
title: Claude Certified Architect — Foundations · Domain 3 Hands-On Exercises
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - hands-on
  - domain-3
status: in-progress
estimated-hours: 4
language: typescript
---

# CCA-F Domain 3 — Hands-On Exercises

Practical fluency in Claude Code configuration, skills, slash commands, custom subagents, plan mode, and CI/CD integration — patterns most heavily tested in Domain 3 (20% of the exam).

> [!note] On language
> Domain 3 is largely Claude Code configuration: Markdown skill files with YAML frontmatter, JSON settings, bash CLI commands, and JSON schemas for headless mode. Where there is code to write, it's TypeScript. There is no Python in these exercises.

| Item | Task statements reinforced | Time |
|------|---------------------------|------|
| Exercise 1 — Custom skill with `context: fork` and `agent: Explore` | 3.2 | ~45 min |
| Exercise 2 — Custom subagent with full frontmatter | 3.2, 3.4 | ~45 min |
| Exercise 3 — CI-style headless run with JSON schema validation | 3.6 | ~60 min |
| Quick drills (×8) | 3.1, 3.3, 3.4, 3.5 | ~90 min |

## Prerequisites

- [ ] **Claude Code** installed and authenticated — verify with `claude --version`
- [ ] **Claude Code v2.1.59+** for auto memory (`claude --version`)
- [ ] A working directory: `mkdir -p ~/cca-prep && cd ~/cca-prep`
- [ ] **Node.js 20+** for the Exercise 3 CI test harness

> [!info] About `code.claude.com`
> The Claude Code docs live at `code.claude.com/docs/en/...`. The frontmatter reference for `SKILL.md` is at `code.claude.com/docs/en/skills`. Bookmark this page — you'll reference it across all three substantial exercises.

---

## Exercise 1 — Custom skill with `context: fork` and `agent: Explore`

### Goal

Build a custom skill that runs in an isolated forked subagent context with read-only tools, observe how verbose exploration stays out of the main conversation, and confirm only the summary returns.

### Why this matters

Task 3.2 tests recognition of the `context: fork` + `agent: <type>` pattern. The exam asks about scenarios where verbose skill output would pollute the main conversation, where read-only exploration is needed without history, and where skill frontmatter fields like `allowed-tools`, `disable-model-invocation`, and `argument-hint` solve specific failure modes.

### Steps

- [ ] Create the project skill directory:
  ```bash
  cd ~/cca-prep
  mkdir -p .claude/skills/dep-analyzer
  ```

- [ ] Create `.claude/skills/dep-analyzer/SKILL.md`:
  ```markdown
  ---
  name: dep-analyzer
  description: Analyze how a dependency is used across the codebase. Use when the user asks where a package is imported, what features of it are used, or whether it's safe to remove or upgrade. Produces a structured summary that fits in one screen.
  argument-hint: <package-name>
  context: fork
  agent: Explore
  allowed-tools: [Read, Grep, Glob]
  ---

  # Dependency Analyzer

  You are analyzing how the package `$ARGUMENTS` is used across the codebase.

  Produce a single structured report with these sections:

  1. **Import sites** — list every file that imports `$ARGUMENTS`, grouped by directory. Use Grep for the import statements; use Glob if you need to narrow by file type first.
  2. **Used surfaces** — for each unique import (named imports, default imports, type-only imports), list which symbols are used and where.
  3. **Risk assessment** — flag any imports inside critical paths (auth, payments, core API routes). Use file-path heuristics to identify these.
  4. **Removal/upgrade summary** — if the user is considering removing or upgrading the dependency, estimate the blast radius in lines of code touched.

  Be terse. Total report should fit in 50 lines or fewer. Do not include code snippets longer than 3 lines.
  ```

- [ ] Verify the skill is discovered. Start Claude Code in this directory:
  ```bash
  cd ~/cca-prep
  claude
  ```
  Then in Claude Code, type `/` and confirm `/dep-analyzer` appears in the list. Type `/dep-analyzer react` (or some package name in a project you have) and run it.

- [ ] **Observe:** the skill runs in a forked Explore subagent. Verbose tool calls (Grep across many files, Read of many files) happen inside the subagent's context — your main conversation only sees the final structured report.

- [ ] Verify the read-only restriction works: try to use the skill on a package and see if Claude attempts to write any files. With `allowed-tools: [Read, Grep, Glob]`, write operations are blocked.

### Verification

- The skill appears in the `/` menu in Claude Code
- Running it produces the structured 4-section report
- Verbose intermediate tool calls do not appear in your main conversation history
- Editing the codebase from within the skill is impossible (no Edit / Write in `allowed-tools`)

### Patterns reinforced

- **Task 3.2 — Skill frontmatter:**
  - `context: fork` — runs the skill in an isolated subagent
  - `agent: Explore` — uses the built-in Explore subagent (read-only, optimized for codebase research)
  - `allowed-tools: [...]` — restricts the tools available to the skill
  - `argument-hint: <package-name>` — prompts the user for an argument
  - `description` — drives auto-invocation; `Use when ...` phrasing is the trigger pattern
- **Skill content becomes the prompt** that drives the subagent. The subagent doesn't see your conversation history — context comes from the skill text and the argument.

### Failure-mode exploration

- [ ] Remove `context: fork` from the frontmatter. Run the skill again. Observe how the verbose Grep/Read output now floods your main conversation.
- [ ] Remove `agent: Explore` but keep `context: fork`. The skill still runs in a forked context, just using the general-purpose subagent (less optimized for read-only research, fewer pre-loaded conventions).
- [ ] Add `Edit` to `allowed-tools`. Now the skill can modify files — useful if you want the skill to fix something it finds, but verify you want that authority.
- [ ] Add `disable-model-invocation: true`. The skill no longer auto-invokes when relevant; only manual `/dep-analyzer` triggers it. Useful for skills with side effects.

---

## Exercise 2 — Custom subagent with full frontmatter

### Goal

Define a custom subagent in `.claude/agents/`, configure it with the full set of frontmatter fields (tools, model, permissionMode, mcpServers, hooks, maxTurns), and invoke it both via `@`-mention and via `--agent <name>` on the CLI.

### Why this matters

Task 3.2 + 3.4 test the distinction between skills, subagents, and slash commands. Subagent frontmatter is a superset of skill frontmatter — `tools`, `disallowedTools`, `permissionMode`, `mcpServers`, `maxTurns`, `memory`, and `isolation` are subagent-only fields. Knowing what each does is heavily testable.

### Steps

- [ ] Create the subagent directory:
  ```bash
  cd ~/cca-prep
  mkdir -p .claude/agents
  ```

- [ ] Create `.claude/agents/security-reviewer.md`:
  ```markdown
  ---
  name: security-reviewer
  description: Reviews code for security vulnerabilities. Use when the user asks for a security review, audit, or vulnerability check.
  tools:
    - Read
    - Grep
    - Glob
    - Bash
  disallowedTools:
    - Edit
    - Write
  model: opus
  permissionMode: default
  maxTurns: 20
  color: red
  ---

  You are a senior security engineer with deep expertise in:
  - OWASP Top 10 vulnerabilities
  - Authentication and authorization flaws (broken access control, session fixation, token leakage)
  - Injection vulnerabilities (SQL, command, XSS, path traversal)
  - Insecure cryptography (weak hashing, hardcoded secrets, predictable randomness)
  - Information disclosure (excessive error messages, debug endpoints in production, secrets in logs)
  - Insecure dependencies (known CVEs, outdated packages)

  When reviewing code:

  1. Identify specific vulnerabilities with line references
  2. Rate severity: Critical / High / Medium / Low / Informational
  3. Suggest concrete fixes with code-level guidance
  4. Distinguish definite issues from potential issues that need verification
  5. Do not modify files — output a report only

  Output structure:
  ```
  ## Findings (N total)

  ### Critical
  - <file>:<line> — <issue>. Fix: <suggestion>.

  ### High
  - ...

  ### Medium / Low / Informational
  - ...

  ## Out-of-scope
  Things you noticed but couldn't verify without runtime testing or context you don't have.
  ```
  ```

- [ ] Confirm the subagent is registered. In Claude Code:
  ```
  /agents
  ```
  You should see `security-reviewer` listed.

- [ ] Invoke via `@`-mention in any conversation:
  ```
  @security-reviewer please review the auth code in src/auth/
  ```
  The subagent runs in its own context with its restricted toolset.

- [ ] Try running an **entire session** as the subagent using the `--agent` flag:
  ```bash
  claude --agent security-reviewer "Review the codebase for hardcoded secrets and credentials."
  ```
  In this mode the subagent's prompt replaces the default Claude Code system prompt entirely — Claude Code defaults like the software-engineering persona are gone.

### Verification

- [ ] Subagent appears in `/agents` list
- [ ] `@security-reviewer` invocation works inline
- [ ] `claude --agent security-reviewer ...` works as a session-wide mode
- [ ] Asking the subagent to modify files is blocked (`disallowedTools: [Edit, Write]` takes effect)
- [ ] The `maxTurns: 20` cap prevents runaway iteration if the subagent gets stuck

### Patterns reinforced

- **Task 3.2 — Subagent frontmatter extends skill frontmatter:**
  - `tools` — allowed tool list (intersection with `allowed-tools` semantics)
  - `disallowedTools` — explicit denylist (takes precedence over `tools`)
  - `model` — override the default model (`opus`, `sonnet`, `haiku`, or specific version strings)
  - `permissionMode` — `default` (ask), `acceptEdits`, `bypassPermissions`, `plan`
  - `maxTurns` — iteration safety cap
  - `mcpServers` — restrict which MCP servers this subagent can use
  - `hooks` — subagent-scoped hooks
  - `memory` — persistent memory directory for cross-conversation continuity
  - `isolation: worktree` — run in a git worktree for filesystem isolation
  - `background`, `effort`, `color` — UI / scheduling fields
- **`--agent <name>`** runs the session as the subagent — the subagent's prompt replaces the default Claude Code system prompt entirely
- **Set `agent` in `.claude/settings.json`** to make a subagent the default for every session in the project
- **Plugin subagents** don't support `hooks`, `mcpServers`, or `permissionMode` (security restriction)

### Failure-mode exploration

- [ ] Remove `disallowedTools`. Ask the subagent to fix a finding by editing the file. With `tools: [Read, Grep, Glob, Bash]` and no `Edit`, modification still fails — but you've now made the restriction less explicit. The exam tests preferring `disallowedTools` for explicit denials over relying on `tools` omissions.
- [ ] Set `model: haiku`. Run a complex security review. Watch quality degrade. Use this to internalize the trade-off: cheaper/faster model for narrow tasks, more capable model for nuanced security analysis.
- [ ] Add `hooks` configuration to enforce that the subagent always runs `npm audit` before producing findings. The hook fires `PreToolUse` on the first tool call.

---

## Exercise 3 — CI-style headless run with JSON schema validation

### Goal

Run Claude Code in non-interactive mode (`-p`) with `--output-format json` and `--json-schema` to produce parseable, schema-validated output suitable for a CI/CD pipeline.

### Why this matters

Task 3.6 tests CI/CD integration. The exam scenarios include "code review running on every PR" where the output must be machine-readable and schema-conformant. The wrong answers reach for regex parsing, prompt-engineering JSON requests, or running Claude twice and diffing. The right answer is `-p --output-format json --json-schema <path>`.

### Steps

- [ ] Create a sample diff to review. In your `~/cca-prep` directory:
  ```bash
  cat > sample_diff.txt <<'EOF'
  diff --git a/src/auth.ts b/src/auth.ts
  --- a/src/auth.ts
  +++ b/src/auth.ts
  @@ -10,7 +10,7 @@ export function login(email: string, password: string) {
     const user = db.users.findOne({ email });
     if (!user) return null;
  -  const valid = bcrypt.compareSync(password, user.passwordHash);
  +  const valid = password === user.passwordHash;
     if (!valid) return null;
     return generateToken(user);
   }
  EOF
  ```

- [ ] Create the JSON schema that defines the review output shape. Save as `review-schema.json`:
  ```json
  {
    "type": "object",
    "properties": {
      "summary": {
        "type": "string",
        "description": "One-paragraph summary of the diff and overall risk assessment."
      },
      "findings": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "severity": {
              "type": "string",
              "enum": ["critical", "high", "medium", "low", "informational"]
            },
            "category": {
              "type": "string",
              "enum": [
                "security",
                "correctness",
                "performance",
                "maintainability",
                "style",
                "other"
              ]
            },
            "category_detail": {
              "type": ["string", "null"],
              "description": "If category is 'other', describe what category. Otherwise null."
            },
            "file": { "type": "string" },
            "line": { "type": ["integer", "null"] },
            "description": { "type": "string" },
            "suggested_fix": { "type": ["string", "null"] }
          },
          "required": [
            "severity",
            "category",
            "category_detail",
            "file",
            "line",
            "description",
            "suggested_fix"
          ],
          "additionalProperties": false
        }
      },
      "blocking": {
        "type": "boolean",
        "description": "Whether any finding blocks merge (any critical or high severity)."
      }
    },
    "required": ["summary", "findings", "blocking"],
    "additionalProperties": false
  }
  ```

- [ ] Run Claude Code in headless mode:
  ```bash
  claude -p \
    --output-format json \
    --json-schema ./review-schema.json \
    "Review the diff in sample_diff.txt for issues. Return findings conforming to the schema." \
    > review_output.json
  ```

- [ ] Inspect the output:
  ```bash
  cat review_output.json | npx -y json-pretty
  ```

  Expected: a JSON object with `summary`, `findings` (a list with at least one critical security finding — the diff replaces a bcrypt comparison with plaintext equality), and `blocking: true`.

- [ ] Now write a small TypeScript CI gate script that parses the output and exits with a non-zero status if `blocking: true`. Save as `ci_gate.ts`:
  ```typescript
  import { readFileSync } from "node:fs";

  type Finding = {
    severity: "critical" | "high" | "medium" | "low" | "informational";
    category: string;
    category_detail: string | null;
    file: string;
    line: number | null;
    description: string;
    suggested_fix: string | null;
  };

  type Review = {
    summary: string;
    findings: Finding[];
    blocking: boolean;
  };

  const raw = readFileSync("review_output.json", "utf-8");
  const review: Review = JSON.parse(raw);

  console.log("Summary:", review.summary);
  console.log("Findings:", review.findings.length);

  for (const f of review.findings) {
    console.log(`  [${f.severity.toUpperCase()}] ${f.file}:${f.line ?? "?"} — ${f.description}`);
    if (f.suggested_fix) {
      console.log(`    Fix: ${f.suggested_fix}`);
    }
  }

  if (review.blocking) {
    console.error("\n✗ Review blocks merge.");
    process.exit(1);
  } else {
    console.log("\n✓ Review passes.");
    process.exit(0);
  }
  ```

- [ ] Run the gate:
  ```bash
  npx tsx ci_gate.ts
  echo "Exit code: $?"
  ```
  Expected: exit code 1 (because the diff has a critical security finding).

- [ ] Now try a clean diff:
  ```bash
  cat > clean_diff.txt <<'EOF'
  diff --git a/README.md b/README.md
  --- a/README.md
  +++ b/README.md
  @@ -1,3 +1,3 @@
   # My Project
  -A short description.
  +A short description of the project.
  EOF
  ```
  Re-run the review with `clean_diff.txt` as input and confirm `blocking: false` and exit code 0.

### Verification

- [ ] `review_output.json` is valid JSON conforming to the schema (no missing fields, no extra properties)
- [ ] `findings` includes a critical security finding for the sample diff (plaintext password comparison)
- [ ] `ci_gate.ts` exits with code 1 for the dangerous diff and code 0 for the clean diff
- [ ] No regex parsing required — `JSON.parse` works directly

### Patterns reinforced

- **Task 3.6 — CI/CD integration flags:**
  - `-p` (or `--print`) — non-interactive mode, prints output and exits
  - `--output-format json` — structured JSON output (vs default `text`)
  - `--json-schema <path>` — validates output against a schema at the protocol level
- **The protocol-level enforcement** is more reliable than prompt-engineering JSON ("RESPOND IN VALID JSON" in the prompt is probabilistic). Schema constraints constrain the generation grammar.
- **Including prior findings** in the next run: pass the previous review output back in the prompt so Claude doesn't re-report the same issues on every PR iteration. (Pattern: maintain a `prior_findings.json` and include it in the next `-p` call.)

### Failure-mode exploration

- [ ] Drop `--json-schema` and try to parse the output. Without the schema, the model still tries to return JSON because `--output-format json` is set, but the structure can drift between runs.
- [ ] Drop `--output-format json` and add "RESPOND IN VALID JSON" to your prompt. Observe how the output sometimes wraps the JSON in markdown code fences or includes a preamble — both break `JSON.parse`.
- [ ] Modify the schema to require a field the model can't actually produce (e.g., a `commit_hash` field that requires git history Claude can't see). Observe how schema-grammar-constrained output handles impossible-to-fill fields.

---

## Quick drills

Eight short drills covering the remaining Domain 3 patterns. Each is a 5–15 minute experiment.

### Quick drill A — CLAUDE.md hierarchy

**Goal:** Set up project / user / local CLAUDE.md and use `@import` to modularize.

- [ ] Create the project-level CLAUDE.md:
  ```bash
  cd ~/cca-prep
  mkdir -p .claude
  cat > .claude/CLAUDE.md <<'EOF'
  # Project Standards
  
  - Use TypeScript strict mode in all new code
  - Prefer named exports over default exports
  - Test files live next to source: `foo.ts` → `foo.test.ts`
  
  @import .claude/standards/api-conventions.md
  @import .claude/standards/error-handling.md
  EOF
  ```

- [ ] Create the imported modules:
  ```bash
  mkdir -p .claude/standards
  cat > .claude/standards/api-conventions.md <<'EOF'
  ## API Conventions
  
  - RESTful naming: nouns for resources, verbs in HTTP methods
  - All errors return JSON: `{ error: { code, message } }`
  - Use HTTP status codes correctly (404 for missing, 422 for validation)
  EOF
  
  cat > .claude/standards/error-handling.md <<'EOF'
  ## Error Handling
  
  - Catch errors at system boundaries (HTTP handlers, queue consumers, external APIs)
  - Trust internal code; don't add defensive checks for "can't happen" scenarios
  - Log structured errors: `{ level, message, error, context }`
  EOF
  ```

- [ ] Create a personal CLAUDE.md for comparison:
  ```bash
  mkdir -p ~/.claude
  cat > ~/.claude/CLAUDE.md <<'EOF'
  # Personal Preferences
  
  - Prefer concise responses
  - Use 2-space indentation in code suggestions
  EOF
  ```

- [ ] In Claude Code, run `/memory` and verify both project and user CLAUDE.md are loaded.

**Pattern reinforced:** Task 3.1 — project-level CLAUDE.md (Git-tracked, shared) vs user-level (personal); `@import` for modularization.

### Quick drill B — Auto memory exploration

**Goal:** Enable auto memory, work with Claude Code, then check what it saved.

- [ ] In Claude Code, run `/memory` and confirm auto memory is enabled (toggle if not).

- [ ] Have a short working session — ask Claude Code to refactor a small file or set up something. Make a few corrections during the work so Claude has something to learn from.

- [ ] After the session, check the auto memory file:
  ```bash
  cat ~/.claude/MEMORY.md
  # or for project-level:
  cat .claude/MEMORY.md
  ```

- [ ] **Observe:** Claude has written notes about your conventions, preferences, or project specifics. These auto-load (first 200 lines / 25 KB) at the start of every future session.

**Pattern reinforced:** Auto memory is Claude-written (vs CLAUDE.md which you write). Loads first 200 lines / 25 KB at session start. Requires Claude Code v2.1.59+.

### Quick drill C — `.claude/rules/` with glob patterns

**Goal:** Path-scoped rules that only load when matching files are touched.

- [ ] Create three rule files:
  ```bash
  mkdir -p .claude/rules
  
  cat > .claude/rules/tests.md <<'EOF'
  ---
  paths:
    - "**/*.test.ts"
    - "**/*.test.tsx"
    - "**/*.spec.ts"
  ---
  
  # Test File Rules
  
  - Use `describe`/`it` from vitest, not jest
  - One assertion per test where possible
  - Test names start with "should" or "when"
  EOF
  
  cat > .claude/rules/terraform.md <<'EOF'
  ---
  paths:
    - "terraform/**/*.tf"
    - "terraform/**/*.tfvars"
  ---
  
  # Terraform Rules
  
  - Always pin provider versions
  - Use `for_each` over `count` when iterating
  - Module outputs must include descriptions
  EOF
  
  cat > .claude/rules/api.md <<'EOF'
  ---
  paths:
    - "src/api/**/*"
  ---
  
  # API Rules
  
  - All endpoints validate input with zod
  - Return `{ data, error }` envelope, never raw data
  - Rate-limit decorators on every public endpoint
  EOF
  ```

- [ ] Start Claude Code and ask it to edit a test file. Run `/context` and confirm the test rule loaded.

- [ ] Ask Claude Code to edit a Terraform file. Run `/context` and confirm the terraform rule loaded (and ideally the test rule is no longer in context, since it's only loaded when a test file is read).

**Pattern reinforced:** Task 3.3 — path-scoped rules with `paths:` frontmatter load only when matching files are touched, preventing context bloat for rules that don't apply.

### Quick drill D — Skill with `disable-model-invocation: true`

**Goal:** Create a skill that only fires on explicit user invocation, never auto-invoked.

- [ ] Create a deploy skill that should never auto-trigger:
  ```bash
  mkdir -p .claude/skills/deploy
  cat > .claude/skills/deploy/SKILL.md <<'EOF'
  ---
  name: deploy
  description: Deploy the application to production. Manual invocation only.
  disable-model-invocation: true
  allowed-tools: [Bash, Read]
  ---
  
  # Production Deployment
  
  Deploy steps:
  
  1. Confirm tests pass: `npm test`
  2. Build the application: `npm run build`
  3. Push to deploy target: `git push production main`
  4. Verify health endpoint
  EOF
  ```

- [ ] In Claude Code, ask "can you deploy this?" — observe that Claude does NOT auto-invoke the skill, even though the description matches the intent.

- [ ] Now invoke it explicitly: `/deploy`. The skill runs.

**Pattern reinforced:** Task 3.2 — `disable-model-invocation: true` makes a skill manual-only. Useful for skills with side effects (deploy, send-email, modify-production).

### Quick drill E — Plan mode vs direct execution

**Goal:** Feel the difference between modes by deliberately misusing them.

- [ ] Take a small, well-scoped task you'd normally do directly:
  > "Add a null check in `src/utils/parseUser.ts` line 42."
  
  Enter plan mode (`/plan`), have Claude propose a plan. Notice the overhead — for a one-line change, the plan-creation phase is heavier than the change itself.

- [ ] Take a larger task you'd normally plan:
  > "Migrate this codebase from CommonJS to ES modules — there are about 70 files."
  
  Try direct execution without plan mode. Notice Claude making decisions you'd want to review (which files first, whether to update package.json, which ESM conventions to use) without giving you a chance to redirect.

- [ ] Now do them correctly: direct execution for the null check, plan mode for the migration.

**Pattern reinforced:** Task 3.4 — plan mode for uncertain / multi-file / architectural; direct execution for small / scoped / familiar. Mismatch in either direction has a cost.

### Quick drill F — Slash command with argument hint

**Goal:** Custom slash command with a typed argument that prompts the user.

- [ ] Create `.claude/commands/review-pr.md`:
  ```markdown
  ---
  name: review-pr
  description: Review a pull request and produce a structured review.
  argument-hint: <pr-number>
  ---
  
  Review pull request #$ARGUMENTS:
  
  1. Use `gh pr view $ARGUMENTS --json title,body,files,additions,deletions`
  2. Use `gh pr diff $ARGUMENTS` to get the full diff
  3. Review the diff for security, correctness, and style issues
  4. Output a structured review with severity-rated findings
  ```

- [ ] In Claude Code, type `/` and select `/review-pr`. Observe the prompt for the argument.

- [ ] Run `/review-pr 123` (or any PR number you have access to).

**Pattern reinforced:** `argument-hint` provides UX guidance for required arguments. `$ARGUMENTS` in the body is the substitution token.

### Quick drill G — Output style experiment

**Goal:** Create a custom output style that turns Claude Code into a teaching tutor while keeping the agent infrastructure.

- [ ] Create `.claude/output-styles/tutor.md`:
  ```markdown
  ---
  name: tutor
  description: Teaching tutor mode — explains concepts before making changes.
  ---
  
  You are a senior engineer mentoring a junior developer.
  
  Before making any code changes:
  
  1. Explain the relevant concept in 2-3 sentences
  2. Describe the approach you're going to take and why
  3. Make the change
  4. Point out what to look for if this pattern comes up again
  
  Use the Socratic method for ambiguous questions — ask the developer one clarifying question before proceeding when the goal is unclear.
  ```

- [ ] Apply the output style: `/output-style tutor` (or via settings).

- [ ] Ask Claude Code to fix something. Observe the tutor framing.

- [ ] Confirm tools, MCP, sub-agents, and slash commands all still work — only the persona changed.

**Pattern reinforced:** Output styles replace the software-engineering portion of the system prompt while preserving infrastructure. Distinct from `--append-system-prompt` (which adds to default) and `--system-prompt` (which fully replaces).

### Quick drill H — Settings layering test

**Goal:** Confirm settings precedence: Managed > User > Project > Local.

- [ ] Set the same setting at three levels with different values:
  
  Project (`~/cca-prep/.claude/settings.json`):
  ```json
  {
    "permissions": {
      "deny": ["Bash(curl *)"]
    }
  }
  ```
  
  Local (`~/cca-prep/.claude/settings.local.json`):
  ```json
  {
    "permissions": {
      "deny": ["Bash(curl *)", "Bash(rm -rf *)"]
    }
  }
  ```
  
  User (`~/.claude/settings.json`):
  ```json
  {
    "permissions": {
      "deny": ["Bash(sudo *)"]
    }
  }
  ```

- [ ] Start Claude Code in `~/cca-prep` and run `/permissions`. Observe the effective rule set is a merge: all three sources contribute their rules, with `Local > Project > User` precedence on conflicts.

- [ ] Override via CLI: `claude --settings ./alt-settings.json`. The CLI flag takes precedence over all of the above for this session.

**Pattern reinforced:** Settings precedence (Managed > User > Project > Local). CLI flags override the file-based hierarchy for the session.

---

## Quick-reference cheatsheet for the patterns drilled

- **Memory hierarchy:** Enterprise → Project → User → Local. `.claude/CLAUDE.md` (project, Git-tracked) vs `~/.claude/CLAUDE.md` (user, personal).
- **`@import`** in CLAUDE.md for modularization; `/memory` to verify loaded; `/context` for live breakdown.
- **Auto memory:** `MEMORY.md` (project) or `~/.claude/MEMORY.md` (user). First 200 lines / 25 KB load at startup. Claude Code v2.1.59+.
- **Skill frontmatter:** `context: fork` for isolation; `agent: <type>` for which subagent; `allowed-tools` for restriction; `argument-hint` for autocomplete; `disable-model-invocation: true` for manual-only; `user-invocable: false` for Claude-only.
- **Subagent frontmatter** extends skill frontmatter: `tools`, `disallowedTools`, `model`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`, `memory`, `isolation: worktree`.
- **Path-scoped rules:** `.claude/rules/<name>.md` with `paths:` glob frontmatter; loads only when matching file touched.
- **Slash commands and skills are merged** — both create `/name`.
- **Plan mode:** read-only tools, creates plan; for large/uncertain. Direct execution for small/scoped.
- **CI/CD:** `claude -p --output-format json --json-schema <path>`. Include prior findings.
- **Settings precedence:** Managed > User > Project > Local. CLI flags override.
- **Output style** replaces software-engineering portion of system prompt; preserves infrastructure. `--append-system-prompt` adds. `--system-prompt` fully replaces.
- **`--agent <name>`** runs whole session as the named subagent (replaces system prompt entirely).

---

## Progress tracker

- [ ] Exercise 1 complete (skill with `context: fork`)
- [ ] Exercise 1 failure-mode exploration complete
- [ ] Exercise 2 complete (custom subagent)
- [ ] Exercise 2 failure-mode exploration complete
- [ ] Exercise 3 complete (CI-style headless run with JSON schema)
- [ ] Exercise 3 failure-mode exploration complete
- [ ] Quick drill A — CLAUDE.md hierarchy
- [ ] Quick drill B — Auto memory exploration
- [ ] Quick drill C — `.claude/rules/` with globs
- [ ] Quick drill D — `disable-model-invocation: true`
- [ ] Quick drill E — Plan mode vs direct execution
- [ ] Quick drill F — Slash command with argument hint
- [ ] Quick drill G — Output style experiment
- [ ] Quick drill H — Settings layering test

---

*Domain 3 weight: **20%** of the exam.*
