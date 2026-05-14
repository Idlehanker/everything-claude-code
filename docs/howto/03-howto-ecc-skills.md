# Chapter 3: Skills — The Primary Workflow Surface

**Level: Beginner** | [← Architecture](./02-howto-ecc-architecture.md) | [Next: Hooks →](./04-howto-ecc-hooks.md)

---

Skills are the **canonical workflow surface** in ECC. They are reusable knowledge modules that Claude loads based on context, providing domain expertise, workflow definitions, and actionable reference material. With 208 skills spanning 12+ language ecosystems, mastering skill usage is the single highest-leverage practice in ECC.

---

## Best Practice #1: Understand Skill Categories

Skills fall into five categories. Knowing which type you need saves time:

| Category | Purpose | Examples |
|----------|---------|---------|
| **Language Standards** | Idioms, conventions, best practices | `python-patterns`, `golang-patterns`, `rust-patterns` |
| **Framework Patterns** | Framework-specific conventions | `django-patterns`, `nextjs-turbopack`, `springboot-patterns` |
| **Workflows** | Step-by-step processes | `tdd-workflow`, `verification-loop`, `eval-harness` |
| **Domain Knowledge** | Specialized expertise | `security-review`, `api-design`, `mle-workflow` |
| **Tool Integration** | Library/service usage | `docker-patterns`, `mcp-server-patterns`, `e2e-testing` |

---

## Best Practice #2: Let Skills Activate Contextually

Skills have a `When to Activate` section that Claude uses for auto-activation. You don't need to invoke them manually — just describe your task:

```
You: "I need to add authentication to this Next.js app"
     → Claude auto-loads: frontend-patterns, security-review, api-design

You: "Fix the failing test suite"
     → Claude auto-loads: tdd-workflow, verification-loop

You: "Review this Go code for production readiness"
     → Claude auto-loads: golang-patterns, coding-standards, security-review
```

> **Anti-pattern**: Don't try to force-load every skill you think might be relevant. Context window is finite. Let Claude's context matching do its job.

---

## Best Practice #3: Know the Essential Skills

Out of 208 skills, these form the core daily toolkit:

### For Every Developer

| Skill | What It Does |
|-------|--------------|
| `coding-standards` | Cross-project naming, readability, immutability |
| `tdd-workflow` | Red-green-refactor with 80%+ coverage |
| `security-review` | Comprehensive security checklist before commits |
| `verification-loop` | Build, test, lint, typecheck, security gate |

### For Frontend

| Skill | What It Does |
|-------|--------------|
| `frontend-patterns` | React/Next.js state, rendering, performance |
| `e2e-testing` | Playwright patterns and Page Object Model |
| `frontend-slides` | HTML presentations from scratch or PPTX |

### For Backend

| Skill | What It Does |
|-------|--------------|
| `backend-patterns` | API design, database, caching patterns |
| `api-design` | REST conventions, pagination, error responses |
| `docker-patterns` | Compose, networking, container security |
| `deployment-patterns` | CI/CD, health checks, rollbacks |

### For Operations

| Skill | What It Does |
|-------|--------------|
| `strategic-compact` | Manual compaction at logical intervals |
| `documentation-lookup` | Live framework docs via Context7 MCP |
| `deep-research` | Multi-source research with citation |

---

## Best Practice #4: Skills Over Commands

Commands (`/plan`, `/code-review`, `/build-fix`) are maintained for backward compatibility, but the durable logic lives in skills. Prefer skills:

```
# ✅ Preferred: skill-based workflows
Use the tdd-workflow skill
Use the security-review skill

# ⚠️ Legacy: still works but is a compatibility shim
/tdd
/eval
/verify
```

New workflow development should always land in `skills/` first. Commands exist only as entry points that invoke the underlying skill.

---

## Best Practice #5: Compose Skills Through Agents

Skills become powerful when composed through agent delegation:

```
Planning Phase:
  └── planner agent → uses coding-standards + api-design skills

Implementation Phase:
  └── tdd-guide agent → uses tdd-workflow + language-patterns skills

Review Phase:
  └── code-reviewer agent → uses security-review + coding-standards skills

Verification Phase:
  └── e2e-runner agent → uses e2e-testing + verification-loop skills
```

Each agent loads only the skills relevant to its scope, keeping context clean.

---

## Best Practice #6: Use Decision Trees in Skills

Well-written skills include decision trees for complex choices:

```
Need to fetch data in React?
├── Single request → use fetch directly
├── Multiple independent → Promise.all()
├── Multiple dependent → await sequentially
├── With caching → use React Query / SWR
└── Server-side only → use Server Components

Database pattern?
├── Simple CRUD → Repository pattern
├── Complex queries → Query builder
├── Cross-service → Event-driven
└── Read-heavy → CQRS with read replicas
```

When using a skill, look for these trees first — they compress decision-making into seconds.

---

## Best Practice #7: Check Skill Quality Signals

Not all 208 skills are equal. Evaluate quality before relying on a skill:

| Signal | Good Skill | Weak Skill |
|--------|-----------|------------|
| `When to Activate` | Specific, actionable triggers | Vague or missing |
| Code examples | Copy-pasteable, tested | Pseudocode or hand-wavy |
| Anti-patterns | Shows what NOT to do | Only positive examples |
| Length | 200–500 lines, focused | >800 lines or too generic |
| Related skills | Links to complementary skills | Isolated |

---

## Best Practice #8: Use the Skill Creator for Custom Skills

Generate skills from your own git history:

```bash
# Local analysis (built-in)
/skill-create                    # Analyze current repo
/skill-create --instincts        # Also generate instincts for learning

# GitHub App (for 10k+ commits, auto-PRs)
# Comment on any issue: /skill-creator analyze
```

This extracts patterns from your actual commit history and creates SKILL.md files that encode your team's conventions.

---

## Best Practice #9: Keep Skills Focused

One skill = one domain. If you find yourself writing a skill that covers multiple unrelated concerns, split it:

```
# ✅ Good: focused skills
skills/
├── react-hook-patterns/      # Just React hooks
├── postgresql-indexing/      # Just PG indexes
└── pytest-fixtures/          # Just pytest fixtures

# ❌ Bad: too broad
skills/
├── react/                    # All of React?
├── databases/                # All databases?
└── python-testing/           # All Python testing?
```

---

## Best Practice #10: Reference Skills from Rules

Rules tell Claude _what_ to do. Skills tell Claude _how_. Link them:

```markdown
# In rules/common/testing.md
...
For the full TDD workflow, see `skills/tdd-workflow/`.
For E2E testing patterns, see `skills/e2e-testing/`.
```

This creates a pull-based knowledge system — rules set expectations, skills deliver the execution detail on demand.

---

## Quick Reference: Skill Discovery

```bash
# List all installed skills
/plugin list ecc@ecc

# Search skills by topic
# (Skills auto-activate, but you can browse)
ls skills/ | grep "pattern"
ls skills/ | grep "security"

# Check skill health
node scripts/skills-health.js
```

---

## What's Next

Skills give Claude knowledge. But how do you **enforce** that knowledge automatically? That's where hooks come in.

[Next: Chapter 4 — Hooks: Event-Driven Automation →](./04-howto-ecc-hooks.md)
