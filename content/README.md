# Content — Technical Writing

Deep-dive articles that explain the *reasoning* behind Eidolon's most distinctive engineering
decisions. Each is grounded in a real ADR and the actual codebase — written the way you'd write for
an engineering blog, not a portfolio filler.

| Article | What it covers | Grounded in |
|---|---|---|
| [Governance as Code](./articles/governance-as-code.md) | Encoding product ethics (ownership, deletion, no-power-for-pay) as a CI-enforced typed state machine | [ADR-004](../docs/DECISIONS/ADR-004-governance-as-code.md), `docs/governance/` |
| [Designing an AI Gateway That Degrades Honestly](./articles/ai-gateway-graceful-degradation.md) | Server-side AI gateway: cost guard, tiered routing, and a fallback chain that's honest about degradation | [ADR-003](../docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md) |

> These double as the basis for thought-leadership posts (engineering blog, LinkedIn). They are
> deliberately specific — the value is in the trade-offs named, not the topic covered.
