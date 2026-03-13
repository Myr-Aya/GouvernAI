# 🛡️ GouvernAI Gate

**Lightweight governance for AI agents. Classify risk, enforce proportional controls, and log every sensitive action.**

Built by [MindXO](https://mind-xo.com) — translating AI policy frameworks into operational governance systems.

## What it does

GouvernAI Gate inserts a structured governance process between the agent's decision to act and the action itself. It covers 35 action types across 8 categories, classifying each into one of four risk tiers and applying proportional controls — from auto-proceed for routine reads to full-stop impact assessments for irreversible operations.

The skill adapts to context: full approval gates when a user is present, audit-only logging when the agent runs autonomously. Tier 4 (critical) actions are always halted in autonomous mode.

## Risk tiers

| Tier | Level | Interactive | Autonomous |
|------|-------|-------------|------------|
| 1 | 🟢 Routine | Auto-proceed, logged | Auto-proceed, logged |
| 2 | 🟡 Standard | Notify + proceed unless objected | Auto-proceed, logged |
| 3 | 🔴 Elevated | Explicit approval required | Execute + alert user (or halt if not pre-approved) |
| 4 | ⚫ Critical | Full impact assessment + confirmation | Always halted, user alerted |

## Action categories

- **File system** — reads, writes, config modifications, deletions, bulk operations
- **Shell execution** — read-only commands, state-modifying, network access, system-level, obfuscated
- **Network and API** — public reads, authenticated reads, external writes, data transmission, unfamiliar endpoints
- **Communication** — drafts, self-notifications, messages to others, group/public posts
- **Credentials** — existence checks, authenticated usage, value display, external transmission
- **Browser automation** — known URLs, external URLs, form interaction, downloads
- **Scheduling** — config reads, cron creation, heartbeat modification, webhook endpoints
- **Financial** — lookups, purchases, billing changes

## Key features

- **Context-adaptive governance:** full gate for interactive sessions, audit-only for autonomous workflows
- **Escalation rules:** autonomous triggers, bulk operations, unfamiliar endpoints, and scope expansion automatically escalate by one tier
- **Pre-approval patterns:** configure known-safe actions that skip the approval gate but still log
- **Hard constraints:** credentials never transmitted, obfuscated commands never executed, skill never self-modified
- **Audit trail:** every classified action logged with timestamp, tier, type, mode, and approval status
- **Slash commands:** `/governance status`, `log`, `policy`, `strict`, `relaxed`
- **Config overrides:** customize mode, autonomous behavior, alert channels, and pre-approved patterns via openclaw.json

## File structure

```
gouvernai-gate/
├── SKILL.md              # Core orchestrator (always loaded, ~800 tokens)
├── TIERS.md              # Risk tier definitions + escalation rules (on demand)
├── ACTIONS.md            # 35 actions × tier × control lookup table (on demand)
├── POLICY.md             # Hard constraints + conflict resolution (on demand)
├── governance_log.md     # Auto-created audit trail (runtime)
└── README.md             # This file (human-only, not loaded by LLM)
```

**Token efficiency:** SKILL.md is always in context (~800 tokens). Reference files are read only when the gate triggers. Most routine actions need only TIERS.md for quick classification. Worst-case per gated action: ~2900 tokens.

## Install

```bash
# Via ClawHub
clawhub install mindxo/gouvernai-gate

# Or manually
mkdir -p ~/.openclaw/skills/gouvernai-gate
# Copy all .md files into the directory
```

## Configuration

Add to your `openclaw.json`:

```json
{
  "skills": {
    "entries": {
      "gouvernai-gate": {
        "enabled": true,
        "env": {
          "GOVERNANCE_MODE": "auto",
          "GOVERNANCE_AUTONOMOUS_MODE": "audit-only",
          "GOVERNANCE_TIER4_AUTONOMOUS": "halt",
          "GOVERNANCE_PRE_APPROVED": "write:briefings/*,send:telegram:self",
          "GOVERNANCE_ALERT_CHANNEL": "auto"
        }
      }
    }
  }
}
```

| Key | Default | Options |
|-----|---------|---------|
| `GOVERNANCE_MODE` | `auto` | `auto`, `full-gate`, `audit-only`, `strict` |
| `GOVERNANCE_AUTONOMOUS_MODE` | `audit-only` | `audit-only`, `full-gate` |
| `GOVERNANCE_TIER4_AUTONOMOUS` | `halt` | `halt`, `alert-and-execute` |
| `GOVERNANCE_PRE_APPROVED` | (empty) | Comma-separated action patterns |
| `GOVERNANCE_ALERT_CHANNEL` | `auto` | `auto`, or specific channel like `telegram:12345` |

## Honest limitations

This is a governance *skill*, not a governance *engine*. It operates through natural language instructions that the LLM interprets probabilistically. There is no runtime enforcement layer that prevents the agent from bypassing the gate.

What it provides: a structured reasoning framework, visible audit trail, decision points for human judgment, and explicit risk classification.

What it cannot guarantee: 100% compliance, prevention under adversarial manipulation, or enforcement equivalent to programmatic access controls.

The audit log is the real enforcement mechanism — it creates accountability after the fact, even when the gate doesn't prevent an action in real time. For a deeper discussion of linguistic vs. programmatic policy enforcement in AI agents, see MindXO's research on the [policy-to-practice gap](https://mind-xo.com).

## Contributing

Issues and PRs welcome. If you've found edge cases where the governance gate doesn't trigger correctly, or action types that should be in the catalog, please report them.

## License

MIT

## About MindXO

MindXO focuses on translating AI policy frameworks into operational governance systems for regulated enterprises and government entities. Our work spans the full AI lifecycle — from risk classification through operational deployment — with particular expertise in bridging the gap between high-level policy and day-to-day implementation.

→ [mind-xo.com](https://mind-xo.com)
