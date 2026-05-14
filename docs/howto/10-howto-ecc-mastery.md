# Chapter 10: Mastery — Building Your Own ECC Ecosystem

**Level: Master** | [← Parallelization](./09-howto-ecc-parallelization.md) | [Index →](./00-howto-ecc-index.md)

---

Mastery is not about using ECC's 208 skills — it's about building your own. This final chapter covers creating custom skills, designing agents, extending the system for your domain, contributing upstream, and scaling ECC across teams.

---

## Best Practice #1: Create Custom Skills from Git History

Your best skills already exist — in your commit history:

```bash
# Analyze current repo, generate skills
/skill-create

# Also generate instincts for continuous-learning-v2
/skill-create --instincts
```

This extracts patterns from actual commits: naming conventions, error handling approaches, architecture decisions, testing patterns. The output is a `SKILL.md` that encodes your team's real practices — not generic advice.

---

## Best Practice #2: Write Skills That Activate Correctly

The `When to Activate` section is the most important part of any skill. Be specific:

```markdown
# ✅ Good: specific, actionable triggers
## When to Activate
- Creating new API endpoints with Express or Fastify
- Adding middleware to the HTTP pipeline
- Implementing request validation with Zod schemas

# ❌ Bad: vague, too broad
## When to Activate
- Working with APIs
- Backend development
```

---

## Best Practice #3: Follow the Skill Development Checklist

Every custom skill should pass this checklist:

- [ ] Focused on one domain (not too broad)
- [ ] Includes `When to Activate` for auto-activation
- [ ] Includes practical, copy-pasteable code examples
- [ ] Shows anti-patterns (what NOT to do)
- [ ] Under 500 lines (800 max)
- [ ] Uses clear section headers
- [ ] Links to related skills
- [ ] No sensitive data (API keys, tokens, paths)
- [ ] YAML frontmatter with `name:` matching directory name
- [ ] `description:` is an inline string (not a block scalar)

---

## Best Practice #4: Design Custom Agents for Your Domain

Create agents that match your team's workflow:

```yaml
---
name: your-domain-reviewer
description: Reviews code for [your-domain] compliance, patterns, and performance
tools: ["Read", "Grep", "Glob"]
model: sonnet
---

You are a [your-domain] specialist reviewer.

## Your Role
- Check [domain-specific patterns]
- Verify [domain-specific compliance]
- Flag [domain-specific anti-patterns]

## Workflow
1. Read the changed files
2. Check against [domain] patterns from [your-skill]
3. Report findings with severity and fix suggestions

## Output Format
Return a structured review with:
- CRITICAL: must fix before merge
- HIGH: should fix soon
- MEDIUM: consider improving
- INFO: suggestion for future
```

### Agent Design Rules

| Rule | Why |
|------|-----|
| Minimum tools | Less scope = more focus |
| One clear input | Agent knows exactly what to do |
| One clear output | Orchestrator knows what it got |
| Reference skills in prompt | Agent follows your conventions |
| Set model appropriately | Don't waste opus on simple reviews |

---

## Best Practice #5: Build Reusable Workflows

The highest-leverage investment is building reusable patterns:

```
Invest once:                      Compound returns:
├── Custom skills           ──►   Auto-activate on relevant tasks
├── Custom agents           ──►   Delegate automatically
├── Project CLAUDE.md       ──►   Every session starts informed
├── Session templates       ──►   Consistent kickoff patterns
├── Custom hooks            ──►   Enforce standards without effort
└── Instinct collections    ──►   Team knowledge compounds
```

> "Early on, I spent time building reusable workflows/patterns. Tedious to build, but this had a wild compounding effect as models and agent harnesses improved." — @omarsar0

---

## Best Practice #6: Contribute Upstream

If your skill or agent is useful beyond your project, contribute it to ECC:

```bash
# Fork and clone
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# Create branch
git checkout -b feat/skill-your-skill-name

# Add your skill
mkdir -p skills/your-skill-name
# Create SKILL.md...

# Validate
npm test

# Submit
git add skills/your-skill-name/
git commit -m "feat(skills): add your-skill-name skill"
git push -u origin feat/skill-your-skill-name
```

### Contribution Conventions

- **Commit format**: `feat(skills): add rust-patterns skill`
- **PR title**: Match commit format
- **Skill adaptation**: If porting from another source, follow the [Skill Adaptation Policy](../docs/skill-adaptation-policy.md)
- **No external dependencies**: Prefer ECC-native solutions over requiring users to install unvetted packages

---

## Best Practice #7: Scale ECC Across Teams

### Team-Level Setup

```bash
# 1. Project-level CLAUDE.md with team conventions
# (checked into the repo)
.claude/
├── settings.local.json    # Team settings
└── rules/
    └── team-conventions.md

# 2. Shared instincts
/instinct-export > team-instincts.json
# Distribute to team, each member imports

# 3. Project-specific skills
skills/
└── your-project-patterns/
    └── SKILL.md           # Encodes team decisions
```

### Per-Developer Customization

```bash
# User-level rules (personal preferences)
~/.claude/rules/ecc/common/    # ECC common rules
~/.claude/rules/ecc/python/    # Language-specific
~/.claude/rules/personal/      # Personal preferences

# User-level settings
~/.claude/settings.json        # Model, env vars, MCPs
```

---

## Best Practice #8: Use agent-sort for Project-Specific ECC

Not every project needs all 208 skills. Use agent-sort to trim ECC to what matters:

```bash
# Sort all ECC components into DAILY vs LIBRARY for your repo
# agent-sort analyzes your actual code and determines relevance
```

This produces an evidence-backed install plan, classifying each skill, command, rule, and hook as either:
- **DAILY**: Use frequently, should always be loaded
- **LIBRARY**: Reference occasionally, load on demand

---

## Best Practice #9: Build Your Evaluation Harness

Masters don't guess if their setup works — they measure:

```
Fork conversation ──► Instance A: with skill
                  └── Instance B: without skill
                       │
                       ▼
                  Compare outputs
                       │
                       ▼
                  Measure: accuracy, token usage, time
```

The `eval-harness` skill formalizes this with:
- **Checkpoint evals**: Verify at defined points
- **Continuous evals**: Run every N minutes
- **pass@k / pass^k metrics**: Measure reliability

---

## Best Practice #10: The Master's Daily Workflow

```
Morning:
  1. claude-dev (loads dev context alias)
  2. Review previous session summary
  3. /instinct-status (check learned patterns)
  4. Plan the day's work with planner agent

Working:
  5. Sonnet for implementation, Opus for complex reasoning
  6. Fork for research, main for code
  7. /compact at logical breakpoints
  8. /learn when discovering non-obvious patterns

Wrapping up:
  9. /code-review on all changes
  10. Session summary auto-generated (Stop hook)
  11. Instincts extracted automatically
  12. /cost to check spending

Weekly:
  13. /evolve to cluster instincts into skills
  14. /prune to clean expired instincts
  15. Review and refine custom skills
  16. Share instincts with team
```

---

## The Mastery Checklist

- [ ] Created custom skills from your team's actual patterns
- [ ] Designed domain-specific agents with tight tool scoping
- [ ] Built reusable session templates and context files
- [ ] Implemented custom hooks for your quality standards
- [ ] Contributed at least one skill upstream to ECC
- [ ] Shared instincts across your team
- [ ] Set up project-specific ECC trim with agent-sort
- [ ] Built an eval harness to measure your setup
- [ ] Established a daily workflow that compounds learning
- [ ] Can explain the full ECC architecture to a newcomer

---

## Final Words

ECC is not a config file you install and forget. It's a **performance system** that compounds over time. The masters don't just use the 208 skills — they build new ones, share them, measure their impact, and continuously refine their workflow.

Start with Chapter 1. Build to Chapter 10. Then keep going — the best ECC setup is the one you're still improving.

---

[← Back to Index](./00-howto-ecc-index.md)
