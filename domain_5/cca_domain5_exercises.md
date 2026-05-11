---
title: Claude Certified Architect — Foundations · Domain 5 Hands-On Drills
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - hands-on
  - domain-5
  - typescript
status: in-progress
estimated-hours: 2
language: typescript
---

# CCA-F Domain 5 — Hands-On Drills

Context management and reliability patterns are cross-cutting — they're tested inside scenarios driven by other domains. Most of the substantial work for Domain 5 is already covered by the Domain 1 exercises (which implement case-facts persistence, structured error propagation, and provenance preservation in real code).

This file is intentionally a shorter set of quick drills focused on the Domain 5 patterns that don't have a natural home in another domain's exercises: memory tool usage, scratchpads, sentiment vs. explicit escalation, stratified sampling for human review, and `/context` exploration.

| Item | Task statements reinforced | Time |
|------|---------------------------|------|
| Cross-references to Domain 1 exercises | 5.1, 5.3, 5.6 | (already done) |
| Quick drills (×6) | 5.1, 5.2, 5.4, 5.5 | ~90 min |

## Prerequisites

- [ ] **Node.js 20+** installed
- [ ] **Anthropic API key** in env: `export ANTHROPIC_API_KEY=sk-ant-...`
- [ ] **Claude Code** installed for `/context` and scratchpad drills
- [ ] A working directory: `mkdir -p ~/cca-prep && cd ~/cca-prep`

---

## Cross-references to Domain 1 exercises

If you completed the Domain 1 exercises, you've already implemented the substantial Domain 5 patterns. Skim the relevant exercises again with a Domain 5 lens:

- [ ] **Domain 1 Exercise 1** reinforces **Task 5.1 (case-facts persistence)** — the customer support agent injects structured case facts (customer ID, order ID, refund amount) outside the conversation summary so they survive across turns
- [ ] **Domain 1 Exercise 2** reinforces **Task 5.3 (structured error propagation)** — when a subagent times out or fails, structured error context (`failureType`, `attemptedQuery`, `partialResults`) propagates up to the coordinator for decision-making
- [ ] **Domain 1 Exercise 2** reinforces **Task 5.6 (provenance preservation)** — every claim in the synthesis carries `source_url`, `document_name`, `publication_date` as structured fields

The Domain 5 quick drills below cover the remaining patterns.

---

## Quick drill A — Memory tool usage (Task 5.1 cross-session)

### Goal

Implement the memory tool in a multi-session agent so it views the memory directory at session start, records progress as it works, and resumes correctly when context is reset.

### Steps

- [ ] Create `memory_tool_demo.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";
  import { mkdirSync, readFileSync, writeFileSync, existsSync, readdirSync, statSync, unlinkSync } from "node:fs";
  import { join } from "node:path";

  const client = new Anthropic();

  // Client-side storage for the memory tool — you control where this lives
  const MEMORY_DIR = "./memory_demo";
  mkdirSync(MEMORY_DIR, { recursive: true });

  // Implement the memory tool's commands locally
  type MemoryCommand =
    | { command: "view"; path: string }
    | { command: "create"; path: string; file_text: string }
    | { command: "update"; path: string; old_str: string; new_str: string }
    | { command: "delete"; path: string };

  function executeMemoryCommand(cmd: MemoryCommand): string {
    // Resolve path: /memories/foo.md → MEMORY_DIR/foo.md
    const resolveMem = (p: string) => join(MEMORY_DIR, p.replace(/^\/memories\/?/, ""));

    if (cmd.command === "view") {
      const path = resolveMem(cmd.path);
      if (!existsSync(path)) {
        return `Path not found: ${cmd.path}`;
      }
      const stat = statSync(path);
      if (stat.isDirectory()) {
        const entries = readdirSync(path);
        return entries.map((e) => `${e}`).join("\n");
      } else {
        return readFileSync(path, "utf-8");
      }
    }
    if (cmd.command === "create") {
      writeFileSync(resolveMem(cmd.path), cmd.file_text);
      return `Created ${cmd.path}`;
    }
    if (cmd.command === "update") {
      const path = resolveMem(cmd.path);
      const content = readFileSync(path, "utf-8");
      const updated = content.replace(cmd.old_str, cmd.new_str);
      writeFileSync(path, updated);
      return `Updated ${cmd.path}`;
    }
    if (cmd.command === "delete") {
      unlinkSync(resolveMem(cmd.path));
      return `Deleted ${cmd.path}`;
    }
    return "Unknown command";
  }

  // The memory tool definition the model sees
  const memoryTool: Anthropic.Tool = {
    name: "memory",
    description:
      "Read and write files in the /memories directory. ALWAYS view /memories before starting work to check for prior context.",
    input_schema: {
      type: "object",
      properties: {
        command: {
          type: "string",
          enum: ["view", "create", "update", "delete"],
        },
        path: { type: "string" },
        file_text: { type: ["string", "null"] },
        old_str: { type: ["string", "null"] },
        new_str: { type: ["string", "null"] },
      },
      required: ["command", "path"],
    },
  };

  // Run an agent loop that processes one task; memory persists across calls
  async function runTask(taskDescription: string) {
    const messages: Anthropic.MessageParam[] = [
      {
        role: "user",
        content: `${taskDescription}\n\nIMPORTANT: Before starting, view /memories to check for any prior context or progress on this work.`,
      },
    ];

    let stopReason: string = "";
    let turns = 0;
    while (stopReason !== "end_turn" && turns < 10) {
      turns++;
      const response = await client.messages.create({
        model: "claude-sonnet-4-6",
        max_tokens: 2048,
        tools: [memoryTool],
        messages,
      });
      stopReason = response.stop_reason ?? "";

      const toolUses = response.content.filter((b) => b.type === "tool_use");
      if (toolUses.length === 0) {
        // Print the final text response
        const text = response.content
          .filter((b) => b.type === "text")
          .map((b) => (b as { text: string }).text)
          .join("\n");
        console.log(`\n[Agent final response]\n${text}`);
        break;
      }

      messages.push({ role: "assistant", content: response.content });

      const results: Anthropic.ToolResultBlockParam[] = [];
      for (const tu of toolUses) {
        if (tu.type !== "tool_use") continue;
        const result = executeMemoryCommand(tu.input as MemoryCommand);
        console.log(`[memory.${(tu.input as MemoryCommand).command}] ${(tu.input as MemoryCommand).path}`);
        results.push({
          type: "tool_result",
          tool_use_id: tu.id,
          content: result,
        });
      }
      messages.push({ role: "user", content: results });
    }
  }

  // Run two sequential sessions — memory persists between them
  console.log("=== Session 1 ===");
  await runTask(
    "Begin a research project: compare the lifespans of three model organisms — C. elegans, killifish, and naked mole rats. Start by writing a /memories/research_plan.md with the comparison structure and initial notes.",
  );

  console.log("\n=== Session 2 (memory persists) ===");
  await runTask(
    "Continue the model organisms research. Look at what's in /memories and pick up where you left off.",
  );
  ```

- [ ] Run it:
  ```bash
  npx tsx memory_tool_demo.ts
  ```

- [ ] Inspect what Claude saved:
  ```bash
  ls -la memory_demo/
  cat memory_demo/research_plan.md
  ```

### Verification

- Session 1 creates files in `./memory_demo/`
- Session 2 views the memory directory first, finds the prior work, and continues from there
- The two sessions share no conversation history — only the contents of `/memories`

### Patterns reinforced

- **Task 5.1 — Memory tool (cross-session persistence):** Claude actively uses `view`, `create`, `update`, `delete` commands to manage the `/memories` directory. Distinct from case facts (which the application injects deterministically) and from auto memory (which is Claude-written but only loaded at session start in Claude Code).
- **Memory protocol pattern:** the system prompt or task description instructs Claude to view memory first ("ALWAYS view /memories before starting"). This canonical prompt comes from the Anthropic docs.
- **Client-side storage:** you control where memory is physically stored — local filesystem, S3, database, whatever fits your deployment.

---

## Quick drill B — Scratchpad pattern (Task 5.4)

### Goal

In a Claude Code session, write key findings to a scratchpad file. Let context auto-compact. Confirm Claude can resume by re-reading the scratchpad.

### Steps

- [ ] In a real codebase (or `~/cca-prep`), start Claude Code:
  ```bash
  cd ~/cca-prep
  claude
  ```

- [ ] Ask Claude to explore the codebase and write findings to a scratchpad:
  ```
  Explore this codebase. As you work, write key findings (architecture, conventions, gotchas) to .claude/notes/exploration.md. After exploration, summarize what you found.
  ```

- [ ] After exploration, verify the scratchpad exists:
  ```bash
  cat .claude/notes/exploration.md
  ```

- [ ] Run `/context` to see your current context utilization.

- [ ] Now intentionally fill context with verbose work — ask Claude to read 10+ large files in sequence.

- [ ] Trigger compaction: `/compact`.

- [ ] After compaction, ask Claude to "continue where we left off." Notice it may not remember specifics from before compaction.

- [ ] Now point Claude back to the scratchpad: "Re-read `.claude/notes/exploration.md` and continue from there."

- [ ] Confirm Claude resumes with the earlier findings intact.

### Verification

- The scratchpad file is created with structured findings
- After `/compact`, conversation history is lost but the scratchpad survives
- Re-reading the scratchpad restores Claude's context on the earlier work

### Patterns reinforced

- **Task 5.4 — Scratchpad pattern:** for findings that should survive compaction, write to external files (`.claude/notes/<topic>.md`). These are lossless (not summarized) and re-readable on demand.
- **Compaction trade-off:** `/compact` summarizes conversation history (lossy). Scratchpads are external files (lossless). They complement each other.
- **The "use a bigger context window" trap:** context windows always have limits, and context rot degrades accuracy long before the limit. What's *in* context matters more than how much space is available.

---

## Quick drill C — Sentiment vs. explicit escalation (Task 5.2)

### Goal

Confirm the documented pattern: escalate on **explicit request**, not sentiment.

### Steps

- [ ] Create `escalation_test.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const escalateTool: Anthropic.Tool = {
    name: "escalate_to_human",
    description:
      "Escalate the conversation to a human support agent. Use ONLY when the customer explicitly asks for a human, supervisor, manager, or escalation. Do not use based on sentiment alone — frustrated customers who do not explicitly ask for a human should be helped, not escalated.",
    input_schema: {
      type: "object",
      properties: { reason: { type: "string" } },
      required: ["reason"],
    },
  };

  const respondTool: Anthropic.Tool = {
    name: "respond_to_customer",
    description: "Respond to the customer with a helpful message.",
    input_schema: {
      type: "object",
      properties: { message: { type: "string" } },
      required: ["message"],
    },
  };

  const testMessages = [
    // Should NOT escalate — frustrated but not asking for human
    "This is so annoying. I've been on hold for 20 minutes and your website keeps crashing.",
    // Should NOT escalate — venting
    "Honestly, this is the worst customer service experience I've ever had. I'm furious.",
    // SHOULD escalate — explicit request
    "I want to speak to a human, please.",
    // SHOULD escalate — explicit request (different phrasing)
    "Can you connect me to a manager?",
    // SHOULD escalate — explicit request (subtle)
    "Is there someone else I can talk to about this?",
    // Should NOT escalate — calm question
    "Could you help me reset my password?",
  ];

  for (const msg of testMessages) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 512,
      tools: [escalateTool, respondTool],
      tool_choice: { type: "any" },
      messages: [
        {
          role: "system" as never, // (system is normally a top-level field; using user-role for the system context here)
          content: "You are a customer support agent. Help customers resolve their issues. Only escalate when explicitly asked.",
        } as never,
        { role: "user", content: msg },
      ],
    });
    const toolUse = response.content.find((b) => b.type === "tool_use");
    const action =
      toolUse && toolUse.type === "tool_use" ? toolUse.name : "no_tool";
    console.log(`Message: "${msg.slice(0, 60)}..."`);
    console.log(`  → ${action}`);
    console.log();
  }
  ```

  (Note: a proper system prompt is set via the top-level `system` field on `messages.create()`; the snippet above uses an inline message for brevity. In production, use `system: "..."` at the top level.)

- [ ] Run it:
  ```bash
  npx tsx escalation_test.ts
  ```

- [ ] Verify:
  - Messages 1 (frustrated about hold time), 2 (venting), 6 (calm question) → `respond_to_customer`
  - Messages 3, 4, 5 (explicit requests for human/manager/someone else) → `escalate_to_human`

### Patterns reinforced

- **Task 5.2 — Explicit escalation:** sentiment is unreliable as an escalation signal. False positives on frustrated venting; false negatives on calm explicit requests. The tool description encodes the rule, but the agent's reasoning depends on the description making the distinction clearly.
- **The wrong answers in scenarios:** "analyze sentiment and escalate on frustration threshold" or "use a sentiment subagent to gate escalation."

---

## Quick drill D — Stratified sampling for review (Task 5.5)

### Goal

For a batch of extractions with field-level confidence scores, design a sampling strategy that prioritizes low-confidence fields for human review.

### Steps

- [ ] Create `stratified_sampling.ts`:
  ```typescript
  type FieldConfidence = "high" | "medium" | "low";

  type ExtractedRecord = {
    id: string;
    fields: {
      vendor_name: { value: string; confidence: FieldConfidence };
      invoice_number: { value: string; confidence: FieldConfidence };
      invoice_date: { value: string | null; confidence: FieldConfidence };
      total: { value: number; confidence: FieldConfidence };
      expense_category: { value: string; confidence: FieldConfidence };
    };
  };

  // Simulated batch of 100 extractions with synthetic confidence distributions
  function generateBatch(n: number): ExtractedRecord[] {
    const records: ExtractedRecord[] = [];
    for (let i = 0; i < n; i++) {
      const conf = (): FieldConfidence => {
        const r = Math.random();
        return r < 0.7 ? "high" : r < 0.92 ? "medium" : "low";
      };
      records.push({
        id: `inv_${String(i).padStart(4, "0")}`,
        fields: {
          vendor_name: { value: `Vendor ${i}`, confidence: conf() },
          invoice_number: { value: `INV-${i}`, confidence: conf() },
          invoice_date: { value: "2026-01-01", confidence: conf() },
          total: { value: 100 + i, confidence: conf() },
          expense_category: { value: "services", confidence: conf() },
        },
      });
    }
    return records;
  }

  // NAIVE strategy — sample every Nth record
  function naiveSample(records: ExtractedRecord[], n: number): ExtractedRecord[] {
    const step = Math.floor(records.length / n);
    return records.filter((_, i) => i % step === 0).slice(0, n);
  }

  // STRATIFIED strategy — prioritize records with low-confidence fields
  function stratifiedSample(records: ExtractedRecord[], n: number): Array<ExtractedRecord & { reason: string }> {
    // Score each record by number of low-confidence fields, then medium
    const scored = records.map((r) => {
      const fields = Object.entries(r.fields);
      const low = fields.filter(([, f]) => f.confidence === "low").length;
      const medium = fields.filter(([, f]) => f.confidence === "medium").length;
      const lowFields = fields.filter(([, f]) => f.confidence === "low").map(([k]) => k);
      const mediumFields = fields.filter(([, f]) => f.confidence === "medium").map(([k]) => k);
      return {
        ...r,
        _score: low * 10 + medium,
        reason: low > 0
          ? `low: ${lowFields.join(", ")}`
          : medium > 0
            ? `medium: ${mediumFields.join(", ")}`
            : "all-high",
      };
    });

    // Sort descending by score (highest review-need first)
    scored.sort((a, b) => b._score - a._score);
    return scored.slice(0, n).map(({ _score, ...rest }) => rest);
  }

  const batch = generateBatch(100);
  const reviewBudget = 10; // we can manually review 10 of 100

  console.log("=== Naive sampling (every 10th record) ===");
  const naive = naiveSample(batch, reviewBudget);
  console.log(
    `Captured low-confidence records: ${naive.filter((r) => Object.values(r.fields).some((f) => f.confidence === "low")).length} / ${reviewBudget}`,
  );

  console.log("\n=== Stratified sampling (lowest-confidence first) ===");
  const stratified = stratifiedSample(batch, reviewBudget);
  console.log(
    `Captured low-confidence records: ${stratified.filter((r) => Object.values(r.fields).some((f) => f.confidence === "low")).length} / ${reviewBudget}`,
  );
  console.log("\nReview queue:");
  for (const r of stratified) {
    console.log(`  ${r.id}: ${r.reason}`);
  }
  ```

- [ ] Run it:
  ```bash
  npx tsx stratified_sampling.ts
  ```

### Verification

- Naive sampling captures roughly 10-30% of low-confidence records (matches the population's base rate)
- Stratified sampling captures **100%** of low-confidence records within the review budget (assuming budget is large enough to cover all low-confidence cases)
- The stratified queue identifies *which fields* need review per record, not just which records

### Patterns reinforced

- **Task 5.5 — Field-level confidence + stratified sampling:**
  - Record-level confidence ("80% confident in this whole invoice") doesn't tell reviewers which fields to check
  - Field-level confidence ("95% on vendor_name, 60% on line_items, 30% on tax_amount") routes reviewers to specific fields
  - Stratified sampling prioritizes records by review need, not by position
- **The wrong answers:** "review every Nth record," "use record-level confidence only," "have Claude review its own output."

---

## Quick drill E — `/context` exploration

### Goal

In Claude Code, work through a long session. Run `/context` periodically to see live context utilization and observe which categories grow.

### Steps

- [ ] Start Claude Code in a codebase with non-trivial complexity:
  ```bash
  cd ~/cca-prep   # or a real project
  claude
  ```

- [ ] Run `/context` immediately at session start. Note baseline: CLAUDE.md, auto memory, MCP tool names, skill descriptions, system prompt — typically a few thousand tokens.

- [ ] Have Claude do progressively heavier work:
  1. Read one file — note context growth
  2. Read 5 more files — note larger growth
  3. Use a verbose skill (or have Claude do a deep exploration) — note significant growth
  4. Use a subagent for exploration — note context stays roughly stable in main (subagent has its own)

- [ ] Run `/context` after each step. Watch which categories grow:
  - **Conversation history** grows linearly with messages
  - **Tool results** grow with each file read or command output
  - **CLAUDE.md / skills / MCP** stay roughly flat (they're loaded once)
  - **Subagent work** doesn't appear in main context — only the summary that subagent returns

- [ ] When utilization hits ~60%, run `/compact` proactively. Then run `/context` again to see the post-compaction breakdown.

### Verification

- The `/context` output shows category-by-category breakdown
- Conversation history is the dominant grower
- Subagent work stays isolated in subagent context
- `/compact` reduces total but preserves auto memory and static CLAUDE.md

### Patterns reinforced

- **Task 5.4 — Context observability:**
  - `/context` for live breakdown
  - `/memory` for which CLAUDE.md and auto memory files are loaded
  - `/compact` proactively at ~60% (auto-compact triggers around 64-75% in current Claude Code versions)
  - Subagent isolation is a context-management tool, not just a task-routing tool

---

## Quick drill F — Provenance break experiment (Task 5.6, optional)

### Goal

If you completed Domain 1 Exercise 2, revisit it with a focused break-and-fix experiment for provenance preservation.

### Steps

- [ ] Open the research coordinator from Domain 1 Exercise 2.

- [ ] **Strong synthesis prompt** (baseline): every claim in the synthesis output carries `source_url`, `document_name`, `publication_date` as required fields. Run on a test query. Verify attribution is preserved.

- [ ] **Weaken the synthesis prompt** to "cite sources where possible." Re-run on the same test query. Observe attribution become inconsistent — sometimes present, sometimes missing, sometimes just a URL with no document name.

- [ ] **Remove the schema requirement** for source fields. Re-run. Attribution often disappears entirely; the synthesis becomes prose with occasional URLs.

- [ ] **Restore the strict schema and prompt.** Attribution returns to 100%.

### Verification

The exercise gives you a concrete feel for why "cite sources where possible" is a documented anti-pattern. Probabilistic instructions ("where possible") fail under load; structural requirements (every claim has source fields in the schema) hold.

### Patterns reinforced

- **Task 5.6 — Provenance through synthesis:**
  - Structural requirements (schema demands `source_url` on every claim) are deterministic
  - Probabilistic instructions ("cite where possible") are unreliable
  - Synthesis output = **collection of attributed claims**, not prose summary that occasionally references sources
- **Conflict annotation:** when subagents disagree, surface the conflict with attribution ("Source A reports X; Source B reports Y") rather than silently reconciling. (Implementing this is a small extension to Exercise 2.)

---

## Quick-reference cheatsheet for the patterns drilled

- **Case facts (Task 5.1):** structured JSON block injected into every prompt; survives compaction; for must-not-drift identifiers. Application-injected, deterministic.
- **Memory tool (Task 5.1 cross-session):** Claude actively uses `view`/`create`/`update`/`delete` commands on `/memories` directory. Client-side storage you control.
- **Auto memory (Claude Code):** MEMORY.md auto-written by Claude; first 200 lines / 25 KB load at session start.
- **Escalation (Task 5.2):** explicit request only. Sentiment unreliable.
- **Error propagation (Task 5.3):** structured `errorCategory`, `isRetryable`, `description`, partial results. Never silently suppress.
- **Scratchpads (Task 5.4):** external files (`.claude/notes/<topic>.md`); re-read on demand; survive compaction; lossless.
- **`/context` and `/memory`** for observability; `/compact` proactively at ~60%.
- **Subagent context isolation (Task 5.4):** verbose work in subagent stays there; only summary returns.
- **Compaction beta:** `compact-2026-01-12` header, strategy `compact_20260112` in `context_management.edits`.
- **Context editing beta:** `context-management-2025-06-27` header, strategies `clear_tool_uses_20250919` + thinking-block clearing + client-side compaction.
- **Field-level confidence + stratified sampling (Task 5.5):** route reviewers to specific fields. NOT record-level. NOT same-session self-review.
- **Provenance (Task 5.6):** structural claim-source mappings preserved through synthesis. Every claim carries `source_url`, `document_name`, `publication_date`.
- **Conflict annotation:** surface disagreements with attribution; don't silently reconcile.

---

## Progress tracker

- [ ] Cross-references to Domain 1 exercises reviewed (5.1, 5.3, 5.6)
- [ ] Quick drill A — Memory tool usage
- [ ] Quick drill B — Scratchpad pattern
- [ ] Quick drill C — Sentiment vs. explicit escalation
- [ ] Quick drill D — Stratified sampling for review
- [ ] Quick drill E — `/context` exploration
- [ ] Quick drill F — Provenance break experiment (optional)

---

*Domain 5 weight: **15%** of the exam — smallest single domain, but cross-cutting. Most patterns reinforced inside Domain 1 exercises; this file fills the remaining gaps.*
