# Chapter 2: Understanding the Architecture

**Level: Newbie** | [← First Steps](./01-howto-ecc-first-steps.md) | [Next: Skills →](./03-howto-ecc-skills.md)

---

ECC is not a config file. It is a **layered performance system** with five distinct component types, each serving a precise role. Understanding how they interact is the foundation for everything that follows.

---

## The Five Layers

```
┌─────────────────────────────────────────────┐
│                  RULES                      │  Always-on guidelines
│         (loaded every session)              │  "What to always do"
├─────────────────────────────────────────────┤
│                  HOOKS                      │  Event-driven automations
│     (fire on tool calls & lifecycle)        │  "What to enforce automatically"
├─────────────────────────────────────────────┤
│                 SKILLS                      │  Domain knowledge & workflows
│      (context-activated or invoked)         │  "How to do things"
├─────────────────────────────────────────────┤
│                AGENTS                       │  Specialized subassistants
│       (delegated via Task tool)             │  "Who does the work"
├─────────────────────────────────────────────┤
│               COMMANDS                      │  Legacy slash entries
│        (user-invoked shortcuts)             │  "Compatibility shims"
└─────────────────────────────────────────────┘
```

### How They Interact

| Component | Activation | Scope | Relationship |
|-----------|-----------|-------|-------------|
| **Rules** | Always active | Global or language-scoped | Set the standards everything else follows |
| **Hooks** | Event-triggered | Tool calls, lifecycle events | Enforce rules automatically at runtime |
| **Skills** | Context-matched or explicit | Task-scoped knowledge bundles | Provide the knowledge agents and commands use |
| **Agents** | Delegated by orchestrator | Limited tools, focused role | Execute work using skills and rules |
| **Commands** | User-invoked via `/` | Session-scoped actions | Legacy entry points that invoke skills/agents |

---

## Best Practice #1: Understand the Component Hierarchy

**Rules are foundational.** They tell Claude _what_ standards to follow. Everything else builds on them.

**Skills are the canonical workflow surface.** They tell Claude _how_ to do things. New workflow logic should always land in `skills/` first.

**Commands are legacy compatibility.** They still work, but the durable logic lives in skills. Commands are maintained shims.

**Agents are delegated workers.** They receive a narrow scope and execute within it. They consume skills and follow rules.

**Hooks are automated enforcement.** They make sure rules are followed without human intervention.

```
User Request
    │
    ▼
Orchestrator (main Claude instance)
    │
    ├── Loads RULES (always)
    ├── Activates relevant SKILLS (contextual)
    ├── Delegates to AGENTS (when needed)
    │       └── Agent uses SKILLS + follows RULES
    └── HOOKS fire on tool calls throughout
```

---

## Best Practice #2: Know Where Everything Lives

```
everything-claude-code/
├── agents/           # 55 agent definitions (markdown + YAML frontmatter)
├── skills/           # 208 skill directories (SKILL.md per skill)
├── commands/         # 72 legacy slash commands
├── hooks/            # hooks.json + memory-persistence scripts
├── rules/            # Layered guidelines: common/ + language-specific/
│   ├── common/       # Universal (always install)
│   ├── typescript/   # TS/JS specific
│   ├── python/       # Python specific
│   ├── golang/       # Go specific
│   └── ...           # 12+ language ecosystems
├── scripts/          # Hook implementations, CLI tools, CI validators
├── mcp-configs/      # MCP server presets
├── contexts/         # Dynamic system prompt injection files
└── examples/         # Real-world project CLAUDE.md examples
```

### The Key Distinction: User vs Project Scope

| Scope | Location | When to Use |
|-------|----------|-------------|
| **User-level** | `~/.claude/rules/`, `~/.claude/agents/` | Applies to all your projects |
| **Project-level** | `.claude/rules/`, `.claude/settings.local.json` | Applies to one repo only |
| **Plugin** | Loaded from plugin manifest | Managed by the plugin system |

> **Best practice**: Keep ECC rules at user level (`~/.claude/rules/ecc/`). Keep project-specific overrides at project level (`.claude/`).

---

## Best Practice #3: Understand Agent Anatomy

Every agent is a markdown file with YAML frontmatter:

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

You are a senior code reviewer...
```

The four frontmatter fields matter:

| Field | Purpose | Best Practice |
|-------|---------|---------------|
| `name` | Lowercase identifier | Match the filename |
| `description` | When to invoke | Be specific — this drives auto-delegation |
| `tools` | Allowed capabilities | Minimum necessary — scope tightly |
| `model` | Complexity level | `haiku` for simple, `sonnet` for coding, `opus` for complex |

---

## Best Practice #4: Understand Skill Anatomy

Skills live in directories under `skills/`, each with a `SKILL.md`:

```yaml
---
name: tdd-workflow
description: Test-driven development with 80%+ coverage
origin: ECC
---

# TDD Workflow

## When to Activate
- Writing new features
- Fixing bugs
- Refactoring code

## Core Concepts
...
```

The `When to Activate` section is **critical** — it tells Claude when to load the skill automatically.

---

## Best Practice #5: Understand the Rules Layer

Rules are organized in a layered architecture:

```
rules/
├── common/              # Language-agnostic (always install)
│   ├── coding-style.md  # Immutability, file organization
│   ├── testing.md       # TDD, 80% coverage
│   ├── security.md      # Secret management, validation
│   ├── git-workflow.md  # Commit format, PR process
│   ├── patterns.md      # Design patterns
│   ├── hooks.md         # Hook architecture
│   ├── agents.md        # Delegation rules
│   └── performance.md   # Model selection, context mgmt
└── typescript/          # Extends common with TS specifics
```

**Priority rule**: When language-specific rules conflict with common rules, **language-specific wins**. This follows standard layered configuration (like CSS specificity).

---

## Best Practice #6: Understand the Hook Lifecycle

```
User sends request
    │
    ▼
Claude picks a tool (e.g., Bash, Edit, Write)
    │
    ▼
PreToolUse hooks fire ──► Can BLOCK (exit 2) or WARN (stderr)
    │
    ▼
Tool executes
    │
    ▼
PostToolUse hooks fire ──► Can analyze/format, cannot block
    │
    ▼
Claude responds
    │
    ▼
Stop hooks fire ──► Session end cleanup, pattern extraction
```

Hooks use exit codes to communicate:
- `0` — Continue (warnings go to stderr)
- `2` — Block the tool call (PreToolUse only)
- Other — Error (logged, does not block)

---

## Best Practice #7: Know the Cross-Harness Story

ECC is not Claude Code exclusive. The same components work across multiple harnesses:

| Harness | Support Level | Config Location |
|---------|--------------|-----------------|
| Claude Code | Native (primary) | Plugin or `~/.claude/` |
| Codex (macOS + CLI) | First-class | `.codex/` + `AGENTS.md` |
| Cursor | Full | `.cursor/` |
| OpenCode | Full plugin | `.opencode/` |
| Gemini CLI | Experimental | `.gemini/` |
| Antigravity | Integrated | `.agent/` |

> **Best practice**: If you work across harnesses, use `skills/` as your canonical source. Skills translate better than hooks across platforms.

---

## Mental Model Summary

Think of ECC as an operating system for your AI coding workflow:

| OS Concept | ECC Equivalent |
|-----------|----------------|
| Kernel config | Rules (always loaded) |
| System services | Hooks (event-driven daemons) |
| Applications | Skills (domain knowledge) |
| User processes | Agents (scoped workers) |
| Shell aliases | Commands (convenience shortcuts) |

When you internalize this mental model, every chapter that follows will click into place.

---

## What's Next

Now that you understand the architecture, let's dive into the most important surface: **skills** — what they are, how to use them, and how to write your own.

[Next: Chapter 3 — Skills: The Primary Workflow Surface →](./03-howto-ecc-skills.md)
