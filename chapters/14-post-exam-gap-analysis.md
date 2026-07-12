---
title: Post-Exam Gap Analysis
tags: [claude, certification, exam-review, certification, #ai]
---

# Chapter 14: Post-Exam Gap Analysis

> Source: Weston's actual CCAR-F score report (passed 804/720, July 9 2026) plus recalled exam questions, debriefed the night before and morning after.

---

## Table of Contents

- [Score Summary](#score-summary)
- [0% Objectives - Real Gaps](#0-objectives---real-gaps)
- [50% Objectives - Partial Gaps](#50-objectives---partial-gaps)
- [Trap Pattern: Skill vs Subagent](#trap-pattern-skill-vs-subagent)
- [Trap Pattern: Command/Skill Vocabulary](#trap-pattern-commandskill-vocabulary)
- [Trap Pattern: True-but-Irrelevant Constraint](#trap-pattern-true-but-irrelevant-constraint)
- [Trap Pattern: Resume vs Restart vs Targeted Re-analysis](#trap-pattern-resume-vs-restart-vs-targeted-re-analysis)
- [Trap Pattern: Concrete vs Vague Edge Cases](#trap-pattern-concrete-vs-vague-edge-cases)
- [Two Different "Avoid Bloat" Axes](#two-different-avoid-bloat-axes)
- [Scenario Coverage Log](#scenario-coverage-log)

---

## Score Summary

**Result: 804/1000 (Pass, threshold 720).** Exam code CCAR-F, proctored, July 9 2026.

Strong across nearly all of Domain 1 (Agent architecture), Domain 4 (Prompt engineering/structured output), and most of Domain 5 (Context/reliability). Weak spots clustered almost entirely in two places: **Claude Code configuration mechanism selection** and **long-horizon reliability/feedback design** — both under-covered in Chapters 1-13 at exam time.

---

## 0% Objectives - Real Gaps

### 1. Orchestration-layer safeguards guaranteeing resolution-or-escalation

> "Design orchestration-layer safeguards ensuring every agent session ends with a completed resolution or human escalation, regardless of how the agentic loop terminates."

Not just knowing `stop_reason == "end_turn"` ([03 - Agent SDK](03-agent-sdk.md)). This is the layer *above* the loop: a supervisory wrapper that catches every exit path (crash, `max_tokens`, unexpected `stop_reason`, dead subagent) and forces exactly one of two outcomes: **resolved, or escalated — never a silent dead end.** Think `finally` block around the whole agentic loop.

```
Agentic loop runs
  ├─ end_turn, task genuinely done       → resolved
  ├─ end_turn, but unresolved            → must still escalate, not just stop
  ├─ max_tokens / truncated              → escalate, don't silently return partial
  ├─ crash / subagent death              → escalate, don't drop silently
  └─ any other unexpected termination    → default path is ALWAYS escalate
```

### 2 & 3. Choosing among 5 Claude Code config mechanisms (settings permissions was missing)

> "Distinguish instructions that must be enforced through settings permissions or hooks from those appropriately placed in CLAUDE.md..."
> "Select the correct Claude Code configuration mechanism—CLAUDE.md, .claude/rules/ with glob patterns, Skills, hooks, or settings permissions—based on guidance type and when it should apply."

Chapters 1-13 covered 4 of 5 mechanisms. **Settings permissions (`.claude/settings.json` allow/deny lists)** is a 5th, distinct from hooks:

```
CLAUDE.md            → general guidance, probabilistic (~90%), always-loaded
.claude/rules/        → path-scoped guidance, probabilistic, conditionally loaded
Skills                → on-demand task procedures, invoked via /command
Hooks                 → deterministic code execution AT a lifecycle point
                         (lets the call happen, then intercepts/transforms it)
Settings permissions  → deterministic ALLOW/DENY gate on whether a tool/command
                         call is even POSSIBLE (blocks it before it happens)
```

**The hook vs. permission distinction that matters:** hooks react to an attempted call (PreToolUse can redirect it, PostToolUse can transform the result). Settings permissions prevent the call from being an option at all — e.g. denying `Bash(rm -rf *)` outright, or restricting which MCP tools are even available in a given project. If the rule is "never allow X, full stop," that's a permissions problem. If the rule is "when X happens, do Y instead," that's a hook.

### 4. Structured feedback loops for long-horizon improvement

> "Design feedback loop mechanisms that capture structured metadata about model errors and use those patterns to improve prompts, schemas, or few-shot examples in future iterations."

Different from single-interaction retry-with-feedback ([06 - Prompt Engineering](06-prompt-engineering.md) 6.5). This is about production-scale iteration:

```
1. Run the system against real/test inputs
2. Capture STRUCTURED metadata on failures — not "it broke," but
   WHICH field, WHICH document type/format, WHAT kind of error
   (fabrication? wrong format? missed edge case?)
3. Group by pattern — e.g. "23% of failures are missing-total invoices"
4. For each pattern, add a TARGETED fix:
   - a concrete few-shot example (6.1)
   - an explicit rule (6.2)
   - a schema change (nullable field, "other"/"unclear" enum) (2.4)
5. Re-test, measure whether that specific failure pattern dropped
6. Repeat — durable, compounding improvement, not a one-time prompt tweak
```

Anti-pattern: patching individual bad outputs conversationally ("loop back and fix the things") — helps once, doesn't prevent recurrence, isn't reproducible.

### 5. tool_choice sequencing across multi-tool prerequisite workflows

> "Configure the tool_choice parameter to guarantee tool invocation when structured output is required, and sequence multi-tool workflows so prerequisite data is obtained before dependent tools are called."

The "forced-then-auto" pattern (see [02 - Tools and tool_use](02-tools-and-tool_use.md) 2.3) extended across **multiple sequential prerequisites**, not just one forced call followed by auto:

```
Call 1: tool_choice = {"type": "tool", "name": "get_customer"}   (forced)
Call 2: tool_choice = {"type": "tool", "name": "lookup_order"}   (forced,
         only reachable once get_customer succeeded)
Call 3+: tool_choice = {"type": "auto"}                          (free choice
         among process_refund / escalate_to_human / etc. now that
         prerequisites are satisfied)
```

Key exam trap: `{"type": "any"}` does NOT guarantee order — it only guarantees *some* tool gets called, any tool, in any order. Only `{"type": "tool", "name": "X"}` guarantees the specific prerequisite happens before dependent tools are even offered as choices.

---

## 50% Objectives - Partial Gaps

- **Iterative refinement workflows** (concrete I/O examples + targeted failure feedback + **batched issue descriptions for consolidated evaluation**) — the "batched" part is a specific sub-pattern not explicit in Ch 6: collect multiple related failures, then do ONE consolidated prompt/schema update addressing all of them, rather than patching one at a time.
- **Why conversation history must be explicit in every API request** — conceptually solid ([01 - Claude API Fundamentals](01-claude-api-fundamentals.md) 1.2), but likely lost points on precise mechanism naming: it's specifically the **client's responsibility** to resend the full `messages[]` array every call; the API is stateless server-side, nothing persists automatically.
- **Tool description disambiguation for semantically similar tools** — direct overlap with [02 - Tools and tool_use](02-tools-and-tool_use.md) 2.2, but the exam wanted more explicit "use X not Y when..." disambiguation guidance than captured, not just "avoid overlapping descriptions."

---

## Trap Pattern: Skill vs Subagent

These are DIFFERENT PRODUCTS, not interchangeable ideas, even though they both mean "delegate a repeatable task to something callable":

| | Skills | Subagents |
|---|---|---|
| Product | Claude Code | Agent SDK |
| Invocation | `/command` | `Task` tool, `AgentDefinition` |
| Use case | On-demand CLI/dev workflows a team runs | Programmatic multi-agent systems |
| Domain weight | Domain 3 (Claude Code config, 20%) | Domain 1 (Agent architecture, 27%) |

**Read the scenario framing to tell which is being tested:** "your team needs to run this repeatable process from the CLI" → Skill. "Build an autonomous system where a coordinator delegates to specialized agents" → Subagent.

## Trap Pattern: Command/Skill Vocabulary

"Command" and "skill" are used interchangeably in casual/exam language for "a slash-invoked custom action." Don't auto-eliminate an answer for using the word "command" — `.claude/commands/` (legacy) and `.claude/skills/` (current) are both triggered by typing `/something`. Read the *behavior described* (project-level + VCS-shared = team-wide; frontmatter config like `context: fork`/`allowed-tools` = Skills-specific) rather than pattern-matching the noun.

## Trap Pattern: True-but-Irrelevant Constraint

Some questions dangle a true fact that isn't the actual constraint governing the decision. Example: "8,000 lines fits comfortably in the context window" is true, but irrelevant if the real problem is attention dilution from loading mostly-irrelevant business logic alongside the relevant cache functions. The fix is targeted Grep-then-Read of relevant sections, not "load it all since it fits." **Always ask: does this fact answer the actual question, or is it just true?**

## Trap Pattern: Resume vs Restart vs Targeted Re-analysis

Recurring scenario type (2+ variations seen): a session is interrupted/mixed-purpose/time has passed, and some but not all of the prior work is still valid.

```
Session interrupted / resumed after time passed / mixed with other work
         │
         ├─ Files/methods UNCHANGED  → trust prior findings, don't re-read
         │
         └─ Files/methods CHANGED    → explicitly flag to the agent,
                                        targeted re-read/re-analysis
                                        of ONLY those, inject as context

NOT: full restart from scratch (wastes valid prior work)
NOT: silent resume/--resume without flagging changes (stale analysis
     on changed parts goes undetected — worse than obviously wrong)
```

Applies whether addressing the session by `--resume <name>` or `--session_id <uuid>` — the flag choice doesn't matter; what matters is whether you (a) verify what changed and (b) inject a condensed relevant summary rather than trusting either full raw history or a blind full restart.

## Trap Pattern: Concrete vs Vague Edge Cases

**Generic edge-case language is always the wrong answer. Concrete edge-case handling is always the right answer.** These can look similar in an answer choice ("update the prompt to handle edge cases") but are opposite in quality:

```
WRONG (vague, matches Ch 6.2 anti-pattern):
  "Handle edge cases appropriately."
  "Be conservative - report only high-confidence findings."

RIGHT (concrete, named, actionable):
  "If total_amount is not stated anywhere, set it to null rather
   than estimating from line items."
  "If invoice_date has no year, use the year of the earliest dated
   document in the same batch."
```

Don't reflexively reject an answer choice for mentioning "edge cases" — check whether it's vague or specific before deciding. This mirrors the command/skill vocabulary trap: read behavior/specificity, not the trigger word.

## Two Different "Avoid Bloat" Axes

A general engineering instinct ("keep things lean/DRY") transfers correctly to one axis and incorrectly to the other:

```
AXIS 1: Conversation/context bloat (CORRECT to minimize)
─────────────────────────────────────────────────────────
Tool RESULTS accumulating across turns, raw subagent dumps,
unbounded conversation history.
→ Fix: PostToolUse trimming hooks, scratchpad files, subagent
  delegation, targeted reads. ([11 - Context Management](11-context-management.md))
→ This instinct is RIGHT.

AXIS 2: Prompt/schema/description completeness (WRONG to minimize)
─────────────────────────────────────────────────────────
Few-shot examples, edge-case handling instructions, format
normalization rules, tool-description disambiguation detail.
→ Fix: ADD MORE concrete examples — this is static, reused-every-
  call content, not accumulating runtime noise.
→ This instinct is WRONG here; it cost points on 3+ objectives.
```

A prompt/tool description is documentation the model reads once per call, not code with runtime overhead. Longer + more example-rich is usually correct when the actual bug is ambiguity, not a cost/bloat problem.

---

## Scenario Coverage Log

Of the 6 possible exam scenarios (4 randomly drawn per real exam), this sitting's actual exam covered:

| # | Scenario | Appeared on real exam |
|---|---|---|
| 1 | Customer Support Agent | Yes |
| 2 | Code Generation with Claude Code | Yes |
| 3 | Multi-Agent Research System | No |
| 4 | Developer Productivity Tools | Yes |
| 5 | Claude Code for CI/CD | No |
| 6 | Structured Data Extraction | Yes |

Next candidate should weight extra prep toward #3 and #5 since they're unconfirmed against a real sitting, while still covering all 6 (selection is random per attempt).

---

Tags: #claude #certification #exam-review #ai
