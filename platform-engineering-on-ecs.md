# ECS Infrastructure Design
## Slide Deck — Architecture Review

---

## Slide 1 — Title

**ECS Infrastructure Design: One Foundation, Every Service**

> *We're introducing ECS. More services will follow. Here's how we're building the foundation right.*

---

## Slide 2 — Why now — and why it matters

**ECS is our new standard container runtime — this is the first service, more will follow.**

- Greenfield adoption: no legacy constraints, but no existing patterns either
- Design decisions made now become the default for every service that follows
- Getting it wrong compounds: each new service inherits the same structural mistakes

> The cost of a wrong design scales with the number of services.
> The cost of a right design is paid once.

---

## Slide 3 — What's blocking self-service deployment

**The goal: any Dev team can deploy a service to ECS from their own repo, via GitHub Actions — no Ops coordination, no platform knowledge required.**

Two structural blockers stand in the way:

**Unclear Ownership**
Every deploy blocks on Ops, or Dev has unguarded access to shared infra.
**Result:** Velocity suffers or risk increases.

**Config Complexity**
20+ fields per service, multiplied across environments; configs drift with every copy-paste.
**Result:** Copy-paste errors at scale.

---

## Slide 4 — Our Answer: Three Design Decisions

**We resolve both blockers with three decisions — made once, inherited by every service.**

| # | Decision | Solves |
|---|---|---|
| 1 | Clean Separation | Unclear Ownership — Ops and Dev deploy independently, no bottlenecks, no unguarded access. |
| 2 | Unified Tooling | Config Complexity — every service deploys from the same template, no reinvention, no drift. |
| 3 | Native Governance | Both — every change is traceable, attributable, and reversible by design, not by process. |

---

## Slide 5 — Decision 1: Clean Separation

**Draw the line between Ops and Dev — and justify it.**

| | Ops | Dev |
|---|---|---|
| **Owns** | Cluster, Capacity Providers, Container Insights, IAM Roles | Service, Task Definition, Log Group |
| **Tooling** | Pulumi | ecspresso |

**Why this boundary?**

- **Responsibility** — Cluster is Ops' domain; Service is Dev's.
- **Tooling** — Cluster lifecycle needs state management; Service deployment needs a fast feedback loop.
- **Governance** — Cluster changes require Ops review. Service changes are owned end-to-end by Dev.

**Risk of getting it wrong:**

- Boundary too tight → Ops becomes a bottleneck; Dev velocity suffers
- Boundary too loose → Dev accidentally modifies shared infra; blast radius expands across all services

---

## Slide 5b — Decision 1: IAM Enforcement

**Self-service without guardrails isn't self-service — it's uncontrolled access.**

| Role | Scope | |
|---|---|---|
| ECS Task Role | Per service | Least privilege enforced by infrastructure. |
| GitHub Actions Role (OIDC) | Per repository | Only repo pipeline can deploy; no long-lived keys. |

**Why Ops owns both:**

- **Deploy authority** — per-repo OIDC role; only the repo's pipeline can deploy, no long-lived credentials
- **Least privilege** — Task Role scope is an infrastructure decision, not an application decision
- **Auditability** — permission changes are version-controlled; Ops review required

**Risk of getting it wrong:**

- Shared deploy role → one compromised repo can deploy to any service
- Dev-managed Task Role → permissions drift without review; blast radius undefined

---

## Slide 6 — Decision 2: Unified Tooling

**Why unified tooling?**

**Three problems that compound with every new service:**

- **High cognitive load** — `task-def` + `service-def`, 20+ fields each
- **Reinventing the wheel** — every team builds the same structure from scratch
- **Config drift** — staging and prod diverge silently with every copy-paste

**The solution: ecspresso**

- **Jsonnet Templates** — fill in image, CPU, memory, ports; baseline handles the rest
- **Per-environment overrides** — env differences declared, not copy-pasted
- **Native Verification** — diff before apply, health check, rollback on failure

---

## Slide 6b — Decision 2: Unified Tooling (continued)

**What this looks like in practice**

One directory per service — structure is standardized, not negotiated:
```
services/my-app/
├── ecspresso.staging.yml
├── ecspresso.prod.yml
├── ecs-service-def.jsonnet
├── ecs-task-def.jsonnet
└── config/
    ├── app.libsonnet
    ├── staging.libsonnet
    └── prod.libsonnet
```

Three commands, every deployment:
```
ecspresso diff     # Preview changes
ecspresso deploy   # Rolling update with health check
ecspresso verify   # Confirm health or trigger rollback
```

> **Onboarding a new service:** copy the template directory, fill in `app.libsonnet`, deploy.

---

## Slide 7 — Decision 3: Native Governance

**You can always answer: who changed what, when, and why.**

*The toolchain isn't only a deploy mechanism — it is the audit trail.*

- **What changed** — Git is the deploy record; the timeline is queryable
- **Who changed it** — OIDC per-repo role; every action is attributed
- **Why it changed** — every change goes through a commit; intent is documented, not inferred

> Production breaks. The answer is already in Git — what changed, who changed it, and why.

---

## Slide 8 — Architecture & Constraints

*(Retain original diagram — overlay ownership annotations)*

**Ownership legend:**

- 🔴 **Ops:** ECS Cluster, Capacity Providers, Task Execution Role
- 🟢 **Dev:** ECS Service, Task Definition, CloudWatch Log Group
- ⬜ **Pre-existing / out of scope:** VPC, Subnets, ALB, Target Group, Security Groups

**What this design works with — and what it does not own**

| Resource | Status | Owner |
|---|---|---|
| VPC / Subnets | Pre-existing | Networking team |
| ALB + Target Group | Pre-existing | Networking team |
| Security Groups | Pre-existing | Networking team |
| ECS Task Execution Role | Separate IaC project | IAM project (Pulumi) |
| ECS Cluster | **This design** | Ops |
| ECS Service + Task Definition | **This design** | Dev |

**Fixed requirements:**
- Region: `ap-northeast-1`
- Network mode: `awsvpc` (Fargate requirement)
- Compute: Fargate only — no EC2 node management

---

## Slide 9 — Design Decisions & Trade-offs

**Why these tools?**

| Choice | Over | Reason |
|---|---|---|
| Pulumi | Terraform | Go-based; consistent with codebase; type-safe config |
| ecspresso | CDK / Terraform | Purpose-built for ECS; native diff/verify; lower cognitive load for Dev |
| Fargate only | EC2 + Fargate | No node management; ops overhead doesn't grow with service count |

**Risk of getting it wrong:**

| Risk | Mitigation |
|---|---|
| FARGATE_SPOT interruption | `Base: 1` on-demand task; rolling updates absorb interruptions |
| Stack Output coupling | Outputs treated as stable API; breaking changes require versioned migration |

---

## Slide 10 — Next Steps

**The foundation is ready. Onboarding the next service requires no platform work.**

- Cluster is ready — no provisioning
- Templates are ready — no boilerplate
- Deploy path is defined — no guesswork
- Ownership is clear — no ambiguity

**To onboard:**
1. Copy the template directory into your repo
2. Fill in `app.libsonnet` (image, CPU, memory, ports)
3. Open a PR — Ops reviews IAM, Dev owns the rest
4. Merge → GitHub Actions deploys

