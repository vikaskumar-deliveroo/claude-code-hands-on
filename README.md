# Claude Code Practical Workshop (60 min)

> Hands-on session for software engineers. Each chapter builds on the previous one.
> Go backend examples throughout. Follow along in your terminal.

## Chapters

| # | Chapter | Time | Difficulty | What You'll Build |
|---|---------|------|------------|-------------------|
| 0 | [Prerequisites](chapters/00-prerequisites.md) | Pre-work | - | Verify tools: Claude Code, Go, goimports, gh |
| 1 | [CLAUDE.md](chapters/01-claude-md.md) | 8 min | Beginner | Global + project instruction files |
| 2 | [Memory](chapters/02-memory.md) | 7 min | Beginner | Persistent cross-session knowledge |
| 3 | [Hooks](chapters/03-hooks.md) | 10 min | Intermediate | Auto-goimports + block generated files |
| 4 | [Agents](chapters/04-agents.md) | 10 min | Intermediate | Code reviewer agent (read-only, diff-based) |
| 5 | [Skills](chapters/05-skills.md) | 10 min | Intermediate | `/commitandcreatepr` - full PR workflow |
| 6 | [MCP & Plugins](chapters/06-mcp-plugins.md) | 10 min | Advanced | Atlassian (Jira + Confluence) integration |
| 7 | [Putting It All Together](chapters/07-summary.md) | 5 min | Recap | Quick reference + cheat sheet |

## How to Use This Workshop

**If you're an attendee**: Start at Chapter 0 (do it before the session). Then follow Chapters 1-7 in order.

**If you're a facilitator**: Each chapter has `[Live Exercise]` sections for hands-on and `[Reference - Try Later]` sections to skip during the session. Chapter 6's Atlassian demo should be shown on the facilitator's screen.

## Key Concepts by Chapter

```
Ch 1: CLAUDE.md     -> Tell Claude your rules (once, not every session)
Ch 2: Memory        -> Claude remembers what it learns across sessions
Ch 3: Hooks         -> Automated guardrails that always run
Ch 4: Agents        -> Specialized workers with restricted tools
Ch 5: Skills        -> Reusable workflows that orchestrate agents
Ch 6: MCP/Plugins   -> Connect Claude to Jira, Datadog, CI, etc.
```

## Full Workshop (single file)

The complete workshop is also available as a single file: [WORKSHOP.md](WORKSHOP.md)
