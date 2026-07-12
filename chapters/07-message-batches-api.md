---
title: Message Batches API
tags: [claude, api, batches, certification, #ai]
---

# Chapter 7: Message Batches API

---

## 7.1 Overview

| Attribute | Value |
|---|---|
| Savings | **50%** compared to synchronous calls |
| Processing window | Up to **24 hours** (no latency SLA guarantee) |
| Multi-turn tool calling | **Not supported** (one request = one response) |
| Correlation | `custom_id` field to link request and response |

---

## 7.2 When to Use Batch API vs Synchronous API

| Task | API | Why |
|---|---|---|
| Pre-merge PR check | **Synchronous** | Developer is waiting; 24 hours is unacceptable |
| Overnight tech-debt report | **Batch** | Result needed by morning; 50% savings |
| Weekly security audit | **Batch** | Not urgent; 50% savings |
| Interactive code review | **Synchronous** | Immediate response required |
| Processing 10,000 documents | **Batch** | Bulk processing; savings are significant |

---

## 7.3 Using `custom_id`

```json
{
  "custom_id": "doc-invoice-2024-001",
  "params": {
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Extract data from: ..."}]
  }
}
```

`custom_id` allows you to:
- Link the result to the original document
- On failure, re-submit only the failed documents
- Avoid re-processing successful documents

---

## 7.4 Handling Failures in Batches

1. Submit a batch of 100 documents
2. 95 succeed; 5 fail (context limit exceeded)
3. Identify failures by `custom_id`
4. Modify strategy (e.g., split long documents into chunks)
5. Re-submit only the 5 failed documents

---

## 7.5 SLA Planning

If you need a result in 30 hours and Batch API can take up to 24 hours:
- Submission window: 30 - 24 = **6 hours**
- Batches must be submitted no later than 24 hours before the deadline
