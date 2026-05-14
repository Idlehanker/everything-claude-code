# Chapter 4: Hooks — Event-Driven Automation

**Level: Beginner** | [← Skills](./03-howto-ecc-skills.md) | [Next: Agents →](./05-howto-ecc-agents.md)

---

Hooks are the enforcement layer of ECC. While rules tell Claude _what_ to do and skills tell it _how_, hooks make sure it **actually happens** — automatically, without human intervention.

---

## Best Practice #1: Understand Hook Types and Timing

ECC hooks fire at specific points in the tool lifecycle:

```
User Request
    │
    ▼
Claude selects a tool
    │
    ├── PreToolUse ──► Validate, warn, or BLOCK (exit 2)
    │
    ▼
Tool executes (Bash, Edit, Write, Read, etc.)
    │
    ├── PostToolUse ──► Analyze, format, give feedback
    │
    ▼
Claude responds
    │
    ├── Stop ──► Audit, persist state, extract patterns
    │
    ▼
Session lifecycle
    ├── SessionStart ──► Load previous context
    ├── SessionEnd ──► Cleanup
    └── PreCompact ──► Save state before compaction
```

### Exit Code Contract

| Code | Meaning | Available In |
|------|---------|-------------|
| `0` | Success — continue | All hooks |
| `2` | Block the tool call | PreToolUse only |
| Other | Error (logged, no block) | All hooks |

---

## Best Practice #2: Know the Built-In Hooks

ECC ships battle-tested hooks. Know what's already running before writing your own:

### PreToolUse (Before Execution)

| Hook | What It Does | Exit |
|------|-------------|------|
| Dev server blocker | Blocks `npm run dev` outside tmux | 2 (blocks) |
| Tmux reminder | Suggests tmux for long commands | 0 (warns) |
| Git push reminder | Review before pushing | 0 (warns) |
| Pre-commit quality | Lints staged files, checks secrets | 2/0 |
| Doc file warning | Warns about non-standard .md files | 0 (warns) |
| Strategic compact | Suggests `/compact` every ~50 tool calls | 0 (warns) |

### PostToolUse (After Execution)

| Hook | What It Does |
|------|-------------|
| Prettier format | Auto-formats JS/TS after edits |
| TypeScript check | Runs `tsc --noEmit` after `.ts`/`.tsx` edits |
| Console.log warning | Warns about debug statements |
| Quality gate | Fast quality checks after edits |
| Design quality | Warns when UI drifts toward generic |

### Lifecycle

| Hook | Event | What It Does |
|------|-------|-------------|
| Session start | SessionStart | Loads previous context |
| Pre-compact | PreCompact | Saves state before compaction |
| Pattern extraction | Stop | Evaluates for extractable patterns |
| Session summary | Stop | Persists state when transcript available |

---

## Best Practice #3: Use Runtime Controls, Not File Edits

Never edit `hooks.json` directly for temporary changes. Use environment variables:

```bash
# Set strictness profile
export ECC_HOOK_PROFILE=minimal    # Lifecycle + safety only
export ECC_HOOK_PROFILE=standard   # Balanced (default)
export ECC_HOOK_PROFILE=strict     # Extra guardrails

# Disable specific hooks by ID
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"

# Control session start context
export ECC_SESSION_START_MAX_CHARS=4000
export ECC_SESSION_START_CONTEXT=off
```

This keeps the hook source clean and makes it easy to switch profiles per project or session.

---

## Best Practice #4: Write Hooks in Node.js, Not Bash

Hook scripts should be cross-platform. ECC uses Node.js scripts for all hook implementations:

```javascript
// scripts/hooks/my-hook.js
let data = '';
process.stdin.on('data', chunk => data += chunk);
process.stdin.on('end', () => {
  const input = JSON.parse(data);
  
  // Access tool info
  const toolName = input.tool_name;       // "Edit", "Bash", "Write"
  const toolInput = input.tool_input;     // Tool-specific params
  const filePath = toolInput?.file_path;
  const command = toolInput?.command;
  
  // Warn (non-blocking): write to stderr
  if (/console\.log/.test(toolInput?.new_string || '')) {
    console.error('[Hook] Warning: console.log detected');
  }
  
  // Block (PreToolUse only): exit with code 2
  // process.exit(2);
  
  // Always output original data to stdout
  console.log(data);
});
```

### Key Rules for Hook Scripts

1. **Keep under 200 lines** — extract helpers to `scripts/lib/`
2. **Exit 0 on non-critical errors** — never crash the workflow
3. **Use `run-with-flags.js` wrapper** — enables `ECC_HOOK_PROFILE` and `ECC_DISABLED_HOOKS`
4. **Blocking hooks (PreToolUse): keep fast (<200ms)** — no network calls
5. **Async hooks**: mark `"async": true` with timeout ≤30s

---

## Best Practice #5: Use Matchers Precisely

Hook matchers determine when a hook fires. Be specific:

```javascript
// ✅ Specific: only catches git push
tool == "Bash" && tool_input.command matches "git push"

// ✅ Specific: only TypeScript files
tool == "Edit" && tool_input.file_path matches "\\.tsx?$"

// ❌ Too broad: fires on every Bash command
tool == "Bash"

// ❌ Too broad: fires on every edit
tool == "Edit"
```

Overly broad matchers create noise. Every unnecessary hook invocation burns context and adds latency.

---

## Best Practice #6: Use Async for Slow Operations

Hooks that take more than a few hundred milliseconds should run asynchronously:

```json
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"npm|yarn|pnpm\"",
  "hooks": [{
    "type": "command",
    "command": "node scripts/hooks/build-analysis.js",
    "async": true,
    "timeout": 30
  }],
  "description": "Background analysis after build commands"
}
```

Async hooks run in the background. They **cannot** block tool execution, but they can log findings.

---

## Best Practice #7: Useful Hook Recipes

### Warn About TODO Comments

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "command",
    "command": "node -e \"let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const i=JSON.parse(d);if(/TODO|FIXME|HACK/.test(i.tool_input?.new_string||'')){console.error('[Hook] New TODO/FIXME added')}console.log(d)})\""
  }],
  "description": "Warn when adding TODO/FIXME comments"
}
```

### Block Large File Creation

```json
{
  "matcher": "Write",
  "hooks": [{
    "type": "command",
    "command": "node -e \"let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const i=JSON.parse(d);const lines=(i.tool_input?.content||'').split('\\n').length;if(lines>800){console.error('[Hook] BLOCKED: File exceeds 800 lines');process.exit(2)}console.log(d)})\""
  }],
  "description": "Block creation of files larger than 800 lines"
}
```

### Auto-Format Python with Ruff

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "command",
    "command": "node -e \"let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const i=JSON.parse(d);if(/\\.py$/.test(i.tool_input?.file_path||'')){require('child_process').execFileSync('ruff',['format',i.tool_input.file_path],{stdio:'pipe'})}console.log(d)})\""
  }],
  "description": "Auto-format Python files after edits"
}
```

---

## Best Practice #8: Never Edit Plugin hooks.json for Manual Installs

The `hooks/hooks.json` in the repo is plugin-oriented. For manual installs, use the installer:

```bash
# ✅ Correct: installer rewrites paths for your system
bash ./install.sh --target claude --modules hooks-runtime

# ❌ Wrong: raw copy breaks path resolution
cp hooks/hooks.json ~/.claude/hooks/hooks.json
```

The installer replaces `${CLAUDE_PLUGIN_ROOT}` with your actual install root.

---

## Best Practice #9: Start with `minimal`, Graduate to `strict`

| Profile | What Runs | Best For |
|---------|-----------|----------|
| `minimal` | Lifecycle + safety hooks only | Learning ECC, debugging hook issues |
| `standard` | Balanced quality + safety | Daily development (default) |
| `strict` | Extra reminders + guardrails | Production-critical work, team enforcement |

Graduate through profiles as you get comfortable:

```bash
# Week 1: minimal — get used to the system
export ECC_HOOK_PROFILE=minimal

# Week 2+: standard — let quality checks run
export ECC_HOOK_PROFILE=standard

# Production work: strict — full enforcement
export ECC_HOOK_PROFILE=strict
```

---

## Common Mistakes to Avoid

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Writing bash hooks | Breaks on Windows | Use Node.js scripts |
| Network calls in PreToolUse | Latency on every tool call | Move to PostToolUse or async |
| Forgetting `console.log(data)` on stdout | Hook silently breaks tool | Always pass through stdin to stdout |
| Not wrapping with `run-with-flags.js` | Runtime controls don't work | Use the wrapper for all hooks |
| Broad matchers | Noise, wasted context | Match specific tools and patterns |

---

## What's Next

Hooks enforce rules automatically. But who does the actual work? That's where agents come in — specialized subassistants that Claude delegates tasks to.

[Next: Chapter 5 — Agents & Subagent Orchestration →](./05-howto-ecc-agents.md)
