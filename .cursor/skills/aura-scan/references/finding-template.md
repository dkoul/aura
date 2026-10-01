# AURA finding template

Copy this structure for each finding in `/aura` output.

```text
🔴 CRITICAL | 🟠 HIGH | 🟡 MEDIUM | 🟢 LOW
<Title>

Primary dimension(s): Agency | Understanding | Authority | Accountability
(Secondary dimensions if applicable)

Explanation:
<What is happening in plain language; who/what initiates; what is at stake.>

Evidence:
- path/to/file.ext (optional: function, line range, config key)
- ...

Reasoning:
<How dimensions connect; whether approval is cosmetic; declared vs enforced policy.>

Why it matters:
<Operational or safety impact in 1–3 sentences.>

Suggested remediation:
1. ...
2. ...

Suggested tests:
- ...
```

## Example (illustrative)

```text
🔴 CRITICAL
Human-as-AI-Custodian Risk

Primary dimension(s): Agency, Authority, Accountability
Secondary: Understanding

Explanation:
The deploy agent can turn a broad health objective into production Kubernetes changes.
Approval UI offers only Deploy / Cancel with no diff or blast radius.

Evidence:
- agents/deploy-agent.ts
- tools/kubernetes.ts
- ui/DeploymentApproval.tsx
- infra/rbac.yaml

Reasoning:
Agency: agent selects scale/restart actions from monitoring signals without per-action user intent.
Authority: credentials include production deploy and secret write per rbac.yaml.
Understanding: UI exposes binary approve/reject only.
Accountability: audit attributes deployment to approver; rollback requires platform-admin.

Why it matters:
The approver carries production outcome risk without context or recovery parity.

Suggested remediation:
1. Restrict agent credentials to staging; separate production break-glass role.
2. Surface resource list, manifest diff, and rollback plan before approval.
3. Require explicit production authority grant per deployment.
4. Add owner-initiated rollback in audit trail.

Suggested tests:
- Production deploy fails when agent lacks production role.
- Reject prevents job enqueue / API call (not only UI state).
- Rollback succeeds for accountable owner role without platform-admin.
```
