# Chapter 5: Agents & Subagent Orchestration

**Level: Intermediate** | [← Hooks](./04-howto-ecc-hooks.md) | [Next: Token Optimization →](./06-howto-ecc-token-optimization.md)

---

Agents are the workforce of ECC. While the main Claude instance orchestrates, agents handle delegated tasks with limited scope, focused tools, and specialized knowledge. With 55 agents spanning code review, security, building, testing, and ML workflows, knowing how to delegate effectively is a force multiplier.

---

## Best Practice #1: Delegate Proactively, Not Reactively

Don't wait for the user to ask. Use agents proactively based on the task:

| Situation | Delegate To | Why |
|-----------|-------------|-----|
| Complex feature request | `planner` | Breaks down work before coding starts |
| Code just written/modified | `code-reviewer` | Catches issues before they compound |
| Bug fix or new feature | `tdd-guide` | Enforces test-first discipline |
| Architectural decision | `architect` | Evaluates scalability and tradeoffs |
| Security-sensitive code | `security-reviewer` | Catches vulnerabilities humans miss |
| Build failure | `build-error-resolver` | Systematic error resolution |
| Database changes | `database-reviewer` | Schema design and query optimization |

---

## Best Practice #2: Scope Agent Tools Tightly

Every agent gets a `tools` list. Give them the **minimum** they need:

```yaml
# ✅ Good: scoped to read-only exploration
---
name: code-explorer
tools: ["Read", "Grep", "Glob"]
model: haiku
---

# ✅ Good: needs to edit but not run commands
---
name: doc-updater
tools: ["Read", "Write", "Edit", "Grep", "Glob"]
model: sonnet
---

# ❌ Bad: full access for a review agent
---
name: code-reviewer
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "WebFetch", "Task"]
model: opus
---
```

Tight scoping serves three purposes:
1. **Security**: Limits blast radius if something goes wrong
2. **Focus**: Fewer tools = more focused execution
3. **Cost**: Fewer tool descriptions = less context consumed

---

## Best Practice #3: Choose the Right Model Per Agent

Model selection per agent is one of the highest-leverage cost optimizations:

| Model | Cost | Best For | Example Agents |
|-------|------|----------|---------------|
| `haiku` | Lowest | Exploration, lookups, simple edits | `code-explorer`, `doc-updater` |
| `sonnet` | Medium | Day-to-day coding, reviews, tests | `code-reviewer`, `tdd-guide` |
| `opus` | Highest | Complex reasoning, security, architecture | `architect`, `security-reviewer` |

**Rule of thumb**: Start with `sonnet` for 90% of agents. Upgrade to `opus` only when:
- First attempt with `sonnet` failed
- Task spans 5+ files
- Architectural decisions are involved
- Security-critical analysis is needed

---

## Best Practice #4: Use the Iterative Retrieval Pattern

The biggest mistake with subagents is fire-and-forget. Subagents lack the orchestrator's semantic context. Use iterative retrieval:

```
1. Orchestrator sends task + objective context to subagent
2. Subagent returns initial results
3. Orchestrator evaluates the return
4. If insufficient: ask follow-up questions
5. Subagent goes back to source, refines, returns
6. Loop until sufficient (max 3 cycles)
```

**Key**: Pass the **objective context** ("we need this for the auth migration"), not just the query ("find all auth files").

---

## Best Practice #5: Use Sequential Phase Orchestration

For complex features, orchestrate agents in sequential phases:

```
Phase 1: RESEARCH
  └── code-explorer agent → produces research-summary.md

Phase 2: PLAN
  └── planner agent → reads research-summary.md → produces plan.md

Phase 3: IMPLEMENT
  └── tdd-guide agent → reads plan.md → produces code changes

Phase 4: REVIEW
  └── code-reviewer agent → reads changes → produces review-comments.md

Phase 5: VERIFY
  └── build-error-resolver (if needed) → fixes issues → done or loop back
```

### Rules for Sequential Orchestration

1. Each agent gets **one clear input** and produces **one clear output**
2. Outputs become inputs for the next phase
3. Never skip phases
4. Use `/clear` between agents to reset context
5. Store intermediate outputs in files

---

## Best Practice #6: Know the Agent Roster

ECC's 55 agents cover major development domains. Here are the ones you'll use most:

### Core Development

| Agent | Purpose | Model |
|-------|---------|-------|
| `planner` | Implementation planning | opus |
| `architect` | System design decisions | opus |
| `tdd-guide` | Test-driven development | sonnet |
| `code-reviewer` | Quality and security review | opus |
| `refactor-cleaner` | Dead code cleanup | sonnet |

### Language-Specific Reviewers

| Agent | Language | Model |
|-------|----------|-------|
| `typescript-reviewer` | TypeScript/JavaScript | sonnet |
| `python-reviewer` | Python | sonnet |
| `go-reviewer` | Go | sonnet |
| `rust-reviewer` | Rust | sonnet |
| `java-reviewer` | Java/Spring Boot | sonnet |
| `kotlin-reviewer` | Kotlin/Android/KMP | sonnet |
| `cpp-reviewer` | C/C++ | sonnet |

### Build Error Resolvers

| Agent | Language | Model |
|-------|----------|-------|
| `build-error-resolver` | General | sonnet |
| `go-build-resolver` | Go | sonnet |
| `rust-build-resolver` | Rust | sonnet |
| `java-build-resolver` | Java/Maven/Gradle | sonnet |
| `kotlin-build-resolver` | Kotlin/Gradle | sonnet |
| `pytorch-build-resolver` | PyTorch/CUDA | sonnet |

### Specialized

| Agent | Purpose | Model |
|-------|---------|-------|
| `security-reviewer` | Vulnerability detection | opus |
| `database-reviewer` | PostgreSQL/Supabase | sonnet |
| `e2e-runner` | Playwright E2E testing | sonnet |
| `mle-reviewer` | ML pipeline review | opus |
| `doc-updater` | Documentation sync | haiku |
| `docs-lookup` | API/docs questions | haiku |

---

## Best Practice #7: Agent Output Discipline

Agents should return **concise, actionable summaries** — not raw dumps:

```markdown
# ✅ Good agent return
## Review Summary
- 3 critical issues found (security)
- 2 high-priority improvements
- All tests passing
- Coverage: 84%

### Critical: SQL injection in user_service.py:47
  - Raw string interpolation in query
  - Fix: use parameterized queries

### Critical: Missing auth check in /api/admin endpoint
  - No @login_required decorator
  - Fix: add authentication middleware
```

```markdown
# ❌ Bad agent return
I reviewed the code and found some issues. The user_service.py file has
a potential SQL injection vulnerability. Also, there might be some
missing authentication... [500 more lines of prose]
```

---

## Best Practice #8: Pass Skills Into Agent Prompts

When spawning subagents, pass conventions from the relevant skill into the agent's prompt:

```
Orchestrator → "Review this Python code using the patterns from the
               python-patterns skill. Check against security-review
               checklist. Focus on the auth module."

Agent receives:
  - Task description
  - Relevant skill content (inline or referenced)
  - Specific scope to focus on
```

This ensures agents follow the same conventions as the rest of the system.

---

## Best Practice #9: Use Parallel Agents for Independent Work

When tasks are independent, launch agents simultaneously:

```
Independent tasks → parallel agents:
  ├── Agent A: Review frontend code
  ├── Agent B: Review backend code
  └── Agent C: Check database schemas

Dependent tasks → sequential agents:
  1. Agent A: Research → output.md
  2. Agent B: Plan (reads output.md) → plan.md
  3. Agent C: Implement (reads plan.md)
```

---

## Common Agent Anti-Patterns

| Anti-Pattern | Why It's Bad | Better Approach |
|-------------|-------------|-----------------|
| Using `opus` for exploration | Expensive overkill | Use `haiku` for file reading |
| Giving all tools to review agents | Over-scoped, risky | Read + Grep + Glob only |
| Fire-and-forget delegation | Misses context, low quality | Iterative retrieval (3 cycles max) |
| Huge context dumps to agents | Overwhelms, degrades output | Focused inputs, one clear task |
| Skipping the plan phase | Implementation without direction | Always start with planner agent |

---

## What's Next

Agents do the work, but they cost tokens. How do you optimize model selection, context management, and MCP hygiene to get maximum value from every dollar?

[Next: Chapter 6 — Token Optimization & Context Management →](./06-howto-ecc-token-optimization.md)
