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
npm install -g @anthropic-ai/claude-code
or 
brew install --cask claude-code


# GitHub CLI (macOS)
brew install gh
```

## Auth Setup
- You need an Anthropic API key or organization SSO configured
- Run `claude` once and complete the login flow before the workshop

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
