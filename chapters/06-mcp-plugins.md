# Chapter 6: MCP & Plugins - External Integrations (10 min)

**What**: MCP (Model Context Protocol) servers and plugins connect Claude to external tools - Jira, Confluence, Datadog, CI, etc. Plugins are the easiest way since they bundle MCP servers + skills in one install.

**Why**: Instead of copy-pasting Jira tickets, Datadog traces, or CI logs into Claude, it can query them directly. This is a massive productivity multiplier.

## Two Ways to Add Integrations

| Method | How | Best For |
|--------|-----|----------|
| **Plugins** | `/plugins` in Claude Code | Official integrations (Atlassian, Sentry, GitHub) |
| **Raw MCP** | Edit `.claude/mcp.json` | Custom/self-hosted servers |

## [Live Exercise] Browse and Install a Plugin (3 min)

In Claude Code, run `/plugins` to see the official plugin registry. Browse what's available. Optionally install one that doesn't require external accounts (like `code-review` or `gopls-lsp`).

## [Facilitator Demo] Atlassian Plugin

> **Why a demo, not hands-on**: OAuth flows, corporate SSO, and missing Atlassian accounts will stall 30% of the room. Watch the facilitator, then set it up at your desk after the workshop.

The Atlassian plugin connects Claude to Jira + Confluence via OAuth. No API tokens to manage.

**Setup** (facilitator shows):
1. Run `/plugins` -> search `atlassian` -> install
2. On first use, browser opens for OAuth 2.1 login
3. Authorize with your Atlassian account -- done

**What you get**: An MCP server (`https://mcp.atlassian.com/v1/mcp`) + 5 built-in skills:

| Skill | Trigger | What It Does |
|-------|---------|-------------|
| `/atlassian:search-company-knowledge` | "search our docs for X" | Searches Confluence + Jira in parallel, synthesizes cited answers |
| `/atlassian:triage-issue` | "triage this error" | Searches Jira for duplicates, creates bug tickets with context |
| `/atlassian:spec-to-backlog` | "create tickets from this spec" | Reads Confluence spec, creates Epic + linked Jira tickets |
| `/atlassian:capture-tasks-from-meeting-notes` | "create tasks from notes" | Extracts action items, looks up assignees, creates Jira tasks |
| `/atlassian:generate-status-report` | "generate status report" | Queries Jira, categorizes by status/priority, publishes to Confluence |

## Try It: Search Company Knowledge

After installing, try:

```
Search our Confluence for Confettura
```

Or:

```
/atlassian:triage-issue

I'm seeing this error in production:
"context deadline exceeded" in FindRestaurantsByDrnID
at internal/repo/restaurant.go:245
```

Claude will:
1. Search Jira for similar past issues
2. Check if it's a known duplicate
3. Offer to create a new ticket or add context to an existing one

## Try It: Spec to Backlog

```
/atlassian:spec-to-backlog

Create Jira tickets from this Confluence page:
https://yoursite.atlassian.net/wiki/spaces/ENG/pages/12345
```

Claude will read the spec, break it into tasks, create an Epic, and generate linked child tickets.

## Raw MCP Config (for custom servers)

For integrations without a plugin, use `.claude/mcp.json` (project) or `~/.claude/mcp.json` (global):

```json
{
  "mcpServers": {
    "your-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@some-org/mcp-server"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

## Popular Plugins & MCP Servers

| Integration | Type | Install |
|------------|------|---------|
| **Atlassian** (Jira + Confluence) | Plugin | `/plugins` -> `atlassian` |
| **GitHub** | Plugin | `/plugins` -> `github` |
| **Sentry** | Plugin | `/plugins` -> `sentry` |
| **Datadog** | MCP | `.claude/mcp.json` (API key required) |
| **CircleCI** | MCP | `.claude/mcp.json` (API token required) |
| **PostgreSQL** | MCP | `.claude/mcp.json` (connection string) |

## Key Tips
- Plugins auto-update and handle auth via OAuth - prefer them over raw MCP
- Restart Claude Code after changing `mcp.json` (plugins take effect immediately)
- Use `env` for secrets in raw MCP config - never hardcode tokens
- Plugins live in `~/.claude/plugins/` - check installed ones with `/plugins`
- The Atlassian MCP respects your existing Jira/Confluence permissions - no extra access

---

Next: [Chapter 7 - Putting It All Together](./07-summary.md)
