# AMI Build Automation

---

## Slide 1 — Hand-rolled AMIs drifted — one declarative pipeline now bakes them all

**Two layers. Two clouds. Same template.**

Problem framing → Architecture → Onboarding → What's next

Pilot service: `flux`

Speaker note: 60 seconds. Promise the onboarding checklist by minute 15. Anchor on `flux` for the worked example (slides 9–14).

---

## Slide 2 — Split the build: stable base feeds per-service app

```
┌─────────────────────────────────────────┐
│  Base AMI                               │
│  Ubuntu 22.04 · Docker CE               │
│  · nvidia-driver · AWS CLI v2           │
└──────────────────┬──────────────────────┘
                   │  source_ami_filter
                   ▼
┌─────────────────────────────────────────────┐
│  App AMI  (one per service)                 │
│  docker-compose.yml · pre-pulled images     │
└─────────────────────────────────────────────┘
```

Speaker note: This is *the* mental model for the rest of the deck — pause here and check for nods. Kernel-level churn never re-runs for a service-only change; say it out loud. Every later slide maps onto one of these two boxes.

---

## Slide 3 — Base AMI: one template + driver-variant matrix → both clouds

```
deployment/packer/base/
├── base.pkr.hcl
├── 01-core-setup.sh
├── 02-nvidia-driver.sh
├── 03-docker-nvidia.sh
├── 04-aws-tools.sh
└── 05-validate.sh
```

Matrix: `{aws, gcp}` × `{nvidia-driver-580, nvidia-driver-580-open}` → **4 jobs**

Speaker note: When someone asks "where does the GPU driver come from?" the answer is `02-nvidia-driver.sh`, one file, one place. This is the only slide where we name scripts 01–05.

---

## Slide 4 — App AMI: sources base, bakes compose + images

```
deployment/packer/app/
├── app.pkr.hcl
├── 00-stage-samples.sh
├── 01-setup-runtime.sh
├── 02-gcp-credentials.sh
├── 03-docker-pull.sh
└── 04-validate.sh
```

Sources base via: `source_ami_filter { name = local.base_image_name }` (exact name, not glob)

> No first-boot install step — rollback is just an AMI swap.

Speaker note: `base_image_name` is `"ami-base-ubuntu2204-${var.nvidia_driver_variant}-${var.base_image_version}"` — interpolated local, exact match. Flag `02-gcp-credentials.sh` here so the GCP slide lands smoothly.

---

## Slide 5 — GCP: same pipeline, only auth + fleet differ

| Concern | AWS | GCP |
|---|---|---|
| Auth into ECR | Native OIDC | metadata JWT → STS → temp creds |
| Fleet primitive | ASG | MIG |
| Packer template | shared | shared |
| Image tags | shared | shared (with optional hardware prefix) |

Pointers: `flux.gcp.pkrvars.hcl`, `02-gcp-credentials.sh`

Speaker note: 90 seconds, only GCP slide in the main flow. Auth chain in full: GCE metadata JWT → STS `assume-role-with-web-identity` → temporary credentials. Bust the common misconception that GCP pulls from a separate Artifact Registry — it doesn't, both clouds pull from the same AWS ECR.

---

## Slide 6 — Two workflows, one per layer, both gated on `workflow_dispatch`

|  | `packer-base-image.yml` | `packer-app-image.yml` |
|---|---|---|
| **Matrix** | 4-job (cloud × driver variant) | dynamic, from `services.yml` |
| **CI auth** | OIDC | OIDC |
| **Trigger** | `workflow_dispatch` | `workflow_dispatch` (`services` input) |

Speaker note: Nothing runs on merge — pre-empt "why no auto-build on PR merge?" with the blast-radius / cost answer. Say out loud: matrix is data, not workflow YAML you hand-edit — sets up `services.yml` on slide 11.

---

## Slide 7 — Smoke test fails the build, not the fleet

```hcl
# <service>.pkrvars.hcl
smoke_test_entrypoint = "./Tests/smoke.sh"
```

`deployment/packer/smoke-test.sh` launches a test instance from the build manifest; non-zero exit → AMI is **not** published.

Speaker note: The value: fail at smoke (one instance) instead of at instance refresh (the fleet). Smoke test runs L1 (host/Docker/GPU checks) + L2 (the entrypoint), then tags the artifact `SmokeStatus=passed/failed` (AWS) / `smoke-status=...` (GCP). Go unit tests run separately in AolCompu via `make test`; integration suite is deferred — mention but don't dwell.

---

## Slide 8 — Add a service: 3 files + 1 YAML entry + 1 dispatch

*Architecture done — checklist time.*

**Files you author**
1. `<service>.pkrvars.hcl` — Packer contract
2. `ServerSetup/<Service>/docker-compose.yml` — runtime source of truth
3. *(optional)* `<service>.<cloud>.pkrvars.hcl` — cloud overlay

**YAML entry**
4. `<service>` block in `deployment/packer/app/services.yml`

**Dispatch**
5. `gh workflow run packer-app-image.yml -f services=<service>`

Speaker note: Everything else is convention — emphasize that point. This is the slide a peer screenshots before opening their PR. The next six slides walk this checklist by dependency, not in list order, using `flux` as the worked example; reset the room before diving in.

---

## Slide 9 — [1] `<service>.pkrvars.hcl` declares identity, sizing, and images — no template edits

```hcl
# deployment/packer/app/flux.pkrvars.hcl
service_name          = "flux"
disk_size_gb          = 165
aws_instance_type     = "g6e.xlarge"
gcp_machine_type      = "g4-standard-48"
nvidia_driver_variant = "nvidia-driver-580"   # picks the base AMI
docker_compose_path   = "ServerSetup/Flux/docker-compose.yml"
docker_image_tags = {
  image_tag = "v1.26.1"   # ↔ ${image_tag} in compose (slide 12)
}
```

Speaker note: Walk top to bottom. Pause on `docker_image_tags` — that's where the pre-pull comes from and where slide 12 will plug in. Tag convention: semver `v1.26.1`; GCP overlay may prefix hardware, e.g. `6000pro-v1.26.1`. Defaults live in `defaults.auto.pkrvars.hcl`; full schema in `variables.pkr.hcl`.

---

## Slide 10 — Cross-reference mismatches silently bake the wrong artifact

| Source | → | Target |
|---|---|---|
| each key of `docker_image_tags` | → | same-named `${...}` in compose *(slide 12)* |
| `service_samples_paths.dest` | → | `smoke_test_entrypoint` path prefix *(slide 7)* |
| `services.yml` `region` | → | `<region>.pkrvars.hcl` must exist *(slide 11)* |
| `nvidia_driver_variant` | → | base AMI name `ami-base-ubuntu2204-<driver-variant>-<version>` *(slide 4)* |

> Build succeeds — wrong image.

Speaker note: This is the #1 reviewer catch and the slide reviewers will quiz on. Each row is a real failure mode we've seen.

---

## Slide 11 — [4] `services.yml` entry — cloud × region per build drives the matrix

```yaml
services:
  flux:
    builds:
      - cloud: aws
        region: us-west-2
        replicate_to: []                 # AWS-only knob; populate to fan out, e.g. [ap-northeast-1]
      - cloud: gcp
        region: us-west-2                # region key also picks <region>.pkrvars.hcl
```

Speaker note: Inline YAML comments carry the semantics — let the audience read them. Region cross-reference enforcement (must match an on-disk `<region>.pkrvars.hcl`) is on slide 10's mismatch table; point at it rather than restating. On GCP, `replicate_to` is ignored — Packer's `ami_copy` is hardcoded to `[]`.

---

## Slide 12 — [2] `docker-compose.yml` owns runtime; Packer renders `${...}` bindings

```yaml
# ServerSetup/Flux/docker-compose.yml (excerpt)
services:
  flux:
    image: ${repository}/rd-ai_flux:${image_tag}
    # ${repository}  ← docker_ecr_registry         (set in defaults / overlay)
    # ${image_tag}   ← docker_image_tags.image_tag (slide 9)
```

Speaker note: Packer renders a `/tmp/.env` at bake time with both bindings; `docker compose` picks them up via standard `${VAR}` substitution. Runtime contract lives next to the service — Packer is a dumb shipper. `${image_tag}` ↔ `docker_image_tags.image_tag` is the failure mode slide 10 named; call back to it explicitly.

---

## Slide 13 — [3] AWS-only = one file; multi-cloud adds an overlay, never a fork

```
flux.pkrvars.hcl                  (base — required)
   └─ flux.gcp.pkrvars.hcl        (cloud overlay — optional, ~5–10 lines)

us-west-2.pkrvars.hcl             (region overlay)
ap-northeast-1.pkrvars.hcl        (region overlay)

  merge order:  base ← cloud ← region
```

Speaker note: GCP overlay is typically 5–10 lines — point this out so the cross-cloud cost feels realistic, not scary.

---

## Slide 14 — [5] Dispatch with `gh workflow run`; instance refresh / MIG update is Ops

```
$ gh workflow run packer-app-image.yml -f services=flux
        │
        ▼
   AMI published
        │
        ▼  (separate pipeline, owned by Ops)
┌──────────────────────────────────────────────────────┐
│ AWS: ASG instance refresh                            │
│      infra-aws-computing-provision-{beta,prod}.yml   │
│ GCP: MIG rolling update                              │
│      infra-gcp-computing-provision-{beta,prod}.yml   │
└──────────────────────────────────────────────────────┘
```

> Your responsibility ends at *AMI published*.

Speaker note: Instance refresh (AWS) / MIG rolling update (GCP) onto the new AMI happens inside the provision workflows above; the exact rollout cadence is an Ops concern — point people at `#infra-ops` rather than explaining it here.

---

## Slide 15 — Recap: 3 files + 1 YAML entry + 1 dispatch

| Onboarding checklist | Appendix (Q&A only) |
|---|---|
| 1. `<service>.pkrvars.hcl` | A1. Capacity-retry / multi-AZ |
| 2. `docker-compose.yml` | A2. `runUpdater.sh` boundary |
| 3. *(optional)* cloud overlay | |
| 4. `services.yml` entry | |
| 5. `gh workflow run` | |

Speaker note: The title-only readthrough of slides 8–14 — the checklist plus its worked example — *is* the take-home; say so on the close. Pause for Q&A. Pull appendix slides only if asked. Expect questions on rollback and how runtime updates differ from AMI rebuilds.

---

## Slide A1 (appendix) — Capacity-retry re-rolls the subnet per attempt

```hcl
# app.pkr.hcl — random pick happens INSIDE the template, every invocation
aws_subnet_id = element(
  var.aws_subnet_ids,
  parseint(substr(uuidv4(), 0, 7), 16)   # fresh uuid each packer run → different AZ
)
```

Outer loop in `packer-app-image.yml` re-invokes `packer build` for up to 3h with 60s pause between attempts.

Speaker note: GCP path is identical, with `gcp_zones` instead of `aws_subnet_ids`. Explains why builds occasionally take 40+ minutes but rarely need manual re-runs. Pull only if someone asks about long build times or AZ failures.

---

## Slide A2 (appendix) — `runUpdater.sh` runs at boot — out of scope for AMI rebuild

`AolCompu/runUpdater.sh` runs at boot, after the AMI is already live. Anything it touches is **out of scope** for AMI rebuild.

Speaker note: Bootstrap updates ≠ image rebuilds — say it plainly. Pull if someone conflates the two. Useful boundary to draw when discussing rollback semantics.
