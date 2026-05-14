# Chapter 8: Continuous Learning & Memory Persistence

**Level: Advanced** | [← Security](./07-howto-ecc-security.md) | [Next: Parallelization →](./09-howto-ecc-parallelization.md)

---

The most powerful ECC workflows don't just execute — they **learn**. Continuous learning and memory persistence turn every session into training data for the next one, compounding your efficiency over weeks and months.

---

## Best Practice #1: Understand the Memory Architecture

ECC has three memory layers:

| Layer | Mechanism | Persistence |
|-------|-----------|-------------|
| **Session memory** | Context within a single conversation | Lost on `/clear` or new session |
| **File-based memory** | `.claude/` context files, session summaries | Persists across sessions |
| **Instinct-based learning** | Patterns extracted into reusable skills | Permanent, evolving |

### The Hook Lifecycle for Memory

```
SessionStart hook → Load previous context
    ↓
Working session (hooks fire on tools)
    ↓
PreCompact hook → Save state before compaction
    ↓
Stop hook → Persist session state + extract patterns
    ↓
SessionEnd hook → Cleanup
```

---

## Best Practice #2: Use Session Summaries for Continuity

When ending a session, have Claude create a summary file:

```markdown
# Session Summary — 2026-05-13

## What Was Accomplished
- Implemented OAuth2 flow for /api/auth
- Added rate limiting middleware
- Fixed 3 failing test cases

## What Worked (Verified)
- Token refresh pattern using httpOnly cookies
- Redis-backed rate limiter with sliding window

## What Was Attempted But Failed
- JWT in localStorage (rejected — XSS risk)
- In-memory rate limiting (rejected — won't scale)

## What Remains
- [ ] Add CSRF protection to form endpoints
- [ ] Write E2E tests for login flow
- [ ] Update API documentation
```

**Key fields**: What worked (with evidence), what failed (and why), what's left. This prevents the next session from repeating failed approaches.

---

## Best Practice #3: Use the Memory Persistence Hooks

ECC includes pre-built hooks for automatic memory management at `hooks/memory-persistence/`:

| Hook | Event | What It Does |
|------|-------|-------------|
| Session start | SessionStart | Loads previous context automatically |
| Pre-compact | PreCompact | Saves important state before compaction |
| Session end | Stop | Persists learnings to a file |

These hooks create a `.claude/` context chain that bridges sessions without manual effort.

---

## Best Practice #4: Master Continuous Learning v2

ECC's instinct-based learning system (v2) automatically extracts patterns from your sessions:

```bash
# View learned instincts with confidence scores
/instinct-status

# Import instincts from teammates
/instinct-import <file>

# Export your instincts for sharing
/instinct-export

# Cluster related instincts into skills
/evolve

# Delete expired pending instincts
/prune
```

### How Instincts Work

1. **Stop hook** evaluates each session for extractable patterns
2. Patterns are saved as **instincts** with confidence scores
3. As confidence grows (repeated observations), instincts become reliable
4. `/evolve` clusters related instincts into formal skills

### Why Stop Hook, Not UserPromptSubmit

The pattern extraction uses the **Stop hook**, not UserPromptSubmit. UserPromptSubmit fires on every message (adding latency). Stop fires once at session end — lightweight, no runtime cost.

---

## Best Practice #5: Create Session Context Files

For complex, multi-day work, create explicit context files:

```bash
~/.claude/contexts/
├── dev.md           # Development mode
├── review.md        # Code review mode
├── research.md      # Research/exploration
└── sessions/
    ├── 2026-05-12-auth-refactor.md
    ├── 2026-05-13-auth-testing.md
    └── active.md    # Current session state
```

Start each session by pointing Claude at the relevant context:

```bash
claude --system-prompt "$(cat ~/.claude/contexts/sessions/active.md)"
```

---

## Best Practice #6: Use `/learn` and `/learn-eval` Mid-Session

Don't wait for session end to capture patterns:

```bash
# Extract patterns from current session
/learn

# Extract, evaluate, and save patterns with scoring
/learn-eval
```

Use these when you discover:
- A debugging technique that wasn't obvious
- A workaround for a framework quirk
- A project-specific pattern worth remembering

---

## Best Practice #7: Separate Project and Global Memory

| Scope | Location | Content |
|-------|----------|---------|
| **Global** | `~/.claude/` | Cross-project patterns, personal preferences |
| **Project** | `.claude/` | Project-specific conventions, architecture decisions |
| **Session** | `.claude/sessions/` | Temporary state for multi-session work |

Don't pollute global memory with project-specific details. Don't duplicate global patterns in every project.

---

## Best Practice #8: Rotate Memory After Untrusted Work

After working with foreign repos, external content, or untrusted data:

1. Review memory files for injected content
2. Reset or rotate project-level memory
3. Keep global memory narrow — only verified patterns

This prevents memory poisoning attacks (see Chapter 7).

---

## Best Practice #9: Build a Verification Loop

Combine learning with verification for compound improvement:

```
Session 1: Implement feature
    → Extract patterns via /learn
    → Save session summary

Session 2: Review and test
    → Load previous context
    → Verify against patterns
    → Refine instincts

Session 3: Next feature
    → Instincts auto-apply
    → Skip known-bad approaches
    → Faster, more accurate
```

The `verification-loop` and `eval-harness` skills formalize this:

- **Checkpoint evals**: Set explicit checkpoints, verify, fix before proceeding
- **Continuous evals**: Run every N minutes or after major changes
- **pass@k**: At least ONE of k attempts succeeds (use when you just need it to work)
- **pass^k**: ALL k attempts must succeed (use when consistency is essential)

---

## Best Practice #10: Share Instincts Across Teams

Instincts are portable. Export and share them:

```bash
# Export your instincts
/instinct-export > team-instincts.json

# Teammate imports
/instinct-import team-instincts.json
```

This creates a shared learning layer where one team member's discovery benefits everyone.

---

## What's Next

You've built a system that learns and remembers. Now let's scale it — running multiple agents in parallel, coordinating across worktrees, and orchestrating multi-model workflows.

[Next: Chapter 9 — Parallelization & Multi-Agent Workflows →](./09-howto-ecc-parallelization.md)
