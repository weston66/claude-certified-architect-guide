---
title: Tools and tool_use
tags: [claude, tools, json-schema, certification, #ai]
---

# Chapter 2: Tools and `tool_use`

---

## 2.1 What is `tool_use`

`tool_use` allows Claude to call external functions. The model does **not** run code directly - it generates a structured tool call request; your code executes it and returns the result.

---

## 2.2 Tool Definition

```json
{
  "name": "get_customer",
  "description": "Finds a customer by email or ID. Returns the customer profile, including name, email, order history, and account status. Use this tool BEFORE lookup_order to verify the customer's identity.",
  "input_schema": {
    "type": "object",
    "properties": {
      "email": {"type": "string", "description": "Customer email"},
      "customer_id": {"type": "integer", "description": "Numeric customer ID"}
    },
    "required": []
  }
}
```

**Critical aspects of a tool description:**

1. **The description is the primary selection mechanism.** Claude chooses tools based on descriptions. Minimal descriptions lead to mistakes when tools overlap.
2. **Include:** what the tool does and returns, input formats and example values, edge cases and constraints, when to use vs similar alternatives.
3. **Avoid** identical or overlapping descriptions across tools.
4. **Built-in vs MCP tools:** agents may prefer built-in tools (Read, Grep) over MCP tools with similar functionality. Strengthen MCP tool descriptions to highlight concrete advantages.

---

## 2.3 The `tool_choice` Parameter

| Value | Behavior | When to use |
|---|---|---|
| `{"type": "auto"}` | Model decides whether to call a tool or answer in text | Default for most cases |
| `{"type": "any"}` | Model **must** call some tool | When you need guaranteed structured output |
| `{"type": "tool", "name": "extract_metadata"}` | Model **must** call a specific tool | When you need a forced first step |

---

## 2.4 JSON Schemas for Structured Output

Using `tool_use` with JSON schemas is the **most reliable** way to obtain structured output. It:
- Guarantees syntactically valid JSON
- Enforces required structure
- Does **not** guarantee semantic correctness (values can still be wrong)

**Schema design rules:**

```json
{
  "type": "object",
  "properties": {
    "category": {
      "type": "string",
      "enum": ["bug", "feature", "docs", "unclear", "other"]
    },
    "category_detail": {
      "type": ["string", "null"],
      "description": "Details if category = 'other' or 'unclear'"
    },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    }
  },
  "required": ["category", "severity"]
}
```

1. **Required vs optional:** mark required only if information is always available. Required fields push the model to fabricate values when data is missing.
2. **Nullable fields:** use `"type": ["string", "null"]` for information that may be absent.
3. **Enums with `"other"`:** add `"other"` + a detail string to avoid losing data outside predefined categories.
4. **Enum `"unclear"`:** for cases where the model cannot confidently pick - honest `"unclear"` is better than a wrong category.

---

## 2.5 Syntax vs Semantic Errors

| Error type | Example | Mitigation |
|---|---|---|
| **Syntax** | Invalid JSON, wrong field type | `tool_use` with a JSON schema (eliminates) |
| **Semantic** | Totals don't add up, wrong field, hallucination | Validation checks, retry with feedback, self-correction |
