# Claude Code Practical Workshop (60 min)

> Hands-on session for software engineers. Each chapter builds on the previous one.
> Total time: ~60 minutes. Follow along in your terminal.

---

## Chapter 0: Prerequisites (do before the workshop)

Make sure these are set up **before** the session starts. We won't have time to debug installs.

### Required Tools

```bash
# Verify everything is installed:
git --version           # Git CLI
claude --version        # Claude Code CLI
go version              # Go toolchain
goimports -help 2>&1 | head -1   # goimports (go install golang.org/x/tools/cmd/goimports@latest)
gh --version            # GitHub CLI (for PR workflows)
```

### Install if missing

```bash
# Git (macOS)
xcode-select --install          # or: brew install git

# Claude Code
npm install -g @anthropic-ai/claude-code

# goimports
go install golang.org/x/tools/cmd/goimports@latest

# GitHub CLI (macOS)
brew install gh
```

### Auth Setup
- You need an Anthropic API key or organization SSO configured
- Run `claude` once and complete the login flow before the workshop

### Permission Prompts (important for first-time users)

Claude Code asks for permission before running commands or editing files. When you see a prompt:
- Press **y** to allow once
- Press **a** to always allow that tool for this session
- Press **n** to deny

For a smoother workshop experience, start Claude with pre-approved tools:
```bash
claude --allowedTools "Edit,Write,Bash(git *),Bash(make *),Bash(go *),Bash(gh *)"
```

---

## Chapter 1: CLAUDE.md - Global & Project Instructions (8 min)

**What**: A markdown file that gives Claude persistent instructions. Think of it as a `.editorconfig` but for AI behavior. It works at three levels:

| Level | Path | Who sees it | Use case |
|-------|------|-------------|----------|
| **Global** | `~/.claude/CLAUDE.md` | Only you, all projects | Your personal workflow, git identity, editor prefs |
| **Project (shared)** | `./CLAUDE.md` in repo root | Whole team (committed) | Build commands, test patterns, code conventions |
| **Project (personal)** | `./.claude/CLAUDE.md` in repo | Only you, this project | Your shortcuts, WIP notes |

**Why**: Without it, you repeat the same instructions every session ("run `make lint` not `golangci-lint`", "use Confettura not Florence for feature flags", etc.)

### Hands-on Exercise: Create a Global CLAUDE.md (5 min)

This is your **personal** instruction file that applies to every project. Open a Claude session:

```bash
claude
```

Then say:

```
Create ~/.claude/CLAUDE.md with these rules:

## Git
- Branch prefix: `<your-name>/` (e.g., `vikas/fix-cache-ttl`)
- Always create PRs in draft mode
- Never use --author flag, rely on git config

## Commit
- Pre-push checklist: `make lint` then `make test/unit` must pass
- Never commit files with only formatting or import changes
- Never add Claude as co-author

## Go Conventions
- Run `goimports` before committing Go files
- Use `testify/assert` for tests, not raw `if` checks
- Prefer table-driven tests
- Feature flags: use Confettura, NOT Florence (deprecated)

## Workflow
- Always run relevant tests before considering a task done
- When I say "commit", run lint + tests first
```

**Verify**: Read `~/.claude/CLAUDE.md` and confirm the rules. Now in **every** project, Claude will follow your branch naming, commit, and Go conventions automatically.

### Real-World Example: What a Mature Global CLAUDE.md Looks Like

Here's a condensed version of a real global CLAUDE.md used daily on a Go backend service:

```markdown
## Quick Reference
- Branch prefix: `vikas/`
- Pre-push: `make lint && make test/unit` must pass
- Feature flags: Confettura (not Florence)

## Commit
- Do NOT use `--author` flag. Git config already set.
- Never commit formatting-only changes
- Pre-push: lint → unit tests → self-review → push

## Query Optimisation
- First explain what each part of the query does
- Share complete runnable psql queries (no "paste IDs here")

## Implementation
- Run relevant tests before marking any task complete
- After committing, review with a subagent before pushing
```

### [Reference - Try Later] Create a Project-Level CLAUDE.md

In your actual Go repo, create `CLAUDE.md` at the root with team-shared rules:

```
Create a CLAUDE.md for this Go project with:

## Build & Test
- Build: `make build`
- Lint: `make lint` (uses golangci-lint)
- Unit tests: `make test/unit`
- DB tests: `make test/database` (needs `make test/infra` first)

## Code Style
- Use UberFX for dependency injection
- Redis cache wrappers go in `internal/repo/cache/`
- All repo methods use `context.Context` as first parameter
- Use `mock.AnythingOfType("[]string")` for StatsD tag mocks

## Architecture
- GraphQL API layer → Service layer → Repo layer → PostgreSQL
- Cache wrappers wrap repo layer, gated by feature flags
- Kafka consumers handle cache invalidation
```

### Key Tips
- Global CLAUDE.md = your personal defaults across ALL repos
- Project CLAUDE.md = team conventions (commit this to git)
- Keep both concise - Claude reads them every session
- Global rules are overridden by project rules when they conflict

---

## Chapter 2: Memory - Persistent Learning (7 min)

**What**: Auto-memory lets Claude remember things across sessions. It lives in `~/.claude/projects/<project-hash>/memory/`.

**Why**: Claude forgets everything between sessions. Memory fixes that. It learns your patterns, project quirks, and past mistakes.

### Hands-on Exercise (5 min)

Start a Claude session and say:

```
Remember: In this project, the database is Aurora PostgreSQL.
Repo layer is in internal/repo/. Cache wrappers are in internal/repo/cache/.
DB tests need `make test/infra` before running `make test/database`.
Unit tests are `make test/unit` and don't require a database.
```

Claude will save this to its `MEMORY.md`. Now verify:

```
What do you know about our test setup?
```

Claude should recall the DB vs unit test distinction without you repeating it.

### Real-World Memory: What Grows Organically Over Time

Here's what a real project memory looks like after weeks of use. Claude builds this automatically as you work:

```markdown
# Project Memory

## Topic Index
| Topic File | What It Covers | Last Updated |
|---|---|---|
| caching-patterns.md | Redis cache wrapper, invalidation, Confettura flags | 2026-02-14 |
| testing-patterns.md | StatsD mock patterns, DB vs unit tests | 2026-02-10 |
| service-metrics.md | All Datadog metric names: RPS, latency, Redis cache | 2026-02-15 |
| restaurant-query-patterns.md | FindRestaurants variants, indexes, feature flags | 2026-02-13 |

## Quick Patterns
### Composite Wrapper for Interface Satisfaction
type cachedXxxGetter struct {
    repo  fullInterface      // delegates uncached methods
    cache partialInterface   // caches specific methods
}
```

Notice how it's **organized by topic** with separate files for deep dives. Claude loads only the relevant memory file when you work on caching vs testing vs metrics.

### Practical Patterns to Save

```
Remember: StatsD mock tests - use mock.AnythingOfType("[]string") for tag
params instead of []string{}. Prevents cascading test failures when tags change.
Files affected: repo/get_in_parallel_test.go, branchavailabilities/api_test.go

Remember: Redis pipe_exec latency is driven by PAYLOAD SIZE not command count.
Cold cache (0 payload) = 4-8ms, warm cache (7.5MB) = 50-55ms.
Compression via snappy with 0x01 version byte prefix for backward compat.

Remember: DB connection pool wait_duration is a CUMULATIVE counter in Go's
sql.DBStats. Raw values grow monotonically. Don't trust percentiles directly.
```

### Key Tips
- Memory is per-project - it won't leak across repos
- Organize into topic files for large projects (Claude does this automatically)
- Keep memories factual and include file paths/metric names
- Say "forget about X" to remove outdated memories
- Memory older than a week may be stale - verify before relying on it

---

## Chapter 3: Hooks - Automated Guardrails (10 min)

**What**: Shell commands that run automatically before/after Claude performs actions. They act as guardrails and automation triggers.

**Why**: Enforce rules without relying on Claude to remember them. Hooks are deterministic - they always run.

**Hook Types**:
| Hook | When it Runs |
|------|-------------|
| `PreToolUse` | Before Claude calls any tool |
| `PostToolUse` | After a tool finishes |
| `Notification` | When Claude sends a notification |
| `Stop` | When Claude finishes its response |
| `UserPromptSubmit` | When you send a message |

### [Live Exercise] Create a Combined hooks.json (8 min)

Create `.claude/hooks.json` in your repo (or `~/.claude/hooks.json` for global).

**Important**: All hooks go in a **single file**. Each hook type (`PreToolUse`, `PostToolUse`) is an array - add multiple entries to the same array.

Here's a combined file with two hooks - auto-goimports + block generated files:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE '(go\\.sum|vendor/|\\.pb\\.go|_generated\\.go|mock_.*\\.go)'; then echo 'BLOCK: Do not edit generated or lock files directly. Use go mod tidy, protoc, or mockgen.' && exit 1; fi",
            "timeout": 5000
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.file_path // .filePath // empty'); if [ -n \"$FILE\" ] && echo \"$FILE\" | grep -qE '\\.go$'; then goimports -w \"$FILE\" 2>/dev/null; fi",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

This does two things:
1. **PreToolUse** - blocks edits to `go.sum`, `.pb.go`, generated files (exit 1 = block)
2. **PostToolUse** - auto-runs `goimports` after any `.go` file is written

**Test it**: Ask Claude to edit a Go file. Verify goimports ran by checking the imports are grouped (stdlib, external, internal).

### [Reference - Try Later] More Hook Ideas

Add more entries to the same arrays above:

```json
// Add to "PreToolUse" array - remind about lint before git commit:
{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'git commit'; then echo 'REMINDER: Run make lint && make test/unit before committing'; fi",
    "timeout": 3000
  }]
}
```

Other ideas:
- Block writes to `main`/`master` branch files
- Auto-run `go vet` after Go file changes
- Log all Bash commands Claude runs (audit trail for security)

---

## Chapter 4: Custom Agents (Subagents) (10 min)

**What**: Specialized AI agents defined in `.claude/agents/` that handle specific types of work. They run in their own context and can have restricted tools.

**Why**: Different tasks need different capabilities. A code reviewer shouldn't need Bash. A researcher doesn't need file editing. Agents also keep your main conversation clean - heavy work happens in a separate context window.

### How Agents Work

```
You (main conversation)
  │
  ├── Task(subagent_type="code-reviewer")  → reads code + diff, can't edit
  ├── Task(subagent_type="go-test-writer") → reads + writes + runs tests
  └── Task(subagent_type="Explore")        → built-in: codebase search
```

Agents are markdown files in `.claude/agents/`:
```
~/.claude/agents/my-agent.md     ← global agent
.claude/agents/my-agent.md       ← project agent
```

### [Live Exercise] Code Reviewer Agent (8 min)

This agent reviews your **local changes** - either unstaged work or the diff between your branch and main. It reads the diff, reads the full files for context, and produces a structured review. It cannot edit files - it only reports.

**File**: `.claude/agents/code-reviewer.md`

```markdown
---
name: code-reviewer
description: >
  Reviews local code changes (unstaged or branch diff) for bugs,
  security issues, and Go-specific problems. Read-only - cannot
  modify files. Outputs a structured review report.
allowed_tools: Glob, Grep, Read, Bash
---

# Code Reviewer Agent

You review local code changes and produce a structured review.

## Step 1: Get the Diff

Determine what to review based on context:

Option A - Unstaged changes (default):
  git diff

Option B - All changes on current branch vs main/master:
  git diff main...HEAD

Option C - Last commit only:
  git diff HEAD~1

If the diff is empty, inform the user and stop.

## Step 2: Read Full Files for Context

For each changed file in the diff:
- Read the FULL file (not just the diff) to understand surrounding code
- Check for related test files (*_test.go) to understand expected behavior
- Look at interfaces/types used by the changed code

## Step 3: Analyze Changes

For each changed file, review for:
- **Bugs**: Logic errors, nil pointer derefs, off-by-one, race conditions
- **Security**: SQL injection, missing auth checks, hardcoded secrets
- **Performance**: N+1 queries, unbounded allocations, missing indexes
- **Go-specific**: Unchecked errors, defer in loop, goroutine leaks,
  context not propagated, mutex not unlocked
- **Tests**: Are new code paths covered? Are existing tests broken?

## Step 4: Output Report

For each finding:

### <SEVERITY> <file_path>:<line_number>
**Issue:** <one-line description>
**Why it matters:** <brief explanation>
**Suggested fix:**
  <code snippet>

At the end, output a summary:

## Summary
| Severity | Count |
|----------|-------|
| CRITICAL | X     |
| WARNING  | X     |
| SUGGESTION | X   |

## Rules
- Only flag real issues — not style preferences or nits
- Every finding MUST include file path, line number, and a fix
- Read the full file before commenting — don't misunderstand partial diffs
- If the code looks good, say "No issues found" — don't invent problems
- Ignore pre-existing issues not introduced by this diff
- Focus on the CHANGED lines, use surrounding code only for context
```

### Using It

Review your unstaged work before committing:

```
Use the code-reviewer agent to review my unstaged changes
```

Review everything on your feature branch:

```
Use the code-reviewer agent to review all changes on this branch vs main
```

Skills like `/commitandcreatepr` use this agent internally at Step 6:

```
Use Task tool with subagent_type="code-reviewer"
Prompt: Review the changes in the last commit (git diff HEAD~1).
```

The agent runs in a separate context - it reads your diff and full files, then reports back findings without touching your code.

### [Reference - Try Later] Go Test Generator Agent

**File**: `.claude/agents/go-test-writer.md`

```markdown
---
name: go-test-writer
description: Generates table-driven Go tests with testify
allowed_tools: Glob, Grep, Read, Write, Bash
---

# Go Test Writer Agent

Generate comprehensive Go unit tests for the given code.

## Rules
- Use table-driven tests with t.Run(tc.name, ...)
- Use testify/assert and testify/require, NOT raw if checks
- Use testify/mock for interfaces
- Follow existing *_test.go patterns in the same package
- Test file goes next to source: foo.go → foo_test.go

## Process
1. Read the target .go file
2. Find existing *_test.go files for patterns
3. Write tests: happy path, error cases, edge cases, nil inputs
4. Run go test ./path/to/package/... to verify all pass
5. Run make lint to catch linting issues

## Template
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name     string
        input    InputType
        expected OutputType
        wantErr  bool
    }{
        {name: "happy path", ...},
        {name: "nil input returns error", ...},
    }
    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            result, err := FunctionName(tc.input)
            if tc.wantErr {
                require.Error(t, err)
                return
            }
            require.NoError(t, err)
            assert.Equal(t, tc.expected, result)
        })
    }
}
```

### Key Concept: `allowed_tools` = Least Privilege

Notice how the two agents have different tool access:

| Agent | allowed_tools | Why |
|-------|--------------|-----|
| `code-reviewer` | Glob, Grep, Read, Bash | Reads code + runs `git diff`. Can't edit files. |
| `go-test-writer` | Glob, Grep, Read, **Write**, Bash | Needs to create test files. |

If you omit `allowed_tools`, the agent gets all tools. Always restrict to what's actually needed.

### Built-in Subagent Types (No Config Needed)

Claude Code ships with these agents out of the box - no `.md` file required:

| Type | Use Case |
|------|----------|
| `Explore` | Fast codebase search and exploration |
| `Plan` | Design implementation plans |
| `feature-dev:code-reviewer` | Review code for bugs and quality |
| `feature-dev:code-explorer` | Deep-dive into existing features |
| `feature-dev:code-architect` | Design feature architectures |

```
Use the Explore agent to find all Redis cache implementations in the codebase
```

---

## Chapter 5: Skills - Reusable Workflows (10 min)

**What**: Markdown files in `.claude/skills/` that teach Claude specialized workflows. Invoked with `/skill-name`. Skills can orchestrate agents from Chapter 4.

**Why**: Codify your team's processes. Instead of explaining "how we do PRs" every time, make it a skill. Claude follows the exact steps every time - no forgetting, no shortcuts.

### Anatomy of a Skill

A skill is just a markdown file with a YAML frontmatter header:

```
~/.claude/skills/my-skill/SKILL.md    ← global skill
.claude/skills/my-skill/SKILL.md      ← project skill
```

The frontmatter tells Claude:
- `name` - the `/slash-command` name
- `description` - when to use it (Claude reads this to auto-suggest)
- `user_invocable: true` - enables `/name` trigger

### Real Example: `/commitandcreatepr` (Production Skill)

This is a real skill used daily on a Go backend. It automates the entire workflow from code changes to a draft PR - 9 steps that would otherwise be manual. Notice how **Step 6 delegates to the code-reviewer agent** from Chapter 4:

> **Note**: This is a real production skill shown for illustration. **You must customize** the branch prefix (`<your-name>/`), known test failures, and PR template path for your own project before using it.

**File**: `~/.claude/skills/commitandcreatepr/SKILL.md`

```markdown
---
name: commitandcreatepr
description: |
  Validate code (goimports, lint, unit tests), commit, sub-agent review,
  push, and create a draft PR. End-to-end workflow from local changes to PR.
user_invocable: true
---

# Commit and Create PR

Validate, commit, review, push, and create a draft pull request.

## Steps

### Step 1: Check for changes
git status --short
- If no changes, inform the user and stop.

### Step 2: Format Code
make goimports
- Fix any formatting issues, re-run until clean.

### Step 3: Run Lint
make lint
- CRITICAL: Must exit zero. Watch for missing tools causing partial lint.
- Fix issues and re-run until clean.

### Step 4: Run Unit Tests
make test/unit
- All tests must pass. Fix failures before proceeding.
- Known pre-existing failures on master: TestCollectionView,
  TestHomeFeedDeepLinking, TestTranslations — ignore if they also
  fail on master.

### Step 5: Stage and Commit
git add <specific-files>   # NEVER use git add -A or git add .
git commit -m "<concise message>"

Rules:
- Stage only files related to the change. Never git add -A.
- Do NOT use --author flag. Git config is already set.
- NEVER add Claude as co-author.

### Step 6: Sub-Agent Code Review (MANDATORY)         ← uses agent!
After committing, launch a code-reviewer sub-agent:
- Review the changes in the last commit (git diff HEAD~1)
- Focus on bugs, logic errors, and security issues
- Address any valid HIGH-confidence issues
- If changes were made, go back to Step 2 and re-validate

### Step 7: Push to Remote
git push -u origin <current-branch-name>
- Ensure branch follows <your-name>/ prefix convention.

### Step 8: Create Draft PR
- Copy the PR template as-is from .github/PULL_REQUEST_TEMPLATE.md
- After "Why" section, explain why changes were important
- In "What are the consequences?", describe changes and impact
- Always create in draft mode

### Step 9: Monitor CI
gh pr checks <PR_NUMBER> --watch
- Ignore approval-related checks
- Fix real CI failures, push fixes, re-monitor
- Repeat until all non-approval checks pass

## Blocker Rules
- DO NOT commit if goimports, lint, or unit tests fail
- DO NOT push if code review has unaddressed issues
- DO NOT skip the sub-agent review
- DO NOT create a non-draft PR
```

### Using It

After writing your code, just type:

```
/commitandcreatepr
```

Claude handles all 9 steps automatically: format → lint → test → commit → **agent review** → fix → push → PR → CI monitoring. If anything fails, it stops and fixes before proceeding.

### How Skills and Agents Work Together

This is the key insight - **agents are the workers, skills are the orchestrators**:

```
/commitandcreatepr (skill - orchestrates the workflow)
  │
  ├── Step 2: make goimports        (Claude runs directly)
  ├── Step 3: make lint              (Claude runs directly)
  ├── Step 4: make test/unit         (Claude runs directly)
  ├── Step 5: git commit             (Claude runs directly)
  ├── Step 6: code-reviewer agent    (delegated to subagent) ← Chapter 4!
  ├── Step 7: git push               (Claude runs directly)
  └── Step 9: gh pr checks --watch   (Claude runs directly)
```

### Quick Starter: Create Your Own Skill

**File**: `.claude/skills/quick-review/SKILL.md`

```markdown
---
name: quick-review
description: Quick code review of staged changes
user_invocable: true
---

# Quick Code Review

1. Run `git diff --cached` to see staged changes
2. For each file, check for: bugs, security issues, N+1 queries
3. Output a table: | File | Issue | Severity | Line |
4. If clean, confirm the code looks good.

Rules:
- Only review staged changes
- Be specific - cite line numbers
- Skip style nits unless they affect correctness
```

### Key Tips
- Skills are project-specific (`.claude/skills/`) or global (`~/.claude/skills/`)
- Use `user_invocable: true` to enable `/slash-command` syntax
- Skills can delegate to agents (like the code review step above)
- Skill files support a `references/` subdirectory for templates, examples, etc.
- Great for: commit workflows, PR processes, deploy checklists, incident response

---

## Chapter 6: MCP & Plugins - External Integrations (10 min)

**What**: MCP (Model Context Protocol) servers and plugins connect Claude to external tools - Jira, Confluence, Datadog, CI, etc. Plugins are the easiest way since they bundle MCP servers + skills in one install.

**Why**: Instead of copy-pasting Jira tickets, Datadog traces, or CI logs into Claude, it can query them directly. This is a massive productivity multiplier.

### Two Ways to Add Integrations

| Method | How | Best For |
|--------|-----|----------|
| **Plugins** | `/plugins` in Claude Code | Official integrations (Atlassian, Sentry, GitHub) |
| **Raw MCP** | Edit `.claude/mcp.json` | Custom/self-hosted servers |

### [Live Exercise] Browse and Install a Plugin (3 min)

In Claude Code, run `/plugins` to see the official plugin registry. Browse what's available. Optionally install one that doesn't require external accounts (like `code-review` or `gopls-lsp`).

### [Facilitator Demo] Atlassian Plugin

> **Why a demo, not hands-on**: OAuth flows, corporate SSO, and missing Atlassian accounts will stall 30% of the room. Watch the facilitator, then set it up at your desk after the workshop.

The Atlassian plugin connects Claude to Jira + Confluence via OAuth. No API tokens to manage.

**Setup** (facilitator shows):
1. Run `/plugins` → search `atlassian` → install
2. On first use, browser opens for OAuth 2.1 login
3. Authorize with your Atlassian account — done

**What you get**: An MCP server (`https://mcp.atlassian.com/v1/mcp`) + 5 built-in skills:

| Skill | Trigger | What It Does |
|-------|---------|-------------|
| `/atlassian:search-company-knowledge` | "search our docs for X" | Searches Confluence + Jira in parallel, synthesizes cited answers |
| `/atlassian:triage-issue` | "triage this error" | Searches Jira for duplicates, creates bug tickets with context |
| `/atlassian:spec-to-backlog` | "create tickets from this spec" | Reads Confluence spec, creates Epic + linked Jira tickets |
| `/atlassian:capture-tasks-from-meeting-notes` | "create tasks from notes" | Extracts action items, looks up assignees, creates Jira tasks |
| `/atlassian:generate-status-report` | "generate status report" | Queries Jira, categorizes by status/priority, publishes to Confluence |

### Try It: Search Company Knowledge

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

### Try It: Spec to Backlog

```
/atlassian:spec-to-backlog

Create Jira tickets from this Confluence page:
https://yoursite.atlassian.net/wiki/spaces/ENG/pages/12345
```

Claude will read the spec, break it into tasks, create an Epic, and generate linked child tickets.

### Raw MCP Config (for custom servers)

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

### Popular Plugins & MCP Servers

| Integration | Type | Install |
|------------|------|---------|
| **Atlassian** (Jira + Confluence) | Plugin | `/plugins` → `atlassian` |
| **GitHub** | Plugin | `/plugins` → `github` |
| **Sentry** | Plugin | `/plugins` → `sentry` |
| **Datadog** | MCP | `.claude/mcp.json` (API key required) |
| **CircleCI** | MCP | `.claude/mcp.json` (API token required) |
| **PostgreSQL** | MCP | `.claude/mcp.json` (connection string) |

### Key Tips
- Plugins auto-update and handle auth via OAuth - prefer them over raw MCP
- Restart Claude Code after changing `mcp.json` (plugins take effect immediately)
- Use `env` for secrets in raw MCP config - never hardcode tokens
- Plugins live in `~/.claude/plugins/` - check installed ones with `/plugins`
- The Atlassian MCP respects your existing Jira/Confluence permissions - no extra access

---

## Chapter 7: Putting It All Together (5 min)

Here's how all pieces work together in a real Go backend workflow:

```
~/.claude/CLAUDE.md   → "Branch prefix <name>/, pre-push: make lint && make test/unit"
./CLAUDE.md           → "Go + PostgreSQL, UberFX DI, cache wrappers in internal/repo/cache/"
Memory                → "Redis pipe_exec latency is payload-driven. Use snappy compression."
Hooks                 → Auto-run goimports on every .go file edit, block go.sum edits
Agents                → code-reviewer checks diff for bugs, security, Go-specific issues
Skills                → /commitandcreatepr runs imports → lint → tests → review → PR
Plugin (Atlassian)    → "Create Jira tickets from the Confluence spec page"
```

### Recommended Setup for Your Team

**Day 1** (10 min):
1. Add `CLAUDE.md` with your team's conventions
2. Commit it so the whole team benefits

**Day 2** (15 min):
3. Add hooks for your language (formatter, linter)
4. Create a `/review` skill for your PR process

**Day 3** (20 min):
5. Set up MCP for your monitoring tools (Datadog, Sentry)
6. Create a custom agent for your most common task

### Quick Reference

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

---

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
```

---

*Workshop complete! Start with CLAUDE.md and hooks - they give the most value
for the least effort. Add skills and agents as you discover repeated workflows.*
