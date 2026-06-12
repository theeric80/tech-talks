# Situation
Mode: A-Decision
Purpose: Proposal review — walk the room through the proposed cloud security architecture for the Remotion Agent, invite changes, and get sign-off.
Audience: Mixed engineering + management + the CT dev team, non-native English speakers; most attended the prior architecture meeting and know the agreed direction and threat model.
Context: ~30-min live talk + Q&A; no demo; ~11 main slides + 3 appendices (A1–A3); live discussion in Chinese, slides and speaker notes in English.
Success criteria: The room reviews the proposal and signs off — endorsing the proposed design as-is (including the two-tier session memory), deciding the one open egress choice (Option B vs C, recommendation C), and accepting the named residual risks.

# Storyline
Structure: Problem → Options compared → Recommendation
1. Last meeting agreed the direction — isolation, access via proxy, default-deny egress, isolated session memory — and this proposal secures each agreed item, leaving egress (Option B vs C) as the only open decision today.
2. The proposed design puts the agent in its own private subnet and moves containment to the agent's egress side, because the current shared VPC cannot contain code that may be hostile.
3. No single layer is the boundary; five independent layers each hold even if another fails.
4. The sandbox runs under gVisor with no IAM role and IMDS disabled, so a compromised agent holds only a per-turn token that is useless outside the VPC.
5. The sandbox never holds real credentials: every model and tool call goes through a gateway that checks the per-turn token.
6. Containment is enforced on the agent's own way out with a default-deny source-side allowlist, not on the internal services it might reach.
7. That allowlist has two valid implementations, and we recommend Option C (AWS Network Firewall) for transparent filtering and managed operations at about $288/month over Option B (Squid proxy, about $44/mo).
8. Hot session state lives in a dedicated Valkey reached only through the tool gateway, isolated per session by deriving the key prefix from the per-turn token's claim, so one session cannot read another's keys.
9. Large session files live on per-session EFS directories, isolating unprivileged containers from each other but not against a host-root escape.
10. Three risks remain that this network design does not stop — co-location escape, per-turn-token replay, content-level exfiltration — each owned by another control and accepted with stated limits.
11. Sign off: endorse the proposed design as-is, decide egress (recommend C), and accept the residual risks.

# Ghost deck

## Slide 1 — Agreed direction & today's decision

Evidence: two-block layout — "Agreed last meeting" (isolation: sandbox + container, ephemeral per turn · access: LLM/VLM via proxy + per-turn token · egress: default-deny, proxy only · session memory: isolated per session) and "Decision today" (sign off the proposed architecture).
So what: recalls the agreed direction the room already endorsed; the only true decision today is egress, the rest is endorse-as-proposed.
Speaker note: Everyone attended — recall the agreed direction in one breath, then frame the proposal as securing each agreed item via the five-layer defense (slide 3) and the details (slides 4–9). Ask: review, flag changes, sign off; egress is the only open part. Gloss IMDS and gVisor once.

## Slide 2 — Proposed architecture

Evidence: the proposed-architecture flow diagram — GenAI VPC with agent subnet (EC2: launcher on host + gVisor sandbox), tool-gateway subnet (tool gateway + dedicated Valkey), egress path (Network Firewall + internet gateway), GenAI subnet (RabbitMQ + agent-relay); EFS and S3 Gateway endpoint; agent reaches model via egress, tool/memory via the tool gateway, storage via EFS, presigned-URL bytes via S3 endpoint.
So what: every following slide is one layer of this picture; the boundary is the agent's egress, not the services it might reach.
Speaker note: This is the mental model — pause for nods. The current shared VPC cannot refuse the agent (Security Groups allow inbound 10.0.0.0/8, cannot DENY), so containment moves to control-plane objects at the subnet edge. Valkey holds hot session state via the tool gateway (slide 8); large files on EFS (slide 9). Diagram shows recommended Option C; Option B swaps in a Squid proxy. Before/after is in A1.

## Slide 3 — Defense-in-depth — five layers

Evidence: five-row table — 1 Isolation (gVisor around the sandbox) · 2 IMDS (disabled, no instance role) · 3 Credentials (gateways hold the real secrets) · 4 Domain allowlist (default-deny, allowed domains only) · 5 Network (subnet allowlist on outbound traffic).
So what: defense-in-depth — a gVisor escape is low-probability but not impossible, so no layer is trusted alone.
Speaker note: Keep the body to the five layer names + one phrase each. Each layer still works if another fails. This frames slides 4–6, which each open one layer. Gloss gVisor once.

## Slide 4 — Sandbox isolation & zero-credential

Evidence: three-block body — Isolation (gVisor sandbox, new per turn, launcher on host) · Hardening (non-root, read-only, cap-drop ALL) · Credentials (no IAM role, IMDS disabled; only the per-turn token). Tag: agreed last meeting — isolation.
So what: even if the agent is fully compromised, there is no instance role to steal and no standing secret in the sandbox.
Speaker note: Isolation strength is proposed for endorsement — last meeting only agreed to use a sandbox. Zero-credential beats least-privilege: arbitrary code completes the IMDS handshake legitimately, so the only safe role is one that does not exist. gVisor over runc: runc had escape CVEs in 2024–25 (e.g. CVE-2024-21626), gVisor none — keep in the note.

## Slide 5 — Credential mediation

Evidence: three-row table — Agent sandbox holds only the per-turn token · agent-relay (model gateway, already public) holds provider API keys · Tool gateway holds the AWS role + tool keys. Tag: agreed last meeting — access via proxy + per-turn token.
So what: real secrets never enter the sandbox; a compromised agent can only make calls its per-turn token already allows.
Speaker note: One idea — credential mediation. The per-turn token is signed by the GenAI API server at enqueue and checked by both gateways against a JWKS key set; one queue message = one turn. Two S3 paths: presigned-URL issuance via the tool gateway, bulk bytes straight to the S3 Gateway endpoint (skips the gateways). Session memory (Valkey) uses the same tool-gateway path (slide 8). Gloss JWT/JWKS once; say "per-turn token" throughout.

## Slide 6 — Egress containment

Evidence: principle line (agent's outbound is a source-side allowlist, not the destination — everything else denied) + egress-slice diagram — agent reaches only the tool-gateway subnet, EFS, and the egress path → internet (agent-relay); rest of VPC denied.
So what: the agent cannot reach any internal service that is not on its own outbound allowlist, regardless of what that service permits.
Speaker note: Egress slice of slide 2, same colours. RabbitMQ is the launcher's task I/O, not the agent's — sandbox has no path to it (gVisor blocks it even though the instance SG allows the EC2 host, A2). Layer 5 (subnet allowlist, NACL/SG) pins allowed subnets and denies the rest of 10.0.0.0/8; layer 4 (domain allowlist) filters which domains the Network Firewall lets out. Valkey sits behind the tool gateway, never on the agent's allowlist directly. DoH exfiltration covered on slide 10.

## Slide 7 — Egress options — the decision

Evidence: B-vs-C decision table — Option B (Squid proxy): fixed cost ≈ $44/mo (t3.medium; ≈ $88 with HA), self-built (patch/scale/HA), proxy variables required, stronger encrypted-DNS-bypass block. Option C (AWS Network Firewall): ≈ $288/month, AWS-managed, transparent (no agent config), weaker DNS-bypass block. Recommendation line: Option C.
So what: Option C filters transparently (catches generated code that ignores proxy settings), is fully managed, and is multi-AZ by config; we recommend it.
Speaker note: The only real decision today — walk the criteria, then the pick. Be fair to B: its deny is the strongest possible (no internet route at all), it is cheapest, but we patch/scale/run HA ourselves and one instance is a single point of failure (~$88/mo HA, still under C); ap-northeast-1 on-demand. On C cost: the ≈$288/mo is the firewall endpoint; the NAT gateway charges are waived in the same service chain. B→C later is a control-plane change only. Remaining slides are implementation, no further decisions.

## Slide 8 — Session memory (Valkey)

Evidence: three-block body — Store (session state) · Access (via tool gateway, no standing credential, per-session keys isolated) · Limit (cross-session needs a sandbox escape — then one session at a time → slide 10). Tag: session memory — small, hot state.
So what: on the normal path the agent reaches only its own session and holds no Valkey credential; reaching another session takes a sandbox escape, and even then one session at a time (slide 10 co-location).
Speaker note: For the dev team — session state lives in a dedicated Valkey reached only through the tool gateway; the sandbox holds no Valkey credential and never connects directly. The gateway derives the key prefix (`session:{id}:…`) from the per-turn JWT's `session` claim and ignores any agent-supplied session_id, so a session reads/writes only its own keys and can't forge another's. Dedicated instance, not the shared GenAI Valkey — separate keyspace/eviction/SG keep the adversarial agent out of the shared cache's blast radius. Cross-session takes a sandbox escape: only host root can replay a live token through the gateway, and even then reaches just that one session's keys — no direct or bulk access, bounded by token expiry (slide 10 co-location). Committed design baseline, settled with the dev team — not a new decision.

## Slide 9 — Session storage (EFS)

Evidence: three-block body — Store (workspace files) · Access (direct mount of own session dir, standing mount, per-session dir isolated) · Limit (cross-session needs a sandbox escape — then every session at once → slide 10). Tag: session storage — large files.
So what: the mount namespace (plus cap-drop, no remount) separates sessions on the normal path; no second permission layer, so a sandbox escape reads every session's files at once (slide 10 co-location).
Speaker note: For the dev team — write only under `/mnt/workspace` (own EFS session dir, persists across turns) and per-task tmp scratch; everything else read-only. Isolation is the mount namespace: each container bind-mounts only its own `/mnt/efs/sessions/<id>`, so there is no in-container path to another session's directory, and cap-drop ALL (CAP_SYS_ADMIN removed) stops it remounting the parent. No per-session UID is assigned — containers share one UID — so the namespace is the only layer between sessions; if it is ever crossed, POSIX permissions give no fallback. Cross-session takes a sandbox escape: a gVisor escape gives host root, which reads every session under `/mnt/efs` directly (slide 10 co-location).

## Slide 10 — Residual risks

Evidence: three-row table — risk / bounded by: (1) sandbox breakout reaches other sessions on the shared host (gVisor, breakout rare; reduce later via one host per session); (2) stolen per-turn token stays valid until expiry (kept short-lived, no mid-life revoke); (3) data leaks hidden inside an allowed model call (size + rate limits + full logging at agent-relay). Footer: this design limits the damage of a compromise — it does not stop prompt injection itself (owned by human review + least-privilege tools).
So what: the network design limits blast radius; it does not remove these, and we ask the room to accept them with their stated bounds.
Speaker note: Honest slide — keep plain, detail only if asked. Co-location is widest: a gVisor escape gives host root → every session's per-turn token (including live tokens in queued RabbitMQ payloads), all EFS data under `/mnt/efs`, and result-queue forgery; bounded by AMQP-over-TLS, least-privilege RabbitMQ user, tight expiry; reducible via one-host-per-session. Valkey is not in that blast radius directly (gateway-mediated, separate subnet, no agent-held credential) — but host root can replay a live token through the tool gateway to reach that one session's keys, so bounded not immune; no direct/bulk Valkey access. Token replay: one token serves many calls, can't be revoked after first use, valid until expiry. Content-level exfil: agent encodes data into an allowed model request with its legitimate token — bounded by per-session size/rate caps + full prompt logging at agent-relay and by denying agent-supplied tool definitions. DoH lives here, owned by the allowlist. Presigned-URL replay is deliberately not raised — existing S3 practice bounds it. Prompt injection is the root cause; the design only limits blast radius.

## Slide 11 — Decision summary & sign-off

Evidence: "Sign-off means agreeing to" three-item list — (1) the proposed design (gVisor sandbox · zero-credential · two gateways · per-turn token · Valkey + EFS); (2) egress — AWS Network Firewall; (3) the residual risks (slide 10). "Not today" line: agent-on-Fargate deferred. "The ask" line: review · flag any changes · sign off.
So what: the explicit ask — endorse the design as-is, pick egress (recommend C), accept the residuals.
Speaker note: Close on the ask, not "thank you / questions" — keep on screen during Q&A. The only decision that must land today is egress; everything else is endorsement + accepting residuals. Deferred target: separate VPC + Fargate move that removes the co-location residual.

## Appendix A1 — Current architecture (before)

Evidence: the current-state diagram — shared, unsegmented VPC (ap-northeast-1) with ALB, API tier (API, RabbitMQ, etcd), ASG (Launcher, AI Engine), managed services (DB, Cache, EFS), S3.
So what: shows the shared, unsegmented layout the proposal replaces; the agent would run alongside these services with no boundary.
Speaker note: Pull up only if the room wants the before/after contrast. The proposal adds the private agent subnet, the tool gateway, and the egress path.

## Appendix A2 — Subnet boundary: Network ACL + Security Group

Evidence: two tables — Security Group outbound (stateful, allow-only; default rule removed): NFS 2049 → EFS, Custom TCP 5671 → RabbitMQ (AMQPS), HTTPS 443 → 0.0.0.0/0 (NFW filters domains, NACL pins internal). Network ACL outbound (stateless, by rule #): 100 NFS 2049 → EFS /32 Allow · 110 5671 → RabbitMQ /32 Allow · 120 443 → tool-gateway subnet CIDR Allow · 200 UDP 443/80 Deny (QUIC) · 300 all → 10.0.0.0/8 Deny · 400 443 → 0.0.0.0/0 Allow (→ firewall).
So what: the per-rule detail behind slides 3 and 6; the agent reaches only EFS / RabbitMQ / tool-gateway subnet internally, internet 443 routes to the firewall.
Speaker note: NACL = subnet firewall (stateless, needs inbound ephemeral 1024–65535); SG = instance firewall (stateful). Order matters: in-VPC allows (100–120) and QUIC + 10.0.0.0/8 denies (200, 300) precede the broad internet allow (400). Under Option B there is no default route, so the SG becomes a tight allowlist (no 0.0.0.0/0). The allowlist is instance-level (launcher + sandbox share the ENI), so the sandbox's inability to reach RabbitMQ rests on gVisor (slide 10). Ports / rule numbers illustrative.

## Appendix A3 — Option C cost

Evidence: cost table (ap-northeast-1) — Network Firewall endpoint $0.395/h ≈ $288/month · Traffic processed +$0.065/GB · NAT gateway per-hour & per-GB waived (same service chain as Network Firewall) · S3 bulk bytes off this path (via S3 Gateway endpoint).
So what: backs the slide-7 recommendation — the firewall endpoint is effectively the only added egress cost.
Speaker note: NAT per-hour and per-GB charges are waived in the Network Firewall service chain; S3 bulk bytes bypass via the Gateway endpoint, so processed volume is mainly model calls and package installs. Cost detail if finance asks.
