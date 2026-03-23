# GouvernAI Skill — Runtime Guardrails for AI Agents

Runtime guardrails that any AI agent can load as a skill. Classifies every sensitive action by risk tier, enforces proportional controls through natural language instructions, and logs a full audit trail.

**Drop-in, zero dependencies.** Copy these files into your agent's skill directory. No plugins, no hooks, no packages to install.

## How it works

GouvernAI is a set of markdown files that teach an AI agent to gate its own actions. When the agent is about to do something sensitive — write a file, run a shell command, send an email, transmit credentials — it follows an 8-step gate process: identify the action, classify its risk tier, check escalation rules, verify hard constraints, apply the appropriate control (notify / pause / halt), and log the decision.

This is **linguistic enforcement** — the agent reads and follows the instructions with judgment. There is no programmatic enforcement layer. The audit log is the accountability mechanism.

> For deterministic enforcement (hard blocking of obfuscated commands, credential exfiltration, etc.), see the [GouvernAI Claude Code Plugin](https://github.com/Myr-Aya/GouvernAI-claude-code-plugin), which adds a PreToolUse hook layer on top of this skill.

## Risk tiers

| Tier | Level | Interactive mode | Autonomous mode |
|------|-------|-----------------|-----------------|
| 1 | Routine | Excluded from gate | Excluded from gate |
| 2 | Standard | Notify + proceed | Auto-proceed + log |
| 3 | Elevated | Pause + require approval | Auto-proceed + alert + log |
| 4 | Critical | Full stop + risk assessment | HALT + alert + log |

## Install

### Claude Code (standalone skill)

```bash
# Create the skill directory
mkdir -p .claude/skills/gouvernai

# Copy all skill files
cp SKILL.md ACTIONS.md TIERS.md POLICY.md GUIDE.md guardrails_log.md .claude/skills/gouvernai/
```

Add to your project's `AGENTS.md` or equivalent instructions file:

```markdown
Before your first tool call each session, read and apply .claude/skills/gouvernai/SKILL.md
```

### Any agent that reads markdown

Copy the skill files into whatever directory your agent loads instructions from. The only requirement is that the agent can:

1. Read markdown files as instructions
2. Write to a log file (for audit trail)
3. Pause and ask the user for approval (for Tier 3/4 actions)

Point your agent's system prompt or instructions to load `SKILL.md` at session start. The skill references the other files by name and loads them on demand.

### OpenClaw

```bash
# Copy into your OpenClaw skills directory
mkdir -p ~/.openclaw/workspace/skills/gouvernai
cp SKILL.md ACTIONS.md TIERS.md POLICY.md GUIDE.md guardrails_log.md ~/.openclaw/workspace/skills/gouvernai/
```

## File structure

```
gouvernai-skill/
├── SKILL.md             # Gate orchestrator (always loaded, ~1,800 tokens)
├── ACTIONS.md           # Action → tier classification lookup (~700 tokens)
├── TIERS.md             # Universal controls + escalation rules (~800 tokens)
├── POLICY.md            # Hard constraints — NEVER rules (~1,000 tokens)
├── GUIDE.md             # Output format templates (~500 tokens)
├── guardrails_log.md    # Audit trail (auto-populated at runtime)
└── README.md            # This file
```

**Token budget:** SKILL.md is always loaded (~1,800 tokens). The other files are read on demand and cached — total on-demand cost is ~3,000 tokens, paid once per session.

### File responsibilities (MECE)

| File | Owns | Does NOT contain |
|------|------|------------------|
| SKILL.md | Gate flow, triggers, exclusions, mode detection, unlisted actions | Tier definitions, action classifications, hard constraints |
| TIERS.md | Universal controls (tier × mode), escalation rules, de-escalation | Per-action classifications, policy constraints |
| ACTIONS.md | Action → base tier classification | Controls, rationale, policy rules |
| POLICY.md | Hard constraints (NEVER rules), pre-approval rules, conflict resolution | Tier definitions, action classifications, controls |
| GUIDE.md | Output format templates, self-improvement logging | Gate logic, classifications, policy |

## Slash commands

| Command | What it does |
|---------|-------------|
| `/guardrails` | Show current mode, tier distribution, approvals/denials |
| `/guardrails log` | Display recent audit log entries |
| `/guardrails strict` | All tiers +1 |
| `/guardrails relaxed` | Tier 2 skips gate |
| `/guardrails audit` | Audit-only mode (for CI/unattended) |
| `/guardrails reset` | Return to default full-gate mode |
| `/guardrails policy` | Display hard constraints |

Mode changes persist to `guardrails-mode.json` in the working directory.

## What it catches vs. what it doesn't

**Catches** (through linguistic classification):
- Unintentional scope expansion (escalation rules)
- Credential exposure in file writes or transmissions
- Obfuscated command execution (decoded and re-classified)
- Bulk operations without explicit approval
- Communication to unfamiliar recipients
- Financial transactions without approval

**Does NOT catch** (limitations of linguistic enforcement):
- If the agent skips or ignores the skill instructions entirely
- Multi-step exfiltration across separate commands with no keyword matches
- Prompt injection that convinces the agent to bypass the gate
- Novel patterns not covered by the action catalog or sequential detection

**This is an operational safety layer, not a security boundary.** It catches mistakes, enforces consistent approval workflows, and creates an audit trail. For production or high-security environments, complement it with infrastructure-level controls: network egress policies, secret vaults, sandboxed execution, and DLP monitoring.

## Customization

### Adding pre-approved patterns

Write patterns to `guardrails-mode.json`:

```json
{
  "mode": "full-gate",
  "audit_only": false,
  "pre_approved": [
    "git commit",
    "write to src/**",
    "npm install from package.json"
  ]
}
```

Pre-approved actions still undergo classification, escalation, and hard-constraint checks. They skip only the approval step.

### Adding actions to the catalog

Edit `ACTIONS.md` to add new actions under the appropriate category with a base tier. The skill will pick them up on the next classification.

### Adjusting escalation rules

Edit `TIERS.md` to modify when escalation triggers. For example, change the bulk threshold from 5 to 10, or add new escalation conditions.

The skill adds ~2 extra messages per session compared to no guardrails.

## Relation to the Claude Code Plugin

This standalone skill is the **linguistic layer** extracted from the [GouvernAI Claude Code Plugin](https://github.com/Myr-Aya/GouvernAI-claude-code-plugin). The plugin adds:

- **Deterministic hook enforcement** — a Python script that blocks obfuscated commands, credential transmission, and catastrophic system commands via PreToolUse hooks (exit code 2 = hard block)
- **85 unit tests** covering hook patterns
- **Plugin infrastructure** — plugin.json, hooks.json, marketplace install

If you're using Claude Code, the plugin gives you both layers. If you're using any other agent platform, this skill gives you the linguistic layer — which handles the nuanced risk classification and approval workflow.

## License

MIT — see [LICENSE](LICENSE)

## Built by Myr-Aya

[MindXO](https://mind-xo.com) — translating AI policy frameworks into operational guardrails systems.
