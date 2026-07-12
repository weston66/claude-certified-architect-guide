---
title: Task Decomposition Strategies
tags: [claude, agent-sdk, prompt-chaining, certification, #ai]
---

# Chapter 8: Task Decomposition Strategies

---

## 8.1 Fixed Pipelines (Prompt Chaining)

Each step is defined in advance:

```
Document -> Metadata extraction -> Data extraction -> Validation -> Enrichment -> Final output
```

**When to use:**
- The task structure is predictable
- All steps are known up front
- You need stability and reproducibility

---

## 8.2 Dynamic Adaptive Decomposition

Subtasks are generated based on intermediate results:

```
1. "Add tests for a legacy codebase"
2. -> First: map the structure (Glob, Grep)
3. -> Found: 3 modules with no tests, 2 with partial coverage
4. -> Prioritize: start with the payments module (high risk)
5. -> During work: discovered a dependency on an external API
6. -> Adapt: add a mock for the external API before writing tests
```

**When to use:**
- Open-ended investigative tasks
- When the full scope is unknown up front
- When each step depends on the results of the previous step

---

## 8.3 Multi-pass Code Review

For pull requests with 10+ files:

```
Pass 1 (per-file): Analyze auth.ts -> list local issues
Pass 1 (per-file): Analyze database.ts -> list local issues
...
Pass 2 (integration): Analyze relationships between files
  -> Cross-file issues: inconsistent types, circular dependencies
```

**Why a single pass over 14 files is bad:**
- Attention dilution: deep analysis for some files, shallow for others
- Inconsistent comments: a pattern flagged in one file but approved in another
- Missed bugs: obvious errors get skipped due to cognitive overload
