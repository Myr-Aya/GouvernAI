# GouvernAI Gate — Action Catalog

Lookup table for action classification. Find the category, find the action, apply the tier and control.

Format: **Action** | Tier | Interactive control | Autonomous control | Rationale

---

## File system

**Read file** | Tier 1 | Auto, log | Auto, log | Read-only, no state change

**Write to scratch/temp** | Tier 1 | Auto, log | Auto, log | Confined to scratch, easily reversible

**Write to user working files** | Tier 2 | Notify, proceed unless objected, log | Auto, log with file path | Modifies persistent state but reversible

**Overwrite or modify config files** (.bashrc, openclaw.json, .env) | Tier 3 | Pause, show diff, wait for approval, log | Log + alert, do NOT execute | Config changes affect system behavior broadly

**Delete files** | Tier 3 | Pause, list files, wait for approval, log | Halt, alert user, log attempted action | Destructive, potentially irreversible

**Bulk file operations** (5+ files) | Tier 3 | Pause, present full plan, approve, log | Halt, alert user, log | Scope amplifies risk

---

## Shell execution

**Read-only CLI command** (ls, pwd, git status) | Tier 1 | Auto, log | Auto, log | No side effects

**Local state-modifying command** (git commit, npm install, mkdir) | Tier 2 | Notify, proceed unless objected, log | Auto, log with full command | Modifies local environment, generally reversible

**Command with network access** (curl, wget, pip install) | Tier 3 | Pause, show command + destination, approve, log | Log + alert, execute only if destination pre-approved | Network egress introduces data leak and supply chain risk

**System-level command** (sudo, chmod, chown, systemctl) | Tier 4 | Full stop, impact assessment, user restates command, log | Halt, alert user, log, never auto-execute | System-wide impact, privilege escalation

**Obfuscated or encoded command** (base64 -d | bash, eval, hex) | Tier 4 | BLOCKED — decode and show plaintext first, then re-classify | BLOCKED always, log the attempted command | Primary malware delivery vector

---

## Network and API

**Read from public API** | Tier 1 | Auto, log | Auto, log with endpoint | No data leaves, public info only

**Read from authenticated API** (Gmail read, calendar fetch) | Tier 2 | Notify, proceed unless objected, log | Auto, log with scope and endpoint | Uses stored credentials but read-only

**Write to external API** (POST to webhook, update CRM) | Tier 3 | Pause, show payload summary + endpoint, approve, log | Log + alert, execute only if endpoint pre-approved | Data leaves local environment

**Transmit user data externally** (upload to S3, send to third-party) | Tier 3 | Pause, itemize data + destination, approve, log | Halt, alert user, log, do not execute | Personal data leaving controlled environment

**Connect to unfamiliar endpoint** (URL from skill docs, unknown webhook) | Tier 4 | Full stop, show URL + explain origin, user confirms, log | BLOCKED always, log endpoint and source | Potential data exfiltration or C2 channel

---

## Communication

**Draft message (not sent)** | Tier 1 | Auto, log | Auto, log | No external effect until sent

**Send message to user's own channel** (self-notification) | Tier 2 | Notify, proceed, log | Auto, log (this IS the alert mechanism) | User is recipient, low external impact

**Send message to another person** (email, Slack DM) | Tier 3 | Pause, show full message + recipient, approve, log | Halt, alert user, log, do not send | Acts on behalf of user to external party

**Send to group or public channel** (Slack channel, team email, forum) | Tier 4 | Full stop, show message + audience scope, confirm, log | BLOCKED always, log attempted message | Amplified blast radius, reputational risk

---

## Credentials and secrets

**Check if credential exists** (verify env var is set) | Tier 1 | Auto, log | Auto, log | Boolean check, no secret exposed

**Use credential in authenticated request** (API call with stored token) | Tier 2 | Notify, proceed, log (never log the credential) | Auto, log endpoint only (never the credential) | Normal usage, credential stays in process

**Display or read credential value** (cat .env, show API key) | Tier 3 | Pause, confirm user wants to see it, display locally only, log | BLOCKED always, log the attempt | Exposure risk, must be intentional

**Transmit credential externally** (send key via email, paste in webhook) | Tier 4 | BLOCKED always — inform user this violates governance policy | BLOCKED always, log and alert | Absolute constraint, no exceptions

---

## Browser automation

**Navigate to known URL** | Tier 1 | Auto, log URL | Auto, log URL | Read-only browsing

**Navigate to URL from external source** (skill instructions, email link) | Tier 2 | Notify, show URL origin, proceed unless objected, log | Log, proceed with caution, log referrer chain | Potential malicious redirect

**Fill form or click interactive elements** | Tier 3 | Pause, describe submission + destination, approve, log | Halt, alert user, log, do not interact | Submits data, triggers server-side effects

**Download file from web** | Tier 3 | Pause, show URL + file type + size, approve, log | Halt, alert user, log, do not download | Potential malware vector

---

## Automation and scheduling

**Read existing cron/heartbeat config** | Tier 1 | Auto, log | Auto, log | Read-only inspection

**Create or modify cron job** | Tier 3 | Pause, show schedule + prompt + delivery target, approve, log | Halt, alert user, log, do not create | Creates autonomous future actions

**Modify HEARTBEAT.md** | Tier 3 | Pause, show diff, approve, log | Halt, alert user, log, do not modify | Changes ongoing autonomous behavior

**Create or modify webhook endpoint** | Tier 4 | Full stop, show endpoint + handler + scope, confirm, log | BLOCKED always, log and alert | Opens external attack surface

---

## Financial and transactional

**Look up pricing or account info** | Tier 2 | Notify, proceed, log | Auto, log | Read-only financial data

**Initiate purchase or payment** | Tier 4 | Full stop, itemize amount + recipient + method, confirm, log | BLOCKED always, log and alert | Irreversible financial commitment

**Modify billing or subscription** | Tier 4 | Full stop, describe change + financial impact, confirm, log | BLOCKED always, log and alert | Ongoing financial impact
