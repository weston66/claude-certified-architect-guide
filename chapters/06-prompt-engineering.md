---
title: Prompt Engineering - Advanced Techniques
tags: [claude, prompt-engineering, few-shot, certification, #ai]
---

# Chapter 6: Prompt Engineering - Advanced Techniques

---

## 6.1 Few-shot Prompting

Include 2-4 input/output examples in a prompt to demonstrate expected behavior.

**Why more effective than textual descriptions:** a vague instruction like "be more precise" can be interpreted many ways. An example unambiguously shows the expected format and decision logic.

**Types of few-shot examples:**

1. **For ambiguous scenarios:**
```
Request: "My order is broken"
Action: Call get_customer -> lookup_order -> check status.

Request: "Get me a manager"
Action: Immediately call escalate_to_human.
Rationale: Customer explicitly requests a human. Do not attempt to solve autonomously.
```

2. **For output formatting** - show the exact JSON structure you expect.

3. **For acceptable vs problematic code** - show a passing example and a flagged example.

4. **For informal measurements:**
```
Text: "about two handfuls of rice"
-> {"amount": "~100g", "original_text": "two handfuls", "precision": "approximate"}
```

**Format normalization rules in prompts:**
```
Normalization:
- Dates: always ISO 8601 (YYYY-MM-DD); "yesterday" -> compute an absolute date
- Currency: numeric amount + currency code; "five bucks" -> {"amount": 5, "currency": "USD"}
- Percentages: decimal fraction; "half" -> 0.5
```

---

## 6.2 Explicit Criteria vs Vague Instructions

**Bad (vague):**
```
Check code comments for accuracy.
Be conservative - report only high-confidence findings.
```

**Good (explicit criteria):**
```
Flag a comment as problematic ONLY if:
1. The comment describes behavior that CONTRADICTS the actual code behavior
2. The comment references a non-existent function or variable
3. A TODO/FIXME comment refers to a bug already fixed in code

Do NOT flag:
- Comments that are merely stylistically outdated
- Missing comments (that is a separate category)
```

**Define severity criteria with examples:**
```
CRITICAL: Runtime failure for users
  Example: NullPointerException while processing a payment

HIGH: Security vulnerability
  Example: SQL injection, XSS, missing authorization checks
```

---

## 6.3 Prompt Chaining

Break a complex task into a sequence of focused steps:

```
Step 1: Analyze auth.ts (local issues only)
       -> Output: list of issues in auth.ts

Step 2: Analyze database.ts (local issues only)
       -> Output: list of issues in database.ts

Step 3: Integration pass (cross-file dependencies)
       -> Output: issues at module boundaries
```

**Why this matters:** avoids **attention dilution** - when the model receives too many files at once, it may miss bugs in some while providing shallow commentary on others.

**Chaining vs dynamic decomposition:**
- **Prompt chaining** - predictable, repeatable tasks (code review, file migrations)
- **Dynamic decomposition** - open-ended investigations where subtasks become clear only during execution

---

## 6.4 The "Interview" Pattern

Before implementing a solution, Claude asks clarifying questions:

```
Claude: "Before implementing caching for the API, a few questions:
1. Which cache invalidation strategy do you prefer - TTL or event-based?
2. Is stale data acceptable when the cache is unavailable?
3. Should caching be per-user or global?"
```

**When useful:**
- Unfamiliar domain (fintech, healthcare, legal systems)
- Tasks with non-obvious implications (cache strategies, failure modes)
- Multiple viable approaches where the best choice depends on context

---

## 6.5 Validation and Retry-with-Feedback

```
Step 1: Extract data from the document
Step 2: Validate (Pydantic, JSON Schema, business rules)
Step 3: If error - retry with context:
  - The original document
  - The previous (incorrect) extraction
  - The specific error: "Field 'total' = 150, but sum(line_items) = 145."
```

**When retry will be effective:**
- Format errors (date in the wrong format)
- Structural errors (field placed in the wrong location)
- Arithmetic inconsistencies

**When retry will NOT help:**
- The information is absent from the source document
- The required context is external

**Pydantic as a validation tool:**
- Structural validation: types, requiredness, enum constraints
- Semantic validation: custom validators for business logic (sum of items = total; start_date < end_date)
- JSON Schema generation from Pydantic models for `tool_use`

---

## 6.6 Self-correction

Pattern for detecting internal contradictions - the model extracts both the stated value and a computed value:

```json
{
  "stated_total": "$150.00",
  "calculated_total": "$145.00",
  "conflict_detected": true,
  "line_items": [
    {"name": "Widget A", "price": 75.00},
    {"name": "Widget B", "price": 70.00}
  ]
}
```

If they differ, `conflict_detected` allows you to handle the discrepancy.
