# Chapter 1: CLAUDE.md - Global & Project Instructions (8 min)

**What**: A markdown file that gives Claude persistent instructions. Think of it as a `.editorconfig` but for AI behavior. It works at three levels:

| Level                  | Path                          | Who sees it            | Use case                                           |
| ---------------------- | ----------------------------- | ---------------------- | -------------------------------------------------- |
| **Global**             | `~/.claude/CLAUDE.md`         | Only you, all projects | Your personal workflow, git identity, editor prefs |
| **Project (shared)**   | `./CLAUDE.md` in repo root    | Whole team (committed) | Build commands, test patterns, code conventions    |
| **Project (personal)** | `./.claude/CLAUDE.md` in repo | Only you, this project | Your shortcuts, WIP notes                          |

**Why**: Without it, you repeat the same instructions every session ("run `make lint` not `golangci-lint`", "use Confettura not Florence for feature flags", etc.)

## Hands-on Exercise: Create a Global CLAUDE.md (5 min)

This is your **personal** instruction file that applies to every project. Open a Claude session:

```bash or zsh
claude
```

Then say:

```
Create ~/.claude/CLAUDE.md with these rules:

# Must Follow
- Must reply to my queries using Formal English
- Branch prefix: `<your-name>/`, Example: 'vikas/'
```

In your project directory. Create an empty CLAUDE.md, if it already exists then use the existing CLAUDE.md

```
# Must Follow
- Must reply to my queries using Pirated Slang
- Never read .env file directly. It's strictly prohibited.
```

**Verify**: Read `~/.claude/CLAUDE.md` and confirm the rules. Now in **every** project, Claude will follow this instruction

## Real-World Example: What a Mature Global CLAUDE.md Looks Like

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

## [Reference - Try Later] Create a Project-Level CLAUDE.md

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

## Key Tips

- Global CLAUDE.md = your personal defaults across ALL repos
- Project CLAUDE.md = team conventions (commit this to git)
- Keep both concise - Claude reads them every session
- Global rules are overridden by project rules when they conflict

---

Next: [Chapter 2 - Memory](./02-memory.md)
