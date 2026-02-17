# Chapter 0: Prerequisites (do before the workshop)

Make sure these are set up **before** the session starts. We won't have time to debug installs.

## Required Tools

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

- brew install --cask claude-code
- From self-service install NetSkope SSL. 



- Use API Billing to login into Claude(You must activate your Claude code account before login)

Slack Channel: #premium-builder-ai-claude-code

# GitHub CLI (macOS)
brew install gh
```

## Auth Setup
- You need to activate Claude account before you can login.
- Use API Billing based login and then sign-in using SSO
- Run `claude` once and complete the login flow before the workshop
- If you dont see NetSkope, then run `NODE_TLS_REJECT_UNAUTHORIZED=0 claude` in terminal 

## Permission Prompts (important for first-time users)

Claude Code asks for permission before running commands or editing files. When you see a prompt:
- Press **y** to allow once
- Press **a** to always allow that tool for this session
- Press **n** to deny

For a smoother workshop experience, start Claude with pre-approved tools:
```bash
claude --allowedTools "Edit,Write,Bash(git *),Bash(make *),Bash(go *),Bash(gh *)"
```

---

Next: [Chapter 1 - CLAUDE.md](./01-claude-md.md)
