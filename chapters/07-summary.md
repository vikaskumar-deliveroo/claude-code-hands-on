# Chapter 7: Putting It All Together (5 min)

Here's how all pieces work together in a real Go backend workflow:

```
~/.claude/CLAUDE.md   -> "Branch prefix <name>/, pre-push: make lint && make test/unit"
./CLAUDE.md           -> "Go + PostgreSQL, UberFX DI, cache wrappers in internal/repo/cache/"
Memory                -> "Redis pipe_exec latency is payload-driven. Use snappy compression."
Hooks                 -> Auto-run goimports on every .go file edit, block go.sum edits
Agents                -> code-reviewer checks diff for bugs, security, Go-specific issues
Skills                -> /commitandcreatepr runs imports -> lint -> tests -> review -> PR
Plugin (Atlassian)    -> "Create Jira tickets from the Confluence spec page"
```

## Recommended Setup for Your Team

**Day 1** (10 min):
1. Add `CLAUDE.md` with your team's conventions
2. Commit it so the whole team benefits

**Day 2** (15 min):
3. Add hooks for your language (formatter, linter)
4. Create a `/review` skill for your PR process

**Day 3** (20 min):
5. Set up MCP for your monitoring tools (Datadog, Sentry)
6. Create a custom agent for your most common task

## Quick Reference

| Feature | Location | Scope |
|---------|----------|-------|
| CLAUDE.md | `./CLAUDE.md` | Project (shared) |
| CLAUDE.md | `./.claude/CLAUDE.md` | Project (personal) |
| CLAUDE.md | `~/.claude/CLAUDE.md` | Global |
| Memory | `~/.claude/projects/*/memory/` | Auto-managed |
| Hooks | `.claude/hooks.json` | Project |
| Hooks | `~/.claude/hooks.json` | Global |
| Skills | `.claude/skills/*/SKILL.md` | Project |
| Skills | `~/.claude/skills/*/SKILL.md` | Global |
| Agents | `.claude/agents/*.md` | Project |
| MCP | `.claude/mcp.json` | Project |
| MCP | `~/.claude/mcp.json` | Global |
| Plugins | `~/.claude/plugins/` | Global (auto-managed) |

## Cheat Sheet

```bash
# Start Claude Code
claude

# Start with specific permission mode
claude --allowedTools "Edit,Write,Bash(git *)"

# Resume last conversation
claude --continue

# Print last response (for piping)
claude -p "explain this error" < error.log

# Check current config
claude config list
```

### Useful Commands Inside Claude Code
```
/help           - Show all commands
/compact        - Compress conversation (saves context)
/clear          - Clear conversation history
/model          - Switch model (opus/sonnet/haiku)
/cost           - Show token usage and cost
/memory         - View/edit memory
/plugins        - Browse and install plugins
```

---

*Workshop complete! Start with CLAUDE.md and hooks - they give the most value
for the least effort. Add skills and agents as you discover repeated workflows.*

---

Back to: [Workshop Index](../README.md)
