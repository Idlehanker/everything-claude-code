# Chapter 1: First Steps — Installation & Configuration

**Level: Newbie** | [← Index](./00-howto-ecc-index.md) | [Next: Architecture →](./02-howto-ecc-architecture.md)

---

Getting started with ECC is deceptively simple — but the difference between a clean install and a broken one usually comes down to following one rule: **pick one install path, and stick with it.**

---

## Best Practice #1: Choose Exactly One Install Path

ECC offers three ways to install. They are **not** stackable. The most common broken setup is layering multiple methods:

| Path | When to Use | What It Does |
|------|-------------|--------------|
| **Plugin install** | Recommended default | Loads skills, commands, hooks, agents automatically via Claude Code's plugin system |
| **Manual installer** | Want fine-grained control or plugin issues | Copies components into `~/.claude/` directories |
| **Selective manual** | Minimalist / specific languages only | Copy only the exact files you need |

```bash
# ✅ Plugin install (recommended)
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install ecc@ecc

# ✅ Manual install (alternative — do NOT combine with plugin)
./install.sh --profile full

# ✅ Minimal install (no hooks)
./install.sh --profile minimal --target claude
```

> **Critical**: Never run `./install.sh --profile full` after `/plugin install ecc@ecc`. This creates duplicate skills, duplicate hooks, and runtime confusion.

---

## Best Practice #2: Install Rules Separately

The Claude Code plugin system **cannot** distribute rules. This is an upstream limitation, not an ECC choice. After a plugin install, manually copy only the rule sets you need:

```bash
# Clone the repo first
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# Create a namespaced rules directory
mkdir -p ~/.claude/rules/ecc

# Copy common rules (always needed)
cp -R rules/common ~/.claude/rules/ecc/

# Copy ONLY the language packs you use
cp -R rules/typescript ~/.claude/rules/ecc/   # for TS/JS projects
cp -R rules/python ~/.claude/rules/ecc/       # for Python projects
cp -R rules/golang ~/.claude/rules/ecc/       # for Go projects
```

### Why Namespace Under `ecc/`?

Copying rules into `~/.claude/rules/ecc/` prevents collisions with your own project rules or other plugins. The directory structure preserves internal references between common and language-specific files.

> **Anti-pattern**: Don't flatten files with `cp rules/**/*.md ~/.claude/rules/`. Common and language-specific directories share filenames — flattening overwrites common rules with language-specific ones.

---

## Best Practice #3: Use `consult` Before Installing

Not sure what you need? Use the built-in advisor:

```bash
npx ecc consult "security reviews" --target claude
npx ecc consult "mlops training model deployment" --target claude
```

This returns matching components, related profiles, and preview/install commands. **Always preview before installing.**

---

## Best Practice #4: Verify Your Installation

After installing, validate immediately:

```bash
# Check what's installed
/plugin list ecc@ecc

# Or via CLI
node scripts/ecc.js list-installed

# Run diagnostics
node scripts/ecc.js doctor

# Repair if needed
node scripts/ecc.js repair
```

If something looks duplicated or broken:

```bash
# Preview what would be removed
node scripts/uninstall.js --dry-run

# Then clean up
node scripts/uninstall.js
```

---

## Best Practice #5: Configure Hook Runtime Controls Early

Hooks are powerful but can be overwhelming for beginners. Control them from day one:

```bash
# Set strictness level (default: standard)
export ECC_HOOK_PROFILE=standard   # Options: minimal | standard | strict

# Disable specific hooks if they annoy you
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder"

# Cap session start context (default: 8000 chars)
export ECC_SESSION_START_MAX_CHARS=4000

# Or disable session start context entirely (for low-context setups)
export ECC_SESSION_START_CONTEXT=off
```

Add these to your shell profile (`~/.bashrc`, `~/.zshrc`) so they persist across sessions.

---

## Best Practice #6: Set Up Your Package Manager

ECC auto-detects your package manager, but explicit configuration prevents surprises:

```bash
# Set globally
node scripts/setup-package-manager.js --global pnpm

# Set per-project
node scripts/setup-package-manager.js --project bun

# Check current detection
node scripts/setup-package-manager.js --detect
```

Or use the command inside Claude Code:

```
/setup-pm
```

---

## Best Practice #7: Start Minimal, Expand Deliberately

Don't install everything at once. Start with the core profile and add modules as you need them:

```bash
# Start minimal — rules, agents, skills, no hooks
./install.sh --profile minimal --target claude

# Add hooks later only if you want runtime enforcement
./install.sh --target claude --modules hooks-runtime

# Add specific capability bundles
npx ecc install --profile minimal --target claude --with capability:machine-learning
```

This approach keeps your context window clean and your cognitive load low while you learn the system.

---

## Common Mistakes to Avoid

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Stacking plugin + manual install | Duplicate hooks, double enforcement | Uninstall one path completely |
| Copying all rule directories | Loading 12 language packs you don't use | Copy only `common/` + your stack |
| Ignoring `ECC_HOOK_PROFILE` | Hooks fire on everything, confusing output | Set to `minimal` while learning |
| Skipping `doctor` after install | Silent misconfigurations persist | Run `node scripts/ecc.js doctor` |
| Editing `hooks.json` directly | Raw file is plugin-oriented, paths may break | Use installer or runtime env vars |

---

## Quick Verification Checklist

After your first install, confirm these:

- [ ] `plugin list ecc@ecc` shows agents, commands, and skills
- [ ] Rules exist at `~/.claude/rules/ecc/common/`
- [ ] `node scripts/ecc.js doctor` reports no critical issues
- [ ] Running `/ecc:plan "test"` produces a planner response
- [ ] `ECC_HOOK_PROFILE` is set in your shell profile

---

## What's Next

You're installed. Now understand **why** ECC is built the way it is — the architecture behind agents, skills, hooks, rules, and commands.

[Next: Chapter 2 — Understanding the Architecture →](./02-howto-ecc-architecture.md)
