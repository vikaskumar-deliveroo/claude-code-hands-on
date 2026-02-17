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

## [Live Exercise] Create a Combined hooks.json (8 min)

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

Next: [Chapter 4 - Agents](./04-agents.md)
