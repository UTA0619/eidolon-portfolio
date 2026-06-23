# Eidolon — Portfolio Proof Layer

> This folder is not the product. It's the **evidence layer**: it frames the real Eidolon
> codebase as proof of specific, hireable capabilities — for recruiters, architects, AI product
> leaders, and investors. Everything here points back to verifiable artifacts in the repo.

**The product in one line:** an AI-native mobile RPG that is secretly a *personal AI ("Twin")* —
the game harvests the personality, memory, and reality data a personal AI needs, willingly and
daily. Pre-launch. Built solo.

**Read in order:** [**Executive Summary**](./EXECUTIVE_SUMMARY.md) (60 sec) →
[**Case Study**](./CASE_STUDY.md) (full) → [**Architecture Deep-Dive**](./ARCHITECTURE_DEEP_DIVE.md)
(C4 + sequences) → [**Demo**](./DEMO.md) (walkthrough + capture plan).

---

## Start here, by who you are

| You are a… | Read this first | What you'll verify in ~5 min |
|---|---|---|
| **Recruiter / Sourcer** | [Recruiter Guide](./recruiter-guide.md) (5-min skim path + pitch) | Role fit, green flags, how to position |
| **Hiring Manager / Eng Lead** | [Hiring Manager Guide](./hiring-manager-guide.md) (deep read + interview Qs) | What each artifact proves about seniority |
| **Enterprise Architect / CTO** | [Architecture Deep-Dive](./ARCHITECTURE_DEEP_DIVE.md) (C4 + sequences) + [Case Study §3, §4, §6](./CASE_STUDY.md#3-architecture-at-a-glance) | Separation of concerns, security/cost boundary, governance-as-code |
| **AI Product Leader** | [Case Study §2, §5](./CASE_STUDY.md#5-ai-product-sense--three-features-that-show-the-thesis-working) | Features mapped to a North-star tree; AI grounded in real user state |
| **Founder / VC** | [Case Study §2, §7, §9](./CASE_STUDY.md#2-market--strategy--and-a-pivot-i-made-against-myself) + [`docs/strategy/STRATEGY.md`](../docs/strategy/STRATEGY.md) | A de-risked wedge→OS roadmap and a moat that compounds |

## The 30-second proof

- **Real & verifiable:** 13 features in Clean Architecture · 245 Dart source files · 46 test files ·
  98 commits · 2 ADRs · a ~960-LOC governance state machine under CI.
- **AI done right:** tiered model routing (Haiku/Sonnet/GPT-4o-mini) behind a cost-guarded proxy,
  with a *local-template final fallback* and a test asserting the AI **admits** when it's degraded.
- **Strategy with teeth:** the founder pivoted the entire thesis against his own sunk cost
  ([STRATEGY.md](../docs/strategy/STRATEGY.md)) and refuses vanity metrics (North-star = *Twin Depth
  × Retained Users*, not MAU).
- **Honest:** pre-launch. No traction is claimed. [Gaps are stated plainly.](./CASE_STUDY.md#9-honest-gaps-the-most-credible-section)

## What this proves (and where to see it)

| Capability | Strongest evidence |
|---|---|
| Problem framing | [Case Study §1](./CASE_STUDY.md#1-the-problem-worth-solving) |
| Strategy & founder judgment | [STRATEGY.md](../docs/strategy/STRATEGY.md) · [Case Study §2](./CASE_STUDY.md#2-market--strategy--and-a-pivot-i-made-against-myself) |
| Enterprise architecture | [Architecture Deep-Dive](./ARCHITECTURE_DEEP_DIVE.md) (C4 + sequences) · [ARCHITECTURE.md](../docs/ARCHITECTURE.md) |
| Execution quality | [Case Study §4](./CASE_STUDY.md#4-engineering-rigor-the-part-thats-verifiable) (verifiable in tree) |
| AI product leadership | [Case Study §5](./CASE_STUDY.md#5-ai-product-sense--three-features-that-show-the-thesis-working) |
| AI governance / systems thinking | [`docs/governance/`](../docs/governance/) · [Case Study §6](./CASE_STUDY.md#6-governance-as-architecture-the-unusual-part) |
| Business & unit economics | [Case Study §7](./CASE_STUDY.md#7-business-model--unit-economics-the-model-stated-as-a-model) |

---

## Expansion roadmap (depth-first)

This layer leads with the flagship case study. Planned next artifacts, each only added if it
clears the bar of *real evidence, not volume*:

- [x] [`EXECUTIVE_SUMMARY.md`](./EXECUTIVE_SUMMARY.md) — one-page, board-ready
- [x] [`ARCHITECTURE_DEEP_DIVE.md`](./ARCHITECTURE_DEEP_DIVE.md) — C4 (Context/Container/Component) + sequences + deployment (Mermaid, all 7 diagrams render-validated)
- [x] ADR backfill — 4 new ADRs + index in [`docs/DECISIONS/`](../docs/DECISIONS/README.md) (backend split, AI gateway/routing, governance-as-code, Clean Architecture)
- [x] [`DEMO.md`](./DEMO.md) — scripted walkthrough + capture plan for the morning-report viral-K funnel *(asset capture pending)*
- [x] [`recruiter-guide.md`](./recruiter-guide.md) / [`hiring-manager-guide.md`](./hiring-manager-guide.md)

> Quantity is the trap. Each item ships only when it adds *signal* a skeptical reader can verify.
