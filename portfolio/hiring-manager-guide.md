# Hiring Manager Guide — The Technical Deep Read

> For the engineering leader / hiring manager evaluating depth and seniority. Assumes you'll open
> the artifacts. Companion to the [Recruiter Guide](./recruiter-guide.md).

---

## What to evaluate, and where the evidence is

| Capability you're hiring for | Evidence | Verdict signal |
|---|---|---|
| System design | [Architecture Deep-Dive](./ARCHITECTURE_DEEP_DIVE.md) — C4 L1–L3 + sequences | Server-side AI gateway with a cost guard; pgvector chosen to serve the moat, not by default |
| Decision-making under uncertainty | [`docs/DECISIONS/`](../docs/DECISIONS/README.md) — 5 ADRs | Options weighed, trade-offs *accepted explicitly*, one ADR supersedes an earlier choice |
| Code organization at scale | [Case Study §4](./CASE_STUDY.md#4-engineering-rigor-the-part-thats-verifiable) | Clean Architecture repeated across 13 features; dependency rule enforced; `Result<T,E>` |
| AI engineering maturity | [ADR-003](../docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md) + Architecture §4a | Tiered routing, fallback chain to a local template, and a test asserting honest degradation |
| Product judgment | [Case Study §2, §5](./CASE_STUDY.md#5-ai-product-sense--three-features-that-show-the-thesis-working) | Features map to a North-star tree; refuses MAU as a vanity metric |
| Risk / governance awareness | [`docs/governance/`](../docs/governance/) + [ADR-004](../docs/DECISIONS/ADR-004-governance-as-code.md) | Ethics encoded as a CI-enforced typed state machine — ahead of the regulatory curve |
| Self-awareness / seniority | [Case Study §9](./CASE_STUDY.md#9-honest-gaps-the-most-credible-section) | Names his own gaps and the plan to close them |

## The three artifacts that tell you the most

1. **ADR-003 (AI gateway).** Read how he reasons about *where* AI calls live (next to the grounding
   data), *cost* as a designed variable (tiered routing + guard), and *graceful degradation*. This
   is the difference between "uses an LLM" and "engineers an AI system."
2. **The governance state machine** (`docs/governance/state/`, ~960 LOC TS under CI). Encoding
   product ethics as runtime invariants is unusual and senior. Probe whether he can extend it.
3. **Case Study §9 (Honest Gaps).** If a candidate volunteers exactly where the work is thin, your
   interview can go deep instead of fencing.

## Interview questions grounded in this repo

These let you verify the work is his and probe the ceiling:

- *"Walk me through ADR-003. What would change if latency, not cost, were the binding constraint?"*
- *"Your fallback chain ends in a local template flagged as degraded. Why is that flag worth a
  dedicated test? What breaks if it silently lies?"*
- *"The governance rules are a typed state machine. Add a new agentic action (Act 3) — what states,
  transitions, and invariants does it need?"*
- *"You pivoted from 'AI RPG' to 'personal AI.' What evidence would have made you pivot *back*?"*
- *"North-star is Twin Depth × Retained Users. How would you actually instrument Twin Depth, and
  what's the failure mode of optimizing it?"*
- *"13 features, solo. Where did Clean Architecture cost you more than it gave, and would you do it
  again?"*

## Honest calibration (so you're not surprised)

- **Pre-launch, solo.** No production-traffic war stories, no team-scale collaboration yet — both
  stated openly in [§9](./CASE_STUDY.md#9-honest-gaps-the-most-credible-section). Screen for these
  directly rather than assuming.
- **Practices outrun results.** The architecture/testing/decision discipline is senior; the metrics
  are targets. That's the right asymmetry for a builder pre-PMF, but verify execution speed and how
  he behaves with real users / a real team.
- **Breadth over narrow depth.** If your role needs a deep specialist in one stack, probe that area
  specifically; his strength is end-to-end ownership.

## Bottom line

A candidate who can own a product from VC narrative to typed error handling, makes decisions on the
record, builds AI responsibly, and tells you the truth about what isn't done. Strongest fit for
consulting, architect, AI-product, and founding-engineer / CTO-track roles.
