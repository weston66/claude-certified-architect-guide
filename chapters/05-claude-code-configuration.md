---
title: Claude Code Configuration and Workflows
tags: [claude, claude-code, CLAUDE.md, skills, ci-cd, certification, #ai]
---

# Chapter 5: Claude Code - Configuration and Workflows

---

## 5.1 The CLAUDE.md Hierarchy

Three-level hierarchy:

```
1. User-level: ~/.claude/CLAUDE.md
   - Applies only to that user
   - NOT shared via VCS
   - Personal preferences and working style

2. Project-level: .claude/CLAUDE.md or a root CLAUDE.md
   - Applies to all project contributors
   - Managed via VCS
   - Coding standards, testing standards, architectural decisions

3. Directory-level: CLAUDE.md in subdirectories
   - Applies when working with files in that directory
   - Conventions specific to that part of the codebase
```

**Common mistake:** placing project instructions in `~/.claude/CLAUDE.md` (user-level) instead of `.claude/CLAUDE.md` (project-level) - new team members won't get them.

---

## 5.2 `@path` Syntax (File Imports)

CLAUDE.md can reference external files using `@path`:

```markdown
# Project CLAUDE.md
Coding standards are described in @./standards/coding-style.md
Test requirements are in @./standards/testing-requirements.md
```

**Rules:**
- Use `@` immediately before the file path (no space)
- Relative paths are resolved relative to the file containing the import
- Maximum import nesting depth is 5

---

## 5.3 The `.claude/rules/` Directory

Alternative to a monolithic CLAUDE.md, organized by topic with YAML frontmatter for conditional loading:

```yaml
---
paths: ["**/*.test.tsx", "**/*.test.ts"]
---

Tests must use describe/it blocks.
Use data factories instead of hardcoding.
Do not mock the database - use a test database.
```

**How it works:** a rule is loaded **only** when Claude Code edits a file matching the `paths` pattern - saves context and tokens.

**When to use `.claude/rules/` with `paths` vs directory-level CLAUDE.md:**
- `.claude/rules/` with `paths` - conventions that apply to files spread across many directories (tests, migrations)
- Directory-level CLAUDE.md - conventions tied to a specific directory

---

## 5.4 Custom Slash Commands and Skills

**`.claude/commands/` format (legacy, supported):**
```
.claude/commands/
  review.md        -- /review -- standard code review
  test-gen.md      -- /test-gen -- test generation
```

**`.claude/skills/` format (current):**
```
.claude/skills/
  review/SKILL.md  -- /review -- with frontmatter configuration
```

**Project commands** - stored in VCS, available to everyone who clones the repo.
**User commands** (`~/.claude/commands/` or `~/.claude/skills/`) - personal, not shared.

---

## 5.5 Skills Frontmatter

```yaml
---
context: fork
allowed-tools: ["Read", "Grep", "Glob"]
argument-hint: "Path to the directory to analyze"
---

Analyze the code structure in the specified directory.
```

| Parameter | Description |
|---|---|
| `context: fork` | Runs the skill in an isolated subagent - verbose output doesn't pollute the main session |
| `allowed-tools` | Restricts which tools are available |
| `argument-hint` | Hint asking for an argument when invoked without parameters |

**Skill vs CLAUDE.md:**
- **Skill** - on-demand invocation for a specific task (review, analysis, generation)
- **CLAUDE.md** - always-loaded general standards and conventions

---

## 5.6 Planning Mode vs Direct Execution

**Planning mode:**
- Model only investigates and plans; does not make changes
- Uses Read, Grep, Glob to explore the codebase
- Produces an implementation plan the user approves
- Safe exploration with no side effects

**When to use planning mode:**
- Large changes (dozens of files)
- Multiple plausible approaches
- Architectural decisions
- Unfamiliar codebase
- Library migrations affecting 45+ files

**When to use direct execution:**
- Single-file fixes with a clear stack trace
- Adding one validation check
- Well-understood, unambiguous changes

---

## 5.7 `/compact` and `/memory` Commands

**`/compact`** - compresses context by summarizing prior history:
- Risk: exact numeric values, dates, and specific details can be lost during summarization

**`/memory`** - opens CLAUDE.md for editing:
- Information persists across sessions and is automatically loaded on startup
- Alternative to re-explaining the same instructions every session

---

## 5.8 Claude Code CLI for CI/CD

**The `-p` (or `--print`) flag:**
```bash
claude -p "Analyze this pull request for security issues"
```
- Non-interactive mode: processes the prompt, prints to stdout, exits
- The **only correct way** to run Claude in CI/CD pipelines

**Structured output for CI:**
```bash
claude -p "Review this PR" --output-format json --json-schema '{"type":"object",...}'
```

**Session context isolation:** the same Claude session that generated code is often less effective at reviewing it - use an independent instance for review.

**Preventing duplicate comments:** include prior review results in context and instruct Claude to report only new or unresolved issues.
