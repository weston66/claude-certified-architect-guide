---
title: Claude API Fundamentals
tags: [claude, api, certification, #ai]
---

# Chapter 1: Claude API - Fundamentals of Model Interaction

---

## 1.1 API Request Structure

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {"role": "user", "content": "Hi!"},
    {"role": "assistant", "content": "Hello!"},
    {"role": "user", "content": "How are you?"}
  ],
  "tools": [...],
  "tool_choice": {"type": "auto"}
}
```

**Key fields:**
- `model` - model selection (`claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5`)
- `max_tokens` - maximum tokens in the response
- `system` - system prompt (defines model behavior)
- `messages` - conversation history (**must send full history** to maintain coherence)
- `tools` - definitions of available tools
- `tool_choice` - tool selection strategy

---

## 1.2 Message Roles

- `user` - user messages
- `assistant` - model responses (included when sending history)
- `tool` - tool call results (appears as a `tool_result` content block)

**Critical:** on every API request you must send the **full conversation history**. The model does not persist state between requests.

---

## 1.3 The `stop_reason` Field

| Value | Description | Action |
|---|---|---|
| `"end_turn"` | Model finished its response | Show result to user |
| `"tool_use"` | Model wants to call a tool | Execute the tool and return the result |
| `"max_tokens"` | Token limit reached | Response is truncated; may need to increase limit |
| `"stop_sequence"` | A stop sequence was encountered | Handle based on application logic |

For agentic systems, `"tool_use"` and `"end_turn"` are the most important.

---

## 1.4 System Prompt

- Passed separately in the `system` field, not part of `messages`
- Has priority over user messages
- Loaded once, applies throughout the conversation
- Defines role, constraints, output format

**Exam note:** system prompt wording can create unintended tool associations. "Always verify the customer" can cause overuse of `get_customer` even when unnecessary.

---

## 1.5 Context Window

The total amount of text (in tokens) the model can process at once. Includes:
- System prompt
- Full message history
- Tool definitions
- Tool results

**Key problems:**

1. **Lost-in-the-middle effect** - models reliably process info at the start and end of long input but can miss details in the middle. Mitigation: place key info near the beginning or end.

2. **Accumulation of tool results** - if a tool returns 40+ fields but only 5 matter, most context is wasted.

3. **Progressive summarization** - when compressing history, numeric values, percentages, and dates often get lost and become vague.
