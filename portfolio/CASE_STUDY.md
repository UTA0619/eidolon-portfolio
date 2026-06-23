# Eidolon — Engineering & Product Case Study

> **One line:** A pre-launch, AI-native mobile product (Flutter, 13 features, 245 source files,
> 46 test files, 98 commits) built solo with the architecture, governance, and go-to-market
> rigor of a funded startup — and the strategic honesty to pivot its own thesis before a line
> of marketing was written.

**Status:** Pre-launch. Every revenue/retention figure in this document is a **target or model
output**, never claimed traction. The proof here is *capability under realistic constraint*, not
results that don't exist yet. That distinction is itself the point: the work demonstrates someone
who can tell the difference, which is the trait every reader below is actually screening for.

**Who this is for:**

| Reader | Skip to | The question they're really asking |
|---|---|---|
| Recruiter / Hiring Manager | §1, §8 | "Is this person's bar high, and is it real?" |
| Enterprise Architect / CTO | §3, §4, §6 | "Would I let them design a system my team depends on?" |
| AI Product Leader | §2, §5, §7 | "Do they ship AI that serves the user, not the demo?" |
| Founder / VC | §2, §7, §9 | "Do they see the market and own the hard trade-offs?" |

---

## 1. The problem worth solving

Personal AI is arriving for everyone. The losing assumption is that the *smartest model* wins —
but frontier models commoditize on a ~12-month cycle, and a chatbot has zero switching cost: your
relationship with it is reconstructable by any competitor from a single export.

**The durable question is not "which model is smartest" but "which AI knows you best and which one
you trust enough to act for you."** Both of those are *accumulated*, not bought — and accumulation
is the one thing a better-funded competitor cannot fast-forward.

The hard part is the cold start: people will not sit down and "train their AI." It feels like work,
and the AI-companion category carries stigma. You need a reason to show up daily, for years, that
doesn't *feel* like feeding a model.

**Eidolon's wedge:** a soul-bound RPG that is genuinely fun to open each morning — and whose real
job is to harvest, willingly and daily, the three signals a personal AI needs: **personality**
(Big Five), **episodic memory**, and **reality data** (steps / sleep / HRV / mood). The game is the
funnel. The Twin is the company.

> *Capability proven: problem framing — separating the commodity (model IQ) from the moat
> (accumulated, un-cloneable user context), and finding a wedge that solves the cold-start.*

---

## 2. Market & strategy — and a pivot I made against myself

The repo contains a real strategy document ([`docs/strategy/STRATEGY.md`](../docs/strategy/STRATEGY.md))
written *after* an unsentimental self-review killed the original framing. That review is the most
important artifact in this project, because it shows a founder who will cut his own favorite idea.

**The pivot:** "An AI RPG" → "A persistent personal AI (a *Twin*) that the game merely bootstraps."

| Framed as a game | Reframed as a Twin |
|---|---|
| Competes on dungeon/content volume | Competes on how well it knows you |
| Moat = content variety (LLM-cloneable) | Moat = your private personality + memory + reality graph (un-cloneable) |
| Monetize power (kills D3 retention) or cosmetics (low ARPU) | Monetize relationship / cognition / agency (high LTV, retention-safe) |
| Comp: mobile game studio (~3–5× revenue) | Comp: personal-AI / identity platform |

**Sequenced so no single bet sinks the company** — each act funds and de-risks the next:

- **Act 1 — Wedge (now → 12mo):** the game harvests the bond. North-star = *Twin Depth × Retained
  Users*, deliberately **not MAU** (vanity metric for this thesis).
- **Act 2 — Bridge (6–18mo):** the Twin reflects the user's *real* life back — journaling, mood,
  a weekly "what I noticed about you." *This act is already shipping* (see `weekly_reflection`,
  `morning_report` features — §5).
- **Act 3 — OS (18mo+):** the Twin *acts*, agentically and with consent, bounded by a written
  governance constitution (§6).

**Beachhead: Japan first** — JP-native i18n and team, existing cultural traction for AI companions,
privacy expectations that *reward* the governance-first posture, and a less brutal competitive field
than the US Replika / Character.AI fight. Expand to EN once Act-1 retention is proven, not before.

> *Capability proven: strategy & founder judgment — TAM-shaping (commodity vs. moat), a
> de-risked roadmap, a defensible beachhead, and the rarer skill of pivoting away from your own
> sunk cost on evidence.*

---

## 3. Architecture at a glance

Full document: [`docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md). The shape:

```
Client (Flutter — iOS / Android / Web PWA)   Flame · Riverpod · GoRouter
        │ HTTPS / WSS
   ┌────┴─────┐
Firebase     Supabase (Postgres + pgvector + Edge Functions/Deno + Realtime)
(Auth,        │
 Firestore,   └── pgvector = the "you-graph": embeddings of memory + personality
 Storage,
 Crashlytics)
        │
Cloudflare Workers  ← LLM proxy: auth, rate-limit, cost guard
        │
   AI Layer:  Claude Sonnet 4.5 (story/critical) · Claude Haiku 4.5 (events/dialogue/classify)
              · GPT-4o-mini (fallback) · Replicate SDXL (assets) · ElevenLabs (voice)
```

**Three decisions a reviewing architect should notice:**

1. **No AI call ever leaves the client directly.** Every model call routes through a Cloudflare
   Worker that owns auth, rate-limiting, and a hard cost guard. The client cannot leak a key or
   run up a bill. (CLAUDE.md policy, enforced in code.)
2. **Right model for the job, with a fallback chain.** Haiku for low-latency classification
   (tags, emotion), Sonnet for generation/reasoning, GPT-4o-mini as fallback, and a **local
   template as the final fallback** — so the product degrades gracefully instead of erroring when
   every provider is down. (See the "fallback honesty invariant" test, §4.)
3. **pgvector is the moat, made concrete.** The "you-graph" isn't a slogan — it's personality
   vectors × episodic memory embeddings × a reality timeline, in a queryable store. The
   architecture is literally shaped around the thing that compounds.

> *Capability proven: enterprise architecture — separation of concerns, a security/cost boundary
> at the right layer, graceful degradation, and infrastructure chosen to serve the business moat.*

---

## 4. Engineering rigor (the part that's verifiable)

Anyone can claim quality. Here it's checkable in the tree:

| Signal | Evidence in repo |
|---|---|
| **Clean Architecture, consistently** | Every one of 13 features splits `data / domain / presentation` ([CLAUDE.md](../CLAUDE.md) folder contract) |
| **Test discipline** | 46 `*_test.dart` files against 245 source files; quality gate set at ≥80% coverage |
| **Honest failure modes tested** | `b655b25 test(overnight): … fallback honesty invariant` — a test that asserts the AI *admits* when it's running on a fallback, instead of pretending |
| **Typed error handling** | `Result<T,E>` mandated in the domain layer — no naked exceptions across boundaries |
| **i18n from day one** | All strings in ARB files; the AI even *mirrors the user's language* (`3847e80`) rather than defaulting |
| **Decision records** | 5 ADRs + index in [`docs/DECISIONS/`](../docs/DECISIONS/README.md) — monorepo, backend split, AI gateway/routing, governance-as-code, Clean Architecture |
| **Real growth instrumentation** | `62400cd feat(analytics): instrument the 3 growth metrics — retention, morning habit, viral K` — events like `morning_report_viewed`, `morning_share_completed`, `overnight_dispatch_tapped` |

The commit history reads like a product team's, not a tutorial's: features land, then get
*refactored for error visibility and de-duplication* (`37c9505`), then *tested*. That sequence —
ship, harden, cover — is the tell of someone who's maintained software, not just written it.

> *Capability proven: execution quality — the discipline that separates a demo from a system a
> team can build on. The "fallback honesty invariant" in particular signals AI-product maturity:
> truthfulness treated as a testable property.*

---

## 5. AI product sense — three features that show the thesis working

The strategy (§2) isn't aspirational; Act-2 mechanics are already in the tree:

- **`morning_report`** — the Twin reports what it did "overnight" and the user can share it. This is
  the *retention + viral* engine: a daily reason to open the app, instrumented with
  `morning_report_viewed` → `morning_share_initiated` → `morning_share_completed` (a measurable
  viral-K funnel).
- **`weekly_reflection`** — "what I noticed about you." The exact Act-2 bridge mechanic: the product
  stops being a game and becomes a mirror. This is where Twin Depth becomes *felt* by the user.
- **`away_report` / overnight dispatch** — "while you were away" idle rewards and an on-demand
  "send your Twin adventuring" loop — async engagement that accrues memory without demanding
  presence.

Each is grounded in the user's *actual* accumulated data (personality + recent memories injected
into every Eidolon prompt — CLAUDE.md LLM policy), not generic LLM output. That grounding is the
difference between "an app with AI in it" and "an AI product."

> *Capability proven: AI product leadership — features that each map to a named metric in the
> North-star tree, AI grounded in real user state, and a viral loop that's instrumented, not hoped for.*

---

## 6. Governance as architecture (the unusual part)

Most consumer apps treat trust as a marketing line. Eidolon treats it as **a typed state machine
under test.** `docs/governance/` contains a written constitution, an ontology, and ~960 lines of
TypeScript implementing the rules:

- `state_schema.ts` (436 LOC) — the formal state of the system
- `transitions.ts` (256 LOC) — legal moves between states
- `soundness_check.ts` (117 LOC) — invariants that must always hold
- `current_state.test.ts` — the soundness checks, executed in CI

The doctrines this enforces are also the *positioning*: **you own your Twin, it can't be sold, real
deletion means real deletion, and no power-for-pay.** In an era of extractive AI — and especially in
the JP/EU beachhead — "the AI that is *constitutionally* yours" is a moat, not overhead. Encoding
ethics as runtime invariants is exactly the kind of thinking enterprises now demand of AI systems
(EU AI Act, model governance), shown here on a consumer product before it was fashionable.

> *Capability proven: systems thinking + AI governance — translating values into enforceable,
> testable invariants. This reads as both EA-grade and ahead of the regulatory curve.*

---

## 7. Business model & unit economics (the model, stated as a model)

Monetize the *relationship*, never *power* (power-for-pay kills D3 retention and violates the
constitution):

- **Free** — the game + a basic Twin (cheaper model, bounded memory) → top-of-funnel + data harvest.
- **Paid tiers** — deeper memory, a smarter Twin, reality-data features, and (Act 3) agentic
  actions. Revenue is designed to **follow Twin Depth, not MAU** — willingness-to-pay rises with
  switching cost, which rises every day the Twin remembers more.

The repo's stated targets — $40K MRR at 25K MAU, D1 ≥55% / D7 ≥30% / D30 ≥18% — are **planning
inputs**, the numbers a seed deck would defend, not metrics being claimed. The credible part is that
the *cost guard* (§3) and the *tiered model routing* (Haiku vs. Sonnet) mean COGS-per-user is a
designed-in variable, not an afterthought — the difference between an AI product with a path to
margin and one that's a venture-subsidized demo.

> *Capability proven: founder/VC literacy — a retention-safe revenue model tied to the moat, and
> COGS treated as an architectural decision (model routing) rather than a hope.*

---

## 8. What I'd tell a hiring manager in 60 seconds

- **Bar:** Clean Architecture across 13 features, typed errors, 46 test files, ADRs, CI-checked
  governance invariants — solo, pre-launch. The bar is high *and* it's real (clone the repo).
- **Range:** strategy doc, architecture doc, governance engine, AI integration, growth analytics,
  i18n — the full stack from VC narrative down to a `Result<T,E>` return type.
- **Judgment:** pivoted the entire thesis against personal sunk cost; refuses vanity metrics;
  tests the AI for *honesty*; designs cost in. These are seniority signals, not output volume.
- **Honesty:** this document never claims traction it doesn't have. That's deliberate.

---

## 9. Honest gaps (the most credible section)

A portfolio that claims no weaknesses is a red flag. Real ones, with the plan:

| Gap | Reality | Plan |
|---|---|---|
| **No live users yet** | All metrics are targets | Act-1 launch in JP beachhead; the analytics to read retention are already wired |
| ~~Only 1 substantive ADR~~ **(closed 2026-06-23)** | `docs/DECISIONS/` now has 5 ADRs + an index | Backfilled ADR-002 (Firebase+Supabase split), 003 (AI gateway / model routing), 004 (governance-as-code), 005 (Clean Architecture). See [`docs/DECISIONS/`](../docs/DECISIONS/README.md) |
| **Solo project** | No team-scale collaboration signal yet | The issue/PR templates and branch workflow exist to *demonstrate* the practices; real proof needs collaborators |
| **Twin Depth is defined, not yet measured** | The North-star is specified but pre-data | Instrument the composite (memory richness × reality-days × personality confidence) as Act-1 ships |

> *Capability proven: the single most senior trait in this document — knowing exactly where the
> work is thin, and saying so before anyone asks.*

---

*This case study is anchored entirely in the real Eidolon repository. Every file, commit hash, and
metric reference above is verifiable in the tree. Where a number is a target rather than a result,
it says so.*
