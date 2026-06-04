# Situation
Purpose: Status / alignment review — report the (already-approved) human cloud-access control mechanism and confirm a current user inventory still matches it. Mechanism is primary (~70%); some management are seeing it for the first time. The user re-inventory is supporting evidence (~30%). NOT a decision meeting, NOT an audit.
Audience: Mixed — management + security reviewers; varying prior exposure; not all are deep on AWS/GCP IAM (the presenter included). Avoid IAM jargon (SCP, AWS permissions-boundary feature) on main slides.
Context: 30 min live. Multi-cloud scope (AWS + GCP), HUMAN-personnel permissions only ("Cloud User"); machine/CI identities are out of scope (a separate concern, kept only as a Q&A fallback in speaker notes). BOTH AWS and GCP accounts sit under a reseller's organization → cannot use org-wide guardrails on either cloud. Leave ~3–5 min Q&A buffer.
Success criteria: Management leaves understanding the human-permission model, agreeing it is sound and being followed, and aware of the two structural constraints (no org-wide hard guardrail; per-account independent IAM users inventoried by hand) plus the improvement direction (central federation + automated over-grant detection). No approval required today.

# Storyline
Structure: Status / top-down (BLUF) — claim first, then the model (who → what → how), then detection, then proof, then honest constraints, then close.

1. Human access to our cloud runs on an already-approved, role-based access model; this review confirms current users still match it. (Scope: human personnel only; inventory-match is a co-deliverable.)
2. We govern human access by three rules: least privilege, individual named identities, and just-in-time privileged access — high-risk actions require a 2FA role-switch to short-lived credentials, never standing.
3. Every human is a named, individually-attributable identity, and root is sealed behind a two-person break-glass rule.
4. What each person can do is fixed by one of three roles — Developer, Operator, Administrator — whose permissions widen by role, with the Developer tier tightening by environment.
5. High-risk actions are scope-bounded and exception-controlled, keeping blast radius and cost bounded.
6. No high-risk access is standing — Operator and Administrator hold no standing privileges at all, and even a Developer's create/delete actions require a 2FA role-switch to short-lived 1h sessions (the 12h web figure is only the unprivileged console login shell).
7. We prevent where we can with least-privilege role policies, and detect where we can't — every IAM permission change and failed login emails an alert.
8. Our manual re-inventory shows current users map cleanly onto the three roles, with nobody outside the model.
9. Two structural constraints remain — no org-wide hard guardrail (reseller org) and per-account independent IAM users inventoried by hand — both contained today and on a clear improvement path.
10. Bottom line: the approved model holds, current access matches it, and the residual risks are known and managed.

# Ghost deck

## Slide 1 — Our human cloud access runs on an approved, role-based access model, and this review confirms current users still match it

Evidence: one summary box — 3 roles (Dev/Operator/Admin) × 2 clouds (AWS+GCP); scope line "human personnel only"; verdict line "mechanism unchanged & approved · users match model · 2 structural constraints managed"
So what: the rest walks the model (who→what→how), shows detection, proves the match, then names the honest constraints — top-down.
Speaker note: BLUF. Re-presented because part of the audience is new to it. State up front the two deliverables: (1) report the approved mechanism, (2) confirm the user re-inventory. Say explicitly this is a review, not an audit, and not a request for a new decision.

## Slide 2 — We govern human access by three rules: least privilege, individual named identities, and just-in-time privileged access

Evidence: 3-principle strip (least privilege · individual named identities, no shared accounts · just-in-time privileged access — high-risk actions need a 2FA role-switch to short-lived credentials)
So what: these three rules drive every preventive control that follows, and detection (Slide 7) backs them up where prevention can't reach — so nothing is ad-hoc.
Speaker note: This is the "why" frame for first-time and non-technical attendees; every later control maps back to one of these three. Rule 3 precisely: high-risk access is just-in-time — Operator & Administrator hold nothing standing, while Developer keeps only baseline development permissions, with all high-risk (create/delete) via 2FA role-switch (detail on Slide 6).

## Slide 3 — Every human is a named, individually-attributable identity, and root is sealed behind a two-person break-glass rule

Evidence: diagram — each human is a distinct named identity (no shared accounts), so every action is attributable to a person; [root] drawn locked: normally unused, login requires password + 2FA held by two different people (password holder ≠ 2FA holder)
So what: we always know who is acting, and the single most dangerous credential cannot be used by any one person alone.
Speaker note: Call it "two-person rule / dual control." If asked "what about service accounts / CI?": machine identities are out of today's scope and are MORE locked down (federated, no stored keys, no interactive human path) — keep this as a verbal fallback only, not on the slide. If asked, root MFA should be a hardware token; a root login alert is a recommended addition (see Slide 7).

## Slide 4 — What you can do is fixed by one of three roles — Developer, Operator, Administrator — widening by role, with Developer tightening by environment

Evidence: simplified role × environment gradient matrix — rows Dev/Operator/Admin, columns staging/prod, cells showing escalation (Dev: read-most + scoped write/create in staging → Operator: CRUD most envs, IAM read-only, no firewall → Admin: full + identity mgmt). Full verbatim table in Appendix A.
So what: permissions are predictable and bounded by role, granted by assuming a role — not by hand-editing policies.
Speaker note: Show the gradient only — the full text table lives in the appendix. Three roles = three escalating tiers is the one thing to land.

## Slide 5 — High-risk actions are scope-bounded and exception-controlled, keeping blast radius and cost bounded

Evidence: callout group — create/delete only in staging AND only with prior security-meeting approval (currently: EC2, S3 Object, Parameter Store); credential-store (Secrets/Parameter Store) read limited to staging; PassRole limited to roles within the user's own permission scope; cost/usage monitoring required on create-permission resources and usage-based services (Athena / Gemini)
So what: even the widest developer permissions carry an approval gate, a scope limit, and a cost guardrail.
Speaker note: "Permission scope" means the role's own allowed permissions — distinct from the AWS permissions-boundary feature, which is not in use. The approved create/delete exception list is unchanged: EC2 / S3 Object / Parameter Store.

## Slide 6 — No high-risk access is standing — every high-risk action requires a 2FA role-switch to short-lived 1h credentials

Evidence: per-role + per-channel layout —
By role: Operator / Administrator hold NO standing privileges (switch role for everything); Developer keeps only baseline development permissions standing, while all create/delete (EC2 / S3 Object / Parameter Store) require a 2FA role-switch.
By channel: Web — login + 2FA → console shell 12h (holds no privileges) → switch role → role session 1h. CLI — long-term access key (baseline/low-risk perms only) → switch role + 2FA on each switch → role session 1h.
GCP note: same pattern via service-account impersonation / short-lived tokens.
So what: a leaked credential cannot perform any high-risk action without the user's 2FA — an Operator/Admin base credential is effectively powerless, and a leaked Developer credential exposes only baseline, staging-scoped dev permissions, never create/delete; real privileged access is short-lived.
Speaker note: The precise claim: high-risk access is just-in-time, never standing — CLI users do keep a long-term AKSK and the Developer baseline is standing, but neither reaches anything high-risk. Assume-role enforces an MFA condition, so a leaked credential cannot escalate to high-risk actions without the user's MFA. Role sessions are 1h on both channels (AWS default; allowed range 1h–12h) — the 12h figure is only the console login shell, which holds no privileges; re-switching needs no re-login, while CLI prompts 2FA on every switch. Extending just-in-time enforcement to the Developer baseline is on the roadmap (Slide 9).

## Slide 7 — We prevent where we can with least-privilege role policies, and detect where we can't — IAM changes and failed logins email an alert

Evidence: two-part layout. Prevent: least-privilege role policy + 2FA-gated role switch. Detect: monitored-events table — IAM permission change → email; failed login → email; (recommended additions: root login, CloudTrail-disabled → top-priority alert).
So what: misuse surfaces quickly even where we have no org-wide preventive guardrail.
Speaker note: Detection is the compensating backstop where there is no org-wide hard guardrail (see Slide 9). Today exactly two alerts fire: IAM permission change and failed login; root-login and CloudTrail-disable are recommended additions, not yet live. Reviewer will ask "what stops someone disabling the alert?" — restricting who can stop CloudTrail and alerting on disable is the answer; flag it as the planned hardening.

## Slide 8 — Our manual re-inventory shows current users map cleanly onto the three roles, with nobody outside the model

Evidence: role distribution table — count of human users per role (Dev / Operator / Administrator), Administrator count highlighted; verdict badges "identities outside model: <count — confirm 0>"; "as-of <date>"
So what: current role assignments match the model — not just on paper.
Speaker note: Fill in actual headcounts, inventory as-of date, and method (how the list was gathered). Management cares most about the Administrator count — keep it small and explainable. Note the inventory is manual (ties to Slide 9 direction).

## Slide 9 — Two structural constraints remain — no org-wide hard guardrail and per-account independent users — both contained today and on a clear improvement path

Evidence: two-column honesty layout.
Constraints we live with: (a) both AWS and GCP sit under the reseller's org → we cannot set ONE org-wide hard guardrail across accounts on either cloud; we rely on per-role least-privilege + alerting. (b) Human users are independent per-account IAM users, inventoried by hand.
Direction: central IdP federation for humans → single identity source + joiner/mover/leaver; automated over-grant detection (e.g. last-accessed / unused-permission analysis) to replace manual inventory; extend just-in-time enforcement to the Developer baseline (today only Developer's low-risk dev permissions are standing — Operator/Administrator are already fully just-in-time).
So what: residual risks are known, bounded, and have a path — nothing is hidden from the room.
Speaker note: De-jargoned on purpose — keep "SCP" out of the slide; if a knowledgeable reviewer names it, acknowledge it must be set by the reseller (org owner), and we compensate with per-role policy + alerts. GCP is also under the reseller, so the no-org-guardrail constraint applies symmetrically to both clouds — no asymmetry to claim. The federation contrast (machines already OIDC-federated, humans next) makes "humans not federated" read as a planned next step, not a hole.

## Slide 10 — Bottom line: the approved model holds, current access matches it, and the residual risks are known and managed

Evidence: 3-point recap (model sound & unchanged · inventory matches · 2 constraints contained + roadmap)
So what: no decision required today; next review cadence and the two improvement items are tracked.
Speaker note: Mirror of Slide 1 to close the loop. End on "known and managed," not "perfect."

## Appendix A — Full Developer / Operator / Administrator permission table

Evidence: the complete role definitions and per-environment rules as approved. Footnote: "‘permission scope’ in this table means each role’s own allowed permissions — not the AWS permissions-boundary feature, which we do not currently use."
So what: backup for detail questions; not presented unless asked.
Speaker note: Reference only — presented when a detail question lands.

## Appendix B — AWS ↔ GCP concept mapping

Evidence: mapping of the constructs we actually use — PassRole → iam.serviceAccounts.actAs (Service Account User); Secrets Manager / Parameter Store → Secret Manager; security group → VPC firewall rule; assume-role short-term creds → service-account impersonation / short-lived tokens. (No SCP / Org Policy rows — those terms stay off the deck.)
So what: shows the single model maps to both clouds despite different primitives; pre-empts "but GCP is different."
Speaker note: Reference only — the mapping covers constructs in actual use; GCP has no exact equivalent of the AWS permissions-boundary feature, which is not in use on AWS either.

# Flagged inputs (gather before the rewrite stage)
- Slide 8: actual human-user headcounts per role (Dev / Operator / Administrator; Administrator count), inventory as-of date, how the list was gathered, and any identities found outside the model.
