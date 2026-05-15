# How to Start a Fresh Next.js Project with ECC

A step-by-step guide using Everything Claude Code agents, skills, hooks, and commands.

---

## 1. Scaffold the Project

```bash
cd ~/Projects
npx create-next-app@latest my-app
```

Choose App Router, TypeScript, Tailwind CSS, ESLint — the defaults work fine.

## 2. Initialize ECC for the Project

Change into the project directory and run:

```bash
cd my-app
```

Then invoke ECC project setup:

```
/ecc:project-init
```

This configures `~/.claude/settings.json` with relevant hooks (auto-format, lint, type-check).

## 3. Plan the Architecture

Before writing code, use the planner to design your app structure:

```
/ecc:plan
```

The planner produces a plan covering:
- Route structure (App Router layout)
- Data fetching strategy (RSC vs client components)
- State management approach
- Component tree design
- Dependencies and risks

## 4. Apply Relevant Skills During Development

| Phase | Skill | Purpose |
|-------|-------|---------|
| Architecture | `/ecc:frontend-patterns` | Component composition, state patterns |
| Performance | `/ecc:nextjs-turbopack` | RSC, server components, Turbopack dev speed |
| Design | `/ecc:frontend-design-direction` | Style system, visual direction |
| UI Motion | `/ecc:motion-ui` or `/ecc:motion-foundations` | Animation patterns |
| Styling | `/ecc:design-system` | Design tokens, component library setup |
| Accessibility | `/ecc:accessibility` | WCAG compliance, a11y patterns |

## 5. Feature Development Workflow

For each feature, follow the ECC development loop:

```
/ecc:plan          → Plan the feature
/ecc:feature-dev   → Implement with TDD loop
/ecc:code-review   → Review code quality
/ecc:security-scan → Security check before commit
```

## 6. Key Agents Available

| Agent | When to Use |
|-------|-------------|
| `architect` | System design decisions |
| `tdd-guide` | Writing tests first (RED → GREEN → REFACTOR) |
| `code-reviewer` | After writing code |
| `typescript-reviewer` | TypeScript-specific issues |
| `security-reviewer` | Before commits |
| `e2e-runner` | Critical user flow E2E tests |

## 7. Hooks That Help

Once ECC is configured, these hooks fire automatically in the background:

- **PostToolUse on Edit/Write**: Auto-format (Prettier), lint (ESLint), type-check (tsc --noEmit)
- **PreToolUse on Write**: Blocks oversized files (>800 lines)
- **Stop**: Final build verification at session end

## 8. ECC Rules That Apply

ECC installs rules tailored to your project's tech stack. For Next.js:

- `rules/typescript/` — TypeScript coding style, testing, security
- `rules/web/` — Web-specific patterns, performance budgets, CSP
- `rules/common/` — Git workflow, testing, security (all projects)

## Quick Reference: Common Commands

```bash
# Dev server (Turbopack by default in Next.js 16+)
npm run dev

# Build
npm run build

# ECC: plan a feature
/ecc:plan

# ECC: implement a feature
/ecc:feature-dev

# ECC: code review
/ecc:code-review
```
