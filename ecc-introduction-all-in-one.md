# Everything Claude Code (ECC) — Introduction

> A performance optimization system for AI agent harnesses. **Not just configs. A complete system.** Evolved over 10+ months of intensive daily use by an Anthropic hackathon winner.

---

## What Is ECC?

ECC is an **open-source plugin and performance system** that extends AI coding assistants (Claude Code, Cursor, Codex CLI, OpenCode, Gemini, and others) with:

- **55 specialized agents** for delegation
- **208 domain-specific workflow skills**
- **72 production slash commands**
- **20+ automated hook scripts** (session persistence, cost tracking, security, etc.)
- **12 language ecosystems of curated rules**
- **14 MCP server configurations**
- **A continuous learning system** that auto-extracts patterns from your sessions
- **Cross-harness support** — same configs work across Claude Code, Cursor, Codex, OpenCode, Gemini, and more

Published as `ecc-universal` on npm, `ecc@ecc` on the Claude marketplace, and `affaan-m/everything-claude-code` on GitHub.

---

## Quick Start

Choose **exactly one** install path:

### Plugin (Recommended for Claude Code)
```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install ecc@ecc
```
Then manually copy only the `rules/` folders you need:
```
mkdir -p ~/.claude/rules/ecc
cp -R rules/common ~/.claude/rules/ecc/
cp -R rules/typescript ~/.claude/rules/ecc/
```

### Manual Install (Fallback)
```
npx ecc-install --profile full
```

### Minimal (No Hooks)
```
npx ecc-install --profile minimal --target claude
```

---

## How It Works — The Components

ECC organizes its functionality into these layers:

### Agents (`agents/`)
Pre-built subagent definitions you can delegate tasks to. Each is a Markdown file with YAML frontmatter specifying name, description, allowed tools, and model.

**Examples:** `planner`, `code-reviewer`, `tdd-guide`, `security-reviewer`, `build-error-resolver`, `e2e-runner`, `rust-reviewer`, `go-reviewer`, `python-reviewer`, `database-reviewer`, `loop-operator`, `harness-optimizer`

**How to use:** Spawn an agent when a task fits its specialty:
- Complex feature → spawn `planner`
- Code just written → spawn `code-reviewer`
- Build failing → spawn `build-error-resolver`

### Skills (`skills/`)
The primary workflow surface. Each skill is a reusable, structured guide (or multi-file bundle) for a specific domain or workflow pattern.

**Examples:** `tdd-workflow`, `security-review`, `django-patterns`, `postgres-patterns`, `rust-testing`, `continuous-learning`, `e2e-testing`, `api-design`, `deployment-patterns`

Skills are invoked via slash commands (legacy) or loaded directly when their domain is relevant.

### Commands (`commands/`)
Slash-command entry points like `/plan`, `/tdd`, `/code-review`, `/e2e`, `/build-fix`, `/learn`, `/skill-create`. These are convenience shims that invoke underlying skills. The durable logic lives in skills; commands are the migration-compatible surface.

### Hooks (`hooks/`)
Trigger-based automations that fire on tool and lifecycle events. ECC provides 20+ hook scripts:

| Hook Type | What It Does |
|-----------|-------------|
| PreToolUse (Bash) | Commit quality checks, tmux reminders, dev server blocking, git push reminders |
| PostToolUse | Formatting, type checking, command logging, build completion detection |
| SessionStart | Session persistence, context loading, observer startup |
| Stop | Session summaries, cost tracking, governance capture |
| PreCompact | Manual compaction suggestions |

All hooks route through a `run-with-flags.js` wrapper that enables runtime gating (`ECC_HOOK_PROFILE=minimal|standard|strict` and `ECC_DISABLED_HOOKS`).

### Rules (`rules/`)
Always-active guidelines organized by language ecosystem: `common/`, `typescript/`, `python/`, `golang/`, `rust/`, `java/`, `kotlin/`, `cpp/`, `csharp/`, `dart/`, `swift/`, `php/`, `perl/`, plus framework-specific subdirectories.

### MCP Configs (`mcp-configs/`)
Pre-configured MCP server definitions for external integrations.

### ECC CLI (`npx ecc`)
A command-line tool for managing the ECC installation:
```
npx ecc status          # Show installation state
npx ecc doctor          # Diagnose issues
npx ecc repair          # Restore ECC-managed files
npx ecc consult         # Ask which components to install
npx ecc uninstall       # Remove ECC-managed files
```

### Dashboard GUI
```
npm run dashboard       # Tkinter desktop app
```
Visually explore all agents, skills, commands, and rules with dark/light theme.

---

## How ECC Differs from Claude Code's Built-in Features

Claude Code ships with a _framework_ for hooks, skills, agents, commands, and rules. ECC is a **production-hardened collection** built on top of that framework. Here's the concrete difference:

### Skills: 208 curated vs. a handful built-in

| Vanilla Claude Code | ECC |
|---------------------|-----|
| Ships a few built-in skills (`/help`, `/clear`, etc.) | Adds 208 domain-specific skills |
| Skills are loaded individually from your project | Ships complete skill directories: `tdd-workflow`, `security-review`, `api-design`, `django-patterns`, `postgres-patterns`, `rust-testing`, `e2e-testing`, and 200+ more |
| No multi-file skill support | Skills can be multi-file bundles with codemaps, supporting files, and structured SKILL.md definitions |
| No built-in workflow for most domains | Covers frontend, backend, mobile, ML, security, devops, compliance, healthcare, finance, and more |

### Agents: 55 pre-built subagents vs. writing your own

| Vanilla Claude Code | ECC |
|---------------------|-----|
| You define agents from scratch in `.claude/agents/` | Ships 55 ready-to-use agents with tested prompts, tool scoping, and model assignments |
| No language-specific reviewers | Includes reviewers and build-resolvers for TypeScript, Python, Go, Rust, Java, Kotlin, C++, C#, Swift, Dart, and more |
| No orchestration patterns | Includes planner, architect, loop-operator, harness-optimizer, and other orchestration agents |
| Basic subagent spawning | Agents include tool restrictions and preferred model assignments (Opus for planning, Sonnet for reviews) |

### Hooks: 20+ production scripts vs. writing from scratch

| Vanilla Claude Code | ECC |
|---------------------|-----|
| You write hook scripts from scratch | Ships 20+ production-tested hook scripts |
| No built-in session persistence | SessionStart/Stop hooks save/load context across sessions automatically |
| No cost tracking | Cost tracker hooks monitor token spending during sessions |
| Manual hook JSON editing | Hook runtime gating via `ECC_HOOK_PROFILE` and `ECC_DISABLED_HOOKS` env vars |
| No security scanning hooks | Security monitor hooks with AgentShield integration |
| No commit quality checks | Pre-Bash hooks enforce commit conventions and block `--no-verify` |
| No MCP health monitoring | MCP health check hooks validate server connectivity |

### Rules: 12 language ecosystems vs. your own

| Vanilla Claude Code | ECC |
|---------------------|-----|
| You write CLAUDE.md rules per project | Ships rule directories for 12 language ecosystems |
| No language-specific defaults | Each language directory covers coding style, security, testing, and common patterns |
| No framework-specific rules | Includes directories for Django, Spring Boot, Laravel, Next.js, Angular, Kotlin Ktor/Exposed, and more |

### Cross-Harness: Works beyond Claude Code

ECC is not limited to Claude Code. The same configs adapt to:

| Harness | How ECC Works There |
|---------|---------------------|
| **Claude Code** | Native — primary target. Plugin + hooks + commands |
| **Cursor** | Rules via `.cursor/rules/`, agents via AGENTS.md, hooks via Cursor's hook system |
| **Codex CLI** | AGENTS.md-based agent definitions, config.toml for MCPs |
| **OpenCode** | Plugin module with 6 native custom tools, 35 commands, 37 skills |
| **Gemini** | Adapted agents via `scripts/gemini-adapt-agents.js` |
| **Qwen / Trae / Kiro** | Community-adapted configs |

### Beyond Built-in Features

ECC also includes things that don't correspond to any built-in Claude Code feature:

- **Continuous Learning System** — Auto-extracts patterns from your sessions into reusable skills via instinct-based learning with confidence scoring, import/export, and evolution
- **Selective Install System** — Manifest-driven install pipeline with state tracking, incremental updates, and profiles (minimal, core, full). Tracks exactly what's installed and enables clean uninstall
- **ECC CLI & Dashboard** — Command-line management tools (`ecc doctor`, `ecc repair`, `ecc consult`) and a desktop GUI for exploring components
- **Multi-Model Orchestration** — `ccg-workflow` runtime for orchestrating across multiple model instances
- **Security Scanning** — AgentShield integration with CVE scanning, secret detection, and vulnerability analysis
- **Operator Workflows** — brand-voice, social-graph-ranker, connections-optimizer, and other operator-level automations

---

## Architecture Overview

```
everything-claude-code/
  agents/          # 55 subagent definitions (Markdown + YAML frontmatter)
  skills/          # 208 domain skills (one subdirectory per skill)
  commands/        # 72 slash command shims
  hooks/           # Hook configs + hooks.json
  rules/           # 12 language ecosystems of guidelines
    common/        #   Universal rules (security, coding style)
    typescript/    #   TypeScript-specific rules
    python/        #   Python-specific rules
    golang/        #   Go-specific rules
    ...            #   (rust, java, kotlin, cpp, csharp, dart, swift, php, perl)
  scripts/         # Node.js utilities, hook handlers, CI validators
    hooks/         #   Individual hook handler scripts
    lib/           #   Shared helper modules
    ci/            #   CI validation scripts
  mcp-configs/     # MCP server configuration presets
  schemas/         # JSON Schema files for validation
  tests/           # Test suite mirroring scripts/ structure
```

---

## When to Use Which Component

| Goal | Use This |
|------|----------|
| Plan a complex feature | `/plan` command → spawns planner agent |
| Write tests first | `/tdd` command → tdd-guide skill/agent |
| Review code quality | `/code-review` → code-reviewer agent |
| Fix build errors | `/build-fix` → build-error-resolver agent |
| Run E2E tests | `/e2e` → e2e-runner agent |
| Security audit | `/security-scan` → security-reviewer agent |
| Learn from sessions | `/learn` → continuous-learning skill |
| Generate a skill from git history | `/skill-create` |
| Check installation health | `npx ecc doctor` |
| Find the right install profile | `npx ecc consult "topic"` |
| Explore components visually | `npm run dashboard` |

---

## Getting Help

- **Shorthand Guide** — Setup, foundations, philosophy. Read this first.
- **Longform Guide** — Token optimization, memory persistence, evals, parallelization.
- **Security Guide** — Attack vectors, sandboxing, sanitization, CVEs, AgentShield.
- **GitHub Issues** — Bug reports and feature requests.
- **GitHub Discussions** — Questions and community support.
