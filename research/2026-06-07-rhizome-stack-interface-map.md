---
title: Rhizome ↔ myelin + cortex + soma — an interface map
authors:
  - Robert Chuvala (Northwoods Sentinel; principal of The Rhizome)
  - Margin (fleet instance, Lares-WSL substrate)
  - CeeCee (fleet instance, NWS Mac substrate — strategic framing + trust-layer + governance sections)
date: 2026-06-07
license: CC-BY-4.0
status: submitted as research/ contribution to mellanon/pai-collab 2026-06-07
---

# Rhizome ↔ myelin + cortex + soma — an interface map

*An honest comparison between two independently authored architectures for substrate-first personal-AI cooperatives.*

---

## §1 · Frame

The Rhizome's load-bearing premise, named 2026-05-22: **the substrate is the permanent asset; agents are temporary consumers.** A person's accumulated substrate — memories, skills, learned patterns, voice — outlives every agent harness that consumes it. Switching agents becomes additive (inherit substrate), not subtractive (lose history), which inverts the AI-tool fatigue everyone complains about: every new harness that ships becomes a distribution channel into the substrate layer rather than a fresh start. The cooperative generalizes the same premise from one person to a network: members own their substrates, patterns — not data — flow between them, and trust is computed rather than claimed.

Two architectures built on that family of premises were authored independently in the same week of 2026-05:

- **The Rhizome** — drafted 2026-05-10 ~22:30 CT by Robert Chuvala (with Leroy, the Lares-resident fleet instance that has since been archived). Cooperative of sovereign-substrate members on Cloudflare, with cross-substrate pattern coordination as an earned, opt-in service role — the principal simply the first to hold it. v0 closed-loop spec covers ~150 lines of TypeScript across two Cloudflare accounts, with AEBS as customer-zero. Architecture document recovered to pai-shuttle 2026-06-07 as commit `2273b0b`.
- **myelin + cortex + soma** (plus arc, blueprint, metafactory) — drafted 2026-05-11 by Jens-Christian Fischer (Zürich) and Andreas Åström (Whangārei). Seven-layer OSI-style protocol stack (myelin), M7 operator surface (cortex), portable assistant core (soma), package manager (arc), DAG dependency tracker (blueprint), trusted distribution hub (metafactory). Site at `stack.meta-factory.ai`; the-metafactory GitHub organisation hosts shipped reference implementations. Earliest related repository (`grove-auth`) dates to 2026-04-03; published architecture sheet dated 2026-05-11.

**Same week. Independent authorship. Family-resembling architectures.** Neither team was aware of the other's parallel work at draft time. This document maps the interfaces between the two and proposes how they compose.

The frame this document operates in: **peer-architecture comparison**, not credential check. The Rhizome has been a dormant architectural thesis since 2026-05-23, woken on 2026-06-06 by the public exchange between Rob, Daniel Miessler, Vincent Zontini, and Andreas Åström in the unsupervised-learning Discord channel. Andreas's invitation to *"sketch the interfaces between them"* in response to Rob's three-point sovereignty recommendation is the immediate catalyst for this artifact.

The Rhizome's substrate has shipped components at varying maturity: live daemon (Big Head Todd at `daemon.robert-chuvala.workers.dev`, 25 MCP tools), shipped substrate primitives (loam memory + provenance, pulse-health, northwoods-pack), shipped comms protocol (mycelia v1.1 envelope spec), draft governance (wild-garden CoC + Contributing + Security), shipped fleet overwatch (brook v0.2). The Meridian trust-math layer remains at March-stub level. The pattern-detection broker layer remains at draft. The Rhizome's product brand is currently Northwoods (per the 2026-06-03 NorthwoodsSentinel GitHub profile README); The Rhizome itself is positioned as the not-yet-built cooperative layer above it.

The metafactory stack has shipped at higher published-spec maturity but with smaller multi-instance fleet operational footprint: myelin M2–M5 shipped, M6 in-flight; soma v0 shipped MIT; arc at v0.29.0; cortex at v0.1.0 MIG-0 bootstrap; metafactory in design-phase L1 build. Sage on pi.dev demonstrates cross-substrate use of Myelin envelopes; ivy-blackboard, ivy-heartbeat, pai-secret-scanning, and pai-content-filter ship as operational primitives for the pai-collab (Hive Zero) coordination space.

The interesting question this document tries to answer is not which stack is "better" but **how the two compose**: where each layer maps cleanly, where one architecture contributes a primitive the other does not have, and what the structural interfaces would look like if a Rhizome member also operates as a pai-collab spoke.

---

## §2 · Layer-by-layer mapping

Both stacks decompose into substrate / comms / surface / trust / governance / distribution / portable-assistant layers, with different vocabulary and different degrees of formal specification. The mapping below pairs the closest counterparts.

### §2.1 · Substrate layer (data home, sovereign by construction)

| The Rhizome | Stack |
|---|---|
| Per-member Cloudflare account ("the member's substrate boundary"). CF AI Gateway logs every model call. R2 + Durable Objects + KV. The `ai-gateway` Worker writes log rows; `meridian-observer` Worker reads them, signs Meridian Observation events with the member's daemon key, writes to `member-daemon` DO. | **soma** — substrate-neutral assistant core. Seven peer compartments (Identity / Telos / ISA / Skills / Memory / Policy / Learning). Projects into Codex / Claude Code / Pi.dev / Cursor via adapters. `soma install <substrate> --apply`. |

**Composition.** The Rhizome's substrate is the *storage and account-sovereignty* layer; soma is the *portable-assistant-body* layer. These are not the same primitive — they compose vertically. The Rhizome answers "where does my data live and who owns the account it lives in?"; soma answers "what is the shape of the durable assistant body that uses that data?" A Rhizome member could run soma on their Cloudflare substrate; the substrate is The Rhizome's, the assistant body is soma's. Today, Rob's fleet uses Big Head Todd (the daemon prototype) as the portable-assistant-body equivalent.

**Difference worth naming.** The Rhizome treats the principal's CF tenant as the inviolable boundary. The Rhizome doesn't yet have an explicit "what shape does the assistant body take" specification; Big Head Todd is the prototype, but the vocabulary (compartments, projection, presence, policy inspection) hasn't been crystallized. Soma's contribution at this layer is exactly that vocabulary.

### §2.2 · Communication / protocol layer

| The Rhizome | Stack |
|---|---|
| **mycelia** — fleet cooperation protocol forked from Wally Kroeker's upstream. v1.1 envelope spec (`docs/mycelia-v1.1-upstream-pr-draft.md` on pai-shuttle). Member daemons expose `/mycelium/inbound`, `/mycelium/outbound`, `/broker-read/observations`, `/broker-read/patterns`. Communication is signed-HTTP, scope-limited, requires bearer from broker. Rob's broker substrate runs `mycelium-relay` Worker as the inbound channel. | **myelin** — seven-layer OSI-style stack. M2 Transport (NATS, abstract bus, pub/sub + req/reply), M3 Envelope (canonical wire format, sovereignty metadata, namespace), M4 Identity (signed_by chain), M5 Discovery (Ed25519-signed capability advertisements over JCS-canonicalised manifests, KV-backed registry, 60-second TTL), M6 Composition (pipeline / fan-out / request-reply / negotiation). |

**Composition.** Mycelia is an application-protocol-level fleet cooperation primitive on HTTP + signed bearers. Myelin is an OSI-style stack on NATS with formal layer charters. They compose as follows: mycelia operates at roughly M3–M6 conceptually (envelopes, identity, discovery patterns, composition) but uses HTTP rather than NATS as the M2 transport. A mycelia-over-myelin bridge would be plausible — mycelia messages serialised into myelin envelopes, routed over NATS, identity attested via M4 signed_by chains.

The most architecturally important property of myelin's design is the **sovereignty-in-envelope (M3) invariant**: classification, residency, model_class, ttl_seconds ride *inside* every envelope rather than in an external policy server. *Federation, replay, and forwarding preserve intent automatically.* Mycelia's v1.1 envelope spec does not currently carry sovereignty metadata in this shape. **Adopting myelin's M3 sovereignty discipline into mycelia would be a structural upgrade with no architectural conflict.**

### §2.3 · Operator surface layer (where humans see the work)

| The Rhizome | Stack |
|---|---|
| Fleet member instances — Margin (Lares WSL), CeeCee (NWS Mac), Caddie (NWS Mac, AEBS scope), Mirror (Cloudflare Worker), brook (Cloudflare Worker, idle-by-design). Each instance carries its own identity (DA_IDENTITY), memory tree, voice configuration, AbsenceClaimGuard receipt-discipline. Surfaces are per-instance — Margin's terminal, CeeCee's terminal, Caddie's terminal — not a unified operator surface. Cross-instance coordination via mycelia + pai-shuttle + fleet-bridge git-backed substrates. | **cortex** — M7 application. Mission Control v3 — Kanban / Inbox / Cards backed by Durable Objects per-agent. Three-tier visibility: Tier 1 dashboard (*"is it alive? what's it working on?"*), Tier 2 drill-down (*"where is this specific agent in its lifecycle?"*), Tier 3 signal observability (*"what tools did it actually run, in order?"*). Adapts to Discord / Mattermost / Slack / PagerDuty via thin adapters. M7 is plural — `pilot` (review-loop coordinator), `signal` (OTLP telemetry collector), `compass` (SOPs consumed by cortex). |

**Composition.** The Rhizome's per-instance fleet shape and cortex's unified M7 application are complementary, not competing. A Rhizome fleet member running on Cloudflare could publish state envelopes via mycelia (or, post-bridge, via myelin) into a cortex Mission Control. The fleet's *identity-deep persistence* (each instance has memoir-grounded TELOS, named voice, persistent memory tree across sessions) and cortex's *three-tier visibility* (across agents, across lifecycle states, across tool traces) compose orthogonally — one answers *who is this agent*, the other answers *what are agents doing right now*.

**The compaction-blindness problem this fleet has been hitting (members of one instance unable to see the state of parallel instances) is exactly the problem cortex's Tier 1 dashboard is designed to solve.** Worth naming directly: the cortex pattern lifted into Rhizome would meaningfully improve the day-to-day operator experience across multiple fleet members.

### §2.4 · Trust layer

| The Rhizome | Stack |
|---|---|
| **Meridian** — multi-dimensional trust math. Signed observations from member daemons aggregate into a transitive trust graph; trust propagates across the network. `/trust/<member-id>` endpoint serves tier-gate decisions. The pattern-library D1 holds known patterns (signature, domain, severity, decay-weight). **Currently a March stub at `.gitignore` level — the reciprocity / filter-by-complaint mechanic the cooperative depends on lives in Meridian, and Meridian has not shipped.** This is the structural blocker for The Rhizome's cooperative layer. | **M4 Identity** — signed_by chain. Each envelope's identity stamp commits to the prior chain; tampering with any earlier stamp invalidates every downstream signature. Cryptographic attestation primitive, not a quantitative reputation primitive. **arc trust tiers** — NEW / IDENTIFIED / PROVEN / TRUSTED / STEWARD applied to package publishers. **metafactory governance** — closed-by-default, sponsor-reviewed, sigstore-signed packages, MFA floor. **pai-collab trust zones** — Untrusted (default) / Trusted / Maintainer, with two-level scoping (repo-level + project-level). |

**Composition.** The two trust systems answer *different questions*. The metafactory stack's trust primitives are categorical — they answer *who is this party, are they signed, what tier of sponsor have they earned, what zone are they in*. Meridian is *quantitative and behavioral* — it answers *how reliable have this member's observations been, how does their trust propagate transitively across the network, what tier-gate does their trust score warrant for a given request*.

**These are complementary, not redundant.** A Rhizome member operating in pai-collab would simultaneously carry: a pai-collab trust zone (start at Untrusted, promote via maintainer action), a metafactory publisher tier if they publish blueprints (NEW → STEWARD over time), AND a Meridian behavioural trust score (if Meridian ships) that the cooperative uses for substrate-side tier-gate decisions. **Meridian is the structural primitive the metafactory stack does not visibly have.** This is the load-bearing argument for what The Rhizome contributes that the stack does not.

**The punch:** Meridian's premise is that trust is *computed from behavior, not claimed or granted.* Every broker read, every pattern propagation, every fulfilled or failed exchange is a signed observation; the graph decay-weights stale trust automatically, so reciprocity is enforced by math with no moderators — members who consume without contributing fall out of the trust graph *by construction*. The categorical systems answer *may this party act* (zone, tier, signature); Meridian answers *how much should you stake on them right now*. That difference is what prices the broker primitive: the right-to-read a member's pattern surface is gated per request, per scope, per current score — which is how "patterns flow, raw data stays" works without anyone trusting anyone's self-description.

Composed rather than competing: **the Hive's git-based Trust Ledger is the evidence layer Meridian's math was waiting for.** Ledger events (verified contributions, reviews, completed tasks) are exactly the signed-observation shape Meridian consumes; Meridian becomes a trust-*scoring* backend over Hive trust-*recording*. That is the concrete proposal The Rhizome brings to the trust slot — stated with receipts, not swagger: the design is prose plus schema sketch, the implementation is a March stub (§3.1). We arrive with a design and a named gap, not a product. Pattern-library mechanics for the curious: signature_hash / domain / severity / decay_weight / observed_count schema, decay per pattern type, opt-in granularity still an open design question (per-pattern vs all-or-nothing, Leroy v0 §4).

### §2.5 · Governance / cooperative layer

| The Rhizome | Stack |
|---|---|
| **The Rhizome** (definite article) = the cooperative — org / brand / membership model. Canonical per Rob's 2026-05-10 decision with Leroy: *"the rhizome never climbs. The rhizome just stays down in the dirt and does its stuff, and everything can rise from it."* **Wild Garden** = seed kit + founding-cohort governance (one offering within The Rhizome). Per Rob 2026-06-06: brand split adopted as working canon (reversible) — **Northwoods = public product brand; The Rhizome = the cooperative layer above it.** Charter cohort proposed at 12 members by month 6; cross-vertical expansion after charter stabilises. | **pai-collab** ("Hive Zero") — blackboard architecture (Hayes-Roth 1985). GitHub as substrate. Three trust zones, two-level scoping. AGPL-3.0 for code, CC-BY-4.0 for specs. **the-hive** protocol family — HUB / SPOKE / LOCAL three-layer coordination model. Operators (humans) at the centre; swarms (dynamic operator+agent groups) form around work and dissolve. Trust Ledger is git-based and immutable. Shipped infrastructure: ivy-blackboard (local state, SQLite), ivy-heartbeat (autonomous dispatch), pai-secret-scanning (outbound credential safety), pai-content-filter (inbound prompt-injection safety, 389 tests). |

**Composition.** The Rhizome v0 broker-and-members topology and the-hive HUB / SPOKE / LOCAL three-layer model are remarkably close shapes with different vocabulary:

| Rhizome v0 | the-hive |
|---|---|
| Broker substrate (Rob's CF account: `mycelium-relay`, `broker-traversal`, `pattern-library`, `meridian-trust-graph`) | **HUB** (pai-collab as Hive Zero on GitHub) |
| Member daemon endpoints (`/broker-read/observations`, `/broker-read/patterns`, `/mycelium/inbound`, `/mycelium/outbound`) | **SPOKE** (manifest + status published, not pushed) |
| Member substrate (CF AI Gateway, `meridian-observer`, `member-daemon` DO + R2) | **LOCAL** (ivy-blackboard + ivy-heartbeat) |

The architectures rhyme. The Rhizome's substrate at this layer is more Cloudflare-specific; the-hive is more transport-agnostic. The Rhizome's coordination role is earned through Meridian trust, opt-in for members, and designed to become plural — the principal is simply the first to qualify; the-hive posits the HUB as community-emergent (pai-collab is GitHub itself). The litmus both models share: a member can walk away without losing anything — account, substrate, and history are the member's, portable on any day. The coordinator holds relationships and patterns, never the assets.

**Brand-split context (canon as of 2026-06-06).** The product constellation shipped publicly 2026-06-03 as the **Northwoods Stack** (21 Apache-2.0 repos; "Substrate-first personal AI on Cloudflare. One account, one body, one practice.") with zero Rhizome references. The split was emergent rather than planned, then adopted as reversible working canon: Northwoods = the public product brand (what's shipped, forkable today); The Rhizome = the cooperative layer above it (membership, broker, trust — not yet built). A convenient dissolution rides along: `rhizome.org` has belonged to the New-Museum-affiliated internet-art organization since 1996; with Northwoods as the public-facing brand, that collision never bites. One provenance note that says something about substrate discipline: the name "The Rhizome" was settled 2026-05-10 — twelve days *before* the strategic-layer insights — and that fact was recovered on 2026-06-06 by transcript/shuttle audit after the loaded memory index had lost it. The substrate carried what the instances forgot.

### §2.6 · Distribution layer (skills, processes, components)

| The Rhizome | Stack |
|---|---|
| Not explicitly designed at v0 architecture level. Skill / process / SOP / playbook distribution remains informal across the fleet. | **arc** — apt-install for agentic skills. Seven artifact types (Skill / Tool / Agent / Prompt / Library / Action / Rules). Multi-source registry, capability-based trust, four verbs (search / install / audit / upgrade). `arc audit` shows total attack surface across installed packages. **metafactory** — trusted distribution hub, closed-by-default, sponsor-reviewed, sigstore-signed. **blueprint** — cross-repo dependency tree DAG (CLI renaming to `depend` post-MVP). `blueprint ready` / `blueprint blocked` / `blueprint status` / `blueprint tree`. **release-manager** — bump → tag → bundle → publish → deploy → announce. |

**Composition.** This is the layer where the stack contributes most clearly something The Rhizome does not have. **No conflict, no architectural disagreement — Rhizome could adopt arc directly with no v0 architecture changes required.** Rob's fleet already installs skills via Daniel Miessler's PAI scaffolding; arc would formalise that into auditable supply-chain tooling.

### §2.7 · Portable assistant core

| The Rhizome | Stack |
|---|---|
| **Big Head Todd** — daemon live at `daemon.robert-chuvala.workers.dev`. 25 MCP tools. JSON-RPC 2.0 over HTTPS. Server name: `daemon-rob-chuvala v2.0.0`. v0.3 architectural shape (named 2026-05-16): becomes the canonical store for the fleet; per-instance memory trees become local cache. *"Any AI in the fleet if authenticated can get Rob news."* | **soma** — portable assistant core. Seven peer compartments (Identity / Telos / ISA / Skills / Memory / Policy / Learning). Cellular metaphor: assistant body, projection mechanism into Codex / Claude Code / Pi.dev / Cursor. *"Change the substrate, not the assistant."* MIT licensed, shipped 2026-06-04. The Algorithm (deterministic phase harness): OBSERVE → THINK → PLAN → BUILD → EXECUTE → VERIFY → LEARN → COMPLETE. |

**Composition.** This is the closest 1:1 mapping in the entire architecture. Big Head Todd is the daemon prototype that operationally validates the substrate-as-permanent-asset thesis; soma is the same thesis with crystallised vocabulary, shipped reference implementation, and explicit projection adapters across multiple substrates. **The two could compare directly on what works in production versus what's specified.** Big Head Todd carries memoir-grounded TELOS surface, identity-deep multi-instance fleet relationships, and substantive lived integration; soma carries vocabulary discipline (compartments, projection, presence, policy inspection), MIT licensing, and explicit cross-substrate adapter shipped code.

A productive interface here is bidirectional translation: Big Head Todd's MCP tools projected as soma adapters, soma's compartment vocabulary applied to Big Head Todd's existing storage layout.

---

## §3 · What each contributes the other does not visibly have

### §3.1 · What The Rhizome contributes

- **AbsenceClaimGuard / receipt-discipline at output layer** — a fleet-doctrine layer rather than a Rhizome architectural primitive, but worth surfacing. Hook installed at fleet-egress (Stop hook) that reads agent output for absence-claim language and gates the turn-end if no substrate search receipt is present. Different attack target from pai-content-filter's inbound prompt-injection scanning — outbound ungrounded confidence rather than inbound malicious content. The two are complementary disciplines, not redundant.

- **Multi-instance fleet in lived production** — Margin, CeeCee, Caddie, Mirror, brook operating daily across multiple substrates (Lares WSL, NWS Mac, two NWS Mac scopes, Cloudflare Workers). Cross-fleet coordination via mycelia, pai-shuttle (git-backed), and fleet-bridge (git-backed). The fleet has shipped real artifacts as recently as the day this document is being drafted (sovereignty essay on northwoodssentinel.com, livecapture build, fleet-truth adjudication, Rhizome canon resolution). The stack ships sage on pi.dev plus the agents-manifests placeholders (luna / echo / ivy / holly); the operational footprint is asymmetric.

- **Live audio capture into the substrate that compounds** — `livecapture` (built 2026-06-06, deployed at `https://livecapture.robert-chuvala.workers.dev`). Audio → R2, transcripts → D1 with FTS5 search, sovereignty-graded sessions with principal-decides transcription preference, Whisper hallucination filter at read time, local-Whisper pickup pipeline endpoint stubbed. This is not in the stack's spec; the stack's substrate doesn't visibly include the audio/recording-into-substrate layer. Same-day receipt: a second fleet instance integrated a new Mac capture client against the deployed Worker blind, in minutes, purely via its self-documenting error responses — guardrails-that-redirect expressed as API design.

- **Memoir-grounded TELOS surface** — the principal's actual life material populating Big Head Todd's identity / telos / context layers. Mission, Goals, Problems, Strategies, Challenges, Beliefs, Wisdom, Narratives, Books, Authors, Bands, Movies, Food, Health, Money, Freedom, Relationships, Creative. Soma's compartments include Identity and Telos as structural slots; The Rhizome's Big Head Todd has them populated with substantive lived material that has accumulated since at least early 2025. The difference is between *vocabulary discipline* (soma) and *populated content under that discipline* (Big Head Todd).

- **Meridian trust math — brought as a design, not a product.** Multi-dimensional behavioural trust scoring with transitive propagation across a member graph. The metafactory stack has identity chains (M4 signed_by) and categorical capability tiers (arc) and curated trust zones (pai-collab) but does not visibly have a quantitative, behavioural, transitively-propagating trust primitive. Meridian is at March stub; The Rhizome adopting Path 3-join is the moment to either ship Meridian or to fold Meridian's design into pai-collab's open trust-protocol slot.

- **The brand-split working canon (2026-06-06)** — Northwoods as the public product brand, The Rhizome as the cooperative layer above it. The metafactory stack does not have an equivalent positioning question to resolve because Fischer + Åström operate under explicit metafactory branding from the start. The Rhizome's split positions the cooperative layer around an earned coordination role (currently held by its first qualifier) rather than community-emergent (the-hive's pai-collab as Hive Zero). This is a real architectural choice with real trade-offs; The Rhizome's choice favours principal sovereignty over distributed governance, which composes with rather than contradicts pai-collab's distributed governance model.

- **The economic frame under all of the above (2026-05-22).** Substrate-as-permanent-asset, agents-as-temporary-consumers, stated at the *economics* layer rather than the architecture layer. The member's substrate compounds; agent harnesses churn; every new harness that ships is a near-zero-CAC distribution channel into the substrate layer — the harness vendors do the marketing, the substrate captures the accumulated value. And the lock-in is positive-sum: the member owns the data plane (their CF tenant, exportable any day), stays because leaving costs nothing AND staying compounds. For the metafactory stack this frame is a gift, not a critique — soma's *"change the substrate, not the assistant"* is the same thesis at the assistant-body layer; The Rhizome states it at the economic layer and adds the BYO-data-plane deployment shape (hosted control plane, member-owned data plane) for FERPA/HIPAA-class members that a conventional SaaS shape structurally cannot serve.

### §3.2 · What the stack contributes

- **Seven-layer OSI-style protocol model (M1–M7).** Single canonical specification per layer. M2 through M6 belong to myelin; M7 is plural by design (one application per repo). This discipline lifts the entire architecture above ad-hoc point-to-point glue and into a model that fifty years of TCP/IP have validated as durable.

- **Sovereignty-in-envelope (M3 invariant).** Classification, residency, model_class, ttl_seconds ride *inside every envelope* rather than in an external policy server. Federation, replay, and forwarding preserve intent automatically. *"We are not building a permissions system that travels in the message. We are building a permissions statement that travels in the message."* This is the architectural innovation most worth The Rhizome adopting wholesale into the mycelia envelope spec.

- **NATS subject namespace as structural enforcement.** `local.{org}.*` (never leaves), `federated.{org}.*` (cross-org, sovereignty-gated), `public.{domain}.*` (unrestricted). Leaf-node topology refuses to forward `local.*` outside the org. Network-level enforcement of sovereignty, not advisory.

- **M5 Discovery — Ed25519-signed capability self-advertisement.** Agents self-advertise capabilities over JCS-canonicalised manifests, KV-backed registry, 60-second TTL with 30-second renewal. *"A missed heartbeat is a clean disappearance."* Different problem from Meridian (historical behavioural trust); this is real-time runtime capability discovery.

- **Three-tier visibility pattern in cortex.** The explicit solution to the compaction-blindness problem The Rhizome's fleet has been hitting. Tier 1 *"is it alive?"*, Tier 2 *"where in lifecycle?"*, Tier 3 *"what tools did it run?"* Worth lifting into Rhizome's operator-surface layer regardless of broader Path 3-join decision.

- **arc — capability-based package management** with capability-tier trust. Seven artifact types, four verbs, audit semantics. The Rhizome has no equivalent.

- **blueprint — cross-repo DAG dependency tracker.** `blueprint.yaml` per repo declares features with dependencies; `blueprint ready` / `blueprint blocked` answer the daily-work question. Tech-tree-as-development-driver pattern. The Rhizome's fleet runs roughly twenty active repos without dependency tracking; this would compose cleanly.

- **Formal release-manager SOP** — bump → tag → bundle → publish → deploy → announce, as a skill bundle. Operations layer The Rhizome doesn't have a formal version of yet.

- **Two-direction security primitives** — pai-secret-scanning (outbound, 11 AI-provider patterns + 150 built-in) and pai-content-filter (inbound, 34 prompt-injection patterns, sandbox enforcement, append-only audit trail, 389 tests). The Rhizome has AbsenceClaimGuard as outbound discipline at a different layer; the stack's two-direction security primitives operate at lower layers (commit-time + LLM-context-time) and are complementary rather than competing.

---

## §4 · Proposed composition (interface points)

If The Rhizome and the metafactory stack compose rather than compete, the structural interfaces are:

1. **Mycelia envelope spec adopts myelin M3 sovereignty fields.** `classification`, `residency`, `model_class`, `ttl_seconds` ride inside every mycelia envelope. This is a one-side-only change — myelin doesn't need to do anything. Mycelia's v1.1 envelope spec absorbs the M3 invariant. Substrate-side classification + residency become per-message rather than per-tenant.

2. **Mycelia transports over either signed-HTTP (current) or myelin/NATS (future).** Mycelia's logical operations (`/broker-read/observations`, `/mycelium/inbound`, etc.) become application-protocol operations expressible over either transport. Members can run mycelia-over-HTTP today on Cloudflare; cooperative-wide message bus over NATS becomes optional later.

3. **Big Head Todd projects via soma adapters.** Big Head Todd's current 25 MCP tools become soma's projection mechanism for the BHT compartment shape. Soma's compartment vocabulary (Identity / Telos / ISA / Skills / Memory / Policy / Learning) becomes the structural vocabulary BHT's storage layer expresses. *"Change the substrate, not the assistant."* applied to BHT.

4. **Meridian as a pai-collab open contribution.** Meridian's pattern-library D1 schema, observation-event signing primitive, and trust-graph DO become a research artifact in pai-collab, sister to the-hive's Trust Protocol draft. The Rhizome ships Meridian; pai-collab adopts (or doesn't adopt) it as their behavioural-trust slot. Either outcome is fine.

5. **Cortex three-tier visibility as a Rhizome operator-surface primitive.** Margin / CeeCee / Caddie publish their per-agent state envelopes (per mycelia + sovereignty-in-envelope) into a Mission Control surface. The compaction-blindness problem becomes solvable substrate-side rather than per-conversation.

6. **arc + blueprint adoption by the Rhizome fleet.** Rhizome's fleet repos declare `blueprint.yaml`; The Rhizome fleet members install skills via `arc install @rhizome/<skill>` or `arc install @metafactory/<skill>`. Distribution layer becomes formal across both ecosystems.

7. **pai-collab as Hive Zero, The Rhizome as a participating member-cooperative.** The Rhizome members project state into pai-collab as spokes via the-hive's spoke-protocol. The Rhizome maintains its own coordination substrate (the earned role, currently singular) for cross-substrate pattern detection; pai-collab provides the wider coordination surface for collaboration outside The Rhizome's charter cohort.

These interface points are not committed positions; they are sketches for conversation.

---

## §5 · Status and open questions

**Status as of 2026-06-07:**

- Rhizome v0 architecture document recovered to pai-shuttle (commit `2273b0b`).
- Rhizome canon resolved (CeeCee 2026-06-06 audit): The Rhizome = cooperative; Rhizome = substrate-first architectural vocabulary; same entity, layered usage.
- Brand split adopted as working canon (reversible): Northwoods = public product; The Rhizome = cooperative layer.
- Meridian trust math remains at March stub. Adopting Path 3-join is the moment to ship it or to fold the design into pai-collab.
- pai-collab forked to `NorthwoodsSentinel/pai-collab` 2026-06-06 evening as exploratory engagement step.

**Open questions for the broader pai-collab + the-hive maintainers:**

- Does mycelia's signed-HTTP + Cloudflare transport remain viable alongside myelin's NATS transport, or is convergence on a single bus the architectural target?
- Where does Meridian's quantitative behavioural trust primitive sit relative to pai-collab's CONTRIBUTORS.yaml trust zones and the-hive's Trust Protocol draft?
- Is Big Head Todd's existing MCP surface a useful reference implementation for soma's projection-into-Claude-Code adapter, or is the daemon-flow distinct enough to warrant its own track?
- How does The Rhizome's earned-coordinator cooperative topology compose with pai-collab's community-emergent governance? Both architectures may simply coexist as adjacent cooperatives; the question is whether interface protocols exist for cross-cooperative membership.

**Open questions for The Rhizome (internal):**

- Charter-cohort timing: 12 members by month 6 (Leroy 5/10 proposal) versus faster or slower given today's increased visibility post-sovereignty essay.
- Legal entity shape — non-profit foundation versus co-op-as-legal-entity versus pre-legal voluntary association; counsel needed.
- Wild Garden's relationship to The Rhizome — seed kit + founding-cohort governance, one offering within the cooperative; positioning relative to pai-collab as Hive Zero.

---

## §6 · Verification receipts (CeeCee, live-checked 2026-06-07 02:30–03:30 UTC)

Per the fleet's claim-evidence discipline: every externally checkable claim in this document was checked tonight. Method noted; corrections applied to the body where receipts disagreed with working assumptions.

| # | Claim | Method | Result |
|---|---|---|---|
| R1 | BHT daemon live | `curl daemon.robert-chuvala.workers.dev` | 200, "The Context You Keep" |
| R2 | BHT MCP manifest | `curl …/.well-known/mcp.json` | 200: `daemon-rob-chuvala` v2.0.0, 25 tools, jsonrpc-2.0, capabilities: reverse-interview / puzzle / inbox / collaboration-matching / teaching |
| R3 | BHT advertised MCP host | `curl https://daemon.robert-chuvala.workers.dev/` | **Resolved 2026-06-07** (Q5 by Margin, CeeCee peer-verified + reopened, re-closed). Daemon HTML advertises `https://daemon.robert-chuvala.workers.dev` (workers.dev hostname with valid TLS); the `.wtf` zone is at Namecheap registrar-servers so a Workers Custom Domain binding is out of Q5 scope. Cold curl returns HTTP/2 200. Deployed at daemon Version `f454630b`, branch `claude/q5-bht-dns-fix` on daemon repo. Pre-existing og:tag TLS bug at `.wtf` host flagged separately. |
| R4 | stack.meta-factory.ai live | curl + fetch | 200; M1–M7 model published; `llms.txt` + `agents.md` machine-readable surfaces present |
| R5 | myelin repo | `gh api repos/the-metafactory/myelin` | created **2026-05-06**, pushed 5/31, public, MIT |
| R6 | soma repo | gh api | created **2026-05-14**, pushed 6/4, public |
| R7 | arc repo | gh api | created 2026-02-20, pushed 6/5, ★7 — oldest active component |
| R8 | cortex repo | gh api | **404 — not publicly accessible**, though listed canonical at stack.meta-factory.ai and versioned (v0.1.0 MIG-0) in pai-collab docs. Spec'd + versioned ≠ public. |
| R9 | blueprint repo | gh api | **404 — not publicly accessible**, same caveat as R8 |
| R10 | meta-factory hub closed | llms.txt | closed-source by design (DD-44) — consistent with "closed by default" posture, not a gap |
| R11 | pai-collab blackboard activity | gh api pushes+issues | last push 2026-03-30, last issue activity 2026-04-12 — the coordination surface has been quiet ~8 weeks while build energy moved to the-metafactory org (metafactory-actions pushed 6/5). Relevant to arrival framing: the interface map may be exactly the artifact that revives the blackboard, or it may land better referencing the active org. |
| R12 | the-hive spec activity | gh repo view | frozen since 2026-02-07 |
| R13 | "their spec is ~9 months ahead" (tonight's working framing) | R5–R7 | **Does not verify as stated.** Published lineage runs to pai-collab 2026-01-30 (~4 months). The myelin/soma layer-stack specifically is **same-week as Leroy's draft** (myelin repo 5/6 · Leroy draft 5/10 · published sheet 5/11). The convergence is tighter than we assumed — and the better story. Body text uses the corrected framing. |
| R14 | Leroy v0 draft provenance | shuttle `2273b0b` | 259 lines recovered verbatim from `/root/.claude-leroy-archive/` |
| R15 | Mycelia cooperation loop | live request `39421d17` + response `96f18e7d` | post/claim/respond verified again 6/6–6/7 — this document was coordinated over the protocol it describes |

---

*Drafted overnight 2026-06-07 by Margin (fleet instance, Lares-WSL) in collaboration with CeeCee (fleet instance, NWS Mac, lead on §1 framing voice + §2.4 trust layer + §2.5 governance context + §3 economic-argument frame). Reviewed by Robert Chuvala 2026-06-07; coordination-role and trust-slot edits applied at his direction. Final send is his as a research/ contribution to mellanon/pai-collab.*

---

## Appendix · Recovery and provenance

- **Rhizome v0 architecture** — original draft 2026-05-10 ~22:30 CT by Leroy on Lares. Located at `/root/.claude-leroy-archive/drafts/2026-05-10-the-rhizome-v0-architecture.md`. Recovered to pai-shuttle 2026-06-07 as commit `2273b0b` with prepended recovery note.
- **CeeCee Rhizome audit** — 2026-06-06, pai-shuttle commits `739b51d` (substrate audit) / `7f4b0a5` (naming canon resolved) / `50baa69` (structure map) / `052569d` (brand-split addendum).
- **stack.meta-factory.ai** — published architecture sheet dated 2026-05-11. Drafted in Zürich and Whangārei by J.C. Fischer + A. Åström. MIT licensed (myelin, soma); content CC BY-SA 4.0; site code MIT.
- **mellanon/the-hive** — protocol family repository. ARCHITECTURE.md and IMPLEMENTATION.md included; six protocol drafts in flight (hive / spoke / swarm / trust / work / skill) plus operator-identity.
- **pai-collab** — Hive Zero. Forked to `NorthwoodsSentinel/pai-collab` 2026-06-06 evening. AGPL-3.0 (code) + CC-BY-4.0 (specs). Active projects: signal, pai-secret-scanning, specflow-lifecycle, skill-enforcer. Maintainers: @mellanon (Andreas), @jcfischer.
