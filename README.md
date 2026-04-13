# Salesforce + Claude AI Integration

Two integration paths — from browser-only Apex + Flow to Claude Code with MCP, agent teams, and operational discipline.

**Author:** Chaitanya — Senior Salesforce Engineer  
**Date:** April 2026

---

## Overview

| | Path A | Path B |
|---|---|---|
| **Who** | Admins & declarative devs | Developers with CI/CD needs |
| **Tools** | Browser: Setup, Dev Console, Flow Builder | VS Code, SF CLI, Claude Code, Node.js 18+ |
| **How** | Paste Apex, configure Named Credentials, build Flow | Claude Code generates code, deploys via MCP, raises PRs |
| **Agents** | Single-agent via Flow | Multi-agent team: Admin, Dev, Test sub-agents |
| **Best for** | Quick MVP or proof of concept | Production workflows with CI/CD |

Both paths share the **same Apex classes**, the same **Named Credential**, and the same **Custom Metadata**. The difference is how you create and deploy them — not what they do at runtime.

---

## Path A: Salesforce UI Only (< 30 minutes)

### What users can do
- Summarize Cases for handoffs
- Draft renewal/client emails
- Get admin guidance (before-save vs after-save, etc.)
- Write user stories with acceptance criteria
- Explain validation rule formulas

### Setup Checklist

| Done | Step | Where | Time |
|---|---|---|---|
| ☐ | Remote Site Setting | Setup → Security → Remote Site Settings | 2 min |
| ☐ | External Credential + Principal | Setup → Named Credentials → External Credentials | 5 min |
| ☐ | Named Credential | Setup → Named Credentials | 3 min |
| ☐ | Permission Set + assign to user | Setup → Permission Sets | 3 min |
| ☐ | Apex class (ClaudeService) | Developer Console → New → Apex Class | 5 min |
| ☐ | Screen Flow (3 elements) | Setup → Flows → New Screen Flow | 10 min |
| ☐ | Add Flow to Lightning Page | Edit Page → drag Flow component | 2 min |

#### Step 1 — Remote Site Setting
`Setup → Security → Remote Site Settings → New`

| Field | Value |
|---|---|
| Remote Site Name | `Claude_API` |
| Remote Site URL | `https://api.anthropic.com` |
| Active | ✓ Checked |

#### Step 2 — Named Credentials

**2a — External Credential**  
`Setup → Named Credentials → External Credentials → New`
- Label: `Claude_External`
- Authentication Protocol: `Custom`
- Save → Principals → New → Parameter Name: `x-api-key`, paste your `sk-ant-...` key

**2b — Permission Set**  
`Setup → Permission Sets → New` → Label: `Claude API Access`  
→ External Credential Principal Access → Add `Claude_External` → Assign to users

**2c — Named Credential**  
`Setup → Named Credentials → Named Credentials → New`
- Label: `Claude_API`
- URL: `https://api.anthropic.com/v1/messages`
- External Credential: `Claude_External`
- Enabled for Callouts: ✓

> ⚠️ **Most common mistake:** Skipping the Permission Set. Without it, users get `INSUFFICIENT_ACCESS` even though everything else is correct.

#### Step 3 — Apex Class
Open Developer Console → File → New → Apex Class → Name: `ClaudeService` → paste from `force-app/main/default/classes/ClaudeService.cls` → Save.

#### Step 4 — Screen Flow
`Setup → Flows → New Flow → Screen Flow → Create`

1. **Input Screen** — Long Text Area, API Name: `userPrompt`, Required: true
2. **Action** — Search "Ask Claude", Input: `{!userPrompt}`, Output: `{!claudeResponse}`
3. **Output Screen** — Display Text component showing `{!claudeResponse}`

Then: Save → Activate → Add to any Lightning Page via the Flow component.

---

## Path B: Claude Code + MCP + Agent Teams

### Prerequisites

```bash
# Install tools (one time)
npm install -g @anthropic-ai/claude-code   # Claude Code
# + Node.js 18+ from nodejs.org
# + SF CLI from developer.salesforce.com/tools/salesforcecli
# + VS Code from code.visualstudio.com
```

### Path B Checklist

| Done | Step | Command |
|---|---|---|
| ☐ | Install Node.js 18+ | nodejs.org |
| ☐ | Install Claude Code | `npm install -g @anthropic-ai/claude-code` |
| ☐ | Install SF CLI | developer.salesforce.com |
| ☐ | Create SFDX project | `sf project generate -n ClaudeIntegration` |
| ☐ | Authenticate org | `sf org login web -a MyOrg` |
| ☐ | Create CLAUDE.md | Copy from this repo |
| ☐ | Launch Claude Code | `cd project && claude` |
| ☐ | Add MCP servers | `claude mcp add salesforce-dx [...]` |
| ☐ | Configure allow-list | `.claude/settings.local.json` |

### First-time setup

```bash
sf project generate -n ClaudeIntegration
cd ClaudeIntegration
code .

# In VS Code terminal:
sf org login web -a MyOrg
claude
```

### Org connection

| Org type | Login URL |
|---|---|
| Production / Developer Edition | `https://login.salesforce.com` |
| Sandbox | `https://test.salesforce.com` |
| Custom My Domain | Domain-name part only (no `https://` or `.my.salesforce.com`) |

### MCP Servers

```bash
claude mcp add salesforce-dx \
  npx -y @salesforce/mcp-server-salesforce-dx

claude mcp add jira \
  npx -y @anthropic/mcp-server-jira

claude mcp add github \
  npx -y @anthropic/mcp-server-github

claude mcp list
```

### Agent Teams

```bash
# Enable experimental feature
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
claude
```

Agents are defined in `.claude/agents/` and auto-discovered by Claude Code.

```
# Invoke specific agent
@sf-admin Which permission sets grant edit on Case.Status?
@sf-dev Create a trigger handler for Account
@sf-test Generate a test class for AccountTriggerHandler

# Full team delivery
"Deliver JIRA-456 using the agent team: Admin reviews config,
 Dev writes Apex, Test generates test classes, then deploy and raise a PR."
```

### Operating Model

> **Plan mode first, edit mode second. Always.**

1. Agent explains what it will do (no files modified)
2. You review: correct object names? field API names? trigger timing?
3. Only then allow edit mode

---

## API Quick Reference

| Item | Value |
|---|---|
| Endpoint | `https://api.anthropic.com/v1/messages` |
| Version header | `anthropic-version: 2023-06-01` |
| Auth header | `x-api-key: sk-ant-...` (via Named Credential only) |
| Model | `claude-sonnet-4-20250514` |
| Apex Named Credential prefix | `callout:Claude_API` |
| Recommended Apex timeout | `30000` ms |
| Max callouts per transaction | 100 |
| Custom Metadata Type | `Claude_Prompt_Config__mdt` |

---

## Claude Code Commands

| Command | Purpose |
|---|---|
| `/init` | Initialize Claude Code in current project |
| `/compact` | Reduce context — keeps summary, clears detail |
| `/clear` | Aggressive context reset |
| `/cost` | Show tokens consumed and estimated cost |
| `/doctor` | Diagnose Claude Code configuration issues |
| `claude mcp list` | Show all registered MCP servers |
| `claude mcp add [name] [cmd]` | Register a new MCP server |

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Unauthorized endpoint` | Remote Site Setting missing | Setup → Remote Site Settings → verify `api.anthropic.com` is active |
| `401 Unauthorized` | API key wrong | Check External Credential → Principal → verify `x-api-key` value |
| `INSUFFICIENT_ACCESS` | Missing Permission Set | Create Permission Set → add Principal → assign to user |
| `Read timed out` | Default 10s timeout too short | Add `req.setTimeout(30000)` |
| Action not in Flow | Apex compile error | Verify class in Developer Console |
| Agent not in `@` typeahead | Files not in `.claude/agents/` | Check path (`agents` not `agent`), verify `name:` field in frontmatter |
| Agent team unavailable | Feature flag not set | `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` |
| Deploy looks failed | Noisy CLI output | Run `sf project deploy report --job-id [id]` for actual status |

---

## Security

- API keys via Named Credentials only — never in Apex, LWC, or CLAUDE.md
- External Credential Principal via Permission Set (least privilege)
- Input validation before sending to Claude API
- Output filtering in `ClaudeResponseParser` blocks sensitive data leakage
- Rate limiting: 50 calls/user/hour (configurable in Custom Metadata)
- Audit trail via Platform Events or Custom Object
- Allow-list in `settings.local.json` — do not blanket-allow all commands

---

*Chaitanya — Senior Salesforce Engineer | April 2026*
