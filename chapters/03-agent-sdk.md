---
title: Claude Agent SDK
tags: [claude, agent-sdk, multi-agent, hooks, certification, #ai]
---

# Chapter 3: Claude Agent SDK - Building Agentic Systems

---

## 3.1 What is an Agentic Loop

The core pattern for autonomous task execution:

```
1. Send a request to Claude with tools
2. Receive a response
3. Check stop_reason:
   - "tool_use" -> execute the tool, append result to history, go back to step 1
   - "end_turn" -> task is complete, show result to user
4. Repeat until completion
```

**Anti-patterns (avoid):**
- Parsing assistant text to detect completion ("Task completed")
- Using an arbitrary iteration limit (e.g., `max_iterations=5`) as the primary stop condition
- Checking whether the assistant produced textual content as a completion signal

**Correct approach:** the only reliable completion signal is `stop_reason == "end_turn"`.

---

## 3.2 `AgentDefinition` Configuration

```python
agent = AgentDefinition(
    name="customer_support",
    description="Handles customer requests for returns and order issues",
    system_prompt="You are a customer support agent...",
    allowed_tools=["get_customer", "lookup_order", "process_refund", "escalate_to_human"],
)
```

**Key parameters:**
- `name` / `description` - identification
- `system_prompt` - instructions
- `allowed_tools` - list of allowed tools (principle of least privilege)

---

## 3.3 Hub-and-Spoke: Coordinator and Subagents

```
         Coordinator
        /     |      \
   Subagent1  Subagent2  Subagent3
    (search)   (analysis)   (synthesis)
```

**Coordinator responsibilities:**
- Decomposing the task into subtasks
- Deciding which subagents are needed (dynamic selection)
- Delegating work
- Aggregating and validating results
- Handling errors and retries
- Communicating results to the user

**Critical principle: subagents have isolated context.**
- Subagents do **not** automatically inherit the coordinator's conversation history
- All required context must be **explicitly passed** in the subagent prompt
- All communication flows through the coordinator (for observability and error control)

---

## 3.4 The `Task` Tool for Spawning Subagents

Coordinator's `allowedTools` must include `"Task"`.

```
# Bad: subagent has no context
Task: "Analyze the document"

# Good: full context in the prompt
Task: "Analyze the following document.
Document: [full document text]
Prior search results: [web search results]
Output format requirements: [schema]"
```

**Parallel spawning:** a coordinator can call multiple `Task`s in one response - subagents run in parallel.

---

## 3.5 Hooks in the Agent SDK

Hooks allow interception and transformation at specific points in the agent lifecycle.

**PostToolUse** - intercepts a tool result before it is provided to the model:

```python
@hook("PostToolUse")
def normalize_dates(tool_result):
    # Convert Unix timestamp -> ISO 8601
    return normalized_result
```

**PreToolUse** - intercepts outgoing calls to block policy violations:

```python
@hook("PreToolUse")
def enforce_refund_limit(tool_call):
    if tool_call.name == "process_refund" and tool_call.args.amount > 500:
        return redirect_to_escalation(tool_call)
```

**Hooks vs prompt instructions:**

| Attribute | Hooks | Prompt instructions |
|---|---|---|
| Guarantee | **Deterministic** (100%) | **Probabilistic** (>90%, not 100%) |
| When to use | Critical business rules, financial operations, compliance | General preferences, recommendations, formatting |

**Rule:** when failure has financial, legal, or safety consequences - use hooks, not prompts.

---

## 3.6 `fork_session` and Session Management

**`--resume <session-name>`** resumes a named session:
```bash
claude --resume investigation-auth-bug
```
- Risk: if files changed since the prior session, tool results may be stale

**`fork_session`** creates an independent branch from shared context:
```
Codebase investigation
         |
    fork_session
    /           \
Approach A:      Approach B:
Redux            Context API
```

**When to start a new session instead of resuming:**
- Tool results are stale (files changed)
- A lot of time has passed and context has degraded
- Better to restart with a short summary than resume with old tool data
