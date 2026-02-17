# Chapter 5: Skills - Reusable Workflows (10 min)

**What**: Markdown files in `.claude/skills/` that teach Claude specialized workflows. Invoked with `/skill-name`. Skills can orchestrate agents from [Chapter 4](./04-agents.md).

**Why**: Codify your team's processes. Instead of explaining "how we do PRs" every time, make it a skill. Claude follows the exact steps every time - no forgetting, no shortcuts.

## Anatomy of a Skill

A skill is just a markdown file with a YAML frontmatter header:

```
~/.claude/skills/my-skill/SKILL.md    <- global skill
.claude/skills/my-skill/SKILL.md      <- project skill
```

The frontmatter tells Claude:
- `name` - the `/slash-command` name
- `description` - when to use it (Claude reads this to auto-suggest)
- `user_invocable: true` - enables `/name` trigger

## Real Example: `/commitandcreatepr` (Production Skill)

This is a real skill used daily on a Go backend. It automates the entire workflow from code changes to a draft PR - 9 steps that would otherwise be manual. Notice how **Step 6 delegates to the code-reviewer agent** from [Chapter 4](./04-agents.md):

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

### Step 6: Sub-Agent Code Review (MANDATORY)         <- uses agent!
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

## Using It

After writing your code, just type:

```
/commitandcreatepr
```

Claude handles all 9 steps automatically: format -> lint -> test -> commit -> **agent review** -> fix -> push -> PR -> CI monitoring. If anything fails, it stops and fixes before proceeding.

## How Skills and Agents Work Together

This is the key insight - **agents are the workers, skills are the orchestrators**:

```
/commitandcreatepr (skill - orchestrates the workflow)
  |
  +-- Step 2: make goimports        (Claude runs directly)
  +-- Step 3: make lint              (Claude runs directly)
  +-- Step 4: make test/unit         (Claude runs directly)
  +-- Step 5: git commit             (Claude runs directly)
  +-- Step 6: code-reviewer agent    (delegated to subagent) <- Chapter 4!
  +-- Step 7: git push               (Claude runs directly)
  +-- Step 9: gh pr checks --watch   (Claude runs directly)
```

## Quick Starter: Create Your Own Skill

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

## Key Tips
- Skills are project-specific (`.claude/skills/`) or global (`~/.claude/skills/`)
- Use `user_invocable: true` to enable `/slash-command` syntax
- Skills can delegate to agents (like the code review step above)
- Skill files support a `references/` subdirectory for templates, examples, etc.
- Great for: commit workflows, PR processes, deploy checklists, incident response

---

Next: [Chapter 6 - MCP & Plugins](./06-mcp-plugins.md)
