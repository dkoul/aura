# AURA Scan

AI-powered code scanning skill for detecting **delegated-agency risks** in agentic systems.

> **AURA Scan: Find where your AI has more agency than authority.**

## What it does

The skill helps engineers scan an unfamiliar agentic codebase and answer:

**Where has agency been delegated to AI, and is that delegation properly bounded?**

It uses the **AURA** model:

| Dimension | Question |
| --- | --- |
| **Agency** | What can the AI independently initiate or decide? |
| **Understanding** | Can the accountable human meaningfully evaluate the decision? |
| **Authority** | What permissions are actually enforced (not just declared)? |
| **Accountability** | Who owns outcomes, and can they stop or reverse actions? |

Output is a **prioritized list of findings** with remediation and test ideas—not a compliance score.

## Usage in Cursor

Install or open a repo that contains this skill under `.cursor/skills/aura-scan/`. When the agent loads skills, invoke:

| Command | Purpose |
| --- | --- |
| `/aura` | Full repository scan and prioritized report |
| `/aura explain <finding>` | Deep dive with code evidence |
| `/aura test <finding>` | Generate tests to detect or bound the risk |

You can also ask naturally: *"Run an AURA scan on this repo"* or *"Scan `services/agent` for AURA risks."*

## Skill location

- Main skill: [`.cursor/skills/aura-scan/SKILL.md`](.cursor/skills/aura-scan/SKILL.md)
- Finding template: [`.cursor/skills/aura-scan/references/finding-template.md`](.cursor/skills/aura-scan/references/finding-template.md)
- Discovery checklist: [`.cursor/skills/aura-scan/references/discovery-checklist.md`](.cursor/skills/aura-scan/references/discovery-checklist.md)

## Design note

The skill deliberately avoids checkbox "human-in-the-loop" detection. It analyzes **relationships** between agency, understanding, authority, and accountability—including the **Human-as-AI-Custodian Trap** (high agency and authority on the machine, limited understanding and full accountability on the human).

## Non-goals

No compliance certification, no overall AURA score, no replacement for security scanners, and no assumption that every AI action requires human approval.
