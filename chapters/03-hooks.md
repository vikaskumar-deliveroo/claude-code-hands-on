# Chapter 3: Hooks - Automated Guardrails (10 min)

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

## [Live Exercise] Create a hook in hooks.json (8 min)

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

## [Reference - Try Later] More Hook Ideas

Add more entries to the same arrays above:

```json
// Add to "PreToolUse" array - remind about lint before git commit:
{
  "matcher": "Bash",
  "hooks": [
    {
      "type": "command",
      "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'git commit'; then echo 'REMINDER: Run make lint && make test/unit before committing'; fi",
      "timeout": 3000
    }
  ]
}
```

Other ideas:

- Block writes to `main`/`master` branch files
- Auto-run `go vet` after Go file changes
- Log all Bash commands Claude runs (audit trail for security)

## [Demo] Auto-Save Knowledge Hook (Hooks + Memory)

This is where hooks get really powerful — combining them with Memory from Chapter 2. This `UserPromptSubmit` hook detects when you share project knowledge and nudges Claude to save it automatically.

**How it works**: The hook scans your message for knowledge-sharing patterns ("we use", "remember", "our database", "don't forget", etc.). When detected, it injects a reminder that Claude sees — telling it to save the information to project memory.

**File**: `~/.claude/hooks.json` (or add to your existing hooks)

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "INPUT=\"$CLAUDE_USER_INPUT\"; if echo \"$INPUT\" | grep -qiE '(remember|we use|our (database|db|stack|api|service|cache)|don.t forget|for future reference|FYI|note that|always use|never use|we switched|we migrated|the pattern is|convention is)'; then echo '⚡ KNOWLEDGE DETECTED: The user is sharing project knowledge. Save the key facts to your project memory (MEMORY.md or a topic file). Include: specific names, file paths, versions, and the reason WHY. Do this BEFORE responding to the rest of their message.'; fi",
            "timeout": 3000
          }
        ]
      }
    ]
  }
}
```

**Try it** — send any of these messages to Claude:

```
We use Aurora PostgreSQL 14 with read replicas for the restaurant service.
The main table has 500M rows.
```

```
Remember: Redis cluster mode is enabled in production.
Single-node mode is only for local dev.
```

```
FYI our cache invalidation uses Kafka consumers, not TTL expiry.
The consumer is in internal/kafka/cache_invalidator.go
```

Without the hook, Claude might just acknowledge and move on. **With the hook**, Claude will proactively save this to its memory files before responding — so it retains the knowledge across sessions.

**Why this is powerful**:

- You never have to explicitly say "save this to memory" — the hook handles it
- Knowledge accumulates automatically as you chat naturally
- Combines two features (Hooks + Memory) into something greater than either alone
- The hook is deterministic (regex match) but the action is intelligent (Claude decides what/how to save)

**Key insight for participants**: Hooks don't have to block or format code. They can **inject context** into Claude's prompt, shaping its behavior in real-time. The `echo` output from a hook becomes part of what Claude sees — making hooks a programmable steering mechanism.

---

Next: [Chapter 4 - Agents](./04-agents.md)
