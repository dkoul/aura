---
name: aura-scan
description: Scan agentic codebases for delegated-agency risks using the AURA model (Agency, Understanding, Authority, Accountability). Use when the user runs /aura, asks for an AURA scan, delegated-agency analysis, or /aura explain or /aura test on a finding.
---

# AURA Scan

**Positioning:** Find where your AI has more agency than authority.

Produce a **prioritized list of actionable AURA findings**, not a compliance score. Reason across files and control flows; do not rely on keyword matching alone.

## Commands

| Invocation | Action |
| --- | --- |
| `/aura` | Scan the repository (or scoped paths the user gives) and output prioritized findings. |
| `/aura explain <finding>` | Deep-dive one finding with code evidence, control-flow narrative, and AURA dimension breakdown. |
| `/aura test <finding>` | Generate an executable test (or test plan + skeleton) that would detect or bound the identified risk. |

If the user did not scope paths, scan the whole repository. Respect monorepo boundaries when the user names a package or directory.

## Design principles (required)

1. **Do not optimize for "Human-in-the-Loop."** Evaluate agency, understanding, authority, and accountability **relationships**.
2. Do **not** flag "no human approval found" by itself. Ask whether the action is consequential enough to require **retained human authority**, and whether technical enforcement exists.
3. Do **not** assume an approval button equals control. Trace whether **rejection or cancellation actually prevents** the underlying action, credentials use, or side effects.
4. Treat **declared policy** (prompts, docs, UI copy) separately from **enforced permissions** (RBAC, tool allowlists, network policy, DB roles).
5. Prefer **fewer, higher-quality findings** over exhaustive low-signal noise. Merge related issues into one finding when they share a root cause.

## Core workflow

Execute in order; skip steps only when the repo clearly has no agents (still report "no agentic surfaces found" with what you searched).

```text
Repository
    → Discover agent architecture
    → Identify actors + capabilities
    → Build Agency / Authority graph
    → Locate intervention points (approval, policy, kill switch, rollback)
    → Evaluate AURA per consequential capability
    → Identify mismatches (especially Custodian Trap)
    → Prioritize risks
    → Generate remediation + tests
```

### 1. Discover agent architecture

Find entry points and runtime shape:

- Agent loops, planners, orchestrators, subgraphs, handoffs
- Tool/MCP definitions, dynamic tool registration, subagents
- Background jobs, triggers, webhooks, cron, event consumers that invoke models
- Config: `environment.json`, agent policies, tool allowlists, egress rules
- UI flows that gate or display agent actions

Use search patterns (adapt to stack): `agent`, `tool`, `mcp`, `delegate`, `approve`, `human`, `rollback`, `credential`, `permission`, `autonomous`, `retry`, `subagent`, `workflow`, `execute`, `deploy`.

Read enough context to describe **who initiates** each consequential path (user, schedule, another agent, model).

### 2. Identify actors and capabilities

For each actor (human role, service account, agent identity, shared "bot" user):

- What tools/APIs can it call?
- What data can it read/write?
- What environments (dev/staging/prod)?
- Can it spawn other agents or extend its own tool set at runtime?

### 3. Build Agency / Authority graph

For each **consequential capability** (deploy, merge, spend money, delete data, modify secrets, change IAM, send external comms, etc.):

- **Agency:** Can the model initiate or choose this without a fresh explicit human intent for *this* action?
- **Authority:** What do credentials, RBAC, and tool schemas **actually** permit?
- **Understanding:** What does a responsible human see before assent (diff, blast radius, uncertainty, alternatives)?
- **Accountability:** Who is attributed, who can revoke, rollback, or escalate?

Draw edges: `user intent → agent plan → tool call → side effect`. Note async paths (queued jobs, retries) that bypass the approval moment.

### 4. Locate intervention points

Document where control *could* exist:

- Pre-execution approval, plan review, dual control
- Tool sandboxing, read-only roles, staging-only credentials
- Kill switches, circuit breakers, max spend / rate limits
- Audit logs with actor + agent + tool + resource IDs
- Rollback, compensating transactions, credential revocation

Mark whether each intervention is **enforced in code/infra** or **instruction-only**.

## AURA dimensions

Score each finding qualitatively (not numerically). Use severity: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`.

### Agency (A)

**Question:** What decisions and actions can the AI independently initiate or determine?

Look for: autonomous tool invocation, planning loops, state changes, retries, agent-to-agent delegation, actions without explicit user initiation for that action, broad goals translated into operational steps.

### Understanding (U)

**Question:** Does the person responsible have enough information to meaningfully evaluate the AI's decision?

Look for: evidence in UI/logs, rationale, uncertainty, affected resources, alternatives, confidence, hidden context, explainability gaps, summaries that omit failure modes.

### Authority (R)

**Question:** What capabilities and permissions have actually been delegated to the AI?

Look for: API keys, RBAC, MCP tools, filesystem, cloud IAM, DB write access, production access, declared vs actual permissions, SQL/shell tools with broad credentials.

### Accountability (A)

**Question:** Who owns the outcome, and can they intervene, reverse, or revoke?

Look for: audit trails, attribution (human vs shared agent principal), ownership, rollback, kill switches, credential revocation, escalation, separation of duties, irreversible actions.

## AURA relationships (required reasoning)

Dimensions are not independent. Flag compound patterns explicitly.

**Human-as-AI-Custodian Trap** (prioritize as CRITICAL when present):

```text
High AGENCY + High AUTHORITY + Limited UNDERSTANDING + Human ACCOUNTABILITY
```

The machine gets agency and authority; the human gets responsibility without proportional understanding or control.

When you see this pattern, use the finding title **Human-as-AI-Custodian Risk** and tie evidence across agency, authority, understanding, and accountability in one report.

## Output format

Default report structure for `/aura`:

1. **Executive summary** (3–6 sentences): what the agentic system can do, top risks, strongest enforcement gaps.
2. **Architecture sketch** (short bullet list or mermaid): main agents, tools, human touchpoints.
3. **Prioritized findings** (sorted CRITICAL → LOW).

Each finding MUST use this template (see [finding-template.md](references/finding-template.md)):

- Severity + title
- Primary AURA dimension(s)
- Explanation
- Evidence (file paths and, when useful, symbol/line references)
- Reasoning (how agency/authority connects to understanding/accountability)
- Suggested remediation (ordered, concrete)
- Suggested tests (verifiable behaviors)

Do **not** output an overall AURA score or compliance certification.

## `/aura explain <finding>`

1. Restate the finding in one paragraph.
2. Walk through **code evidence** with citations (read files; cite paths and line ranges when possible).
3. Trace **control flow** from trigger to side effect.
4. Map each AURA dimension to specific artifacts.
5. State **what would need to change** for the risk to be downgraded (without re-scanning the whole repo unless asked).

## `/aura test <finding>`

1. Identify the **risk hypothesis** (one falsifiable sentence).
2. Propose **test type**: unit, integration, policy-as-code, e2e, or manual checklist—pick what fits the repo's test stack.
3. Deliver **executable or copy-paste-ready** test code when the repo has a clear test framework; otherwise a minimal script or `curl`/IAM policy test steps.
4. Include **pass criteria** that prove enforcement (e.g., production deploy fails without role X; reject prevents job enqueue; rollback callable by owner role).
5. Note **fixtures/mocks** needed and **secrets safety** (never use real production credentials in generated tests).

## Non-goals

Do not: certify compliance; replace SAST/DAST; judge model quality; require a specific agent framework; auto-modify production code; assume every AI action needs human approval.

## Success check

After a scan, a reader should be able to answer:

1. What can the AI actually do?
2. What decisions can it make?
3. What authority has been delegated?
4. Does the accountable person understand the decision?
5. Where can the AI exceed its authority?
6. Who can stop or reverse it?
7. What should we test?

## Reference

- Finding template: [references/finding-template.md](references/finding-template.md)
- Discovery checklist: [references/discovery-checklist.md](references/discovery-checklist.md)
