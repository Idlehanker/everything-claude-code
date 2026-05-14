# Chapter 9: Parallelization & Multi-Agent Workflows

**Level: Expert** | [← Continuous Learning](./08-howto-ecc-continuous-learning.md) | [Next: Mastery →](./10-howto-ecc-mastery.md)

---

Running multiple Claude instances is powerful but expensive. The goal is always **maximum output with minimum viable parallelization**. This chapter covers the patterns for scaling agent workflows without losing control.

---

## Best Practice #1: Start with Forks, Not New Sessions

For non-overlapping tasks, use `/fork` before opening new terminals:

```bash
# Main session: implementing feature
# Fork for questions about the codebase
/fork

# Main continues implementation
# Fork explores, answers questions, researches
```

**Rule**: Main session for code changes, forks for questions and research.

---

## Best Practice #2: Use Git Worktrees for Parallel Code Changes

When multiple agents must edit overlapping files, use git worktrees:

```bash
# Create worktrees for parallel work
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-refactor refactor-branch

# Each worktree gets its own Claude instance
cd ../project-feature-a && claude
cd ../project-feature-b && claude
```

Each worktree is an independent checkout — no merge conflicts during development.

> **Critical**: Use `/rename <name>` to label all your parallel sessions. Without labels, you'll lose track within minutes.

---

## Best Practice #3: The Cascade Method

When running multiple instances, organize with the cascade pattern:

```
Tab 1 (oldest) ──► Tab 2 ──► Tab 3 ──► Tab 4 (newest)
     │                │           │          │
  Wrapping up    In progress   Starting   Just opened
```

- Open new tasks in new tabs to the right
- Sweep left to right, oldest to newest
- Focus on at most **3–4 tasks** at a time
- Don't add terminals arbitrarily — each one should arise from necessity

---

## Best Practice #4: The Two-Instance Kickoff

For new projects, start with exactly two instances:

| Instance 1: Scaffolding Agent | Instance 2: Research Agent |
|-------------------------------|---------------------------|
| Lays down project structure | Connects to services, web search |
| Creates configs (CLAUDE.md, rules) | Creates the detailed PRD |
| Sets up directory layout | Compiles architecture diagrams |
| Initializes git, CI/CD | Gathers documentation references |

The scaffolding agent builds the house. The research agent gathers the blueprints. They merge when both are done.

---

## Best Practice #5: Multi-Model Commands

ECC provides multi-model orchestration commands for complex workflows:

```bash
# Multi-agent task decomposition
/multi-plan "Build a user dashboard with analytics"

# Orchestrated execution across models
/multi-execute

# Backend-focused multi-service orchestration
/multi-backend

# Frontend-focused multi-service orchestration
/multi-frontend

# General multi-service workflows
/multi-workflow
```

> **Setup required**: These need the `ccg-workflow` runtime:
> ```bash
> npx ccg-workflow
> ```

---

## Best Practice #6: Use dmux for Agent Coordination

The `dmux-workflows` skill provides patterns for multi-agent orchestration using tmux pane management:

```bash
# Pattern: parallel agents in named tmux panes
tmux new-session -d -s agents
tmux split-window -h
tmux send-keys -t agents:0.0 'claude' Enter
tmux send-keys -t agents:0.1 'claude' Enter
tmux attach -t agents
```

dmux patterns work across Claude Code, Codex, OpenCode, and other harnesses.

---

## Best Practice #7: Define Clear Scope Boundaries

Every parallel instance needs explicit, non-overlapping scope:

```
✅ Good scope boundaries:
  Instance A: Frontend components (src/components/)
  Instance B: API routes (src/api/)
  Instance C: Database migrations (prisma/)

❌ Bad scope boundaries:
  Instance A: "Work on the user feature"
  Instance B: "Also work on the user feature"
  Instance C: "Fix stuff related to users"
```

Overlap = merge conflicts + wasted tokens + inconsistent implementations.

---

## Best Practice #8: Use PM2 for Service Management

For multi-service workflows, use PM2 to manage background processes:

```bash
/pm2                    # PM2 service lifecycle management
```

This keeps dev servers, workers, and background processes organized without consuming terminal tabs.

---

## Best Practice #9: Autonomous Loops with Guardrails

For autonomous agent loops, always implement safety:

```bash
# ✅ With guardrails: loop-operator agent manages execution
/loop-start

# Check loop status
/loop-status

# Monitor for stalls, intervene when needed
```

The `loop-operator` agent provides:
- Stall detection (heartbeat monitoring)
- Automatic intervention when agents loop
- Safety kill switches
- Session telemetry

> **Never**: `claude --dangerously-skip-permissions` on an autonomous loop. This skips all approval boundaries.

---

## Best Practice #10: Know When NOT to Parallelize

More terminals ≠ more productivity. Add a terminal only when:

- The task is truly independent
- Waiting for one task blocks another
- The scope is well-defined and non-overlapping
- You can actually monitor the output

**Your goal**: How much can you get done with the **minimum viable** amount of parallelization.

---

## What's Next

You can orchestrate multiple agents in parallel. Now let's go to the final level — building your own ECC ecosystem, creating custom components, and contributing upstream.

[Next: Chapter 10 — Mastery: Building Your Own ECC Ecosystem →](./10-howto-ecc-mastery.md)
