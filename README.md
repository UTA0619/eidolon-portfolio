<div align="center">

# Eidolon — Capability Proof System

**An AI-native mobile product, built solo with the rigor of a funded startup —
and the documentation to prove it.**

`Status: Pre-launch` · `Docs & evidence layer` · `Strategy · Architecture · ADRs · Governance-as-code`

[Executive Summary](./portfolio/EXECUTIVE_SUMMARY.md) ·
[Case Study](./portfolio/CASE_STUDY.md) ·
[Architecture](./portfolio/ARCHITECTURE_DEEP_DIVE.md) ·
[Recruiter Guide](./portfolio/recruiter-guide.md) ·
[Hiring Manager Guide](./portfolio/hiring-manager-guide.md)

</div>

---

> ### 👀 Reviewing this as a recruiter, architect, AI product leader, or investor?
> This repository is not a tutorial project. It is the **evidence layer** of a real pre-launch
> startup — strategy, cloud architecture, documented decisions, AI engineering, and a product case
> study — packaged to prove a specific set of capabilities. **Everything is grounded in verifiable
> artifacts; no traction is claimed.**
>
> **→ Start at the [Portfolio / Proof Layer](./portfolio/README.md).**

## What is Eidolon?

An AI-native mobile RPG that is secretly a **personal AI ("Twin")**. The game is a *wedge*: it gets
people to feed an AI their personality, episodic memory, and real-life data — willingly, daily — so
the Twin comes to know them better than any competitor's model could reconstruct.

The thesis: frontier models commoditize; a chatbot has zero switching cost. The durable moat is not
model IQ but **accumulated, un-cloneable user context** — and a daily habit is the only way to
accumulate it. Eidolon is the habit engine for that moat. (Full reasoning:
[Case Study §1–2](./portfolio/CASE_STUDY.md).)

## Read by who you are

| You are a… | Start here | You'll verify in ~5 min |
|---|---|---|
| **Recruiter / Sourcer** | [Recruiter Guide](./portfolio/recruiter-guide.md) | Role fit, green flags, a copy-paste pitch |
| **Hiring Manager / Eng Lead** | [Hiring Manager Guide](./portfolio/hiring-manager-guide.md) | What each artifact proves about seniority + interview Qs |
| **Enterprise Architect / CTO** | [Architecture Deep-Dive](./portfolio/ARCHITECTURE_DEEP_DIVE.md) · [Threat Model](./docs/SECURITY_THREAT_MODEL.md) | Trust boundaries, cost guard, governance-as-code |
| **AI Product Leader** | [Case Study §2, §5](./portfolio/CASE_STUDY.md#5-ai-product-sense--three-features-that-show-the-thesis-working) | Features mapped to a North-star tree |
| **Founder / VC** | [Case Study §2, §7, §9](./portfolio/CASE_STUDY.md#2-market--strategy--and-a-pivot-i-made-against-myself) · [STRATEGY](./docs/strategy/STRATEGY.md) | A de-risked wedge→OS roadmap and a compounding moat |

## The proof, in numbers

*All verifiable in the source repository. Pre-launch, so the metrics below are **targets**, never claimed traction.*

| Build evidence | | Capability evidence | |
|---|---|---|---|
| Features (Clean Architecture) | **13** | Architecture Decision Records | **5 + index** |
| Dart source / test files | **245 / 46** | C4 + sequence diagrams (render-validated) | **7** |
| Commits | **98** | AI gateway functions (Deno) | **5** |
| Governance state machine (TS, CI-enforced) | **~960 LOC** | Technical deep-dive articles | **2** |

## What this demonstrates

- **Consultant / strategist** — reframed the company's thesis *against personal sunk cost*
  (RPG → Twin), chose a defensible beachhead, and refuses vanity metrics.
- **Enterprise architect** — server-side AI gateway with a hard cost guard, tiered model routing,
  graceful degradation, and ethics encoded as **CI-enforced runtime invariants**.
- **AI product leader** — features mapped to metrics, an instrumented viral-K loop, AI grounded in
  real user state.
- **Founder** — a wedge→OS roadmap that de-risks each act and a retention-safe revenue model.

## Technology

| Layer | Technology |
|-------|-----------|
| Mobile / Web | Flutter / Flame Engine / Riverpod / GoRouter |
| Backend | Firebase (app plane) + Supabase Postgres + pgvector (the "you-graph") |
| AI gateway | Supabase Edge Functions (Deno) — tiered routing + cost guard ([ADR-003](./docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md)) |
| AI models | Claude Sonnet 4.5 (generate) · Haiku 4.5 (classify) · GPT-4o-mini (fallback) |
| Observability | Sentry + Crashlytics + Mixpanel / GameAnalytics |

## Repository map

```
eidolon-portfolio/
├── portfolio/        ← START HERE — the proof layer
│   ├── README.md            4-audience router + 30-sec proof
│   ├── EXECUTIVE_SUMMARY.md  one-page, board-ready
│   ├── CASE_STUDY.md         flagship; 9 sections, capability-mapped
│   ├── ARCHITECTURE_DEEP_DIVE.md  C4 + sequence + deployment (Mermaid)
│   ├── DEMO.md               walkthrough + capture plan
│   ├── recruiter-guide.md
│   └── hiring-manager-guide.md
├── docs/
│   ├── ARCHITECTURE.md · GDD.md · API.md · RUNBOOK.md · SETUP_GUIDE.md
│   ├── SECURITY_THREAT_MODEL.md   trust boundaries + STRIDE analysis
│   ├── DECISIONS/            5 ADRs + index
│   ├── governance/           constitution + ontology + CI-enforced state machine
│   └── strategy/             the wedge→OS strategy (the pivot)
└── content/articles/         technical deep-dive writing
```

## Scope & honesty

- This repo is the **documentation & evidence layer**. The product source (app / backend /
  packages) is intentionally kept private — what's public is the strategy, architecture, decisions,
  governance, and case study.
- **Pre-launch.** Every revenue/retention figure is a target. The case study's
  [Honest Gaps](./portfolio/CASE_STUDY.md#9-honest-gaps-the-most-credible-section) section names the
  remaining weaknesses and the plan to close them — read it first if you want the fastest read on
  credibility.

## License

Proprietary — All Rights Reserved. See [LICENSE](./LICENSE). This repository is shared for
**portfolio review**; the documents may be read but not reused without permission.
