# Chapter 4: Custom Agents (Subagents) (10 min)

**What**: Specialized AI agents defined in `.claude/agents/` that handle specific types of work. They run in their own context and can have restricted tools.

**Why**: Different tasks need different capabilities. A code reviewer shouldn't need Bash. A researcher doesn't need file editing. Agents also keep your main conversation clean - heavy work happens in a separate context window.

## How Agents Work

```
You (main conversation)
  |
  +-- Task(subagent_type="code-reviewer")  -> reads code + diff, can't edit
  +-- Task(subagent_type="go-test-writer") -> reads + writes + runs tests
  +-- Task(subagent_type="Explore")        -> built-in: codebase search
```

Agents are markdown files in `.claude/agents/`:
```
~/.claude/agents/my-agent.md     <- global agent
.claude/agents/my-agent.md       <- project agent
```

## [Live Exercise] Code Reviewer Agent (8 min)

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

## Using It

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

## [Reference - Try Later] Go Test Generator Agent

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
- Test file goes next to source: foo.go -> foo_test.go

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

## Key Concept: `allowed_tools` = Least Privilege

Notice how the two agents have different tool access:

| Agent | allowed_tools | Why |
|-------|--------------|-----|
| `code-reviewer` | Glob, Grep, Read, Bash | Reads code + runs `git diff`. Can't edit files. |
| `go-test-writer` | Glob, Grep, Read, **Write**, Bash | Needs to create test files. |

If you omit `allowed_tools`, the agent gets all tools. Always restrict to what's actually needed.

## Built-in Subagent Types (No Config Needed)

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

Next: [Chapter 5 - Skills](./05-skills.md)
