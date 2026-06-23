# Eidolon — Executive Summary

> One page. For a recruiter, hiring manager, or executive deciding in 60 seconds whether to read
> further. Full detail: [Case Study](./CASE_STUDY.md) · [Architecture](./ARCHITECTURE_DEEP_DIVE.md).

---

**What it is.** Eidolon is an AI-native mobile RPG that is secretly a *personal AI ("Twin")*. The
game is a wedge: it gets people to feed an AI their personality, memories, and real-life data —
willingly, daily — so the Twin comes to know them better than any competitor's model could
reconstruct. **Status: pre-launch.** Built solo.

**Why it matters.** Frontier models commoditize on a ~12-month cycle; a chatbot has zero switching
cost. The durable moat is not model IQ but *accumulated, un-cloneable user context* — and a daily
habit is the only way to accumulate it. Eidolon is the habit engine for that moat.

---

### The proof, in numbers (all verifiable in the repo)

| | |
|---|---|
| Features (Clean Architecture) | **13** |
| Dart source / test files | **245 / 46** |
| Commits | **98** |
| Architecture Decision Records | **5 + index** |
| Governance state machine (TypeScript, CI-enforced) | **~960 LOC** |
| AI gateway functions (Deno) | **5** (`eidolon-respond`, `overnight-simulate`, `weekly-reflect`, `dungeon-generate`, `reality-sync`) |
| C4 + sequence diagrams (render-validated) | **7** |

*Every revenue/retention figure elsewhere is a **target**, never claimed traction. Saying so is the point.*

---

### What this demonstrates (the four hats)

- **Consultant / strategist** — reframed the company's thesis *against personal sunk cost*
  (RPG → Twin), chose a defensible beachhead (Japan), and refuses vanity metrics (North-star =
  *Twin Depth × Retained Users*, not MAU).
- **Enterprise architect** — server-side AI gateway with a hard cost guard, tiered model routing
  (Haiku/Sonnet/GPT-4o-mini), graceful degradation to a local template, and ethics encoded as
  **CI-enforced runtime invariants**.
- **AI product leader** — shipped features each mapped to a metric: a morning-report **viral-K
  funnel** (instrumented), a weekly "what I noticed about you" reflection, an overnight async loop —
  all grounded in the user's real accumulated state.
- **Founder** — a wedge→OS roadmap that de-risks each act, a relationship-based revenue model that
  protects retention, and COGS designed in via model routing.

---

### The two things to verify first

1. **Clone it.** The numbers above are in the tree, not on a slide.
2. **Read [§9 Honest Gaps](./CASE_STUDY.md#9-honest-gaps-the-most-credible-section).** A portfolio
   that names its own weaknesses — pre-launch, solo, Twin-Depth defined-not-yet-measured — and ships
   the plan to close them is the signal worth hiring for.

---

*Bottom line: this is not a tutorial project. It is a pre-launch startup's strategy, architecture,
governance, and AI engineering — executed solo to a bar that survives a skeptical clone-and-check.*
