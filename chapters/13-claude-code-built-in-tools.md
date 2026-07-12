---
title: Claude Code Built-in Tools
tags: [claude, claude-code, tools, certification, #ai]
---

# Chapter 13: Claude Code Built-in Tools

---

## 13.1 Tool Selection Reference

| Task | Tool | Example |
|---|---|---|
| Find files by name/pattern | **Glob** | `**/*.test.tsx`, `src/components/**/*.ts` |
| Search within files | **Grep** | Function name, error message, import |
| Read a file in full | **Read** | Load a file for analysis |
| Write a new file | **Write** | Create a file from scratch |
| Edit an existing file precisely | **Edit** | Replace a specific snippet via unique text match |
| Run a shell command | **Bash** | git, npm, run tests, build |

---

## 13.2 Incremental Investigation Strategy

Do not read all files at once. Build understanding incrementally:

```
1. Grep: find entry points (function definition, export)
2. Read: read the found files
3. Grep: find usages (import, calls)
4. Read: read consumer files
5. Repeat until you have a complete picture
```

---

## 13.3 Fallback: Read + Write Instead of Edit

When Edit fails due to a non-unique text match:
1. Read - load the full file content
2. Modify the content programmatically
3. Write - write the updated version
