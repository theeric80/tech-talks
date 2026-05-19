# Situation
Purpose: Knowledge transfer + onboarding — peers can submit a new-service PR unaided after the talk
Audience: Same-team senior engineers / reviewers; read Packer HCL and GitHub Actions YAML cold; have seen pieces in code review
Context: 30-min live talk + 10-min Q&A; no demo; ~15 main slides + 2 appendix slides surfaced only on Q&A
Success criteria: A peer leaves able to add a new service to the AMI pipeline (vars file + services.yml entry + compose file + dispatch) without hand-holding

# Storyline
Structure: Problem framing → Architecture → Onboarding → What's next

1. Hand-rolled AMIs drifted across services and clouds, so we built one declarative pipeline that bakes every service image the same way.
2. We split the build into a base AMI (OS + Docker + GPU driver) and a per-service app AMI, so kernel-level churn never re-runs for every service change.
3. The base AMI is one Packer template driven by a driver-variant matrix, so AWS and GCP share the same Ubuntu + Docker + nvidia-driver foundation.
4. The app AMI sources that base and bakes ECR login plus `docker compose pull/up` at image time, so instances boot already running the service.
5. GCP rides the same Packer + ECR pipeline, with workload-identity federation standing in for native OIDC and MIG standing in for ASG, so the AWS mental model transfers with one overlay file.
6. Two GitHub Actions workflows — one per layer — both gated on manual dispatch, so an AMI never appears in production by surprise.
7. Every service ships an optional smoke-test entrypoint that the build runs before publishing, so a broken compose file fails the build instead of the fleet.
8. Adding a service is exactly three files, one YAML entry, and one dispatch — everything else is convention, so onboarding is a checklist rather than a design exercise.
9. The service vars file declares identity, sizing, and the image tags Packer will pull, so Packer knows what to bake without touching the template.
10. Several of those fields cross-reference other files, so the vars file is really a contract — get the cross-references wrong and the build silently picks the wrong artifact.
11. The `services.yml` entry declares which clouds and regions to build for, so the CI matrix expands automatically with no workflow edits.
12. The `ServerSetup/<Service>/docker-compose.yml` is the single source of truth for what runs on the box, so the runtime contract lives with the service, not the pipeline.
13. Cloud and region overlays compose on top of the base vars file, so per-cloud quirks stay isolated to a few lines instead of forking the service.
14. `gh workflow run packer-app-image.yml -f services=<svc>` kicks the build, and instance refresh (AWS) or MIG rolling update (GCP) picks up the new AMI on a separate pipeline you do not own here.
15. The appendix covers multi-AZ retry and the bootstrap updater — pull them in only if Q&A asks.

# Ghost deck

## Slide 1 — Hand-rolled AMIs drifted across services and clouds, so we built one declarative pipeline that bakes every service image the same way

Evidence: title slide; one-line problem statement plus the talk's arc (Problem framing → Architecture → Onboarding → What's next)
So what: sets the frame — this is KT for a system you'll touch, not a pitch
Speaker note: 60 seconds; name `flux` as the pilot service so people anchor to something concrete; promise the onboarding checklist by minute 15.

## Slide 2 — We split the build into a base AMI and a per-service app AMI so kernel churn never re-runs for every service change

Evidence: two-box diagram — Base AMI (Ubuntu 22.04 + Docker CE + nvidia-driver + AWS CLI v2) feeding App AMI (service compose + ECR pull + smoke test); arrow labeled "source_ami_filter"
So what: explains why there are two Packer templates and two workflows, which the rest of the deck will mirror
Speaker note: this is the single most important mental model; pause here and check for nods before moving on.

## Slide 3 — One Packer template plus a driver-variant matrix produces the base AMI for both clouds

Evidence: file listing of `deployment/packer/base/` showing `base.pkr.hcl` and the five provisioners `01-core-setup.sh`, `02-nvidia-driver.sh`, `03-docker-nvidia.sh`, `04-aws-tools.sh`, `05-validate.sh`; callout listing the two driver variants (`nvidia-driver-580`, `nvidia-driver-580-open`)
So what: when someone asks "where does the GPU driver come from?", the answer is one script in one place
Speaker note: emphasize that the 4-job matrix (2 clouds × 2 driver variants) is what keeps base AMIs aligned; this is the only slide where we talk about scripts 01–05.

## Slide 4 — The app AMI sources the base and bakes ECR login plus `docker compose pull/up` so instances boot already running the service

Evidence: file listing of `deployment/packer/app/` showing `app.pkr.hcl` and the five provisioners `00-stage-samples.sh`, `01-setup-runtime.sh`, `02-gcp-credentials.sh`, `03-docker-pull.sh`, `04-validate.sh`; one short HCL excerpt of the `source_ami_filter` block pointing at the base
So what: there is no first-boot install step — the container layer is frozen at image time, which is why rollbacks are just an AMI swap
Speaker note: flag `02-gcp-credentials.sh` here so the GCP slide (next) lands smoothly.

## Slide 5 — GCP rides the same Packer + ECR pipeline; the only real deltas are workload-identity auth and MIG instead of ASG

Evidence: small 2-row table — concern / AWS / GCP — covering auth (native OIDC vs. STS `assume-role-with-web-identity` from GCP into the AWS ECR account) and fleet primitive (ASG vs. MIG); pointer to `flux.gcp.pkrvars.hcl` and `02-gcp-credentials.sh`
So what: if you understand the AWS path, GCP is a one-overlay-file delta — same registry, same image tags, same Packer template
Speaker note: this is the only GCP slide in the main flow; keep it to 90 seconds; explicitly bust the common misconception that GCP pulls from a separate Artifact Registry.

## Slide 6 — Two workflows, one per layer, both gated on manual dispatch so an AMI never appears in production by surprise

Evidence: side-by-side of `.github/workflows/packer-base-image.yml` (4-job matrix, OIDC) and `packer-app-image.yml` (dynamic matrix from `services.yml`, `services` input)
So what: the human-in-the-loop is the workflow_dispatch trigger; nothing in this pipeline runs on merge
Speaker note: pre-empt the question "why no auto-build on PR merge?" — answer is blast radius and cost. Retry / multi-AZ behavior is appendix A1 — pull only if asked.

## Slide 7 — Every service ships an optional smoke-test entrypoint that runs before publishing, so a broken compose fails the build instead of the fleet

Evidence: callout of `smoke_test_entrypoint` field in a `pkrvars.hcl`; one-line description of `deployment/packer/smoke-test.sh` launching a test instance from the manifest
So what: the AMI you publish has at minimum started its containers once — failing here is cheaper than failing instance refresh
Speaker note: Go unit tests run separately in AolCompu via `make test`; integration suite is deferred — mention but don't dwell.

## Slide 8 — Adding a service is three files, one YAML entry, and one dispatch; everything else is convention

Evidence: numbered checklist grouped — **Files you author**: (1) `<service>.pkrvars.hcl`, (2) `ServerSetup/<Service>/docker-compose.yml`, (3) optional `<service>.<cloud>.pkrvars.hcl`; **YAML entry**: (4) entry in `services.yml`; **Dispatch**: (5) `gh workflow run`
So what: this is the slide a peer screenshots before opening their PR
Speaker note: tell the audience the next six slides walk this checklist using `flux` as the worked example; this is the section they came for.

## Slide 9 — `<service>.pkrvars.hcl` declares identity, sizing, and the image tags Packer will pull

Evidence: annotated excerpt of `flux.pkrvars.hcl` highlighting `service_name`, `disk_size_gb`, instance types, `docker_compose_path`, `docker_image_tags` map, `service_samples_paths`, `smoke_test_entrypoint`
So what: this file is the contract between your service and Packer — no template edit is needed
Speaker note: walk top to bottom; pause on `docker_image_tags` since that's where the pre-pull comes from; flag that defaults come from `defaults.auto.pkrvars.hcl` and the full schema lives in `variables.pkr.hcl`.

## Slide 10 — Several of those fields cross-reference other files, so the vars file is really a contract you must satisfy

Evidence: four cross-reference arrows — `docker_image_tags` keys ↔ compose `${var}` substitutions; `service_samples_paths` destination → `smoke_test_entrypoint` path prefix; `services.yml` region → `<region>.pkrvars.hcl` overlay file; `nvidia_driver_variant` → matching base AMI variant
So what: this is the slide that turns onboarding from "copy flux and hope" into "fill in the contract"; mismatches are the #1 reviewer catch
Speaker note: mention the in-repo image-tag convention (semver-style `v1.26.1`; GCP overlay prefixes hardware tag, e.g. `6000pro-v1.26.1`); this is the slide reviewers will quiz on.

## Slide 11 — The `services.yml` entry declares which clouds and regions to build for, so the CI matrix expands automatically

Evidence: excerpt of `services.yml` showing a service block with `cloud: aws`, `region: us-west-2`, `replicate_to: [ap-northeast-1]`, and a GCP leg; authoring-rules callout (region required + selects the regional overlay file; `replicate_to` AWS-only)
So what: you never touch the workflow YAML — the matrix is derived from this file
Speaker note: `replicate_to` is the AWS-only multi-region knob; GCP ignores it. Region cross-reference enforcement (region must match an existing `<region>.pkrvars.hcl`) is covered by slide 10's mismatch table — point at it rather than restating.

## Slide 12 — You author `ServerSetup/<Service>/docker-compose.yml` alongside your service; the pipeline just pulls and starts it

Evidence: short excerpt of `ServerSetup/Flux/docker-compose.yml` showing service definitions and `${image_tag}` substitutions that bind to `docker_image_tags` keys from slide 9; callout "this is file (2) on the slide-8 checklist — you write it"
So what: the runtime contract lives next to the service, not in the pipeline; Packer is a dumb shipper, the compose file is source of truth
Speaker note: stress the `${...}` ↔ `docker_image_tags` cross-reference; this is the failure mode that slide 10's mismatch table already named.

## Slide 13 — Cloud and region overlays compose on top of base vars so per-cloud quirks stay isolated

Evidence: file list — `flux.pkrvars.hcl` + `flux.gcp.pkrvars.hcl` + `us-west-2.pkrvars.hcl` + `ap-northeast-1.pkrvars.hcl`; arrow showing layered merge order
So what: if your service is AWS-only, you write one file; if it spans clouds or regions, you add overlays — never a fork
Speaker note: this is also the slide where you point out that the GCP overlay is usually 5–10 lines.

## Slide 14 — `gh workflow run packer-app-image.yml -f services=<svc>` kicks the build; instance refresh and MIG rolling update pick it up downstream

Evidence: one-line `gh` command; arrow into a small downstream box labeled "ASG instance refresh (AWS) / Pulumi MIG rolling update (GCP) — separate pipeline"
So what: your responsibility ends at "AMI published"; deployment is a separate pipeline owned elsewhere, intentionally
Speaker note: name the downstream owners briefly so people know who to ping.

## Slide 15 — Recap and questions: three files, one YAML entry, one dispatch — and an appendix waiting for the curious

Evidence: condensed version of the slide-8 checklist on the left; bullet list of appendix topics on the right (multi-AZ retry, bootstrap updater)
So what: the title-only readthrough of slides 8–14 is the take-home; everything else is reference
Speaker note: pause for Q&A; pull appendix slides only if asked; expect questions on rollback and how runtime updates differ from AMI rebuilds.

## Slide A1 (appendix) — The capacity-retry loop re-rolls the subnet on each attempt, so a single AZ outage doesn't fail the build

Evidence: pseudocode of the retry loop in `packer-app-image.yml` showing 3h budget and 60s pause; callout that each `packer build` invocation picks a fresh subnet via UUID, so retries naturally land in a different AZ
So what: explains why builds sometimes take 40+ minutes but rarely need manual intervention
Speaker note: pull only if someone asks about long build times or AZ failures.

## Slide A2 (appendix) — `runUpdater.sh` is a runtime concern, not part of the build pipeline

Evidence: one-paragraph callout describing `AolCompu/runUpdater.sh` and where it lives in the boot sequence
So what: clarifies the boundary — anything `runUpdater.sh` touches is out of scope for AMI rebuild
Speaker note: pull if someone conflates bootstrap updates with image rebuilds.
