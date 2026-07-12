---
title: Exam Question Design Style Guide
tags: [claude, certification, exam-review, question-design, #ai]
---

# Chapter 15: Exam Question Design Style Guide

> Purpose: how to WRITE practice questions that actually match the real CCAR-F exam's difficulty, based on Weston's post-exam debrief (passed 804/720, July 9 2026). Use this when building a trainer tool or writing new practice questions for teammates. See [14 - Post-Exam Gap Analysis](14-post-exam-gap-analysis.md) for the content gaps this format is meant to expose.

---

## Table of Contents

- [The Core Problem With Naive Practice Questions](#the-core-problem-with-naive-practice-questions)
- [Format Spec](#format-spec)
- [Rule 1: No Obvious Anti-Pattern Tell](#rule-1-no-obvious-anti-pattern-tell)
- [Rule 2: Bury a True-but-Irrelevant Fact](#rule-2-bury-a-true-but-irrelevant-fact)
- [Rule 3: Vocabulary Overlap Traps](#rule-3-vocabulary-overlap-traps)
- [Rule 4: Product/Domain Confusion Traps](#rule-4-productdomain-confusion-traps)
- [Rule 5: Omit the Textbook Mechanism](#rule-5-omit-the-textbook-mechanism)
- [Rule 6: Require Composing Two Techniques](#rule-6-require-composing-two-techniques)
- [Rule 7: Specificity Discrimination, Not Keyword Presence](#rule-7-specificity-discrimination-not-keyword-presence)
- [Writing Checklist](#writing-checklist)
- [Worked Example](#worked-example)

---

## The Core Problem With Naive Practice Questions

The first round of practice questions built for this exam (see the drilling sessions before [14 - Post-Exam Gap Analysis](14-post-exam-gap-analysis.md)) scored ~87% because they had a structural flaw: **one answer was written to sound like a documented anti-pattern, and that answer was always wrong.** Real test-takers pattern-match on "does this sound like the bad example from the notes" rather than reasoning about the scenario. This inflates practice scores without building real discrimination skill.

Weston's actual exam (804/720, comfortable pass) still surfaced real gaps precisely because it did NOT telegraph the wrong answer this way. The gaps that scored 0-50% were concepts he could articulate correctly when asked directly in conversation, but couldn't reliably pick out under 4-option MC pressure — because the real exam's wrong answers are *plausible*, not *obviously bad*.

**Design goal: every wrong answer should be something a competent, half-attentive candidate could talk themselves into.**

---

## Format Spec

- Single-select, 1 correct of 4 options (matches real exam: no multi-select, no partial credit, no penalty for guessing)
- Scenario-framed stem, not a bare definitional question ("What is X?" tests recall; "Given this situation, what do you do?" tests application — the real exam is almost entirely the latter)
- No answer choice should contain a dead giveaway phrase lifted verbatim from a documented anti-pattern list

---

## Rule 1: No Obvious Anti-Pattern Tell

**Don't do this:**
> A) Use an arbitrary iteration limit as the completion signal (WRONG — obviously matches "anti-pattern" language from notes)
> B) Check `stop_reason == "end_turn"` (correct)

**Do this instead** — make the wrong answer sound like a reasonable engineering choice, not a strawman:
> A) Check `stop_reason == "end_turn"`
> B) Treat the appearance of any text content in the response as completion, since a text response usually means the model is done reasoning
> C) Check for `stop_reason` being anything other than `"tool_use"`
> D) Poll a `is_complete` field the model is instructed to set in its final message

(B sounds plausible — text often does appear at completion. C is close to correct but wrong because `stop_reason` values other than `tool_use` include things like `max_tokens`, which is NOT completion. D relies on model instruction-following for a structural signal, which is the deeper conceptual error — same category as "prompts are probabilistic, not deterministic.")

---

## Rule 2: Bury a True-but-Irrelevant Fact

State a real, verifiable fact in the stem or in an answer choice that does NOT resolve the actual question — bait test-takers into anchoring on it.

**Example pattern (used successfully in drilling):**
> "Token budget isn't a concern — 8,000 lines fits comfortably in the context window."

This fact is true and it's tempting to conclude "so just load it all." The actual governing constraint is attention dilution from irrelevant content, not token capacity. Good questions in this style explicitly state that the tempting-but-wrong constraint (cost, token count, time) is NOT the bottleneck, forcing the candidate to identify the REAL bottleneck (signal-to-noise, attention dilution, staleness, semantic correctness vs syntactic validity).

**Where to source these:** any place in the chapters where two constraints could plausibly be confused — e.g. "does it fit" vs "does loading it help," "is it valid JSON" vs "is it factually correct," "is the model confident" vs "is the model calibrated."

---

## Rule 3: Vocabulary Overlap Traps

Real-world terms get used loosely even by exam writers. Don't assume a keyword maps 1:1 to a specific mechanism.

**Confirmed instance:** "slash command" is used generically for BOTH the legacy `.claude/commands/` format and the current `.claude/skills/` format. A question describing "create a `/api-review` command the team can invoke" is not automatically describing the legacy/wrong mechanism — read the behavioral details (project-level? VCS-shared? frontmatter config like `context: fork`?) to determine if it's actually describing a Skill.

**Design technique:** use the colloquial term ("command," "hook," "rule") in the stem, then differentiate the 4 answers purely by the SPECIFIC MECHANISM/BEHAVIOR each describes, not by which noun they use. Forces candidates to read past the label.

---

## Rule 4: Product/Domain Confusion Traps

Claude has multiple products with conceptually similar features that live in different places and different exam domains:

| Concept | Product | Domain (weight) |
|---|---|---|
| Skills, `/command` | Claude Code | Domain 3 (20%) |
| Subagents, `Task` tool, `AgentDefinition` | Agent SDK | Domain 1 (27%) |
| Hooks | Both (PreToolUse/PostToolUse exist in both Agent SDK and Claude Code) | Domain 1 or 3 depending on framing |
| MCP servers/tools | Both, but configured differently (`.mcp.json` for Claude Code, MCP client setup for Agent SDK apps) | Domain 2 (18%) |

**Design technique:** write a scenario that could plausibly be solved by either a Skill or a Subagent, and make both available as answer choices. Force the candidate to notice which product the scenario is actually set in (is this "my team runs this from a CLI" vs "I'm building an autonomous agent system that gets deployed")? This is a confirmed real trap from Weston's exam (the "8-step review process" question).

---

## Rule 5: Omit the Textbook Mechanism

Sometimes none of the 4 answers name the exact flag/parameter from the notes (e.g. no answer choice literally said `--output-format json --json-schema {...}`). This forces reasoning from the underlying PRINCIPLE (schema-enforced structure vs. free-form prose output) rather than pattern-matching memorized syntax.

**Design technique:** deliberately write a question where the "obviously correct, exact-match" answer is absent, and the 4 choices instead differ on a related-but-not-identical axis (e.g., all 4 choices are about WHERE to configure something, none mention the specific flag, because the flag isn't actually what's being tested — the placement/scope decision is). Use sparingly — this should be maybe 1 in 10 questions, since testing pure recall still has value, but including a few keeps candidates from over-relying on memorized keywords.

---

## Rule 6: Require Composing Two Techniques

Some real exam scenarios can't be solved by picking ONE named pattern — they require recognizing that TWO techniques from possibly the same chapter need to be combined.

**Confirmed instance:** "session degrading after 8 of 45 files, how proceed" — the correct answer combines scratchpad persistence (rescue what's already known) AND subagent delegation (protect the remaining exploration) into ONE answer choice, while the 3 wrong choices each apply only ONE technique in isolation, or apply techniques in the wrong order.

**Design technique:** write a scenario with two distinct sub-problems (e.g., "what's already degrading" + "what's still to come"). Make the correct answer the one that addresses both. Make each wrong answer address only one, or address them in the wrong sequence (e.g., discard-and-restart instead of persist-then-continue).

---

## Rule 7: Specificity Discrimination, Not Keyword Presence

Don't reward or punish an answer for CONTAINING a topic word (edge cases, examples, formatting). Differentiate answers by HOW SPECIFIC/CONCRETE they are.

**Confirmed instance:** "add edge case handling to the prompt" is bad when vague ("handle edge cases appropriately") and good when concrete ("if total_amount is absent, output null rather than estimating from line items"). A naive question would make "mentions edge cases = correct" or "mentions edge cases = prompt bloat, therefore wrong" — both are wrong simplifications. The real skill is distinguishing vague instruction from concrete instruction, independent of topic.

**Design technique:** offer two answer choices about the same underlying fix (e.g. both about improving a tool description, or both about handling missing data) where one is generic/vague language and the other names the SPECIFIC scenario and SPECIFIC correct behavior. Only the concrete one is correct.

---

## Writing Checklist

Before finalizing a practice question, verify:

- [ ] No wrong answer uses language lifted directly from a documented "anti-pattern" list (Rule 1)
- [ ] At least sometimes, a true-but-irrelevant fact is present to test whether the candidate identifies the actual governing constraint (Rule 2)
- [ ] Terminology in the stem doesn't presuppose which mechanism is correct — the answers differentiate on behavior, not label (Rule 3)
- [ ] If the scenario could span two products (Claude Code vs Agent SDK), that ambiguity is deliberately tested (Rule 4)
- [ ] Occasionally, omit the exact textbook flag/parameter to force principle-level reasoning (Rule 5)
- [ ] Occasionally, require the correct answer to combine two named techniques rather than one (Rule 6)
- [ ] When "add more detail/examples" is a plausible-sounding answer, differentiate correct-vs-wrong by concreteness, not by presence of the topic (Rule 7)
- [ ] All 4 options are the length/tone/detail-level of each other — don't let the correct answer be identifiable just because it's longer or more detailed than the distractors

---

## Worked Example

Applying multiple rules to one question (this is the target difficulty level):

**Weak version (naive, don't do this):**
> Q: Should hooks or prompts enforce a $500 refund limit?
> A) Prompts (WRONG — obviously the "unreliable" option from notes)
> B) Hooks (correct — obviously the "reliable" option from notes)

**Strong version (target difficulty):**
> Q: Your CLAUDE.md instructs Claude Code to never run `Bash` commands that modify files outside the current project directory. Testing shows this works correctly in about 9 out of 10 attempts, but occasionally a generated command does touch an external path. Leadership wants a guarantee, not a strong tendency. What should you add?
>
> A) A `PreToolUse` hook on `Bash` calls that inspects the target path and blocks/redirects any command touching outside the project directory
> B) Strengthen the CLAUDE.md wording with more explicit, emphatic language ("NEVER, under any circumstances, modify files outside...")
> C) Add a `.claude/rules/` file scoped to `paths: ["**/*"]` with the same instruction, since rules are more targeted than CLAUDE.md
> D) Configure a `.claude/settings.json` permission rule denying `Bash` commands with path patterns outside the project directory
>
> **Correct: A or D are both plausible — this is intentionally the hardest kind of question, where two mechanisms COULD both work, and the discriminator is which one is more appropriate.** D (settings permission) is arguably the cleaner fit since it PREVENTS the attempt entirely rather than intercepting after the model already decided to try it; A (hook) is also fully valid and deterministic. B and C are both still prompt-tier (probabilistic) despite sounding more "official" — rules are just conditionally-loaded CLAUDE.md content, same reliability tier, not automatically more enforceable.

This mirrors Weston's actual 0%-scored objectives directly: distinguishing hooks from settings permissions, and correctly identifying that `.claude/rules/` does NOT upgrade reliability tier just because it's more targeted.

---

Tags: #claude #certification #question-design #ai
