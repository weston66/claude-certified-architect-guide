---
title: Escalation and Human-in-the-Loop
tags: [claude, agent-sdk, escalation, certification, #ai]
---

# Chapter 9: Escalation and Human-in-the-Loop

---

## 9.1 When to Escalate to a Human

**Escalation triggers (clear rules):**

| Situation | Action |
|---|---|
| Customer explicitly asks "get me a manager" | Escalate immediately; do not attempt to solve |
| Policy does not cover the request | Escalate |
| Agent cannot make progress | Escalate after a reasonable number of attempts |
| Financial operation above a threshold | Escalate (preferably via hook, not prompt) |
| Multiple matches when searching for a customer | Ask for additional identifiers; do not guess |

**What is NOT a reliable trigger:**

| Unreliable method | Why it fails |
|---|---|
| Sentiment analysis | Customer mood does not correlate with case complexity |
| Model self-rated confidence (1-10) | The model can be confidently wrong; calibration is poor |
| An automatic classifier | May require training data you don't have |

---

## 9.2 Escalation Patterns

**Immediate escalation:**
```
Customer: "I want to speak to a manager"
Agent: [immediately calls escalate_to_human]
NOT: "I can help with your issue, let me..."
```

**Nuanced escalation (acknowledge -> resolve -> escalate on reiteration):**
```
Customer: "This is outrageous, I'm very unhappy!"
Agent: "I understand your frustration." [offers replacement or refund]
Customer: "No, I want to talk to someone!"
Agent: [customer insists again -> immediate escalation]
```

Key principle: acknowledge emotion first, then propose a concrete solution, and only escalate if the customer reiterates. Do not escalate on the first expression of dissatisfaction.

---

## 9.3 Structured Handoff Protocols

On escalation, the agent passes a structured summary:

```json
{
  "customer_id": "CUST-12345",
  "customer_name": "Ivan Petrov",
  "issue_summary": "Refund request for a damaged item",
  "order_id": "ORD-67890",
  "actions_taken": [
    "Verified customer via get_customer",
    "Confirmed order via lookup_order",
    "Offered standard replacement - customer insists on refund"
  ],
  "refund_amount": "$89.99",
  "recommended_action": "Approve a full refund",
  "escalation_reason": "Customer requested to speak with a manager"
}
```

The human operator only sees this summary - it must be complete and self-contained.

---

## 9.4 Confidence Calibration and Human Oversight

For data extraction systems:

1. **Field-level confidence scores:** the model outputs a confidence score per extracted field
2. **Calibration:** use labeled validation sets to tune thresholds
3. **Routing:**
   - High confidence + stable accuracy -> automated processing
   - Low confidence or ambiguous sources -> human review

**Stratified random sampling:** even for high-confidence extractions, regularly audit a sample. An aggregate 97% accuracy can hide 40% errors for a particular document type.
