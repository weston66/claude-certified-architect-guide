---
title: Error Handling in Multi-Agent Systems
tags: [claude, agent-sdk, error-handling, certification, #ai]
---

# Chapter 10: Error Handling in Multi-agent Systems

---

## 10.1 Error Categories

| Category | Examples | Retryable | Agent action |
|---|---|---|---|
| **Transient** | Timeout, 503, network failure | Yes | Retry with exponential backoff |
| **Validation** | Invalid input format, missing required field | No (fix input) | Modify request and retry |
| **Business** | Policy violation, threshold exceeded | No | Explain to user; propose an alternative |
| **Permission** | Access denied | No | Escalate |

---

## 10.2 Error-handling Anti-patterns

| Anti-pattern | Problem | Correct approach |
|---|---|---|
| Generic status "search unavailable" | Coordinator can't decide how to recover | Return error type, query, partial results, alternatives |
| Silent suppression (empty result = success) | Coordinator thinks there were no matches, but it was a failure | Distinguish "no results" from "search failure" |
| Aborting the whole workflow on one failure | You lose all partial results | Continue with partial results; annotate gaps |
| Infinite retries inside a subagent | Latency and wasted resources | Local recovery (1-2 retries), then propagate to coordinator |

---

## 10.3 A Structured Subagent Error

```json
{
  "status": "partial_failure",
  "failure_type": "timeout",
  "attempted_query": "AI impact on music industry 2024",
  "partial_results": [
    {"title": "AI Music Generation Report", "url": "...", "relevance": 0.8}
  ],
  "alternative_approaches": [
    "Try a narrower query: 'AI music composition tools'",
    "Use an alternative data source"
  ],
  "coverage_impact": "Not covered: AI impact on music production"
}
```

This provides the coordinator with the information needed to decide: retry with a modified query, use partial results, delegate to a different subagent, or continue with a gap annotation.

---

## 10.4 Coverage Annotations in the Final Synthesis

```markdown
### Visual Art (FULL COVERAGE)
[research results]

### Music (PARTIAL COVERAGE - search agent timeout)
[partial results]
Note: coverage for this section is limited due to a timeout in the search agent.
```
