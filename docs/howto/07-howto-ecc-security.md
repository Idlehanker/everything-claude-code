# Chapter 7: Security — The Non-Negotiable Layer

**Level: Advanced** | [← Token Optimization](./06-howto-ecc-token-optimization.md) | [Next: Continuous Learning →](./08-howto-ecc-continuous-learning.md)

---

Agent security is infrastructure, not an afterthought. In 2026, prompt injection is not a toy problem — it's shell execution, secret exposure, and silent lateral movement. This chapter distills the security practices every ECC user must follow.

---

## The Threat Model

Everything an LLM reads is executable context. There is no meaningful distinction between "data" and "instructions" once text enters the context window.

**Attack vectors include**: poisoned repos, malicious PRs, hidden Unicode in skills, MCP tool poisoning, email/PDF attachments, and memory persistence exploits.

> **One rule**: Never let the convenience layer outrun the isolation layer.

---

## Best Practice #1: Separate Agent Identity

Never give agents your personal accounts:

| Resource | Bad | Good |
|----------|-----|------|
| Email | Your personal Gmail | `agent@yourdomain.com` |
| GitHub | Your personal token | Scoped bot token, short-lived |
| Slack | Your account | Dedicated bot user |
| SSH keys | Your `~/.ssh/` | Separate agent keypair |

If your agent has the same accounts you do, a compromised agent **is you**.

---

## Best Practice #2: Sandbox Untrusted Work

For untrusted repos, attachment-heavy workflows, or foreign content — isolate:

```yaml
# Docker Compose: no egress, minimal privileges
services:
  agent:
    build: .
    user: "1000:1000"
    working_dir: /workspace
    volumes:
      - ./workspace:/workspace:rw
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    networks:
      - agent-internal

networks:
  agent-internal:
    internal: true    # No internet access
```

For quick one-off reviews:

```bash
docker run -it --rm \
  -v "$(pwd)":/workspace \
  -w /workspace \
  --network=none \
  node:20 bash
```

---

## Best Practice #3: Restrict Tools and Paths

Deny access to sensitive locations by default:

```json
{
  "permissions": {
    "deny": [
      "Read(~/.ssh/**)",
      "Read(~/.aws/**)",
      "Read(**/.env*)",
      "Write(~/.ssh/**)",
      "Write(~/.aws/**)",
      "Bash(curl * | bash)",
      "Bash(ssh *)",
      "Bash(scp *)",
      "Bash(nc *)"
    ]
  }
}
```

If a workflow only needs to read a repo and run tests, don't let it read your home directory.

---

## Best Practice #4: Require Approval Boundaries

The model should never be the final authority for:

- Shell execution outside sandbox
- Network egress
- Reads from secret-bearing paths
- Writes outside the repo
- Workflow dispatch or deployment

> **Never** use `--dangerously-skip-permissions` on autonomous loops.

---

## Best Practice #5: Sanitize All Input

### Scan for Hidden Unicode

```bash
# Zero-width and bidi control characters
rg -nP '[\x{200B}\x{200C}\x{200D}\x{2060}\x{FEFF}\x{202A}-\x{202E}]'

# Hidden HTML, base64 payloads
rg -n '<!--|<script|data:text/html|base64,'
```

### Scan Skills and Hooks

```bash
rg -n 'curl|wget|nc|scp|ssh|enableAllProjectMcpServers|ANTHROPIC_BASE_URL'
```

### Sanitize Attachments

- Extract only the text you need
- Strip comments and metadata
- Don't feed live external links to a privileged agent
- Separate parsing agents from action-taking agents

---

## Best Practice #6: Use AgentShield

ECC includes AgentShield — a security scanner for your configuration:

```bash
# Quick scan
npx ecc-agentshield scan

# Auto-fix safe issues
npx ecc-agentshield scan --fix

# Deep adversarial analysis with three Opus agents
npx ecc-agentshield scan --opus --stream

# Generate secure config from scratch
npx ecc-agentshield init
```

It scans CLAUDE.md, settings.json, MCP configs, hooks, agent definitions, and skills for: secrets (14 patterns), permission issues, hook injection, MCP risk, and agent config problems.

---

## Best Practice #7: Log Everything

If you can't see what the agent read, called, and tried to reach, you can't secure it.

Log at minimum:
- Tool name and input summary
- Files touched
- Approval decisions
- Network attempts
- Session/task ID

```json
{
  "timestamp": "2026-03-15T06:40:00Z",
  "session_id": "abc123",
  "tool": "Bash",
  "command": "curl -X POST https://example.com",
  "approval": "blocked",
  "risk_score": 0.94
}
```

---

## Best Practice #8: Implement Kill Switches

For autonomous loops, always have a kill path:

```javascript
// Kill the whole process group, not just the parent
process.kill(-child.pid, "SIGKILL");
```

Implement a heartbeat-based dead-man switch:
1. Supervisor starts task
2. Task writes heartbeat every 30s
3. Supervisor kills process group if heartbeat stalls
4. Stalled tasks get quarantined for log review

---

## Best Practice #9: Keep Memory Narrow

Persistent memory is useful but dangerous:

- Don't store secrets in memory files
- Separate project memory from user-global memory
- Reset or rotate memory after untrusted runs
- Disable long-lived memory for high-risk workflows

A payload doesn't have to win in one shot — it can plant fragments in memory, then assemble later.

---

## Best Practice #10: Treat Skills as Supply Chain

Snyk's ToxicSkills study found prompt injection in **36%** of 3,984 public skills, with 1,467 malicious payloads identified. Skills are supply chain artifacts — treat them accordingly:

- Audit before installing
- Scan for hidden instructions
- Prefer ECC-native skills over unvetted community skills
- Add guardrails next to external references

---

## The Minimum Security Checklist

- [ ] Separate agent identities from personal accounts
- [ ] Use short-lived scoped credentials
- [ ] Run untrusted work in containers/VMs
- [ ] Deny outbound network by default
- [ ] Restrict reads from secret-bearing paths
- [ ] Sanitize files and linked content before privileged agent access
- [ ] Require approval for shell, egress, deployment, off-repo writes
- [ ] Log tool calls, approvals, and network attempts
- [ ] Implement process-group kill and heartbeat dead-man switches
- [ ] Keep persistent memory narrow and disposable
- [ ] Scan skills, hooks, MCP configs like supply chain artifacts

---

## What's Next

Security protects your system. Now let's build the system that **learns from itself** — continuous learning and memory persistence.

[Next: Chapter 8 — Continuous Learning & Memory Persistence →](./08-howto-ecc-continuous-learning.md)
