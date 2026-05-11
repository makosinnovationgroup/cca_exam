---
title: Claude Certified Architect — Foundations · Domain 1 Hands-On Exercises (TypeScript)
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - hands-on
  - domain-1
  - typescript
status: in-progress
created: 2026-05-11
estimated-hours: 3-4
language: typescript
---

# CCA-F Domain 1 — Hands-On Exercises (TypeScript)

> [!info] Linked notes
> [[cca_exam_guide]] · [[cca_docs_roadmap]] · [[cca_progress]] · [[cca_domain1_practice]]

Four exercises that build practical fluency in the Claude Agent SDK patterns most heavily tested in Domain 1 (27% of the exam). TypeScript version. The patterns the exam tests are language-agnostic — pick whichever language you ship production code in.

| Exercise | Task statements reinforced | Time |
|----------|---------------------------|------|
| 0 — Setup + Hello World | 1.1 | ~30 min |
| 1 — Multi-tool agent + hook | 1.1, 1.4, 1.5, 2.1, 2.2 | ~60 min |
| 2 — Multi-agent coordinator with parallel spawn | 1.2, 1.3, 5.3, 5.6 | ~90 min |
| 3 — Session forking | 1.7 | ~30 min |

## Prerequisites

- [ ] **Node.js 20+** installed — verify with `node --version`
- [ ] **Anthropic API key** in env: `export ANTHROPIC_API_KEY=sk-ant-...` — or active Claude Code subscription
- [ ] A working directory: `mkdir -p ~/cca-prep && cd ~/cca-prep`
- [ ] Basic TypeScript comfort (no advanced generics required; the SDK uses async iteration but the patterns are simple)

---

## Exercise 0 — Setup and Hello World

### Goal

Get the SDK installed in a TypeScript project and confirm an agent loop runs end-to-end.

### Steps

- [ ] Initialize the project:
  ```bash
  cd ~/cca-prep
  npm init -y
  npm pkg set type="module"
  ```

- [ ] Install the SDK and a TS runner. `tsx` lets you run `.ts` files directly without a compile step.
  ```bash
  npm install @anthropic-ai/claude-agent-sdk zod
  npm install -D tsx typescript @types/node
  ```

- [ ] Create a minimal `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "target": "ES2022",
      "module": "ESNext",
      "moduleResolution": "Bundler",
      "esModuleInterop": true,
      "skipLibCheck": true,
      "strict": true,
      "allowImportingTsExtensions": true,
      "noEmit": true
    }
  }
  ```

- [ ] Create `hello_agent.ts`:
  ```typescript
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "What's 2 + 2? Reply with just the number.",
    options: {
      systemPrompt: "You are a concise math assistant.",
    },
  })) {
    console.log(message);
  }
  ```

- [ ] Run it:
  ```bash
  npx tsx hello_agent.ts
  ```

- [ ] Verify you see message objects streaming back, ending with a result message that includes `stop_reason: "end_turn"`.

### What to notice

The `for await...of` loop above is the entire agent loop. Internally the SDK is checking `stop_reason` on each turn and continuing while it equals `"tool_use"`, terminating when it sees `"end_turn"`. **You don't have to write the loop yourself**, but the exam tests that you understand what's happening underneath.

### Exam mapping

- **Task 1.1** (agentic loops): you've just used a managed agentic loop. The SDK handles `stop_reason` for you; the exam will test that you understand this is driven by `stop_reason`, not by parsing text or iteration caps.

---

## Exercise 1 — Multi-tool Customer Support Agent with Policy Hook

### Goal

Build a customer support agent with four MCP tools and wire in a `PreToolUse` hook that blocks any `process_refund` above $500. Reinforces Tasks 1.4, 1.5, 2.1, and 2.2.

### Step 1 — Define the MCP tools

- [ ] Create `customer_support_agent.ts`:

```typescript
import {
  query,
  tool,
  createSdkMcpServer,
  type HookCallback,
} from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

// --- TOOLS ---------------------------------------------------------

// Each tool description clearly differentiates itself from the others.
// This is Task 2.1 in practice — descriptions are the primary signal
// the model uses to pick the right tool.

const getCustomer = tool(
  "get_customer",
  "Retrieve customer profile by customer_id. Use this to verify customer " +
    "identity BEFORE any account or financial operation. Returns name, " +
    "membership tier, and verification status.",
  { customer_id: z.string() },
  async (args) => {
    const customers: Record<string, unknown> = {
      "C-100": { name: "Alice Chen", tier: "Gold", verified: true },
      "C-200": { name: "Bob Singh", tier: "Standard", verified: true },
    };
    const cust = customers[args.customer_id];
    if (!cust) {
      // Structured error response (Task 2.2)
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              isError: true,
              errorCategory: "validation",
              isRetryable: false,
              description: "Customer not found",
            }),
          },
        ],
      };
    }
    return {
      content: [{ type: "text", text: JSON.stringify(cust) }],
    };
  }
);

const lookupOrder = tool(
  "lookup_order",
  "Retrieve order details by order_id. Use for order-status questions, " +
    "shipping inquiries, or to identify the order associated with a refund. " +
    "Returns order total, status, and customer reference.",
  { order_id: z.string() },
  async (args) => {
    const orders: Record<string, unknown> = {
      "O-5821": { total: 147.5, status: "delivered", customer: "C-100" },
      "O-5822": { total: 89.0, status: "shipped", customer: "C-200" },
    };
    const order = orders[args.order_id];
    if (!order) {
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({ isError: true, errorCategory: "validation" }),
          },
        ],
      };
    }
    return {
      content: [{ type: "text", text: JSON.stringify(order) }],
    };
  }
);

const processRefund = tool(
  "process_refund",
  "Issue a refund to the customer for a specific order. ONLY call this " +
    "after get_customer has verified identity. Returns success or " +
    "business-rule denial.",
  {
    customer_id: z.string(),
    order_id: z.string(),
    amount: z.number(),
  },
  async (args) => {
    return {
      content: [
        {
          type: "text",
          text: `Refund of $${args.amount.toFixed(2)} processed for order ${args.order_id}.`,
        },
      ],
    };
  }
);

const escalateToHuman = tool(
  "escalate_to_human",
  "Route this case to a human agent with a structured handoff summary. " +
    "Use when policy gaps, customer explicitly requests a human, or " +
    "operation exceeds policy limits.",
  {
    reason: z.string(),
    summary: z.string(),
  },
  async (args) => {
    return {
      content: [
        { type: "text", text: `Escalated to human queue. Reason: ${args.reason}` },
      ],
    };
  }
);

const supportServer = createSdkMcpServer({
  name: "customer-support",
  version: "1.0.0",
  tools: [getCustomer, lookupOrder, processRefund, escalateToHuman],
});
```

### Step 2 — Add the PreToolUse hook for the $500 cap

This is the **core Task 1.4/1.5 pattern**. The hook intercepts `process_refund` before it executes and denies if the amount exceeds policy.

- [ ] Add this to the same file, below the server definition:

```typescript
// --- HOOK: enforce $500 refund cap ---------------------------------

const refundCapHook: HookCallback = async (input, _toolUseId, _context) => {
  const toolName = input.tool_name as string;
  const toolInput = input.tool_input as Record<string, unknown>;

  // SDK MCP tools are namespaced as mcp__<server-name>__<tool-name>
  if (toolName === "mcp__customer-support__process_refund") {
    const amount = (toolInput.amount as number) ?? 0;
    if (amount > 500) {
      return {
        hookSpecificOutput: {
          hookEventName: "PreToolUse",
          permissionDecision: "deny",
          permissionDecisionReason:
            `Refund of $${amount.toFixed(2)} exceeds $500 policy limit. ` +
            "Use escalate_to_human instead.",
        },
      };
    }
  }
  return {};
};
```

> [!tip] Why the hook is essential
> A system prompt rule like "never refund over $500" works most of the time but isn't guaranteed. The hook **guarantees** the block — when deterministic compliance is required (financial policy, identity verification, regulatory), prompts have a non-zero failure rate. This is the exact distinction tested by Task 1.5 and Q3 in your practice bank.

### Step 3 — Run the agent

- [ ] Add the runner at the bottom of the file:

```typescript
// --- RUNNER --------------------------------------------------------

async function runScenario(label: string, prompt: string) {
  console.log(`\n=== ${label} ===\n`);

  for await (const message of query({
    prompt,
    options: {
      systemPrompt:
        "You are a customer support agent. ALWAYS call get_customer first " +
        "to verify identity before any refund or account change. Use the " +
        "structured handoff via escalate_to_human for cases that hit " +
        "policy gaps.",
      mcpServers: { "customer-support": supportServer },
      allowedTools: [
        "mcp__customer-support__get_customer",
        "mcp__customer-support__lookup_order",
        "mcp__customer-support__process_refund",
        "mcp__customer-support__escalate_to_human",
      ],
      hooks: {
        PreToolUse: [
          {
            matcher: "mcp__customer-support__process_refund",
            hooks: [refundCapHook],
          },
        ],
      },
    },
  })) {
    console.log(message);
  }
}

// TEST 1: Small refund — should succeed
await runScenario(
  "TEST 1: $50 refund (should succeed)",
  "Customer C-100 reports order O-5821 arrived damaged. " +
    "Process a $50 partial refund."
);

// TEST 2: Large refund — should be blocked by the hook
await runScenario(
  "TEST 2: $750 refund (should be blocked)",
  "Customer C-200 reports order O-5822 arrived damaged. " +
    "Process a $750 refund for the full amount."
);
```

### Step 4 — Run and verify

- [ ] Run: `npx tsx customer_support_agent.ts`
- [ ] **Test 1 verification**: You should see the agent call `get_customer`, then `process_refund` succeeds, and the agent reports the $50 refund processed.
- [ ] **Test 2 verification**: You should see the agent call `get_customer`, attempt `process_refund` with $750 — and the **hook denies the call**. The agent should then pivot and call `escalate_to_human`. If it just gives up, that's a sign the system prompt isn't strong enough about the fallback.

### Exam mapping

- **Task 1.1** — managed agentic loop with `stop_reason`-driven flow
- **Task 1.4** — programmatic prerequisite enforcement via hook
- **Task 1.5** — `PreToolUse` hook with `permissionDecision: "deny"` redirecting to alternative workflow
- **Task 2.1** — differentiated tool descriptions to prevent selection confusion
- **Task 2.2** — structured error responses with `errorCategory` and `isRetryable`

> [!warning] If the hook signature doesn't work
> The exact callback signature has evolved across SDK versions. If `HookCallback` types don't line up, check `docs.claude.com/en/docs/agent-sdk/hooks` for the current TypeScript signature. The **concept** is what's tested on the exam — block specific tool calls based on inputs, return `permissionDecision: "deny"` to enforce.

---

## Exercise 2 — Multi-Agent Coordinator with Parallel Subagent Spawning

### Goal

Build a research coordinator that spawns two specialized subagents in parallel. Measure the latency improvement vs serial. Reinforces Tasks 1.2, 1.3, 5.3, and 5.6 (provenance preservation).

### Step 1 — Define the subagents

- [ ] Create `research_pipeline.ts`:

```typescript
import {
  query,
  type ClaudeAgentOptions,
} from "@anthropic-ai/claude-agent-sdk";

// --- SUBAGENT DEFINITIONS ------------------------------------------

// Each agent definition has its own system prompt, allowed tools,
// and (optionally) a model override. Note the scoped tool access
// (Task 2.3) — search agent only gets search tools, etc.

const webSearchAgent = {
  description:
    "Searches the web for recent factual information. Returns findings " +
    "as structured records with claim, evidence_excerpt, source_url, " +
    "and publication_date for every claim.",
  prompt:
    "You are a web research agent. For every claim you produce, you " +
    'MUST emit a structured record: { "claim": "...", ' +
    '"evidence_excerpt": "...", "source_url": "...", ' +
    '"publication_date": "..." }. Never produce prose summaries without ' +
    "these fields.",
  tools: ["WebSearch", "WebFetch"],
  model: "sonnet" as const,
};

const docAnalyzerAgent = {
  description:
    "Analyzes provided documents for specific claims and quotes. Returns " +
    "structured records with claim, evidence_excerpt, document_name, " +
    "and page_or_section for every claim.",
  prompt:
    "You are a document analysis agent. For every claim, emit: " +
    '{ "claim": "...", "evidence_excerpt": "...", "document_name": "...", ' +
    '"page_or_section": "..." }. Surface conflicting values from ' +
    "different documents explicitly.",
  tools: ["Read", "Grep"],
  model: "sonnet" as const,
};
```

### Step 2 — The coordinator

- [ ] Continue in the same file:

```typescript
// --- COORDINATOR ---------------------------------------------------

async function runResearch(prompt: string, parallel: boolean): Promise<number> {
  const parallelInstruction = parallel
    ? "Spawn the web_search_agent and doc_analyzer_agent IN PARALLEL by " +
      "emitting BOTH Task tool calls in your very next response."
    : "Spawn the web_search_agent first, wait for its result, then " +
      "spawn the doc_analyzer_agent.";

  const coordinatorPrompt =
    "You are a research coordinator. Your job: decompose the user's " +
    "research question, delegate to specialist subagents, and synthesize " +
    "their findings.\n\n" +
    parallelInstruction +
    "\n\n" +
    "Each subagent will return structured records with attribution. " +
    "Your final synthesis MUST preserve source_url, document_name, and " +
    "publication_date on every claim. If subagents return conflicting " +
    "values, annotate the conflict with attribution rather than " +
    "silently reconciling.";

  const options: ClaudeAgentOptions = {
    systemPrompt: coordinatorPrompt,
    agents: {
      web_search_agent: webSearchAgent,
      doc_analyzer_agent: docAnalyzerAgent,
    },
    // IMPORTANT: Task tool MUST be in allowedTools so the coordinator
    // can spawn subagents. This is Task 1.3 in practice.
    allowedTools: ["Task"],
  };

  const start = Date.now();
  for await (const message of query({ prompt, options })) {
    console.log(message);
  }
  return (Date.now() - start) / 1000;
}

// --- MAIN ----------------------------------------------------------

const researchQuestion =
  "What are the latest documented trends in AI adoption in healthcare, " +
  "and how do industry-report claims compare to peer-reviewed findings?";

console.log("=== SERIAL RUN ===\n");
const serialTime = await runResearch(researchQuestion, false);

console.log("\n=== PARALLEL RUN ===\n");
const parallelTime = await runResearch(researchQuestion, true);

console.log(`\n\nSerial:   ${serialTime.toFixed(1)}s`);
console.log(`Parallel: ${parallelTime.toFixed(1)}s`);
console.log(`Speedup:  ${(serialTime / parallelTime).toFixed(1)}x`);
```

### Step 3 — Run and observe

- [ ] Run: `npx tsx research_pipeline.ts`
- [ ] **Watch for these in the output**:
  1. In the parallel run, **two `Task` tool calls in a single coordinator response** (both subagents spawned at once)
  2. In the serial run, they appear in separate turns (spawn → wait → result → next spawn)
  3. The parallel run should be roughly **30–50% faster** for a research task of this size

### Step 4 — Break the provenance and watch what happens (Task 5.6 reinforcement)

- [ ] Modify the coordinator's system prompt to weaken provenance: change the line about preserving `source_url`/`document_name` to just "cite sources where possible." Re-run.
- [ ] Observe: the final synthesis is now more likely to drop attribution on some claims. This is exactly the failure mode Q11 tests. **Restore the strict instruction** and re-run — attribution preservation comes back.

### Exam mapping

- **Task 1.2** — hub-and-spoke (coordinator → subagents → coordinator synthesis), coordinator-driven decomposition
- **Task 1.3** — `Task` in `allowedTools`, explicit context (we passed the research question in the user prompt, not assumed inheritance), **parallel spawning by emitting multiple Task calls in one response**
- **Task 2.3** — scoped tool access per subagent role
- **Task 5.6** — structured claim-source mappings preserved through synthesis (feel it work and feel it break)

> [!tip] Why doing this beats reading about it
> Until you've watched two `Task` tool calls fire in a single response and seen latency halve, the parallel-spawn pattern is abstract. After you've seen it once, Q10 and Q34 are obvious. Same for provenance — once you've watched a synthesis agent drop attribution because the coordinator's instructions were sloppy, Q11/Q32 land instantly.

---

## Exercise 3 — Session Forking for Divergent Exploration

### Goal

Establish a baseline session, fork it twice, explore two divergent paths, and confirm each fork has independent context. This is the core Task 1.7 pattern.

### Step 1 — Create the baseline and forks

- [ ] Create `session_fork_demo.ts`:

```typescript
import {
  ClaudeSDKClient,
  type ClaudeAgentOptions,
} from "@anthropic-ai/claude-agent-sdk";

const options: ClaudeAgentOptions = {
  systemPrompt:
    "You are a code architect helping evaluate refactoring strategies. " +
    "Be concise. Reference specific findings from earlier in the session.",
};

// --- BASELINE SESSION --------------------------------------------

const baseline = new ClaudeSDKClient({ options });

await baseline.query(
  "I have a 2000-line payment processing module with three " +
    "responsibilities mixed together: validation, billing API calls, " +
    "and audit logging. Summarize the structural problems in three " +
    "bullet points."
);
for await (const msg of baseline.receiveResponse()) {
  console.log("BASELINE:", msg);
}

// Capture the session ID so we can fork from it
const baselineSessionId = baseline.sessionId;
await baseline.close();

// --- FORK A: extract-classes strategy -----------------------

console.log("\n=== FORK A: extract-classes strategy ===\n");
const forkA = new ClaudeSDKClient({
  options,
  resume: baselineSessionId,
  forkSession: true, // Independent branch from the baseline
});
await forkA.query(
  "Evaluate the 'extract three classes' refactoring strategy " +
    "specifically. List risks and migration steps."
);
for await (const msg of forkA.receiveResponse()) {
  console.log("FORK A:", msg);
}
await forkA.close();

// --- FORK B: hexagonal-architecture strategy ----------------

console.log("\n=== FORK B: hexagonal architecture ===\n");
const forkB = new ClaudeSDKClient({
  options,
  resume: baselineSessionId,
  forkSession: true,
});
await forkB.query(
  "Evaluate moving to a hexagonal architecture with ports and " +
    "adapters. List risks and migration steps."
);
for await (const msg of forkB.receiveResponse()) {
  console.log("FORK B:", msg);
}
await forkB.close();
```

### Step 2 — Run and observe

- [ ] Run: `npx tsx session_fork_demo.ts`
- [ ] **What to confirm**:
  1. Both forks reference the **baseline analysis** (the three structural problems) — the fork inherits the baseline context
  2. Fork A's discussion of extract-classes risks does **not** appear in Fork B's reasoning — the forks are isolated from each other
  3. The baseline session itself is unchanged after the forks complete

### Step 3 — Compare with resume (Task 1.7 distinction)

- [ ] Modify the code: remove `forkSession: true` from Fork B, so it becomes a regular resume:
  ```typescript
  const forkB = new ClaudeSDKClient({
    options,
    resume: baselineSessionId,
    // forkSession omitted (defaults to false)
  });
  ```
- [ ] Run again. Now Fork B is a **continuation** of the baseline. If you'd run Fork A first as a fork it stays isolated, but if you do them serially without forking, the second one inherits whatever the first one did.

### Step 4 — The "fresh-start vs resume" decision (Q24 territory)

- [ ] Imagine you ran this analysis yesterday. Today you've **modified the payment module heavily** — 30% of the code has changed. Should you `resume` the baseline session?
- [ ] Answer per Task 1.7: **No** — start a fresh session with an injected summary of the current state. Resuming would carry stale tool results into context.
- [ ] Try it: create a new client with a system prompt that says "Here's the current state of the payment module after refactoring [summary]" rather than resuming. Note how much cleaner the reasoning is.

### Exam mapping

- **Task 1.7** — `resume` for continuation, `forkSession: true` for divergent branches from a shared baseline, and the meta-decision of when fresh-start-with-summary beats resume-with-stale-results

> [!warning] If forkSession syntax differs in your SDK version
> Some versions expose forking as a method (`client.fork()`) or via different parameter names. If `forkSession: true` doesn't exist in your SDK version, check `docs.claude.com/en/docs/agent-sdk/sessions` for the current TypeScript syntax. The **concept** the exam tests: forking creates independent branches from a shared baseline.

---

## Wrap-up — What You Should Have Built

After completing all three exercises, you should have:

- [ ] `hello_agent.ts` — proves SDK installation and managed agent loop
- [ ] `customer_support_agent.ts` — multi-tool agent with `PreToolUse` hook enforcing $500 cap
- [ ] `research_pipeline.ts` — coordinator spawning two subagents in parallel with provenance preservation
- [ ] `session_fork_demo.ts` — baseline session with two forks exploring divergent strategies

Plus the supporting files: `package.json`, `tsconfig.json`.

Total runtime to build all four: roughly 3 hours if you're not stuck. The code is intentionally minimal so the patterns are visible — production versions would have more error handling, logging, and observability.

You can add convenience scripts to `package.json` if you want shorter commands:

```json
{
  "scripts": {
    "hello": "tsx hello_agent.ts",
    "customer": "tsx customer_support_agent.ts",
    "research": "tsx research_pipeline.ts",
    "session": "tsx session_fork_demo.ts"
  }
}
```

Then run with `npm run customer`, etc.

## Common Pitfalls

1. **Forgetting `Task` in `allowedTools`** for the coordinator. Subagents won't spawn. (Task 1.3)
2. **Expecting subagents to inherit conversation history**. They don't. You must pass context in the prompt. (Task 1.3, Q34)
3. **Hooks not firing**. Check the `matcher` field — the tool name format is `mcp__<server-name>__<tool-name>` for SDK MCP tools. Built-in tools use their bare names (e.g., `"Bash"`, `"Read"`).
4. **System prompt overriding the hook intent**. If you tell the agent in the prompt "process all refunds," it may try to work around the hook. The hook still blocks, but the agent gets confused. Make the prompt and hook coherent.
5. **Provenance dropping in synthesis**. If your coordinator says "summarize the findings," attribution gets lost. Say "preserve `source_url` and `document_name` on every claim." Explicit beats implicit. (Task 5.6)
6. **`"type": "module"` missing from package.json**. If you see `Cannot use import statement outside a module`, add `"type": "module"` to your `package.json`.

## TypeScript-Specific Notes

- **Zod** is the recommended way to define tool input schemas. It gives you type inference for free — `args` inside the tool handler is fully typed based on the schema.
- The SDK uses **camelCase** option keys (`systemPrompt`, `allowedTools`, `mcpServers`, `forkSession`). The Python SDK uses snake_case. The wire protocol underneath uses snake_case (`tool_name`, `permission_decision`, `stop_reason`).
- `for await...of` is the only way to consume the streaming `query()` response. There's no callback-style or promise-style alternative.
- `tsx` is the fastest dev loop — it compiles TS on the fly with no separate build step. For production you'd typically compile with `tsc` and run the resulting JS with `node`.

## Next Steps

- Update [[cca_progress]] — mark the Domain 1 Exercises 1 and 4 checkboxes complete in the Hands-On Exercises section.
- Re-take [[cca_domain1_practice]] questions on Tasks 1.4 (Q13–Q17) and 1.7 (Q24–Q25). They should feel different now that you've built the patterns.
- Note any sticking points in [[cca_progress#Session notes]] for follow-up.

---

*Maps to Priority 1 of [[cca_docs_roadmap]]. Domain 1 weighting: 27% of the exam — the largest single domain.*
