---
title: Claude Certified Architect - Foundations Overview
tags: [claude, certification, ai, #ai]
---

# Claude Certified Architect - Foundations

> Source: https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.MD

Confirms a specialist can make sound trade-off decisions when implementing real-world Claude-based solutions.

---

## Target Candidate

Solution architect who designs and ships production applications with Claude. Should have 6+ months hands-on with:

- **Claude Agent SDK** - multi-agent orchestration, delegating to subagents, tool integration, lifecycle hooks
- **Claude Code** - CLAUDE.md, MCP servers, Agent Skills, planning mode
- **Model Context Protocol (MCP)** - tools and resources for backend integration
- **Prompt engineering** - JSON schemas, few-shot examples, data extraction templates
- **Context windows** - working with long documents, multi-agent context passing
- **CI/CD pipelines** - automated code review, test generation
- **Escalation and reliability** - error handling, human-in-the-loop

---

## Exam Format

| Parameter | Value |
|---|---|
| Question type | Multiple choice (1 correct out of 4) |
| Scoring | 100-1000 scale, passing score **720** |
| Guessing penalty | None (answer every question!) |
| Scenarios | 4 out of 6 possible (randomly selected) |

---

## 5 Exam Domains

| Domain | Weight |
|---|---|
| 1. Agent architecture and orchestration | **27%** |
| 2. Tool design and MCP integration | **18%** |
| 3. Claude Code configuration and workflows | **20%** |
| 4. Prompt engineering and structured output | **20%** |
| 5. Context management and reliability | **15%** |

---

## 6 Exam Scenarios

1. **Customer Support Agent** - returns, billing disputes, account issues using Agent SDK + MCP tools
2. **Code Generation with Claude Code** - custom slash commands, CLAUDE.md config, planning mode
3. **Multi-Agent Research System** - coordinator + specialized subagents for web research, doc analysis, synthesis
4. **Developer Productivity Tools** - codebase exploration, boilerplate generation, built-in tools + MCP
5. **Claude Code for CI/CD** - automated code reviews, test generation, PR feedback
6. **Structured Data Extraction** - JSON schemas, validation, edge case handling

---

## Chapter Index

| Chapter | Topic |
|---|---|
| [01 - Claude API Fundamentals](01-claude-api-fundamentals.md) | API structure, message roles, stop_reason, context window |
| [02 - Tools and tool_use](02-tools-and-tool_use.md) | Tool definitions, tool_choice, JSON schemas, error types |
| [03 - Agent SDK](03-agent-sdk.md) | Agentic loop, AgentDefinition, hub-and-spoke, hooks, sessions |
| [04 - MCP](04-mcp.md) | MCP servers, config, isError flag, resources |
| [05 - Claude Code Configuration](05-claude-code-configuration.md) | CLAUDE.md hierarchy, @path, rules dir, slash commands, skills |
| [06 - Prompt Engineering](06-prompt-engineering.md) | Few-shot, explicit criteria, chaining, interview pattern, retry |
| [07 - Message Batches API](07-message-batches-api.md) | Async processing, 50% savings, custom_id, failure handling |
| [08 - Task Decomposition](08-task-decomposition.md) | Fixed pipelines vs dynamic decomposition, multi-pass review |
| [09 - Escalation and Human-in-the-Loop](09-escalation-and-human-in-the-loop.md) | Escalation triggers, patterns, structured handoff |
| [10 - Error Handling](10-error-handling.md) | Error categories, anti-patterns, structured subagent errors |
| [11 - Context Management](11-context-management.md) | Fact extraction, trimming, position-aware input, scratchpad files |
| [12 - Preserving Provenance](12-preserving-provenance.md) | Attribution loss, conflicting data, dates |
| [13 - Claude Code Built-in Tools](13-claude-code-built-in-tools.md) | Tool selection reference, incremental investigation |
| [14 - Post-Exam Gap Analysis](14-post-exam-gap-analysis.md) | Real score report gaps, trap patterns, scenario coverage log |
| [15 - Exam Question Design Style Guide](15-exam-question-design-style-guide.md) | How to write practice questions that match real exam difficulty |
