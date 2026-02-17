# Chapter 2: Memory - Persistent Learning (7 min)

**What**: Auto-memory lets Claude remember things across sessions. It lives in `~/.claude/projects/<project-hash>/memory/`.

**Why**: Claude forgets everything between sessions. Memory fixes that. It learns your patterns, project quirks, and past mistakes.

## Hands-on Exercise (5 min)

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

## Real-World Memory: What Grows Organically Over Time

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

## Practical Patterns to Save

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

## Key Tips
- Memory is per-project - it won't leak across repos
- Organize into topic files for large projects (Claude does this automatically)
- Keep memories factual and include file paths/metric names
- Say "forget about X" to remove outdated memories
- Memory older than a week may be stale - verify before relying on it

---

Next: [Chapter 3 - Hooks](./03-hooks.md)
