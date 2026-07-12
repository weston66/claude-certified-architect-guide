---
title: Model Context Protocol (MCP)
tags: [claude, mcp, certification, #ai]
---

# Chapter 4: Model Context Protocol (MCP)

---

## 4.1 What is MCP

An open protocol for connecting external systems to Claude. Three primary resource types:

1. **Tools** - functions the agent can call to perform actions (CRUD, API calls, command execution)
2. **Resources** - data the agent can read for context (docs, database schemas, content catalogs)
3. **Prompts** - predefined prompt templates for common tasks

---

## 4.2 MCP Servers

An MCP server is a process that implements the MCP protocol and provides tools/resources.
- All tools are discovered automatically on connection
- Tool descriptions determine how the model will use them

---

## 4.3 Configuring MCP Servers

**Project configuration (`.mcp.json`)** - for team usage, stored in version control:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

- Environment variables used for secrets - tokens themselves are NOT committed
- Available to all project contributors

**User configuration (`~/.claude.json`)** - personal/experimental servers:
- Not shared via version control
- For personal experiments and testing

**Choosing servers:**
- For standard integrations (Jira, GitHub, Slack), prefer existing community MCP servers
- Build your own only for unique, team-specific workflows

---

## 4.4 The `isError` Flag in MCP

When an MCP tool encounters an error, it uses `isError: true` in the response.

**Structured error (good):**
```json
{
  "isError": true,
  "content": {
    "errorCategory": "transient",
    "isRetryable": true,
    "message": "Service temporarily unavailable. Timeout while calling the orders API.",
    "attempted_query": "order_id=12345",
    "partial_results": null
  }
}
```

**Generic error (anti-pattern):**
```json
{
  "isError": true,
  "content": "Operation failed"
}
```

A generic error gives the agent no information for decision-making - should it retry, change the query, or escalate?

---

## 4.5 MCP Resources

Data an agent can request to get context without taking actions:
- Content catalogs (list of all project tasks, hierarchical navigation)
- Database schemas (understanding data structure)
- Documentation (API references, internal guides)
- Issue/task summaries

**Resource advantage:** the agent does not need exploratory tool calls to understand what data exists. A resource provides an immediate "map."
