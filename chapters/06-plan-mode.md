# Chapter 6: Plan Mode - Think Before You Code (8 min)

**What**: Plan mode makes Claude explore your codebase, design an approach, and present a plan for your approval _before_ writing any code. It spawns subagents (from [Chapter 4](./04-agents.md)) to do research in parallel without polluting your main conversation.

**Why**: Without plan mode, Claude jumps straight to coding. For non-trivial tasks, this means wasted effort, wrong approaches, and refactoring. Plan mode puts the human in control: you approve the architecture before a single line is written.

## How Plan Mode Works

```
You: "Build a link validator for this repo"
  |
  v
Claude enters PLAN MODE
  |
  +-- Spawns Explore subagent --> scans repo structure, finds .md files
  +-- Spawns Explore subagent --> traces internal links between chapters
  |
  v
Claude presents a PLAN
  |
  +-- "Here's my approach: 3 files, Go CLI, uses filepath.Walk..."
  +-- "Shall I proceed?"
  |
  v
You APPROVE (or reject / ask questions)
  |
  v
Claude IMPLEMENTS the approved plan
  |
  +-- Spawns code-reviewer subagent --> reviews its own implementation
```

Key points:

- **Subagents run in separate context windows** - your main conversation stays clean
- **You can reject or modify the plan** - Claude doesn't write code until you say "go"
- **Plan mode can be triggered automatically** (for complex tasks) or manually with `/plan`

## [Live Exercise] Plan Mode Demo (6 min)

### Option A: Link Validator (Recommended - uses this repo)

Ask Claude to build something non-trivial in this workshop repo:

```
Plan how to build a Go CLI tool that reads all markdown files in this repo and reports
broken internal links (e.g., ./02-memory.md referenced from 01-claude-md.md
but the file doesn't exist). Put it in cmd/linkcheck/main.go. Use multiple sub-agents to speed up the process.
```

**What participants will see**:

1. Claude enters plan mode (visible in the UI)
2. An **Explore subagent** spawns to scan the repo - participants see it working in parallel
3. Claude presents a structured plan:
   - Files to create (`cmd/linkcheck/main.go`, `go.mod`)
   - Approach (filepath.Walk, regex for markdown links, os.Stat to verify)
   - Build target (`make linkcheck`)
4. You approve the plan
5. Claude implements, then spawns a **code-reviewer subagent** to check its own work
6. The tool runs and actually finds real broken links (or confirms all links are valid)

### Option B: Quick Version (if time is tight)

```
This workshop has 9 chapters (00-08). Create a README section that generates
a progress tracker table. Parse the chapter files to extract titles and
time estimates automatically.
```

This still triggers plan mode and subagents but produces a single file edit.

### Forcing Plan Mode

If Claude doesn't enter plan mode automatically, you can force it:

```
/plan Build a Go CLI tool that reads all markdown files in this repo...
```

Or prefix your request:

```
Let's plan this first before implementing: Build a Go CLI tool that...
```

## What the Audience Should Notice

| What They See                                    | What It Demonstrates                                    |
| ------------------------------------------------ | ------------------------------------------------------- |
| Claude pauses to explore before coding           | Plan mode prevents "code first, think later"            |
| Explore subagent appears in the UI               | Subagents work in parallel, in a separate context       |
| A structured plan with file paths and steps      | Claude thinks architecturally, not line-by-line         |
| You press "approve" to proceed                   | Human stays in control - Claude acts only after consent |
| Code-reviewer subagent runs after implementation | Agents compose - reviewer checks implementer's work     |
| The tool runs and produces real output           | Verification built into the workflow                    |

## When Plan Mode Activates

Plan mode triggers automatically when Claude detects:

- **Multi-file changes** - touching 3+ files
- **Architectural decisions** - choosing between approaches
- **New feature implementation** - not a simple bug fix
- **Unclear requirements** - needs exploration first

You can also configure it in CLAUDE.md (from [Chapter 1](./01-claude-md.md)):

```markdown
## Workflow

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately
```

## Plan Mode + Agents + Skills = The Full Loop

This is how all three features compose in a real workflow:

```
/commitandcreatepr (skill from Chapter 5)
  |
  +-- Plan Mode: "What files changed? What's the approach?"
  |     +-- Explore subagent: scans changed files
  |     +-- Plan presented and approved
  |
  +-- Implementation: Claude writes code
  |
  +-- code-reviewer agent (Chapter 4): reviews the diff
  |
  +-- Push + PR creation
```

## Key Tips

- Plan mode is NOT just for big features - use it for any task where the wrong approach wastes time
- Subagents spawned during planning don't count against your main context window
- You can ask questions or push back on the plan before approving
- Rejected plans are not wasted - Claude learns from your feedback and revises
- Combine with CLAUDE.md rules (e.g., "always plan before implementing") to make it the default

---

Next: [Chapter 7 - MCP & Plugins](./07-mcp-plugins.md)
