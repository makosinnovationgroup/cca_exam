---
title: Claude Certified Architect — Foundations · Domain 4 Hands-On Exercises (TypeScript)
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - hands-on
  - domain-4
  - typescript
status: in-progress
estimated-hours: 4-5
language: typescript
---

# CCA-F Domain 4 — Hands-On Exercises (TypeScript)

Practical fluency in prompt engineering, structured output via `tool_use` and native structured outputs, validation-retry loops, the Message Batches API, and multi-instance independent review — patterns most heavily tested in Domain 4 (20% of the exam).

| Item | Task statements reinforced | Time |
|------|---------------------------|------|
| Exercise 1 — Structured extraction with retry loop | 4.1, 4.2, 4.3, 4.4 | ~75 min |
| Exercise 2 — Message Batches API with chunking | 4.5 | ~60 min |
| Exercise 3 — Multi-instance independent review | 4.6 | ~45 min |
| Quick drills (×5) | 4.3, 4.4 | ~60 min |

## Prerequisites

- [ ] **Node.js 20+** installed — verify with `node --version`
- [ ] **Anthropic API key** in env: `export ANTHROPIC_API_KEY=sk-ant-...`
- [ ] A working directory: `mkdir -p ~/cca-prep && cd ~/cca-prep`
- [ ] Messages SDK installed: `npm install @anthropic-ai/sdk` (if not from Domain 2)

> [!note] On the SDK choice
> Domain 4 uses `@anthropic-ai/sdk` (the Messages API SDK), not `@anthropic-ai/claude-agent-sdk` (the Agent SDK). Structured output, Batches API, and multi-instance review are all Messages API features — you don't need the agent harness for these.

---

## Exercise 1 — Structured extraction with retry loop

### Goal

Build an invoice extraction pipeline that:

1. Uses **forced `tool_use`** with `strict: true` for protocol-level schema validation
2. Uses **nullable fields** to prevent fabrication of missing data
3. Uses **enum with `"other"` + detail string** for graceful unknown handling
4. Runs a **bounded retry loop** with feedback on validation failures
5. Falls back gracefully when retries are exhausted

### Why this matters

Tasks 4.1, 4.3, and 4.4 are densely tested. The exam asks:
- How do you force a specific tool? (`tool_choice: {type: "tool", name: "X"}`)
- How do you prevent fabrication of optional fields? (nullable)
- How do you handle edge-case categories? (enum + "other" + detail)
- When does retry help, and when doesn't it? (helps when error gives correction info; doesn't help when source lacks data)
- What components make a robust validation-retry loop? (limit + fallback)

### Steps

- [ ] Create `extract_invoice.ts`. This implements the full pipeline.

  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  // The schema for our extracted invoice.
  // Note the patterns:
  //   - nullable fields ("type": ["string", "null"]) for fields that may be absent
  //   - enum with "other" + detail string for graceful unknown handling
  //   - required: true on every field (so the model can't silently omit)
  //   - additionalProperties: false (strict)
  const extractInvoiceTool: Anthropic.Tool = {
    name: "extract_invoice",
    description:
      "Extract structured invoice data from the provided text. Return null for any field that genuinely does not appear in the source — do not fabricate.",
    input_schema: {
      type: "object",
      properties: {
        vendor_name: { type: "string" },
        invoice_number: { type: "string" },
        invoice_date: {
          type: ["string", "null"],
          description: "YYYY-MM-DD format, or null if not present",
        },
        due_date: {
          type: ["string", "null"],
          description: "YYYY-MM-DD format, or null if not present",
        },
        line_items: {
          type: "array",
          items: {
            type: "object",
            properties: {
              description: { type: "string" },
              quantity: { type: "number" },
              unit_price: { type: "number" },
              line_total: { type: "number" },
            },
            required: ["description", "quantity", "unit_price", "line_total"],
            additionalProperties: false,
          },
        },
        subtotal: { type: "number" },
        tax: { type: ["number", "null"] },
        total: { type: "number" },
        expense_category: {
          type: "string",
          enum: [
            "software",
            "hardware",
            "services",
            "office_supplies",
            "travel",
            "other",
          ],
        },
        expense_category_detail: {
          type: ["string", "null"],
          description:
            "If expense_category is 'other', describe what category. Otherwise null.",
        },
        shipping_address: {
          type: ["string", "null"],
          description: "Full shipping address, or null if absent",
        },
      },
      required: [
        "vendor_name",
        "invoice_number",
        "invoice_date",
        "due_date",
        "line_items",
        "subtotal",
        "tax",
        "total",
        "expense_category",
        "expense_category_detail",
        "shipping_address",
      ],
      additionalProperties: false,
    },
  };

  type Invoice = {
    vendor_name: string;
    invoice_number: string;
    invoice_date: string | null;
    due_date: string | null;
    line_items: Array<{
      description: string;
      quantity: number;
      unit_price: number;
      line_total: number;
    }>;
    subtotal: number;
    tax: number | null;
    total: number;
    expense_category:
      | "software"
      | "hardware"
      | "services"
      | "office_supplies"
      | "travel"
      | "other";
    expense_category_detail: string | null;
    shipping_address: string | null;
  };

  type ValidationError = {
    field: string;
    message: string;
  };

  // Semantic validation (strict mode handles syntax; this handles meaning)
  function validateInvoice(inv: Invoice): ValidationError[] {
    const errors: ValidationError[] = [];

    // Line items must sum to subtotal
    const lineSum = inv.line_items.reduce((s, li) => s + li.line_total, 0);
    if (Math.abs(lineSum - inv.subtotal) > 0.01) {
      errors.push({
        field: "subtotal",
        message: `Line items sum to ${lineSum.toFixed(2)} but subtotal is ${inv.subtotal.toFixed(2)}.`,
      });
    }

    // Subtotal + tax must equal total
    const computedTotal = inv.subtotal + (inv.tax ?? 0);
    if (Math.abs(computedTotal - inv.total) > 0.01) {
      errors.push({
        field: "total",
        message: `subtotal (${inv.subtotal.toFixed(2)}) + tax (${(inv.tax ?? 0).toFixed(2)}) = ${computedTotal.toFixed(2)}, but total is ${inv.total.toFixed(2)}.`,
      });
    }

    // Each line item: quantity * unit_price ~= line_total
    inv.line_items.forEach((li, i) => {
      const expected = li.quantity * li.unit_price;
      if (Math.abs(expected - li.line_total) > 0.01) {
        errors.push({
          field: `line_items[${i}].line_total`,
          message: `${li.quantity} × ${li.unit_price} = ${expected.toFixed(2)}, but line_total is ${li.line_total.toFixed(2)}.`,
        });
      }
    });

    // expense_category == "other" must have a detail string
    if (
      inv.expense_category === "other" &&
      (!inv.expense_category_detail ||
        inv.expense_category_detail.trim() === "")
    ) {
      errors.push({
        field: "expense_category_detail",
        message:
          'When expense_category is "other", expense_category_detail must be a non-empty string.',
      });
    }

    return errors;
  }

  async function extractWithRetry(
    invoiceText: string,
    maxRetries: number = 3,
  ): Promise<
    | { status: "success"; invoice: Invoice; attempts: number }
    | { status: "failure"; lastErrors: ValidationError[]; attempts: number }
  > {
    const messages: Anthropic.MessageParam[] = [
      {
        role: "user",
        content: `Extract structured data from this invoice. Return null for any field that genuinely is not in the source. Do not fabricate values.\n\n<invoice>\n${invoiceText}\n</invoice>`,
      },
    ];

    let attempt = 0;
    let lastErrors: ValidationError[] = [];

    while (attempt <= maxRetries) {
      attempt++;
      const response = await client.messages.create({
        model: "claude-sonnet-4-6",
        max_tokens: 4096,
        tools: [extractInvoiceTool],
        tool_choice: { type: "tool", name: "extract_invoice" }, // FORCE the call
        messages,
      });

      const toolUse = response.content.find((b) => b.type === "tool_use");
      if (!toolUse || toolUse.type !== "tool_use") {
        // Shouldn't happen with forced tool_choice, but defend anyway
        return { status: "failure", lastErrors: [{ field: "_", message: "Model did not call the tool." }], attempts: attempt };
      }

      const invoice = toolUse.input as Invoice;
      const errors = validateInvoice(invoice);

      if (errors.length === 0) {
        return { status: "success", invoice, attempts: attempt };
      }

      lastErrors = errors;
      console.log(`Attempt ${attempt}: ${errors.length} validation error(s):`);
      for (const e of errors) {
        console.log(`  - ${e.field}: ${e.message}`);
      }

      // Decide: is this error retryable? (Heuristic: math errors are correctable; missing data isn't.)
      // For this exercise, we retry on math errors, give up on others.
      const allMathErrors = errors.every((e) =>
        e.message.includes("but") ||
        e.message.includes("="),
      );

      if (!allMathErrors) {
        // Non-math errors (e.g., missing required detail for "other" category)
        // are unlikely to resolve with retry — bail.
        console.log("  (errors not correctable by retry; aborting)");
        return { status: "failure", lastErrors: errors, attempts: attempt };
      }

      // Append the model's response and an error message for the next iteration
      messages.push({ role: "assistant", content: response.content });
      messages.push({
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: toolUse.id,
            content: `Validation failed:\n${errors.map((e) => `- ${e.field}: ${e.message}`).join("\n")}\n\nPlease correct these and re-extract.`,
            is_error: true,
          },
        ],
      });
    }

    return { status: "failure", lastErrors, attempts: attempt };
  }

  // Test cases
  const goodInvoice = `
  ACME WIDGETS LLC
  Invoice #: INV-2026-0042
  Date: 2026-03-15
  Due: 2026-04-14
  
  Line items:
  - Widget Pro (5 × $40.00) ............ $200.00
  - Bracket Mount (2 × $25.00) ......... $50.00
  
  Subtotal: $250.00
  Tax (8%):  $20.00
  Total:    $270.00
  
  Category: Hardware
  `;

  const badMathInvoice = `
  ACME WIDGETS LLC
  Invoice #: INV-2026-0043
  Date: 2026-03-16
  
  Line items:
  - Widget Pro (5 × $40.00) ............ $200.00
  - Bracket Mount (2 × $25.00) ......... $50.00
  
  Subtotal: $300.00  [<-- WRONG: should be $250]
  Tax: $20.00
  Total: $320.00
  
  Category: Hardware
  `;

  const missingDataInvoice = `
  ACME WIDGETS LLC
  Invoice #: INV-2026-0044
  [no date visible]
  
  Items:
  - Consulting Hours (10 × $150.00) .... $1,500.00
  
  Subtotal: $1,500.00
  Total: $1,500.00
  
  [No shipping address — this is a service invoice]
  [Category: not specified — could be 'services' or 'consulting']
  `;

  console.log("\n=== GOOD INVOICE ===");
  console.log(JSON.stringify(await extractWithRetry(goodInvoice), null, 2));

  console.log("\n=== BAD MATH INVOICE (retry should help) ===");
  console.log(JSON.stringify(await extractWithRetry(badMathInvoice), null, 2));

  console.log(
    "\n=== MISSING DATA INVOICE (retry shouldn't help; fallback engages) ===",
  );
  console.log(
    JSON.stringify(await extractWithRetry(missingDataInvoice), null, 2),
  );
  ```

- [ ] Run it:
  ```bash
  npx tsx extract_invoice.ts
  ```

### Verification

For each test case, observe:

- **Good invoice** → success on attempt 1. `shipping_address: null` (correctly, since it's not in the source). `expense_category: "hardware"`.
- **Bad math invoice** → first attempt produces validation errors (the model trusts the source's stated subtotal `$300`, which doesn't match the line-item sum `$250`). Retry attempts give Claude the discrepancy info, and it should correct on attempt 2 or 3.
- **Missing data invoice** → date and shipping address come back as `null` (good — no fabrication). Category may resolve to `"services"` with detail or `"other"` with detail.

### Patterns reinforced

- **Task 4.3 — Structured output:**
  - Force the tool call with `tool_choice: {type: "tool", name: "X"}`
  - **Nullable fields** prevent fabrication of optional data
  - **Enum with `"other"` + detail string** handles unknown cases gracefully
  - `additionalProperties: false` rejects extra fields
- **Task 4.4 — Validation-retry loop:**
  - **Bounded retry** with `maxRetries` so you don't loop forever
  - **Feedback in retry message** — pass the validation errors back so the model has info to correct
  - **Retry-helpfulness heuristic** — retry helps when the error gives the model new info (math mismatch); doesn't help when data is genuinely missing (no amount of retrying produces a date that isn't in the source)
  - **Fallback path** — when retries exhaust or errors aren't retryable, return a structured failure
- **Task 4.1 — Explicit criteria:** the prompt tells the model what to do *and* what not to do ("do not fabricate"). The schema enforces structure; the prompt guides judgment.

### Failure-mode exploration

- [ ] Make `shipping_address` non-nullable (`type: "string"` only). Run the missing-data invoice. The model will fabricate an address to satisfy the schema.
- [ ] Remove `"other"` from the `expense_category` enum and remove `expense_category_detail`. Run the missing-data invoice. The model will force-fit the category into one of the 5 strict options — misclassification.
- [ ] Remove the retry limit (set `maxRetries: Infinity`). Run the missing-data invoice. Watch it loop forever if you also remove the "abort on non-math errors" logic.
- [ ] Remove the error context from the retry message (just say "try again"). Watch retry effectiveness drop — the model has no info to correct on.

---

## Exercise 2 — Message Batches API with chunking

### Goal

Submit a batch of structured-extraction requests to the Message Batches API, poll for completion, handle results (success and failure), and re-submit failed entries in a smaller chunk.

### Why this matters

Task 4.5 is densely tested with verbatim facts: 50% cost savings, 24-hour processing window, no SLA, `custom_id` correlation, no streaming, no multi-turn tool calling, 100K requests / 256 MB max, 29-day result retention. The exam tests both the facts and the appropriate use cases — and traps where Batches is misapplied to real-time scenarios.

### Steps

- [ ] Create `batch_extract.ts`:

  ```typescript
  import Anthropic from "@anthropic-ai/sdk";
  import { writeFileSync, readFileSync } from "node:fs";

  const client = new Anthropic();

  // Simulated batch of invoice documents to process
  const invoices: Array<{ id: string; text: string }> = [
    { id: "inv_001", text: "ACME LLC | INV-001 | 2026-01-15 | Widget × 10 @ $20 = $200 | Total $200" },
    { id: "inv_002", text: "BETA Corp | INV-002 | 2026-01-16 | Service × 5 @ $50 = $250 | Tax $20 | Total $270" },
    { id: "inv_003", text: "GAMMA Inc | INV-003 | 2026-01-17 | Widget × 3 @ $20 = $60 | Total $60" },
    { id: "inv_004", text: "[Corrupted document — unreadable]" },
    { id: "inv_005", text: "DELTA SaaS | INV-005 | 2026-01-18 | Subscription × 1 @ $99 = $99 | Total $99" },
  ];

  // The same tool definition from Exercise 1 (simplified for batches)
  const extractTool: Anthropic.Tool = {
    name: "extract_invoice",
    description: "Extract structured invoice data. Use null for missing fields.",
    input_schema: {
      type: "object",
      properties: {
        vendor_name: { type: "string" },
        invoice_number: { type: "string" },
        invoice_date: { type: ["string", "null"] },
        total: { type: ["number", "null"] },
        confidence: { type: "string", enum: ["high", "medium", "low"] },
      },
      required: ["vendor_name", "invoice_number", "invoice_date", "total", "confidence"],
      additionalProperties: false,
    },
  };

  // Step 1: Submit the batch
  // Note the custom_id pattern — alphanumeric, hyphens, underscores, 1-64 chars
  console.log("Submitting batch...");
  const batch = await client.messages.batches.create({
    requests: invoices.map((inv) => ({
      custom_id: inv.id, // CRUCIAL: this is how we correlate results back
      params: {
        model: "claude-sonnet-4-6",
        max_tokens: 1024,
        tools: [extractTool],
        tool_choice: { type: "tool", name: "extract_invoice" },
        messages: [
          {
            role: "user",
            content: `Extract data from this invoice:\n\n${inv.text}`,
          },
        ],
      },
    })),
  });

  console.log(`Batch submitted: ${batch.id}`);
  console.log(`Status: ${batch.processing_status}`);
  console.log(`Created: ${batch.created_at}`);
  console.log(`Expires: ${batch.expires_at}`);

  // Step 2: Poll for completion (most batches finish in <1 hour; max 24h)
  // For this exercise, poll every 30 seconds
  let current = batch;
  while (current.processing_status !== "ended") {
    console.log(`\nWaiting... status: ${current.processing_status}`);
    console.log(
      `  Processing: ${current.request_counts.processing} | Succeeded: ${current.request_counts.succeeded} | Errored: ${current.request_counts.errored}`,
    );
    await new Promise((resolve) => setTimeout(resolve, 30_000));
    current = await client.messages.batches.retrieve(batch.id);
  }

  console.log(`\nBatch complete: ${current.id}`);
  console.log(
    `Final counts: ${JSON.stringify(current.request_counts, null, 2)}`,
  );

  // Step 3: Retrieve results
  // The results endpoint returns a streaming JSONL file
  const results = await client.messages.batches.results(batch.id);

  type SuccessResult = {
    custom_id: string;
    invoice: unknown;
  };
  type FailureResult = {
    custom_id: string;
    error: string;
  };

  const successes: SuccessResult[] = [];
  const failures: FailureResult[] = [];

  for await (const result of results) {
    if (result.result.type === "succeeded") {
      const msg = result.result.message;
      const toolUse = msg.content.find((b) => b.type === "tool_use");
      if (toolUse && toolUse.type === "tool_use") {
        successes.push({
          custom_id: result.custom_id,
          invoice: toolUse.input,
        });
      } else {
        failures.push({
          custom_id: result.custom_id,
          error: "No tool_use block in response",
        });
      }
    } else {
      failures.push({
        custom_id: result.custom_id,
        error: JSON.stringify(result.result),
      });
    }
  }

  console.log(`\nSuccesses (${successes.length}):`);
  for (const s of successes) {
    console.log(`  ${s.custom_id}: ${JSON.stringify(s.invoice)}`);
  }

  console.log(`\nFailures (${failures.length}):`);
  for (const f of failures) {
    console.log(`  ${f.custom_id}: ${f.error}`);
  }

  // Step 4: For failed entries, re-submit as a smaller chunk
  // (In real workflows you might also drill into individual failures to inspect them)
  if (failures.length > 0) {
    console.log(`\nResubmitting ${failures.length} failed entries...`);

    const failedInvoices = invoices.filter((inv) =>
      failures.some((f) => f.custom_id === inv.id),
    );

    const retryBatch = await client.messages.batches.create({
      requests: failedInvoices.map((inv) => ({
        custom_id: `retry-${inv.id}`,
        params: {
          model: "claude-opus-4-7", // use a more capable model for retries
          max_tokens: 1024,
          tools: [extractTool],
          tool_choice: { type: "tool", name: "extract_invoice" },
          messages: [
            {
              role: "user",
              content: `Extract data from this invoice. If the document is unreadable, return all fields as null/low-confidence:\n\n${inv.text}`,
            },
          ],
        },
      })),
    });

    console.log(`Retry batch submitted: ${retryBatch.id}`);
    // (Poll and process as before — left as practice)
  }

  // Persist results for downstream consumers
  writeFileSync(
    "batch_results.json",
    JSON.stringify({ successes, failures }, null, 2),
  );
  console.log("\nResults written to batch_results.json");
  ```

- [ ] Run it:
  ```bash
  npx tsx batch_extract.ts
  ```
  Note: this will run against the actual Batches API. The batch may take a few minutes (most batches finish in <1 hour, but the exam knows there's no SLA — could be up to 24 hours).

### Verification

- The batch is submitted and returns a `batch.id`
- Polling shows `request_counts` progressing
- Results stream back with `custom_id` matching the input IDs
- Successful extractions are in `successes`; the corrupted document is in `failures`
- The retry batch is submitted for failed entries only

### Patterns reinforced

- **Task 4.5 — Batches API verbatim facts** (exam-day memorization):
  - **50% cost savings** vs interactive API
  - **24-hour max processing window** (most finish in <1 hour, but no SLA)
  - **`custom_id`** for input-output correlation (1-64 chars, regex `^[a-zA-Z0-9_-]{1,64}$`)
  - **Results unordered** — match by `custom_id`
  - **100K requests / 256 MB max** per batch
  - **29-day result retention** after batch creation
  - **No streaming** in batches
  - **No multi-turn tool calling** — single-turn only
  - **Failure isolation** — one request failing doesn't affect others
- **The chunking pattern:** submit large batch → identify failures via `custom_id` → re-submit failures as a smaller batch (possibly with a more capable model)
- **When NOT to use the Batches API:** real-time / customer-facing / multi-turn / streaming — these are 4.5 trap patterns

### Failure-mode exploration

- [ ] Use a `custom_id` longer than 64 chars or with disallowed characters (spaces, slashes). Watch the API reject the batch.
- [ ] Submit two batches with overlapping `custom_id` values (`inv_001` in both). Each batch is independent — results are scoped to the batch.
- [ ] Try to include `stream: true` in batch request params. The API rejects it (no streaming in batches).
- [ ] Try a multi-turn batch (include both user and assistant messages with a tool result). The API rejects it (single-turn only).

---

## Exercise 3 — Multi-instance independent review

### Goal

For a structured-extraction task, run the same input through **three independent instances** and compare. Disagreement among instances surfaces genuine ambiguity in the source — this is the documented pattern for production quality assurance.

### Why this matters

Task 4.6 tests independent multi-instance review as the correct approach to quality assurance. The wrong answers reach for same-session self-review (which isn't independent) or "run twice and use the second result." The exam tests recognition that real disagreement is a signal of ambiguity, not error.

### Steps

- [ ] Create `multi_instance_review.ts`:

  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  type Classification = {
    sentiment: "positive" | "neutral" | "negative" | "mixed";
    intent: "complaint" | "praise" | "question" | "request" | "other";
    intent_detail: string | null;
    urgency: "low" | "medium" | "high";
    confidence: "high" | "medium" | "low";
  };

  const classifyTool: Anthropic.Tool = {
    name: "classify_message",
    description: "Classify a customer support message.",
    input_schema: {
      type: "object",
      properties: {
        sentiment: { type: "string", enum: ["positive", "neutral", "negative", "mixed"] },
        intent: { type: "string", enum: ["complaint", "praise", "question", "request", "other"] },
        intent_detail: { type: ["string", "null"] },
        urgency: { type: "string", enum: ["low", "medium", "high"] },
        confidence: { type: "string", enum: ["high", "medium", "low"] },
      },
      required: ["sentiment", "intent", "intent_detail", "urgency", "confidence"],
      additionalProperties: false,
    },
  };

  async function classifyOnce(message: string): Promise<Classification> {
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 512,
      tools: [classifyTool],
      tool_choice: { type: "tool", name: "classify_message" },
      messages: [
        {
          role: "user",
          content: `Classify this customer support message. Use null for intent_detail unless intent is "other".\n\n<message>\n${message}\n</message>`,
        },
      ],
    });
    const toolUse = response.content.find((b) => b.type === "tool_use");
    if (!toolUse || toolUse.type !== "tool_use") {
      throw new Error("No tool_use in response");
    }
    return toolUse.input as Classification;
  }

  // Run N independent classifications in parallel
  async function classifyIndependent(
    message: string,
    n: number = 3,
  ): Promise<Classification[]> {
    return Promise.all(Array.from({ length: n }, () => classifyOnce(message)));
  }

  function summarizeAgreement(results: Classification[]): {
    agreement: Record<keyof Classification, number>;
    disagreements: string[];
  } {
    const fields: (keyof Classification)[] = [
      "sentiment",
      "intent",
      "intent_detail",
      "urgency",
      "confidence",
    ];
    const agreement: Record<string, number> = {};
    const disagreements: string[] = [];

    for (const field of fields) {
      const values = results.map((r) => r[field]);
      const unique = new Set(values);
      agreement[field] = unique.size === 1 ? 1.0 : (results.length - unique.size + 1) / results.length;
      if (unique.size > 1) {
        disagreements.push(
          `${field}: ${Array.from(unique).map((v) => JSON.stringify(v)).join(" / ")}`,
        );
      }
    }
    return { agreement: agreement as Record<keyof Classification, number>, disagreements };
  }

  const testMessages = [
    // Clear: positive praise, low urgency
    "Thank you so much! Your team's support has been incredible. I'm a customer for life.",
    // Clear: negative complaint, high urgency
    "My account has been charged 3 times for the same order. I need this fixed NOW or I'm reporting fraud.",
    // Genuinely ambiguous: mixed sentiment, unclear urgency
    "I love using your product but the recent UI change is really confusing. Not sure if I should switch to something else. Has anyone else mentioned this?",
    // Subtle: question that could be a polite complaint
    "Could you tell me why my invoice this month is twice last month's?",
  ];

  for (const msg of testMessages) {
    console.log("\n" + "=".repeat(60));
    console.log("MESSAGE:", msg);
    const results = await classifyIndependent(msg, 3);
    console.log("\nResults from 3 independent instances:");
    results.forEach((r, i) => {
      console.log(
        `  Instance ${i + 1}: sentiment=${r.sentiment}, intent=${r.intent}, urgency=${r.urgency}, confidence=${r.confidence}`,
      );
    });

    const { agreement, disagreements } = summarizeAgreement(results);
    console.log("\nField agreement:", JSON.stringify(agreement));
    if (disagreements.length > 0) {
      console.log("Disagreements (route to human review):");
      for (const d of disagreements) console.log("  -", d);
    } else {
      console.log("All instances agree — high confidence in classification.");
    }
  }
  ```

- [ ] Run it:
  ```bash
  npx tsx multi_instance_review.ts
  ```

### Verification

For each message, observe:

- **Clear positive message** → all 3 instances likely agree completely
- **Clear negative complaint** → all 3 likely agree
- **Genuinely ambiguous message** → instances disagree on sentiment ("mixed" vs "negative") or urgency ("low" vs "medium") — this disagreement is the **signal** that human review is needed
- **Subtle message** → may show partial disagreement (intent might split between "question" and "complaint")

### Patterns reinforced

- **Task 4.6 — Multi-instance independent review:**
  - Run the **same** input through **separate, independent** instances (separate API calls, no shared context, ideally parallel)
  - **Disagreement is a signal**, not noise — it surfaces genuine ambiguity in the source
  - Use disagreement to **route to human review**, not to "pick the best one" or "majority vote and ignore"
- **Why not same-session self-review:** the model is anchored on its prior response; "review your own answer" doesn't escape the original framing
- **Why not "run twice and use second":** arbitrary; doesn't surface ambiguity
- The 3% genuine-ambiguity rate cited in the exam guide manifests as the rate at which independent instances disagree

### Failure-mode exploration

- [ ] Run all 3 classifications **in the same session** (multi-turn conversation, asking Claude to classify, then "do it again, independently"). Watch how the second and third are anchored on the first.
- [ ] Run with N=10 instances. Compute disagreement rate. Estimate the genuine-ambiguity rate of your input population.
- [ ] Modify the classification to add a "confidence" field that each instance must self-assess. Compare the model's self-assessed confidence to actual instance-agreement rate. (Self-reported confidence isn't a reliable proxy for instance agreement.)

---

## Quick drills

Five short drills covering remaining Domain 4 patterns.

### Quick drill A — Native structured outputs (beta)

**Goal:** Use `output_config.format` directly on `messages.create()` for JSON output without a tool.

- [ ] Create `native_structured.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  // Note: native structured outputs use a beta header.
  // The output_format parameter is DEPRECATED; use output_config.format.
  const response = await client.messages.create(
    {
      model: "claude-sonnet-4-6",
      max_tokens: 1024,
      messages: [
        {
          role: "user",
          content:
            "Extract: 'Alice Johnson, 34, software engineer at Acme Inc, alice@acme.example'",
        },
      ],
      // @ts-expect-error - output_config is a beta field
      output_config: {
        format: {
          type: "json_schema",
          schema: {
            type: "object",
            properties: {
              name: { type: "string" },
              age: { type: "integer" },
              role: { type: "string" },
              company: { type: "string" },
              email: { type: "string" },
            },
            required: ["name", "age", "role", "company", "email"],
            additionalProperties: false,
          },
        },
      },
    },
    {
      headers: { "anthropic-beta": "structured-outputs-2025-11-13" },
    },
  );

  const text = response.content
    .filter((b) => b.type === "text")
    .map((b) => (b as { text: string }).text)
    .join("");

  console.log("Raw response:", text);
  console.log("Parsed:", JSON.parse(text));
  ```

- [ ] Run it:
  ```bash
  npx tsx native_structured.ts
  ```

**Pattern reinforced:** The newer `output_config.format` API (beta `structured-outputs-2025-11-13`) returns JSON directly without forcing a tool call. The deprecated `output_format` parameter has been replaced. Schemas are cached for 24 hours after first compile (100-300ms compile overhead first time).

### Quick drill B — Few-shot vs schema

**Goal:** Compare consistency: 5 few-shot examples vs. tool_use schema. Schema wins.

- [ ] Create `few_shot_vs_schema.ts` with two functions:
  1. `classifyWithFewShot(message)` — 5 examples in the prompt, no tool, ask for JSON output as text
  2. `classifyWithSchema(message)` — tool_use with forced `tool_choice`, schema validation

- [ ] Run each function 10 times on the same input. Compare:
  - **Few-shot:** consistency 70-90% (occasional drift in field names, extra fields, malformed JSON)
  - **Schema:** consistency 100% (protocol-enforced structure)

**Pattern reinforced:** Few-shot examples shape style and tone (soft constraints). Schemas enforce structure (hard constraints). For consistency, schema wins every time.

### Quick drill C — `tool_choice` + extended thinking incompatibility

**Goal:** Confirm that forced `tool_choice` and `"any"` don't work with extended thinking.

- [ ] Create `thinking_tool_choice.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();
  const tools: Anthropic.Tool[] = [
    {
      name: "lookup",
      description: "Look up a fact.",
      input_schema: {
        type: "object",
        properties: { query: { type: "string" } },
        required: ["query"],
      },
    },
  ];

  // This SHOULD work: tool_choice auto + extended thinking
  try {
    const ok = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 4096,
      thinking: { type: "enabled", budget_tokens: 2048 },
      tools,
      tool_choice: { type: "auto" },
      messages: [{ role: "user", content: "Look up something." }],
    });
    console.log("auto + thinking: ok");
  } catch (e) {
    console.log("auto + thinking: ERROR", (e as Error).message);
  }

  // This SHOULD fail: tool_choice any + extended thinking
  try {
    const any = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 4096,
      thinking: { type: "enabled", budget_tokens: 2048 },
      tools,
      tool_choice: { type: "any" },
      messages: [{ role: "user", content: "Look up something." }],
    });
    console.log("any + thinking: ok (unexpected)");
  } catch (e) {
    console.log('any + thinking: ERROR (expected):', (e as Error).message);
  }

  // This SHOULD fail: tool_choice forced + extended thinking
  try {
    const forced = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 4096,
      thinking: { type: "enabled", budget_tokens: 2048 },
      tools,
      tool_choice: { type: "tool", name: "lookup" },
      messages: [{ role: "user", content: "Look up something." }],
    });
    console.log("forced + thinking: ok (unexpected)");
  } catch (e) {
    console.log('forced + thinking: ERROR (expected):', (e as Error).message);
  }
  ```

- [ ] Run it. Confirm: `auto` works; `any` and forced both error.

**Pattern reinforced:** Extended thinking only supports `tool_choice: "auto"` or `"none"`. The `any` and `{type: "tool"}` options prefill the assistant message, which conflicts with thinking blocks.

### Quick drill D — XML structure for clarity

**Goal:** Compare the same prompt with and without XML structure. Observe the consistency gain.

- [ ] Take a vague prompt:
  > "Look at this email and tell me if it's spam, and why."
  
  Run 10 times. Observe inconsistency in response structure (sometimes "yes" then reasoning, sometimes reasoning first, sometimes a one-word answer).

- [ ] Restructure with XML tags:
  ```
  <task>Classify the following email as spam or not, with reasoning.</task>
  
  <email>
  {{email_body}}
  </email>
  
  <output_format>
  <classification>spam | not_spam</classification>
  <reasoning>One paragraph explaining the classification.</reasoning>
  </output_format>
  ```
  
  Run 10 times. Consistency dramatically improves; output structure matches the XML template.

**Pattern reinforced:** XML tags help Claude parse complex prompts unambiguously. Use distinct tags for distinct content (`<instructions>`, `<example>`, `<input>`, `<output_format>`, `<context>`).

### Quick drill E — Schema grammar cache observation

**Goal:** Observe the 24-hour schema grammar cache by timing the first vs. second request with the same schema.

- [ ] Create `schema_cache_timing.ts`:
  ```typescript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const tool: Anthropic.Tool = {
    name: "extract_person",
    description: "Extract person details.",
    input_schema: {
      type: "object",
      properties: {
        name: { type: "string" },
        age: { type: "integer", minimum: 0, maximum: 200 },
        occupation: { type: ["string", "null"] },
        skills: {
          type: "array",
          items: { type: "string" },
          minItems: 0,
          maxItems: 20,
        },
      },
      required: ["name", "age", "occupation", "skills"],
      additionalProperties: false,
    },
  };

  async function timeRequest(label: string) {
    const t0 = performance.now();
    await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 512,
      tools: [{ ...tool, strict: true } as Anthropic.Tool],
      tool_choice: { type: "tool", name: "extract_person" },
      messages: [
        {
          role: "user",
          content: "Extract: Mary Chen, age 31, software engineer, skills: TypeScript, React, GraphQL",
        },
      ],
    });
    console.log(`${label}: ${(performance.now() - t0).toFixed(0)}ms`);
  }

  await timeRequest("First request (schema compile)");
  await timeRequest("Second request (schema cached)");
  await timeRequest("Third request (schema cached)");
  ```

- [ ] Run it:
  ```bash
  npx tsx schema_cache_timing.ts
  ```
  Expected: the first request is ~100-300ms slower than subsequent requests (grammar compilation). Cache lasts 24 hours.

**Pattern reinforced:** Schema grammars are compiled on first use and cached for 24 hours. For production systems with critical schemas, "warm" the cache by sending a dummy request during deployment.

---

## Quick-reference cheatsheet for the patterns drilled

- **Structured output:** `tool_use` with forced `tool_choice` (classic), OR `output_config.format` (newer beta `structured-outputs-2025-11-13`). NOT "respond in JSON" in prompt.
- **`tool_choice`:** `auto` (default), `any` (force some tool), `{type: "tool", name: "X"}` (force specific), `none` (disable).
- **Extended thinking** only works with `tool_choice: "auto"` or `"none"`. `any` and forced both error.
- **Nullable fields prevent fabrication.** Required-and-non-nullable forces invention.
- **Enum + `"other"` + detail string** for unknown handling.
- **`strict: true`** eliminates syntax errors only. Semantic validation is your responsibility.
- **Schema grammar caching:** 100-300ms first compile; 24-hour cache.
- **Validation-retry:** retry helps when error gives correction info; doesn't help when source lacks data. **Always** have retry limit + fallback.
- **Batches API:** 50% cost, 24-hour window, no SLA, `custom_id` (1-64 chars), NO multi-turn, NO streaming, 100K requests / 256 MB max, 29-day result retention, results unordered.
- **Batches API NOT for:** real-time, customer-facing, multi-turn tool use, streaming.
- **Beta header for 300K output:** `output-300k-2026-03-24` (Opus 4.7/4.6, Sonnet 4.6, Claude API only).
- **Few-shot examples:** 3-5; XML-wrapped (`<example>` / `<examples>` parent); relevant, diverse, structured.
- **XML tags** for distinct content: `<instructions>`, `<example>`, `<formatting>`, `<context>`, `<input>`, `<output_format>`.
- **Independent review:** separate sessions (or separate batch entries); NOT same-session self-review.
- **Multi-instance review:** disagreement among independent runs surfaces genuine ambiguity (~3% in practice).

---

## Progress tracker

- [ ] Exercise 1 complete (structured extraction with retry loop)
- [ ] Exercise 1 failure-mode exploration complete
- [ ] Exercise 2 complete (Batches API with chunking)
- [ ] Exercise 2 failure-mode exploration complete
- [ ] Exercise 3 complete (multi-instance independent review)
- [ ] Exercise 3 failure-mode exploration complete
- [ ] Quick drill A — Native structured outputs
- [ ] Quick drill B — Few-shot vs schema comparison
- [ ] Quick drill C — `tool_choice` + extended thinking incompatibility
- [ ] Quick drill D — XML structure for clarity
- [ ] Quick drill E — Schema grammar cache observation

---

*Domain 4 weight: **20%** of the exam.*
