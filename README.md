<img src="assets/badge.png" alt="Claude Certified Architect Foundations badge" width="160">

# Claude Certified Architect — Study Guide

Study notes, a real post-exam gap analysis, and a seed prompt for training a Claude instance to coach someone through the **Claude Certified Architect Foundations** exam (exam code CCAR-F).

I passed this exam on July 9, 2026 (804/1000, passing threshold 720). This repo is the material I actually used to prep, refined afterward with what the real exam exposed that my prep missed. Related blog post: *(link TBD)*.

## What's Here

- **`chapters/`** — 16 chapters covering the exam's 5 domains: Claude API fundamentals, tools/`tool_use`, Agent SDK, MCP, Claude Code configuration, prompt engineering, Message Batches API, task decomposition, escalation/human-in-the-loop, error handling, context management, provenance, Claude Code built-in tools, plus two chapters written after the exam:
  - **Chapter 14** — a gap analysis built directly from my real score report: the 5 objectives I scored 0% on, 3 at 50%, and 8 named "trap patterns" the real exam uses that generic study material won't prepare you for.
  - **Chapter 15** — a style guide for *writing practice questions that actually match the real exam's difficulty*. My first round of self-written practice questions scored ~87% because the wrong answers were too obviously wrong. The real exam's wrong answers are plausible, not strawmen — this chapter documents the 7 techniques the real exam seems to use and how to replicate them.

- **`SEED.md`** — a prompt you can paste into a fresh Claude conversation to turn it into an exam coach that drills you using these chapters, calibrated against the same 7 question-design rules and the 8 confirmed trap patterns.

- **`LICENSE`** — MIT.

## How to Use This

**Studying solo:** read `chapters/00-overview.md` first for exam format and domain weights, then work through 1-13 for foundational content, then 14 for the specific traps to watch for.

**Getting drilled:** paste `SEED.md` into a new Claude conversation (Claude Code, claude.ai, or via API) along with access to the `chapters/` directory. It will run you through scenario-based multiple-choice questions weighted by domain, in the same style as the real exam, not a softened version of it.

**Contributing gaps back:** if you take the exam using this material and find new traps, patterns, or objectives it didn't prepare you for, add them to `chapters/14-post-exam-gap-analysis.md` and open a PR. The value of this repo compounds with more real exam data points, not more generic content.

## Exam Format (from the Overview chapter)

| Parameter | Value |
|---|---|
| Question type | Multiple choice (1 correct of 4) |
| Scoring | 100-1000 scale, passing score 720 |
| Guessing penalty | None |
| Scenarios | 4 of 6 possible, randomly selected |

**5 Domains:** Agent architecture and orchestration (27%), Claude Code configuration and workflows (20%), Prompt engineering and structured output (20%), Tool design and MCP integration (18%), Context management and reliability (15%).

**6 Possible Scenarios:** Customer Support Agent, Code Generation with Claude Code, Multi-Agent Research System, Developer Productivity Tools, Claude Code for CI/CD, Structured Data Extraction.

## Disclaimer

This is unofficial, community-built study material based on one person's exam experience. It is not affiliated with or endorsed by Anthropic. Exam content, format, and objectives may change over time — treat the specifics here (especially Chapter 14's gap list) as a snapshot from a single sitting, not a guarantee of what you'll see.
