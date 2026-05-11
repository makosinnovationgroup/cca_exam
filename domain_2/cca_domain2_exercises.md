---
title: Claude Certified Architect — Foundations · Domain 2 Hands-On Exercises (TypeScript)
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - hands-on
  - domain-2
  - typescript
status: in-progress
estimated-hours: 3
language: typescript
---

# CCA-F Domain 2 — Hands-On Exercises (TypeScript)

Practical fluency in tool design, MCP integration, structured error handling, and built-in tool selection — patterns most heavily tested in Domain 2 (18% of the exam). The patterns are language-agnostic; TypeScript is used here.

| Item | Task statements reinforced | Time |
|------|---------------------------|------|
| Exercise 1 — Custom tool with structured errors | 2.1, 2.2 | ~45 min |
| Exercise 2 — Tool routing with description quality + `tool_choice` | 2.1, 2.3 | ~45 min |
| Quick drills (×5) | 2.3, 2.4, 2.5 | ~60 min |

## Prerequisites

- [ ] **Node.js 20+** installed — verify with `node --version`
- [ ] **Anthropic API key** in env: `export ANTHROPIC_API_KEY=sk-ant-...`
- [ ] A working directory: `mkdir -p ~/cca-prep && cd ~/cca-prep`
- [ ] If you completed Domain 1 Exercise 0, your `~/cca-prep` is already set up

> [!note] On the SDK choice
> Domain 2 exercises mostly use `@anthropic-ai/sdk` (the Messages API SDK) rather than `@anthropic-ai/claude-agent-sdk` (the Agent SDK). The Agent SDK includes everything in the Messages SDK plus the agentic harness, but for testing tool routing and structured error returns, the Messages SDK is leaner. Quick drills use both.

---

## Exercise 1 — Custom tool with structured errors

### Goal

Define a custom tool that returns **structured error responses** (`is_error: true` + `errorCategory` + `isRetryable` + `description`) and observe how Claude responds differently to each error category.

### Why this matters

Task 2.2 tests that you reach for structured error metadata rather than silently suppressing errors or raising exceptions that crash the loop. The four error categories — `transient`, `validation`, `permission`, `business` — drive different agent behavior, and the exam tests the conceptual distinction.

### Steps

- [ ] Install the Messages SDK if not already:
  ```bash
  cd ~/cca-prep
  npm install @anthropic-ai/sdk
  ```

- [ ] Create `tool_errors.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  // Simulated payments API that fails in different ways
  function processRefund(
    orderId: string,
    amount: number,
  ): {
    is_error: boolean;
    content: string;
    errorCategory?: "transient" | "validation" | "permission" | "business";
    isRetryable?: boolean;
  } {
    // Simulate the four error categories
    if (orderId === "TIMEOUT") {
      return {
        is_error: true,
        content: JSON.stringify({
          errorCategory: "transient",
          isRetryable: true,
          description:
            "Payment gateway timeout. Network or upstream service unavailable.",
        }),
        errorCategory: "transient",
        isRetryable: true,
      };
    }
    if (amount <= 0) {
      return {
        is_error: true,
        content: JSON.stringify({
          errorCategory: "validation",
          isRetryable: false,
          description:
            "Refund amount must be positive. Received: " + amount,
        }),
        errorCategory: "validation",
        isRetryable: false,
      };
    }
    if (orderId === "FORBIDDEN") {
      return {
        is_error: true,
        content: JSON.stringify({
          errorCategory: "permission",
          isRetryable: false,
          description:
            "Refund authorization denied. Order belongs to a different workspace.",
        }),
        errorCategory: "permission",
        isRetryable: false,
      };
    }
    if (amount > 500) {
      return {
        is_error: true,
        content: JSON.stringify({
          errorCategory: "business",
          isRetryable: false,
          description:
            "Refund amount $" +
            amount +
            " exceeds the $500 policy limit. Escalate to a human agent.",
        }),
        errorCategory: "business",
        isRetryable: false,
      };
    }
    // Success path
    return {
      is_error: false,
      content: JSON.stringify({
        refundId: "ref_" + Math.random().toString(36).slice(2, 10),
        orderId,
        amount,
        status: "processed",
      }),
    };
  }

  // Test each error category by sending Claude a different scenario
  async function testScenario(
    label: string,
    userMessage: string,
    orderId: string,
    amount: number,
  ) {
    console.log("\n=== " + label + " ===");
    console.log("User: " + userMessage);

    const tools: Anthropic.Tool[] = [
      {
        name: "process_refund",
        description:
          "Process a refund for an order. Returns refund ID on success, structured error on failure.",
        input_schema: {
          type: "object",
          properties: {
            order_id: { type: "string" },
            amount: { type: "number" },
          },
          required: ["order_id", "amount"],
        },
      },
    ];

    // Round 1: Claude calls the tool
    const first = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 1024,
      tools,
      messages: [{ role: "user", content: userMessage }],
    });

    const toolUse = first.content.find((b) => b.type === "tool_use");
    if (!toolUse || toolUse.type !== "tool_use") {
      console.log("Claude did not call the tool.");
      return;
    }

    // Execute the tool with our scenario inputs
    const result = processRefund(orderId, amount);

    // Round 2: send the structured error back
    const second = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 1024,
      tools,
      messages: [
        { role: "user", content: userMessage },
        { role: "assistant", content: first.content },
        {
          role: "user",
          content: [
            {
              type: "tool_result",
              tool_use_id: toolUse.id,
              content: result.content,
              is_error: result.is_error,
            },
          ],
        },
      ],
    });

    const text = second.content
      .filter((b) => b.type === "text")
      .map((b) => (b as { text: string }).text)
      .join("\n");
    console.log("Claude: " + text);
  }

  // Run all four scenarios
  await testScenario(
    "TRANSIENT (gateway timeout)",
    "Please refund order TIMEOUT for $50.",
    "TIMEOUT",
    50,
  );
  await testScenario(
    "VALIDATION (negative amount)",
    "Please refund order ORD-123 for $-10.",
    "ORD-123",
    -10,
  );
  await testScenario(
    "PERMISSION (forbidden)",
    "Please refund order FORBIDDEN for $50.",
    "FORBIDDEN",
    50,
  );
  await testScenario(
    "BUSINESS (over $500 cap)",
    "Please refund order ORD-456 for $750.",
    "ORD-456",
    750,
  );
  await testScenario(
    "SUCCESS",
    "Please refund order ORD-789 for $100.",
    "ORD-789",
    100,
  );
  ```

- [ ] Run it:
  ```bash
  npx tsx tool_errors.ts
  ```

- [ ] Observe Claude's behavior for each scenario:
  - **TRANSIENT** → Claude often offers to retry (since `isRetryable: true`)
  - **VALIDATION** → Claude asks the user to clarify or fix the input
  - **PERMISSION** → Claude explains it can't perform the action and escalates
  - **BUSINESS** → Claude explains the policy and suggests escalation (the agent doesn't retry; the rule is non-negotiable)
  - **SUCCESS** → Claude confirms the refund with the refund ID

### Verification

You should see **distinctly different responses** for each error category. If all four look the same, the metadata isn't being conveyed effectively — check that `is_error: true` is set on the `tool_result` block and that the JSON content includes the `errorCategory` and `description`.

### Patterns reinforced

- **Task 2.2 — Structured errors:** the four categories (`transient`, `validation`, `permission`, `business`) drive different agent behavior. Each carries `isRetryable` and `description` so the model can reason about next steps.
- **`is_error: true` flag** at the protocol level signals failure to Claude. Raising an exception in your tool implementation instead would crash the loop.
- **Don't suppress errors silently** — the agent needs structured context to decide whether to retry, escalate, ask for clarification, or accept the failure.

### Failure-mode exploration

After the happy path works, deliberately break it:

- [ ] Set `is_error: false` on the tool result even when the structured error is returned → Claude treats the failure as success and continues incorrectly
- [ ] Drop the `errorCategory` field from the JSON content → Claude reasons less specifically about the failure; behavior becomes more generic
- [ ] Change `isRetryable: true` to `false` on the TRANSIENT case → Claude stops offering retry and treats the timeout as permanent

---

## Exercise 2 — Tool routing with description quality and `tool_choice`

### Goal

Build two MCP-style tools with deliberately similar descriptions, watch Claude misroute, then fix the descriptions with explicit preconditions, effects, and discriminating criteria. Then explore `tool_choice` to force specific routing.

### Why this matters

Task 2.1 tests that you reach for **description quality** (preconditions, effects, choice criteria) when tools are confused — not for routing classifiers, longer descriptions in general, or combining tools. Task 2.3 tests `tool_choice` for forcing specific tools or restricting to one tool per turn.

### Steps

- [ ] Create `tool_routing_bad.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  // Two tools with deliberately similar, vague descriptions
  const tools: Anthropic.Tool[] = [
    {
      name: "cancel_order",
      description: "Handle a customer order.",
      input_schema: {
        type: "object",
        properties: {
          order_id: { type: "string" },
        },
        required: ["order_id"],
      },
    },
    {
      name: "refund_order",
      description: "Handle a customer order.",
      input_schema: {
        type: "object",
        properties: {
          order_id: { type: "string" },
          amount: { type: "number" },
        },
        required: ["order_id", "amount"],
      },
    },
  ];

  const testPrompts = [
    "Cancel order ORD-100 — the customer changed their mind.",
    "The customer received the product but it was damaged. Process a $45 refund for order ORD-101.",
    "Please send a refund for order ORD-102, amount $30.",
    "Customer wants to cancel their pending order ORD-103.",
  ];

  for (const prompt of testPrompts) {
    console.log("\nPrompt: " + prompt);
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 512,
      tools,
      messages: [{ role: "user", content: prompt }],
    });
    const toolUse = response.content.find((b) => b.type === "tool_use");
    if (toolUse && toolUse.type === "tool_use") {
      console.log("  → Claude chose: " + toolUse.name);
      console.log("  → Input: " + JSON.stringify(toolUse.input));
    } else {
      console.log("  → Claude responded with text (no tool call)");
    }
  }
  ```

- [ ] Run it:
  ```bash
  npx tsx tool_routing_bad.ts
  ```

  Expected: with such generic descriptions, Claude may pick reasonably for refund-with-amount cases but struggle on edge cases. Note how often it picks the wrong tool.

- [ ] Create `tool_routing_good.ts` — same code, but with rewritten descriptions:
  ```typescript
  const tools: Anthropic.Tool[] = [
    {
      name: "cancel_order",
      description:
        "Cancel an order that has not yet shipped. Use this when the customer wants to stop the order from going out — for example, they changed their mind, found a better price, or the order is no longer needed. The order must be in 'pending' or 'processing' status. Do not use for already-shipped orders (use refund_order with the full amount instead). Effect: marks the order as cancelled and reverses any charge.",
      input_schema: {
        type: "object",
        properties: {
          order_id: { type: "string" },
        },
        required: ["order_id"],
      },
    },
    {
      name: "refund_order",
      description:
        "Issue a refund for an order that has already shipped or been delivered. Use this when the customer received the product but wants money back — for example, the product was damaged, incorrect, defective, or the customer is dissatisfied. Requires an explicit refund amount in USD. Do not use for orders that haven't shipped yet (use cancel_order instead). Effect: credits the customer's original payment method.",
      input_schema: {
        type: "object",
        properties: {
          order_id: { type: "string" },
          amount: { type: "number" },
        },
        required: ["order_id", "amount"],
      },
    },
  ];
  ```

- [ ] Run the improved version:
  ```bash
  npx tsx tool_routing_good.ts
  ```

  Expected: every prompt routes correctly. The descriptions now encode preconditions (`already shipped` vs `not yet shipped`), effects (`marks as cancelled` vs `credits payment method`), and discriminating criteria ("do not use for ... instead").

- [ ] Now create `tool_choice_forced.ts` to test `tool_choice` options:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  // Same tool definitions as tool_routing_good.ts
  const tools: Anthropic.Tool[] = [
    /* ... copy from above ... */
  ];

  const prompt = "Customer ordered item ORD-200 yesterday and wants to undo it.";

  // Auto (default)
  console.log("=== tool_choice: auto ===");
  const auto = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 512,
    tools,
    tool_choice: { type: "auto" },
    messages: [{ role: "user", content: prompt }],
  });
  console.log(
    "  Used:",
    auto.content.find((b) => b.type === "tool_use")?.["name"] ?? "none",
  );

  // Any — must call some tool
  console.log("=== tool_choice: any ===");
  const any = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 512,
    tools,
    tool_choice: { type: "any" },
    messages: [{ role: "user", content: prompt }],
  });
  console.log(
    "  Used:",
    any.content.find((b) => b.type === "tool_use")?.["name"] ?? "none",
  );

  // Forced — must call refund_order
  console.log("=== tool_choice: forced refund_order ===");
  const forced = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 512,
    tools,
    tool_choice: { type: "tool", name: "refund_order" },
    messages: [{ role: "user", content: prompt }],
  });
  console.log(
    "  Used:",
    forced.content.find((b) => b.type === "tool_use")?.["name"] ?? "none",
  );

  // None — disabled
  console.log("=== tool_choice: none ===");
  const none = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 512,
    tools,
    tool_choice: { type: "none" },
    messages: [{ role: "user", content: prompt }],
  });
  console.log(
    "  Used:",
    none.content.find((b) => b.type === "tool_use")?.["name"] ?? "none",
  );
  ```

- [ ] Run it:
  ```bash
  npx tsx tool_choice_forced.ts
  ```

  Expected:
  - `auto` → Claude judges; should pick `cancel_order` for "undo it"
  - `any` → forces some tool; usually `cancel_order` for this prompt
  - `tool: "refund_order"` → forced to refund_order even though cancel was more appropriate
  - `none` → no tool call; Claude responds in text only

### Verification

The "bad" routing version misroutes on at least one prompt. The "good" version routes correctly on all. The `tool_choice` version shows exact behavior for each setting.

### Patterns reinforced

- **Task 2.1 — Tool descriptions** are the primary mechanism Claude uses to select between similar tools. Differentiate with:
  - **Preconditions** (when is this tool valid?)
  - **Effects** (what does it do?)
  - **Discriminating criteria** ("use this when... rather than...")
- **Task 2.3 — `tool_choice`:**
  - `auto` (default) — Claude judges
  - `any` — forces some tool; Claude picks which
  - `{type: "tool", name: "X"}` — forces the specific named tool
  - `none` — disables tool calling for this request
- **Anti-patterns the exam expects you to reject:** longer descriptions in general (specific quality matters, not length), adding a routing classifier as a middle layer, combining the two tools into one with an `action` parameter.

### Failure-mode exploration

- [ ] Try setting `tool_choice: {type: "any"}` with `disable_parallel_tool_use: true` and observe behavior — exactly one tool per turn
- [ ] Add a third tool with another vague description and watch routing degrade — the issue isn't number of tools, it's description quality

---

## Quick drills

Five short drills covering the remaining Domain 2 patterns. Each is a 5–15 minute experiment. Do them in any order.

### Quick drill A — MCP project scoping

**Goal:** Set up `.mcp.json` at a project root with environment variable expansion, confirm it loads without committing the credential.

- [ ] In a project directory, create `.mcp.json`:
  ```json
  {
    "mcpServers": {
      "example": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-everything"],
        "env": {
          "EXAMPLE_TOKEN": "${EXAMPLE_TOKEN}"
        }
      }
    }
  }
  ```

- [ ] Set the env var in your shell:
  ```bash
  export EXAMPLE_TOKEN=test_token_value
  ```

- [ ] Start Claude Code from this directory, then run `/mcp` to see the server listed.

- [ ] **Verify:** the `.mcp.json` is safe to commit (no real token); each developer sets `EXAMPLE_TOKEN` locally.

**Pattern reinforced:** Task 2.4 — `.mcp.json` is project-scoped (Git-tracked); credentials use `${VAR}` env-var expansion at load time.

### Quick drill B — Personal MCP server (`~/.claude.json`)

**Goal:** Confirm both project-level and user-level MCP servers load simultaneously.

- [ ] Add an experimental MCP server to your personal `~/.claude.json`:
  ```json
  {
    "mcpServers": {
      "personal-test": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-time"]
      }
    }
  }
  ```

- [ ] Start Claude Code from the same project as Drill A and run `/mcp`. Both servers should be listed.

- [ ] **Verify:** the personal server is available to you but not committed to the project.

**Pattern reinforced:** Task 2.4 — `~/.claude.json` is user-scoped (personal). Both scopes can be active simultaneously.

### Quick drill C — Built-in tool selection

**Goal:** Practice picking the right built-in tool for each operation type.

For each scenario, identify the tool, then run it in Claude Code to confirm:

- [ ] "Find every `.test.tsx` file in this project" → **Glob** with pattern `**/*.test.tsx`
- [ ] "Find every file containing the string `'TODO: refactor'`" → **Grep** with that pattern
- [ ] "Read the contents of `package.json`" → **Read** with the path
- [ ] "Replace `const x = 1` with `const x = 2` in `src/main.ts` where the anchor is unique" → **Edit** with the unique anchor
- [ ] "Modify a line in a 5000-line file where the anchor isn't unique" → **Read** the full file, modify the contents in memory, **Write** the new contents (Read + Write fallback)
- [ ] "Run `npm run lint` and capture stderr" → **Bash** (last resort when other tools don't fit)

**Pattern reinforced:** Task 2.5 — Grep (content), Glob (paths), Read (load), Write (overwrite), Edit (unique-anchor modify with Read+Write fallback), Bash (last resort).

### Quick drill D — MCP resource pattern

**Goal:** Instead of exposing a tool for every document in a catalog, expose the catalog as an MCP resource.

This is a conceptual drill — you don't need to fully implement an MCP server. Just sketch the design:

- [ ] Imagine a documentation site with 5,000 pages. Sketch two designs:
  - **Bad design:** 5,000 separate tools, one per document (`search_authentication_docs`, `search_database_docs`, ...)
  - **Good design:** one `search_docs` tool that takes a query, plus a `documents` **resource** that lists all available pages with titles and URIs so the agent can pre-filter or directly reference
- [ ] Read the official MCP spec on Resources: `modelcontextprotocol.io/docs/concepts/resources`
- [ ] **Verify understanding:** an MCP resource is *read* by the agent (exposes content, has `uri` and `mimeType`); an MCP tool is *called* by the agent (performs action, has `inputSchema`).

**Pattern reinforced:** MCP type distinctions (Tool vs Resource vs Prompt). Catalogs and indexes belong as resources, not as a flood of search tools.

### Quick drill E — Parallel tool result formatting

**Goal:** Observe the formatting rule that keeps parallel tool use working.

- [ ] Create `parallel_results.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const tools: Anthropic.Tool[] = [
    {
      name: "get_weather",
      description: "Get current weather for a city.",
      input_schema: {
        type: "object",
        properties: { city: { type: "string" } },
        required: ["city"],
      },
    },
    {
      name: "get_time",
      description: "Get the current local time in a timezone.",
      input_schema: {
        type: "object",
        properties: { timezone: { type: "string" } },
        required: ["timezone"],
      },
    },
  ];

  const first = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    tools,
    messages: [
      {
        role: "user",
        content:
          "What's the weather in Paris and what time is it there right now? Use both tools in parallel.",
      },
    ],
  });

  console.log(
    "Number of tool calls:",
    first.content.filter((b) => b.type === "tool_use").length,
  );
  ```

- [ ] Run it:
  ```bash
  npx tsx parallel_results.ts
  ```
  Expected: Claude emits **two** tool_use blocks in one response.

- [ ] When sending the results back, format them in a **single** user message with both `tool_result` blocks. Test the bad pattern too: split the results into two separate user messages and watch parallel tool use degrade across subsequent turns.

**Pattern reinforced:** Tool results for parallel calls go in **one** user message, not two. Splitting them teaches Claude to stop emitting parallel calls.

---

## Quick-reference cheatsheet for the patterns drilled

- **Tool descriptions** are the primary selection signal. Differentiate with preconditions, effects, choice criteria.
- **Structured errors:** `is_error: true` + `errorCategory` (transient/validation/permission/business) + `isRetryable` + `description`.
- **`is_error: true` in tool result, never raise exceptions** (which crash the loop).
- **`tool_choice`:** `auto` (default), `any` (force some tool), `{type: "tool", name: "X"}` (force specific), `none` (disable).
- **`disable_parallel_tool_use: true`** with `auto` / `any` / forced → exactly one tool per turn.
- **MCP scoping:** `.mcp.json` (project, Git-tracked) vs `~/.claude.json` (user, personal).
- **MCP credentials:** env var expansion `${TOKEN}` in `.mcp.json` (never commit tokens).
- **Built-in tools:** Grep (content), Glob (paths), Read (load), Write (overwrite), Edit (unique-anchor modify), Bash (last resort).
- **MCP type distinctions:** Tool (action, called), Resource (content, read), Prompt (template).
- **Parallel tool results** in ONE user message, not separate ones.

---

## Progress tracker

- [ ] Exercise 1 complete (structured errors)
- [ ] Exercise 1 failure-mode exploration complete
- [ ] Exercise 2 complete (tool routing + `tool_choice`)
- [ ] Exercise 2 failure-mode exploration complete
- [ ] Quick drill A — MCP project scoping
- [ ] Quick drill B — Personal MCP server
- [ ] Quick drill C — Built-in tool selection
- [ ] Quick drill D — MCP resource pattern
- [ ] Quick drill E — Parallel tool result formatting

---

*Domain 2 weight: **18%** of the exam.*
