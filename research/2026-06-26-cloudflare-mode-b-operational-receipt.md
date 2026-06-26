---
title: Mode B on Cloudflare — operational receipts from an outside operator already running the shape
subtitle: Field notes contributed against cortex PR #1189 (design-isolated-stack-hosting.md), the "Joining a Network — Sovereign Model B Federation" infographic, and ADR-0015 (community two-tier model)
authors:
  - Robert Chuvala (Northwoods Sentinel; principal of The Rhizome)
  - Margin (fleet instance, Lares substrate)
date: 2026-06-26
license: CC-BY-4.0
status: contribution candidate for mellanon/pai-collab/research/ — submitted 2026-06-26
composes_with:
  - 2026-06-07-rhizome-stack-interface-map.md (prior art, layer-mapping)
  - the-metafactory/cortex PR #1189 — Mode A + Mode B distributed execution
  - the-metafactory/cortex docs/runbook-federation-peering.md (NATS leaf transport via cloudflared)
  - ADR-0015 (community two-tier model: Chat vs Sovereign)
  - the-metafactory/cortex src/runner/execution-backend.ts:33 (existing `cloudflare` placeholder)
posture: vendor-agnostic; CF as one operational reference recipe alongside others. Andreas's 6/15 #off-topic statement holds: *"choice is an important aspect of a vendor agnostic platform."*
---

# Mode B on Cloudflare — operational receipts from an operator already running the shape

## §1 · Frame — naming the convergence first

The 2026-06-07 Rhizome ↔ stack interface map (open as PR #111 against `mellanon/pai-collab`, pending review) documented that I and Andreas independently drafted family-resembling architectures in the same week of 2026-05, neither aware of the other at draft time. That paper named the *protocol-stack ↔ substrate-cooperative* layer mapping and proposed the compositions across substrate / comms / surface / trust / governance / portable-assistant layers. The convergence was real.

Today, Andreas posted the "Joining a Network — Sovereign Model B Federation" infographic in #cortex with an open invitation: *"this is what we're aiming for... Open to ways to make it more plug-and-play."* PR #1189 has since split the work cleanly — Mode A (elastic execution, post-release) and Mode B (isolated stack hosting, pre-release federation test-zone track). The Mode B design doc names exactly the shape The Rhizome has been operationally running on Cloudflare since mid-2025: a sovereign head daemon on the principal's own infrastructure, federating outbound into a network of peers.

This contribution is the substrate-side half of that convergence stated honestly. The Rhizome and cortex Mode B are not competing things — they are two halves of the same architecture that the 6/07 paper already named as composing. Cortex is formalizing the M7 head shape, the federation protocols, the M2–M6 envelope discipline, the trust grammar. The Rhizome has been operationally validating what the head's substrate floor looks like in production on one specific provider's primitives. **What I am contributing is the field-tested CF substrate floor mapped to cortex's emerging Mode B head shape.** Not a competing system; the substrate axis of the architecture the two-author 6/07 paper already proposed composing.

Vendor-agnostic principle holds, and Andreas named it cleanly on 2026-06-15 in #off-topic: *"choice is an important aspect of a vendor agnostic platform."* CF is the substrate I can speak to from 18 months of receipts. The right ecosystem outcome is reference recipes per substrate (CF, AWS, GCP, Hetzner, self-hosted, bare-metal) authored by operators with shipping experience on each. This document is the CF reference recipe candidate, contributed as the substrate-axis complement to cortex Mode B's head-axis design.

## §2 · What's already in cortex on the CF axis

Three concrete observations before adding anything new:

The cortex codebase already declares Cloudflare as a sandbox backend type. `src/runner/execution-backend.ts:33` carries `type: "cloudflare" | "e2b" | "ssh" | "custom"`. The seam is in.

PR #1189's design-distributed-agent-execution.md (Mode A) names CF Containers — *full microVMs, Linux, SSH* — as the target for the managed-backend hands, with Workers VPC and Outbound Workers (Apr 2026 GA) for the credential-injecting egress proxy. Andreas's text: *"`backend: managed` MUST mean run that exact same `claude -p` hand inside a CF Container (full microVM)."* The Mode A architecture is already CF-shaped.

cortex docs/runbook-federation-peering.md documents `cloudflared` TCP tunnel as the recommended NATS leaf transport for the 2-party case (replaced by JC's public hub for steady-state). The doc explicitly says *"hub side already has cloudflared + a CF zone (meta-factory.ai)"* — operational CF infrastructure is already in the federation path.

Mode B (head-on-its-own-infrastructure) is the natural extension of these existing commitments to the *head* axis. The Rhizome has been running this shape for 18 months; the receipts below are what showed up.

## §3 · Layer mapping — Mode B head on Cloudflare primitives

The cortex Mode B head is described as: persistent daemon, identity-bearing, bus presence, surface adapters, dispatch + conversation state, durable session log. Mapping each onto CF primitives:

| Cortex Mode B concern | CF primitive | Operational receipt |
|---|---|---|
| Persistent daemon (long-lived process) | **Workers** for stateless request handling; **Durable Objects** for stateful identity-bearing components | Workers don't run a continuous loop the way a Bun process does; the cortex daemon as designed (`Bun.spawn`-based, holds NATS connections) does NOT port directly. The CF-native shape is *event-driven* — DO actors that wake on incoming messages, hold ephemeral memory between invocations, and persist canonical state to D1/R2. This is a non-trivial port, not a recompile. |
| Stack identity + signing | **Workers Secrets** for NKey seed; **DO** for signing-key-holding actor with explicit storage class | Per-stack signing seed lives in DO Storage (encrypted at rest, per-actor isolation). Public key advertised through standard registry path. Operationally cheap (~zero ongoing cost at idle), durable across CF region failover. |
| Bus presence (NATS leaf node) | **NOT NATIVELY ON CF.** Three options: (a) `cloudflared` TCP tunnel to a NATS server on a small VPS, (b) JC's public hub via leaf dial-out, (c) explore CF Queues + DO + Workers as alternative federation transport for the M2 layer | **This is the unsolved bit.** The Rhizome currently runs NATS-equivalent traffic over signed-HTTP (mycelia envelope spec) on Workers; no JetStream-class guarantees. The decision for cortex Mode B on CF is whether to require an external NATS leaf (breaks "fully CF" promise) or invest in a CF-native federation transport that approximates NATS guarantees for the leaf-link case. |
| Surface adapters (Discord, etc.) | **Workers** with WebSocket support; **Cron Triggers** for scheduled work | Workers' WebSocket support is mature (Discord gateway connection via persistent WebSocket works; ~1 verified Rhizome instance on Workers). The DO pattern handles per-channel state cleanly. Cron Triggers cover scheduled cortex behaviors (digests, retros). |
| Durable session log (Mission Control event stream) | **D1** for structured event log; **R2** for blob attachments; **Analytics Engine** for time-series query | D1's SQLite shape matches Mission Control's per-agent event log well. R2 for attachments (screenshots, large diffs). Analytics Engine for Tier-1 dashboard time-series queries cheaply. |
| Surface adapter for Mission Control dashboard | **Pages** (static + Pages Functions) | The cortex dashboard is a React tree already (`src/surface/mc/dashboard-v2/`); Pages Functions handle the API, deploys via `wrangler pages deploy`. The cortex repo already documents this deploy pattern. |
| LLM routing + observability | **AI Gateway** | Already proposed in The Rhizome's substrate layer (per 2026-06-07 interface map §2.1). Logs every model call, supports caching + rate-limit shaping + cross-provider routing without changing application code. |

The layer mapping is clean for everything except the NATS leaf transport. That's the load-bearing unsolved bit for a "fully CF" Mode B.

## §4 · The NATS-on-CF gap — three operational options

This is the substantive technical question Mode B-on-CF has to answer. Options ordered by friction:

**Option 1 — NATS leaf on a small VPS, exposed via cloudflared.** This is what cortex's runbook-federation-peering.md already documents for the 2-party case. Operational receipt: works reliably (cloudflared TCP tunnels are mature), but it breaks the "join in one wrangler deploy" promise — the operator has to provision a non-CF box. The pre-deployed-tunnel pattern is workable but feels off-thesis for a CF-native Mode B recipe.

**Option 2 — JC's public hub via leaf dial-out.** Cortex's runbook explicitly names this as the steady-state path: *"a public NATS hub (JC's hub) makes the cloudflared 'Reachability' section below moot for the 2-party case — leaf nodes dial outbound to the public hub (NAT-safe)."* For Mode B on CF, this means: the head Worker/DO dials outbound to JC's hub (which CF allows by default for outbound TCP from Workers). One-command join is achievable here. Cost: dependence on JC's hub as critical infrastructure; sovereignty trade-off (your bus traffic flows through someone else's hub even if encrypted at the application layer).

**Option 3 — CF-native federation transport.** Explore replacing NATS-leaf-link with CF Queues + Durable Objects + Workers for the M2 transport between sovereign Mode B stacks. This is a substantial architectural investment — myelin's M2 abstraction would need to grow a CF-Queues backend alongside its NATS backend. Pros: fully CF-native, no external dependencies, leverages CF's own reliability. Cons: doesn't compose with non-CF stacks (they still need NATS); fragments the federation protocol; substantial design + implementation work.

**Operational recommendation from the receipt side:** Option 2 (JC's public hub) is the right Mode-B-on-CF first move because it ships the one-command join immediately and lets operators run as sovereign-substrate-with-shared-hub. Option 1 stays available as the air-gapped fallback (the runbook already covers it). Option 3 is a longer-horizon architectural conversation that probably belongs in a follow-up ADR rather than blocking Mode B Spike B1.

## §5 · What plug-and-play actually looks like in CF terms

The infographic shows a 5-step join: Provision → Register → Pin → Admission → Verify. Mapping each to one wrangler deploy + minimal config:

**Provision the principal seed.** `wrangler secret put NSC_SEED` on the operator's CF account, OR generate at install time via `cortex provision-stack generate` inside the template repo's first-run wizard. The seed lands in Workers Secrets (encrypted at rest, per-Worker access only).

**Register at the registry.** A POST to `https://network.meta-factory.ai/register` from the head Worker after first boot. The Workers fetch call is one line; the proof-of-possession claim is signed in-Worker via the NSC seed. No CLI required if the template's first-boot init does this automatically.

**Pin the registry.** Trust anchor delivery — Luna posted today (16:39 UTC) that PR #1232 just merged: *"default registry trust anchor. The metafactory registry is now pre-pinned into cortex (root-CA model), so 'pin the registry' disappears from the join flow on the happy path."* This step is *already* getting eliminated. The CF recipe inherits the trust-anchor-pinning-by-default automatically.

**Admission.** The community-fleet role grant (per ADR-0015 Chat tier semantics extended to Sovereign tier) — the principal gets admitted by a maintainer action; the bot prepares + executes the grant. The CF recipe needs to expose an "admission status" endpoint the head polls (`GET /admission/<stack-id>`) until it returns granted, then proceeds. Workers fetch + polling pattern is trivial.

**Verify.** The leaf link establishes (via Option 2's public hub dial-out). The head Worker emits a verification event. Operator sees green in Mission Control.

End-to-end: clone the `cortex-stack-cf-template` repo, `wrangler deploy`, paste the admission URL the bot DMs you when granted, done. ~5 minutes including the human-side admission wait.

## §6 · What hurts — install-pain receipts from this morning

Concrete friction from my own cortex install on 2026-06-26 morning (before the Mode B work). Submitted as data for the plug-and-play target:

The community arc registry at `the-metafactory/meta-factory/REGISTRY.yaml` returns 404, so `arc upgrade Cortex` (the canonical install per cortex/README-AGENTS.md §2 Path A) doesn't work for outside operators. I had to fall back to Path B source-install (`git clone the-metafactory/cortex && bun install`). For the Mode B CF template recipe, this means the template can't depend on `arc upgrade Cortex` either — has to vendor or pin cortex itself.

The `agents/CREATING.md` flow documents `grove install agent <name>` as canonical, but `the-metafactory/grove` is private (404 publicly). External operators reading that doc as the install path land on a dead end. The Mode B recipe needs to either point at the current cortex stack flow explicitly OR be the doc that supersedes agents/CREATING.md for external operators.

Privileged Gateway Intents (Server Members + Message Content) toggle at developers.discord.com is irreducible — Discord no-ops API-side PATCH to `/applications/@me` flags. The Mode B template can pre-document the exact two-toggle step but can't automate it. This is the one irreducible human step; documenting it as such (not hiding it) is the honest plug-and-play surface.

PI-001 in `@metafactory/content-filter` (the regex matching bare "you are") false-positives on legitimate principal queries. I patched it for my install (loosened to require "you are now/actually/really/no longer/not supposed/allowed/restricted"); upstream should consider the looser regex. Filed as a separate concern but it composes — first-time operators will hit this on their introduce-the-bot-to-yourself test.

NATS server install was a separate step (`/root/Bin/nats-server v2.14.2` downloaded from GitHub releases — no apt package available on the Lares distro). For Mode B on CF this evaporates if Option 2 ships first (no local NATS needed); it stays a friction point if Option 1 is the recommended path.

## §7 · The composes-with conversation worth having

The Rhizome ↔ stack interface map I co-authored on 2026-06-07 (pending in PR #111) already names per-member CF account as The Rhizome's substrate boundary, with `ai-gateway` Worker + `meridian-observer` Worker + `member-daemon` DO. The mapping in §3 above is the natural extension: cortex's Mode B head deployed onto the same substrate primitives The Rhizome has been operationally validating since mid-2025.

Two specific compositions worth surfacing:

**Soma compatibility.** Soma is the substrate-neutral portable-assistant-body. The interface map names: *"A Rhizome member could run soma on their Cloudflare substrate; the substrate is The Rhizome's, the assistant body is soma's."* Mode B on CF + soma means an operator can pick their substrate (CF here, anything else with a substrate recipe), bring their soma-shaped persona, federate into the network. The three layers compose cleanly.

**Meridian as the trust-scoring backend.** The interface map §2.4 noted Meridian (The Rhizome's quantitative trust-math primitive) as the structural primitive the metafactory stack does not visibly have. Mode B on CF gives Meridian the substrate it was always designed against — DO + D1 + KV — so a working Meridian instance can be authored on the CF recipe to test against. The Hive's git-based Trust Ledger (per the interface map) becomes Meridian's evidence layer.

## §8 · What I can contribute next

Stated as concrete next-step deliverables, not vague offers:

A working `cortex-stack-cf-template` repo with the layer mapping in §3, wrangler config that provisions the DO + D1 + R2 + KV + Workers + AI Gateway primitives in one deploy, head Worker + DO actors implementing the cortex daemon shape. Initial Spike B1 candidate.

A Mode B installation log — what actually happens when I `wrangler deploy` the template from a fresh CF account through to first-mention in The Metafactory guild. Real-time friction documentation that feeds back into the design doc.

The unsolved NATS-on-CF question worth a dedicated design doc — either as Mode B Spike B2 (Option 2 hub-dial-out validation) or a separate ADR proposing the M2 transport extension for the CF-native case (Option 3 — longer horizon).

Authorship on the substrate-aware messaging layer thesis I drafted today (`mellanon/pai-collab/research/2026-06-26-substrate-aware-messaging-layer.md` — staged for upstream PR) — the human-tier communication layer that composes Signal Protocol crypto with mycelia/myelin-style intent envelopes, sitting above the Mode B head per substrate.

## §9 · Posture

I run on Cloudflare and I am not paid by Cloudflare. The Rhizome was authored before cortex existed publicly; I did not design it to be Mode B-shaped, and Andreas did not design Mode B to be Rhizome-shaped. The 6/07 paper documented the independent convergence; this document is the operational substrate-floor receipts feeding the head-shape design now.

What's not in scope here: convincing anyone that CF is the right substrate. The vendor-agnostic principle Andreas named on 2026-06-15 — *"choice is an important aspect of a vendor agnostic platform"* — is the right principle. The right ecosystem outcome is reference recipes per substrate authored by operators who have shipped on each, with the cortex protocols substrate-neutral by design.

What's in scope: surfacing what already works on CF from 18 months of running the shape, naming the unsolved bits honestly, offering the substrate-floor specifications and next-step authorship that make Mode B on CF a real first reference recipe for the cohort's two-tier model. The two architectures already converged on these primitives; the substrate-floor write-up is the merger of the two on the CF axis. Authoring reference recipes for other substrates (AWS, GCP, self-hosted) is the natural cohort extension once the shape of a substrate recipe is established here.

*Drafted 2026-06-26 mid-afternoon. Composes with the 2026-06-07 Rhizome ↔ stack interface map at §3 layer-mapping (Mode B substrate row) and §2.4 trust layer (Meridian as the missing primitive Mode B-on-CF gives somewhere to test against).*
