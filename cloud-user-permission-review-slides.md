# Cloud User Permission Review — Slides

Source: cloud-user-permission-review.md (ghost deck). Audience: mixed management + security reviewers; 30-min live status review.

## Slide 1 — Review scope & status

**Status** Users match the approved role-based model

**Scope** Human users only · 8 AWS accounts · 2 teams

**Agenda**
1. The mechanism
2. The user inventory review

Speaker note: Open with the verdict, then set expectations: this is a status review — nothing to approve today. If asked about GCP: human users live in the 8 AWS accounts; GCP usage is limited to usage-based services (e.g., Gemini) covered by the same cost-monitoring rule. The mechanism was approved previously; since part of the room is new to it, most of the time goes to walking the model (slides 2–7), and the inventory review then shows reality matches (slide 8).

## Slide 2 — Three governing rules

| Rule | Meaning |
|---|---|
| **Least privilege** | Only what the job needs |
| **Individual named identities** | No shared identities — every action traceable to a person |
| **Just-in-time privileged access** | Two-factor (2FA) role-switch → short-lived credentials |

Speaker note: These three rules are the frame — every preventive control on the next slides comes from one of them, with alerting (slide 7) as the backstop. If precision on rule 3 is needed: Operator and Administrator hold nothing standing; Developer keeps only baseline development permissions, and anything high-risk still goes through the 2FA role-switch (detail on slide 6).

## Slide 3 — Named identities & root control

Visual: a row of named individual identities — one per person, no shared identities; beside it, the root account drawn locked.

**Root break-glass**
- Normally unused
- Password + 2FA — held by **two different people**

Speaker note: The root design is a two-person rule: the password holder and the 2FA holder are different people, so nobody can use root alone. If asked about service accounts or CI: they are out of today's scope and even tighter — federated, no stored keys, no human login. If asked about further root hardening: a hardware 2FA token and a root-login alert are the natural next steps. Bridge: named identities are who — the next slide is what each one can touch.

## Slide 4 — Role permission model

| Role | Staging | Production |
|---|---|---|
| **Developer** | Read · update · approved create/delete | **Read only — no credential stores** |
| **Operator** | Full control · identity read-only · no firewall changes | *Same as staging* |
| **Administrator** | Full access · identities · firewall | *Same as staging* |

Full table → Appendix A.

Speaker note: Say it in one line: permissions step up across three tiers, and the Developer tier narrows to read-only in production. Access always comes from switching into a role — there are no hand-edited per-user policies. The Developer "update" cell covers development-related resources; credential-store reads are staging-only, which is why production shows none. Bridge: the riskiest cells of this table get extra controls — next slide.

## Slide 5 — High-risk action controls

| Control | Requirement | Applies to |
|---|---|---|
| **Approval** | Staging only · security-meeting sign-off | create/delete on `EC2` `S3 Object` `Parameter Store` |
| **Cost** | Usage monitoring | resources that allow creation · usage-based services (`AWS Athena` `GCP Gemini`) |

Speaker note: These are the controls on top of the role map — slide 4 said what each role can touch; this slide is what stands between a developer and the riskiest actions. The approved create/delete list is unchanged since the last review. Cost monitoring covers everything with create permissions plus usage-based services. If a reviewer asks about privilege escalation when developers create resources: the AWS PassRole control bounds delegation — nobody can hand out more access than they hold ("permission scope" in Appendix A). These controls bound the scope of damage and cost; the next slide bounds credential theft.

## Slide 6 — Just-in-time privileged access

| High-risk access | All roles |
|---|---|
| **Path** | 2FA role-switch |
| **Lifetime** | 1h session, auto-expires |
| **Standing** | No standing high-risk access — Developer keeps a low-risk baseline |

Speaker note: Voice the nuance: CLI users keep a long-term key and the Developer baseline is standing — but neither reaches anything high-risk, because the role-switch is gated by an MFA condition on the role's trust policy. A leaked Operator or Administrator key is effectively useless; a leaked Developer key reaches only the low-risk baseline — all-environment read (excluding credential stores) plus staging update — never create/delete. 1h is the AWS default role-session duration; the 12h figure is only the unprivileged console login shell. When a role session lapses the user re-switches without logging in again; CLI prompts 2FA on each switch. If asked whether chaining roles buys a longer session: role chaining is hard-capped at 1h by AWS — it shortens, not extends; a longer session comes from re-assuming directly from the long-term identity. Extending just-in-time to the Developer baseline is a future improvement under evaluation — mention only if asked.

## Slide 7 — Prevention & detection

**Prevent** — the role model + just-in-time access

**Detect** — the fallback where we have no org-wide guardrail (reseller org — slide 9)

| Event | Alert | Status |
|---|---|---|
| Identity/permission (IAM) change | Email | Live |
| User login failure | Email | Live |
| Root login | Top-priority | Proposed |
| Audit log disabled | Top-priority | Proposed |

Speaker note: Detection is the backstop for the one thing we cannot prevent organization-wide — slide 9 explains why. Today exactly two alerts fire: IAM permission changes and failed logins; the proposed pair does not exist yet. "Audit log" is CloudTrail. Likely question: "who can turn the alerts off?" — the plan is to restrict CloudTrail and alert-rule changes to Administrator, and to make disabling CloudTrail itself fire a top-priority alert.

## Slide 8 — User inventory review

| Role | Rule | Result |
|---|---|---|
| Developer | everyone not Operator/Admin | ✓ |
| Operator | up to 2 per account | ✓ all 8 accounts |
| Administrator | 2 per team, fixed | ✓ 4 people |

**2 teams · 8 accounts** · outside the model: 0 · as-of **2026-06-01**
Full inventory → [AWS Permission.xlsx](https://cyberlinkcorp-my.sharepoint.com/:x:/r/personal/sophia_huang_cyberlink_com/_layouts/15/Doc.aspx?sourcedoc=%7B773E2C55-BD34-4CAC-AB2A-6527EE0F5676%7D&file=AWS%20Permission.xlsx&wdLOR=c7FF5A92E-45B0-F14E-ACC8-19993307F33E&fromShare=true&action=default&mobileredirect=true)

Speaker note: Lead with the Administrator number — four people, two per team, each explainable by name. Rule column = the approved arrangement; Result column = what the inventory review verified — the match is the point of this slide. Method: manual, account-by-account; people counted uniquely (one person across several accounts counts once); Operator capped at two per account by design. Per-person detail and team/account map: Appendix B plus the linked sheet — answer who-is-in-which-account questions from there. If asked how to avoid manual sweeps next time: automated inventory is under evaluation.

## Slide 9 — Residual constraints

| Constraint | Why | Contained by |
|---|---|---|
| No org-wide hard guardrail | Accounts under the reseller's organization | Least privilege + alerting |
| Per-account users · manual inventory | No central identity source | Role model + this inventory review |

Speaker note: If someone names "SCP": that mechanism belongs to the organization owner — the reseller; our compensation is per-role policy plus the alerting on slide 7. If asked about improvement plans: the direction under evaluation is a central identity provider (one identity source, automatic joiner/mover/leaver), automated over-grant detection to replace manual sweeps, and extending just-in-time to the Developer baseline — concrete plans come to a future review, no commitments today. Useful aside if federation comes up: machine identities already authenticate federated — humans would follow the same path.

## Slide 10 — Conclusion

1. **Model unchanged** — three rules, three roles, just-in-time high-risk access
2. **Users match** — none outside the model
3. **Risks contained** — 2 known constraints

Speaker note: Close on the three conclusions — slide 1 opened on the users-match status; this is the full recap. If asked when the next review happens: to be scheduled — no cadence agreed yet. If asked about improvement plans: direction is under evaluation; concrete plans come to a future review. End on "known and contained" rather than "perfect": the honesty is what makes the rest credible.

## Appendix A — Full permission table (as approved)

**Developer** — design, develop, test and debug services.
- Read permissions allowed in all environments, except credential stores (e.g., `Parameter Store`, `Secrets Manager`), which are limited to staging only.
- Update permissions for development-related resources are restricted to staging.
- Create and delete permissions are restricted to staging and require prior security-meeting approval.
- Resources currently authorized for create/delete: `EC2` · `S3 Object` · `Parameter Store`
- PassRole may be granted, provided it only passes roles within the user's own permission scope.
- Resources with create permissions must be actively monitored to detect unexpected usage and control costs.
- Usage-based managed services (e.g., `AWS Athena`, `GCP Gemini`) require consumption monitoring.

**Operator** — provision and maintain infrastructure; configure and deploy services; monitor performance and availability; respond to and resolve incidents.
- Create, read, update, delete permissions in all environments, except:
  - Read-only access to IAM resources.
  - No create, read, update, or delete permissions for firewall settings (e.g., AWS security groups).
- PassRole may be granted, provided it only passes roles within the user's own permission scope.

**Administrator** — manage user identities and access privilege.
- Configure and maintain firewall settings.
- Full access to all resources.

Footnote: "permission scope" in this table means each role's own allowed permissions — not the AWS permissions-boundary feature, which we do not currently use.

Speaker note: Reference only — bring it up when a detail question lands.

## Appendix B — Teams and accounts

| Team | Accounts |
|---|---|
| **CL-COM** | cl-com · cl-data · cl-cdn · faceme-web · faceme-web-stage |
| **CL-CC** | cl-cc · cl-cloud · cl-uteam |

2 teams · 8 accounts.

Speaker note: Reference only — show when someone asks which teams or which accounts. Each team's two Administrators cover that team's accounts.
