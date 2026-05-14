# Chapter 6: Token Optimization & Context Management

**Level: Intermediate** | [← Agents](./05-howto-ecc-agents.md) | [Next: Security →](./07-howto-ecc-security.md)

---

Token usage is the invisible cost of agentic coding. This chapter covers practices that make the difference between hitting limits in an hour versus working productively all day.

---

## Best Practice #1: Set Optimal Defaults

Add to `~/.claude/settings.json`:

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

| Setting | Default | Recommended | Savings |
|---------|---------|-------------|---------|
| `model` | opus | **sonnet** | ~60% cost reduction |
| `MAX_THINKING_TOKENS` | 31,999 | **10,000** | ~70% hidden cost reduction |
| `CLAUDE_CODE_SUBAGENT_MODEL` | inherits main | **haiku** | ~80% cheaper subagents |

---

## Best Practice #2: Right Model for the Task

| Task | Model | Why |
|------|-------|-----|
| Exploration/search | Haiku | Fast, cheap |
| Simple edits | Haiku | Single-file, clear instructions |
| Multi-file implementation | Sonnet | Best coding balance |
| Complex architecture | Opus | Deep reasoning needed |
| Security analysis | Opus | Can't afford to miss vulnerabilities |
| Writing docs | Haiku | Structure is simple |

Default to Sonnet for 90%. Upgrade to Opus only when first attempt failed, 5+ files, architecture, or security-critical.

---

## Best Practice #3: MCP Context Budget

Each MCP injects tool definitions. With 14 MCPs, your 200K window shrinks to ~70K.

> **Rule**: Have 20–30 MCPs configured. Keep under 10 enabled. Under 80 tools active.

```bash
/mcp                           # Check what's enabled
```

Replace MCPs with CLI skills when possible — `gh` CLI instead of GitHub MCP eliminates 10+ tool definitions.

---

## Best Practice #4: Master Context Commands

| Command | When to Use |
|---------|-------------|
| `/clear` | Between unrelated tasks |
| `/compact` | At logical task breakpoints |
| `/cost` | Check token spending |

Compact after exploration (before implementation), after milestones, after debugging. Never compact mid-implementation or during active debugging.

---

## Best Practice #5: Subagents Protect Main Context

```
Main context reads 20 files → 20 files in your context
Subagent reads 20 files → returns 1 summary → 1 summary in your context
```

Use subagents for exploration, research, and analysis. Your main context stays clean.

---

## Best Practice #6: Dynamic System Prompt Injection

```bash
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'
```

Load different context for different work modes instead of everything via CLAUDE.md every session.

---

## Best Practice #7: Modular Codebase

| File Size | Recommendation |
|-----------|----------------|
| 100–400 lines | Ideal target |
| 400–800 lines | Consider splitting |
| 800+ lines | Split immediately |

Smaller files = fewer tokens per read + higher first-try accuracy.

---

## Best Practice #8: Toggle Extended Thinking

```
Alt+T / Option+T  — toggle thinking on/off
MAX_THINKING_TOKENS=10000   — good default
MAX_THINKING_TOKENS=0       — disable for trivial tasks
```

Enable full thinking for architecture/debugging. Disable for simple edits and docs.

---

## Best Practice #9: Use mgrep Over grep

~50% token reduction compared to grep/ripgrep. In benchmarks, mgrep + Claude Code used ~2x fewer tokens at similar quality.

---

## What's Next

[Next: Chapter 7 — Security: The Non-Negotiable Layer →](./07-howto-ecc-security.md)
