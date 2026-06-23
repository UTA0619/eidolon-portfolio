# Governance as Code: Encoding Product Ethics as CI-Enforced Invariants

*A technical deep-dive from the Eidolon project. Companion to
[ADR-004](../../docs/DECISIONS/ADR-004-governance-as-code.md).*

---

## The problem with a constitution in a markdown file

Eidolon's positioning rests on a promise: *the AI that is constitutionally yours.* Four doctrines
make it concrete — **you own your Twin, it cannot be sold, real deletion means real deletion, and no
power-for-pay.** In an era of extractive AI, that promise is the moat.

But a promise written only in `CONSTITUTION.md` is decoration. Nothing stops a future feature — a
growth experiment, a monetization tweak, an agentic action added in a hurry — from quietly violating
it. Worse, the violation is invisible until a user (or a regulator) finds it. The gap between *what
the docs say* and *what the code does* is exactly where trust dies.

So the design question became: **how do you make a value something the system cannot violate without
a test going red?**

## The approach: a typed state machine with soundness invariants

Instead of scattering ad-hoc checks through feature code, Eidolon models governance as a single,
formal state machine in TypeScript (~960 LOC under `docs/governance/state/`):

| File | Role | LOC |
|---|---|---|
| `state_schema.ts` | The formal shape of the system's governed state | 436 |
| `transitions.ts` | The *only* legal moves between states | 256 |
| `soundness_check.ts` | Invariants that must hold in every reachable state | 117 |
| `current_state.test.ts` | Runs the soundness checks — executed in CI | 52 |

The mental model is the one you'd use for a protocol or a type system, applied to *ethics*:

- **State** is what the system is permitted to be at any moment (what the Twin is, who owns it, what
  consent has been given, what's deletable).
- **Transitions** are the only sanctioned ways that state can change. A change that isn't a defined
  transition simply cannot happen legally.
- **Invariants** are properties that must be true *after every transition* — e.g. "a Twin is never
  in a sellable state," "a deletion request reaches a terminal, irreversible, complete state," "no
  entitlement grants in-game power."

```
        ┌─────────────┐   legal transition    ┌─────────────┐
        │  state  Sₙ  │ ───────────────────▶  │ state Sₙ₊₁   │
        └─────────────┘                        └─────────────┘
               │                                      │
        soundness_check(Sₙ)                    soundness_check(Sₙ₊₁)
               │                                      │
               ▼                                      ▼
        all invariants hold  ──── else ───▶  CI fails the build
```

Because the checks run in GitHub Actions on every push, **a change that could drive the system into
an illegal state never merges.** The constitution stops being a document you're trusted to remember
and becomes a property the pipeline enforces.

## Why this is worth the cost

Encoding ethics as runtime invariants is unusual on a consumer product. Three reasons it earned its
place:

1. **It makes "trust" auditable.** When someone asks "prove deletion is real," the answer is a
   pointer to an invariant and a CI run — not a paragraph of reassurance. That's the difference
   between a marketing claim and a guarantee.

2. **It's the substrate the agentic future needs.** Eidolon's roadmap ends with a Twin that *acts*
   on the user's behalf (drafts messages, prepares the day, eventually represents them in low-stakes
   negotiations). Agency without a formal model of "what actions are legal, bounded, and reversible"
   is how you ship a product that betrays its users by accident. The state machine is where those
   bounds live.

3. **It's ahead of the regulatory curve.** The EU AI Act and the broader model-governance wave are
   moving toward *demonstrable* controls. "Show me where this is enforced" will increasingly have to
   have a code answer. Building that muscle on a consumer app, before it was mandatory, is the kind
   of foresight that compounds.

## The trade-offs (honestly)

- **Modeling tax.** Every powerful new feature must define its legal transitions, not just ship.
  That's friction — deliberately. The friction is the point at exactly the places where moving fast
  would mean moving recklessly.
- **Drift risk.** A state machine can lag what features actually do. The mitigation is twofold: the
  soundness checks run in CI (so drift that breaks an invariant is caught), and capability expansion
  is gated by phase docs (`docs/governance/phases/`, Phase-A enforcement) rather than added ad hoc.
- **Over-modeling.** It's tempting to formalize everything. The discipline is to model only the
  doctrines that are genuinely load-bearing for trust and agency, and to expand per phase rather
  than speculatively.

## Takeaways for other teams

- If a value is core to your product's trust, **make it executable.** A check that runs beats a
  paragraph that's read.
- **Centralize it.** Scattered runtime checks can't be reasoned about globally and are easy to
  forget in a new feature. One state machine is one source of truth.
- **Put it in CI.** An invariant that isn't enforced on every change is an invariant that will
  eventually be violated.
- Governance, done this way, is **not overhead — it's positioning.** "Constitutionally yours,"
  enforced by tests, is a feature competitors with extractive defaults cannot copy without changing
  who they are.

---

*See also: [ADR-004](../../docs/DECISIONS/ADR-004-governance-as-code.md),
[`docs/governance/`](../../docs/governance/), and
[Case Study §6](../../portfolio/CASE_STUDY.md#6-governance-as-architecture-the-unusual-part).*
