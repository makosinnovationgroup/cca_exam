---
title: Claude Certified Architect — Foundations · Domain 1 Hands-On Exercises
tags:
  - exam-prep
  - claude-cca-f
  - anthropic
  - certification
  - hands-on
  - domain-1
status: in-progress
created: 2026-05-08
estimated-hours: 3-4
---

# CCA-F Domain 1 — Hands-On Exercises

> [!info] Linked notes
> [[cca_exam_guide]] · [[cca_docs_roadmap]] · [[cca_progress]] · [[cca_domain1_practice]]

Four exercises that build practical fluency in the Agent SDK patterns most heavily tested in Domain 1 (27% of the exam). Each one maps to specific task statements from the exam guide.

| Exercise | Task statements reinforced | Time |
|----------|---------------------------|------|
| 0 — Setup + Hello World | 1.1 | ~30 min |
| 1 — Multi-tool agent + hook | 1.1, 1.4, 1.5, 2.1, 2.2 | ~60 min |
| 2 — Multi-agent coordinator with parallel spawn | 1.2, 1.3, 5.3, 5.6 | ~90 min |
| 3 — Session forking | 1.7 | ~30 min |

## Prerequisites

- [ ] **Python 3.10+** installed (`python3 --version` should show 3.10 or higher)
- [ ] **Anthropic API key** in env: `export ANTHROPIC_API_KEY=sk-ant-...` — or active Claude Code subscription
- [ ] A working directory: `mkdir -p ~/cca-prep && cd ~/cca-prep`
- [ ] Basic Python comfort (no async wizardry required; the SDK uses async but the patterns are simple)

---

## Exercise 0 — Setup and Hello World

### Goal

Get the SDK installed and confirm an agent loop runs end-to-end. This is the foundation everything else builds on.

### Steps

- [ ] Install the SDK:
  ```bash
  pip install claude-agent-sdk
  ```

- [ ] Create `hello_agent.py`:
  ```python
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions

  async def main():
      async for message in query(
          prompt="What's 2 + 2? Reply with just the number.",
          options=ClaudeAgentOptions(
              system_prompt="You are a concise math assistant."
          )
      ):
          print(message)

  asyncio.run(main())
  ```

- [ ] Run it:
  ```bash
  python hello_agent.py
  ```

- [ ] Verify you see assistant messages streaming back, ending with a result message that includes `stop_reason: "end_turn"`.

### What to notice

This single loop is the foundation. The SDK abstracts away the manual `stop_reason` checking — internally it's looping while `stop_reason == "tool_use"` and terminating when it sees `"end_turn"`. **You don't have to write the loop yourself**, but the exam tests that you understand what's happening underneath.

### Exam mapping

- **Task 1.1** (agentic loops): you've just used a managed agentic loop. The SDK handles `stop_reason` for you; the exam will test that you understand this is driven by `stop_reason`, not by parsing text or iteration caps.

---

## Exercise 1 — Multi-tool Customer Support Agent with Policy Hook

### Goal

Build a customer support agent with four MCP tools and wire in a `PreToolUse` hook that blocks any `process_refund` above $500. This drills Task 1.4 (your weak spot) and Tasks 1.5, 2.1, 2.2.

### Step 1 — Define the MCP tools

- [ ] Create `customer_support_agent.py`:

```python
import asyncio
from claude_agent_sdk import (
    query,
    tool,
    create_sdk_mcp_server,
    ClaudeAgentOptions,
    HookMatcher,
)

# --- TOOLS ---------------------------------------------------------

# Note the description quality: each tool clearly differentiates
# itself from the others. This is Task 2.1 in practice.

@tool(
    "get_customer",
    "Retrieve customer profile by customer_id. Use this to verify customer "
    "identity before any account or financial operation. Returns name, email, "
    "membership tier, and verification status.",
    {"customer_id": str},
)
async def get_customer(args):
    # Fake data store
    customers = {
        "C-100": {"name": "Alice Chen", "tier": "Gold", "verified": True},
        "C-200": {"name": "Bob Singh", "tier": "Standard", "verified": True},
    }
    cust = customers.get(args["customer_id"])
    if not cust:
        # Structured error response (Task 2.2)
        return {
            "content": [{
                "type": "text",
                "text": '{"isError": true, "errorCategory": "validation", '
                       '"isRetryable": false, "description": "Customer not found"}'
            }]
        }
    return {"content": [{"type": "text", "text": str(cust)}]}


@tool(
    "lookup_order",
    "Retrieve order details by order_id. Use for order-status questions, "
    "shipping inquiries, or to identify the order associated with a refund. "
    "Returns order total, status, and item list.",
    {"order_id": str},
)
async def lookup_order(args):
    orders = {
        "O-5821": {"total": 147.50, "status": "delivered", "customer": "C-100"},
        "O-5822": {"total": 89.00, "status": "shipped", "customer": "C-200"},
    }
    order = orders.get(args["order_id"])
    if not order:
        return {"content": [{"type": "text",
                "text": '{"isError": true, "errorCategory": "validation"}'}]}
    return {"content": [{"type": "text", "text": str(order)}]}


@tool(
    "process_refund",
    "Issue a refund to the customer for a specific order. ONLY call this "
    "after get_customer has verified identity. Returns success or business-rule "
    "denial.",
    {"customer_id": str, "order_id": str, "amount": float},
)
async def process_refund(args):
    # In production this would call your billing system.
    return {
        "content": [{
            "type": "text",
            "text": f"Refund of ${args['amount']:.2f} processed for "
                    f"order {args['order_id']}."
        }]
    }


@tool(
    "escalate_to_human",
    "Route this case to a human agent with a structured handoff summary. "
    "Use when policy gaps, customer explicitly requests a human, or "
    "operation exceeds policy limits.",
    {"reason": str, "summary": str},
)
async def escalate_to_human(args):
    return {
        "content": [{
            "type": "text",
            "text": f"Escalated to human queue. Reason: {args['reason']}"
        }]
    }


# Bundle into an SDK MCP server
support_server = create_sdk_mcp_server(
    name="customer-support",
    version="1.0",
    tools=[get_customer, lookup_order, process_refund, escalate_to_human],
)
```

### Step 2 — Add the PreToolUse hook for the $500 cap

This is the **core Task 1.4/1.5 pattern**. The hook intercepts `process_refund` before it executes and denies if the amount exceeds policy.

- [ ] Add this to the same file, below the tool definitions:

```python
# --- HOOK: enforce $500 refund cap ---------------------------------

async def refund_cap_hook(input_data, tool_use_id, context):
    """
    PreToolUse hook. Fires before any tool call.
    If process_refund is called with amount > 500, block it.
    """
    tool_name = input_data.get("tool_name", "")
    tool_input = input_data.get("tool_input", {})

    # SDK MCP tools are namespaced as mcp__<server-name>__<tool-name>
    if tool_name == "mcp__customer-support__process_refund":
        amount = tool_input.get("amount", 0)
        if amount > 500:
            return {
                "hookSpecificOutput": {
                    "hookEventName": "PreToolUse",
                    "permissionDecision": "deny",
                    "permissionDecisionReason": (
                        f"Refund of ${amount:.2f} exceeds $500 policy limit. "
                        "Use escalate_to_human instead."
                    ),
                }
            }
    return {}  # No intervention; let the call proceed.
```

> [!tip] Why the hook is essential
> A system prompt rule like "never refund over $500" works most of the time but isn't guaranteed. The hook **guarantees** the block — when deterministic compliance is required (financial policy, identity verification, regulatory), prompts have a non-zero failure rate. This is the exact distinction tested by Task 1.5 and Q3 in your practice bank.

### Step 3 — Run the agent

- [ ] Add the runner at the bottom of the file:

```python
# --- RUNNER --------------------------------------------------------

async def main():
    options = ClaudeAgentOptions(
        system_prompt=(
            "You are a customer support agent. ALWAYS call get_customer first "
            "to verify identity before any refund or account change. Use the "
            "structured handoff via escalate_to_human for cases that hit "
            "policy gaps."
        ),
        mcp_servers={"customer-support": support_server},
        allowed_tools=[
            "mcp__customer-support__get_customer",
            "mcp__customer-support__lookup_order",
            "mcp__customer-support__process_refund",
            "mcp__customer-support__escalate_to_human",
        ],
        hooks={
            "PreToolUse": [
                HookMatcher(
                    matcher="mcp__customer-support__process_refund",
                    hooks=[refund_cap_hook],
                )
            ]
        },
    )

    # TEST 1: Small refund — should succeed
    print("\n=== TEST 1: $50 refund (should succeed) ===\n")
    async for msg in query(
        prompt=(
            "Customer C-100 reports order O-5821 arrived damaged. "
            "Process a $50 partial refund."
        ),
        options=options,
    ):
        print(msg)

    # TEST 2: Large refund — should be blocked
    print("\n=== TEST 2: $750 refund (should be blocked) ===\n")
    async for msg in query(
        prompt=(
            "Customer C-200 reports order O-5822 arrived damaged. "
            "Process a $750 refund for the full amount."
        ),
        options=options,
    ):
        print(msg)


if __name__ == "__main__":
    asyncio.run(main())
```

### Step 4 — Run and verify

- [ ] Run: `python customer_support_agent.py`
- [ ] **Test 1 verification**: You should see the agent call `get_customer`, then `process_refund` succeeds, and the agent reports the $50 refund processed.
- [ ] **Test 2 verification**: You should see the agent call `get_customer`, attempt `process_refund` with $750 — and the **hook denies the call**. The agent should then pivot and call `escalate_to_human`. If it just gives up, that's a sign the system prompt isn't strong enough about the fallback.

### Exam mapping

- **Task 1.1** — managed agentic loop with `stop_reason`-driven flow
- **Task 1.4** ⚠️ — programmatic prerequisite via hook (your weak spot)
- **Task 1.5** — `PreToolUse` hook with `permissionDecision: "deny"` redirecting to alternative workflow
- **Task 2.1** — differentiated tool descriptions to prevent selection confusion
- **Task 2.2** — structured error responses with `errorCategory` and `isRetryable`

> [!warning] If the hook signature doesn't work
> The exact callback signature has evolved across SDK versions. If `refund_cap_hook(input_data, tool_use_id, context)` fails, check `docs.claude.com/en/docs/agent-sdk/hooks` for the current Python signature. The **concept** is what's tested on the exam — block specific tool calls based on inputs, return `permissionDecision: "deny"` to enforce.

---

## Exercise 2 — Multi-Agent Coordinator with Parallel Subagent Spawning

### Goal

Build a research coordinator that spawns two specialized subagents in parallel. Measure the latency improvement vs serial. This drills Tasks 1.2, 1.3, 5.3, and 5.6 (provenance — your weak spot).

### Step 1 — Define the subagents using AgentDefinition

- [ ] Create `research_pipeline.py`:

```python
import asyncio
import time
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    AgentDefinition,
)

# --- SUBAGENT DEFINITIONS ------------------------------------------

# Each AgentDefinition has its own system prompt, allowed tools,
# and (optionally) a model override. Note the scoped tool access
# (Task 2.3) — search agent only gets search tools, etc.

WEB_SEARCH_AGENT = AgentDefinition(
    description=(
        "Searches the web for recent factual information. Returns findings "
        "as structured records with claim, evidence_excerpt, source_url, "
        "and publication_date for every claim."
    ),
    prompt=(
        "You are a web research agent. For every claim you produce, you "
        "MUST emit a structured record: "
        '{"claim": "...", "evidence_excerpt": "...", '
        '"source_url": "...", "publication_date": "..."}. '
        "Never produce prose summaries without these fields."
    ),
    tools=["WebSearch", "WebFetch"],
    model="sonnet",
)

DOC_ANALYZER_AGENT = AgentDefinition(
    description=(
        "Analyzes provided documents for specific claims and quotes. Returns "
        "structured records with claim, evidence_excerpt, document_name, "
        "and page_or_section for every claim."
    ),
    prompt=(
        "You are a document analysis agent. For every claim, emit: "
        '{"claim": "...", "evidence_excerpt": "...", '
        '"document_name": "...", "page_or_section": "..."}. '
        "Surface conflicting values from different documents explicitly."
    ),
    tools=["Read", "Grep"],
    model="sonnet",
)
```

### Step 2 — The coordinator

- [ ] Continue in the same file:

```python
# --- COORDINATOR ---------------------------------------------------

async def run_research(prompt: str, parallel: bool):
    """
    Run the research coordinator on a prompt.
    parallel=True: instruct coordinator to spawn both subagents in one response
    parallel=False: instruct coordinator to spawn serially (one then the other)
    """
    parallel_instruction = (
        "Spawn the web_search_agent and doc_analyzer_agent IN PARALLEL by "
        "emitting BOTH Task tool calls in your very next response."
        if parallel
        else "Spawn the web_search_agent first, wait for its result, then "
             "spawn the doc_analyzer_agent."
    )

    coordinator_prompt = (
        "You are a research coordinator. Your job: decompose the user's "
        "research question, delegate to specialist subagents, and synthesize "
        "their findings.\n\n"
        f"{parallel_instruction}\n\n"
        "Each subagent will return structured records with attribution. "
        "Your final synthesis MUST preserve source_url, document_name, and "
        "publication_date on every claim. If subagents return conflicting "
        "values, annotate the conflict with attribution rather than "
        "silently reconciling."
    )

    options = ClaudeAgentOptions(
        system_prompt=coordinator_prompt,
        agents={
            "web_search_agent": WEB_SEARCH_AGENT,
            "doc_analyzer_agent": DOC_ANALYZER_AGENT,
        },
        # IMPORTANT: Task tool MUST be in allowedTools so the coordinator
        # can spawn subagents. This is Task 1.3 in practice.
        allowed_tools=["Task"],
    )

    start = time.time()
    async for msg in query(prompt=prompt, options=options):
        print(msg)
    elapsed = time.time() - start
    return elapsed


async def main():
    research_question = (
        "What are the latest documented trends in AI adoption in healthcare, "
        "and how do industry-report claims compare to peer-reviewed findings?"
    )

    print("=== SERIAL RUN ===\n")
    serial_time = await run_research(research_question, parallel=False)

    print("\n=== PARALLEL RUN ===\n")
    parallel_time = await run_research(research_question, parallel=True)

    print(f"\n\nSerial:   {serial_time:.1f}s")
    print(f"Parallel: {parallel_time:.1f}s")
    print(f"Speedup:  {serial_time / parallel_time:.1f}x")


if __name__ == "__main__":
    asyncio.run(main())
```

### Step 3 — Run and observe

- [ ] Run: `python research_pipeline.py`
- [ ] **Watch for these in the output**:
  1. In the parallel run, **two `Task` tool calls in a single coordinator response** (both subagents are spawned at once)
  2. In the serial run, they appear in separate turns (spawn → wait → result → next spawn)
  3. The parallel run should be roughly **30–50% faster** for a research task of this size

### Step 4 — Break the provenance and watch what happens (Task 5.6 reinforcement)

- [ ] Modify the coordinator's system prompt to weaken provenance: change the line about preserving `source_url`/`document_name` to just "cite sources where possible." Re-run.
- [ ] Observe: the final synthesis is now more likely to drop attribution on some claims. This is exactly the failure mode Q11 tests. **Restore the strict instruction** and re-run — attribution preservation comes back.

### Exam mapping

- **Task 1.2** — hub-and-spoke (coordinator → subagents → coordinator synthesis), coordinator-driven decomposition
- **Task 1.3** — `Task` in `allowedTools`, `AgentDefinition`, explicit context (we passed the research question in the user prompt, not assumed inheritance), **parallel spawning by emitting multiple Task calls in one response**
- **Task 2.3** — scoped tool access per subagent role
- **Task 5.6** ⚠️ — structured claim-source mappings preserved through synthesis (your weak spot — feel it work and feel it break)

> [!tip] Why doing this beats reading about it
> Until you've watched two `Task` tool calls fire in a single response and seen latency halve, the parallel-spawn pattern is abstract. After you've seen it once, Q10 and Q34 are obvious. Same for provenance — once you've watched a synthesis agent drop attribution because the coordinator's instructions were sloppy, Q11/Q32 land instantly.

---

## Exercise 3 — Session Forking for Divergent Exploration

### Goal

Establish a baseline session, fork it twice, explore two divergent paths, and confirm each fork has independent context. This is the core Task 1.7 pattern.

### Step 1 — Create the baseline session

- [ ] Create `session_fork_demo.py`:

```python
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

async def main():
    options = ClaudeAgentOptions(
        system_prompt=(
            "You are a code architect helping evaluate refactoring strategies. "
            "Be concise. Reference specific findings from earlier in the session."
        ),
    )

    # --- BASELINE SESSION --------------------------------------------
    async with ClaudeSDKClient(options=options) as client:
        await client.query(
            "I have a 2000-line payment processing module with three "
            "responsibilities mixed together: validation, billing API calls, "
            "and audit logging. Summarize the structural problems in three "
            "bullet points."
        )
        async for msg in client.receive_response():
            print("BASELINE:", msg)

        # Capture the session ID so we can fork from it
        baseline_session_id = client.session_id

        # --- FORK A: extract-classes strategy -----------------------
        print("\n=== FORK A: extract-classes strategy ===\n")
        async with ClaudeSDKClient(
            options=options,
            resume=baseline_session_id,
            fork_session=True,  # Independent branch from the baseline
        ) as fork_a:
            await fork_a.query(
                "Evaluate the 'extract three classes' refactoring strategy "
                "specifically. List risks and migration steps."
            )
            async for msg in fork_a.receive_response():
                print("FORK A:", msg)

        # --- FORK B: hexagonal-architecture strategy ----------------
        print("\n=== FORK B: hexagonal architecture ===\n")
        async with ClaudeSDKClient(
            options=options,
            resume=baseline_session_id,
            fork_session=True,
        ) as fork_b:
            await fork_b.query(
                "Evaluate moving to a hexagonal architecture with ports and "
                "adapters. List risks and migration steps."
            )
            async for msg in fork_b.receive_response():
                print("FORK B:", msg)


if __name__ == "__main__":
    asyncio.run(main())
```

### Step 2 — Run and observe

- [ ] Run: `python session_fork_demo.py`
- [ ] **What to confirm**:
  1. Both forks reference the **baseline analysis** (the three structural problems) — the fork inherits the baseline context
  2. Fork A's discussion of extract-classes risks does **not** appear in Fork B's reasoning — the forks are isolated from each other
  3. The baseline session itself is unchanged after the forks complete

### Step 3 — Compare with resume (Task 1.7 distinction)

- [ ] Modify the code: remove `fork_session=True` from Fork B, so it becomes a regular resume:
  ```python
  async with ClaudeSDKClient(
      options=options,
      resume=baseline_session_id,
      # fork_session=False (default)
  ) as fork_b:
  ```
- [ ] Run again. Now Fork B is a **continuation** of the baseline. If you'd run Fork A first as a fork it stays isolated, but if you do them serially without forking, the second one inherits whatever the first one did.

### Step 4 — The "fresh-start vs resume" decision (Q24 territory)

- [ ] Imagine you ran this analysis yesterday. Today you've **modified the payment module heavily** — 30% of the code has changed. Should you `resume` the baseline session?
- [ ] Answer per Task 1.7: **No** — start a fresh session with an injected summary of the current state. Resuming would carry stale tool results into context.
- [ ] Try it: create a new session with options including a system prompt that says "Here's the current state of the payment module after refactoring [summary]" rather than resuming. Note how much cleaner the reasoning is.

### Exam mapping

- **Task 1.7** — `resume` for continuation, `fork_session=True` for divergent branches from a shared baseline, and the meta-decision of when fresh-start-with-summary beats resume-with-stale-results

> [!warning] If fork_session syntax differs in your SDK version
> Some versions expose forking as a method (`client.fork()`) or via different parameter names. If the `fork_session=True` flag doesn't exist in your SDK, check `docs.claude.com/en/docs/agent-sdk/sessions` for the current syntax. The **concept** the exam tests: forking creates independent branches from a shared baseline.

---

## Wrap-up — What You Should Have Built

After completing all three exercises, you should have:

- [ ] `hello_agent.py` — proves SDK installation and managed agent loop
- [ ] `customer_support_agent.py` — multi-tool agent with `PreToolUse` hook enforcing $500 cap
- [ ] `research_pipeline.py` — coordinator spawning two subagents in parallel with provenance preservation
- [ ] `session_fork_demo.py` — baseline session with two forks exploring divergent strategies

Total runtime to build all four: roughly 3 hours if you're not stuck. The code is intentionally minimal so the patterns are visible — production versions would have more error handling, logging, and observability.

## Common Pitfalls

1. **Forgetting `Task` in `allowedTools`** for the coordinator. Subagents won't spawn. (Task 1.3)
2. **Expecting subagents to inherit conversation history**. They don't. You must pass context in the prompt. (Task 1.3, Q34)
3. **Hooks not firing**. Check the `matcher` field — the tool name format is `mcp__<server-name>__<tool-name>` for SDK MCP tools. Built-in tools use their bare names (e.g., `"Bash"`, `"Read"`).
4. **System prompt overriding the hook intent**. If you tell the agent in the prompt "process all refunds," it may try to work around the hook. The hook still blocks, but the agent gets confused. Make the prompt and hook coherent.
5. **Provenance dropping in synthesis**. If your coordinator says "summarize the findings," attribution gets lost. Say "preserve `source_url` and `document_name` on every claim." Explicit beats implicit. (Task 5.6)

## Next Steps

- Update [[cca_progress]] — mark the Domain 1 Exercises 1 and 4 checkboxes complete in the Hands-On Exercises section.
- Re-take [[cca_domain1_practice]] questions on Tasks 1.4 (Q13–Q17) and 1.7 (Q24–Q25). They should feel different now that you've built the patterns.
- Note any sticking points in [[cca_progress#Session notes]] for follow-up.

---

*Maps to Priority 1 of [[cca_docs_roadmap]]. Domain 1 weighting: 27% of the exam — the largest single domain.*
