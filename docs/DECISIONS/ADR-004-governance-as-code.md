# ADR-004: Governance as Code — the Constitution is a Typed State Machine under CI

**Date:** 2026-06-08  
**Status:** Accepted  
**Deciders:** Founder / Principal Engineer

---

## Context

Eidolon's positioning rests on trust: "the AI that is *constitutionally* yours." The doctrines —
**you own your Twin, it cannot be sold, real deletion means real deletion, no power-for-pay** — are
both ethics and marketing (see [STRATEGY.md](../strategy/STRATEGY.md) §5). They are also exactly the
kind of guarantee that the agentic Act-3 roadmap and emerging regulation (EU AI Act, model
governance) will demand be *enforceable*, not aspirational.

A constitution that lives only in a markdown file is decoration. The question: how do we make the
governance rules something the system *cannot* violate without a test going red?

## Decision Drivers

- Trust must be **enforceable**, not just documented
- Invariants (e.g. "deletion is irreversible and complete", "no entitlement grants in-game power")
  must hold across every state transition
- The agentic future (Act 3) needs a formal, bounded model of legal actions and consent
- Regulatory/audit defensibility — "show me where this is enforced" must have a code answer

## Options Considered

### Option A: Constitution as documentation only
- **Pros:** Zero engineering cost
- **Cons:** Unenforced; drifts from behavior; worthless under audit; no protection as the agentic
  surface grows

### Option B: Ad-hoc runtime checks scattered in feature code
- **Pros:** Pragmatic, incremental
- **Cons:** No single source of truth; invariants can't be reasoned about globally; easy to forget a
  check in a new feature

### Option C: A formal, typed state machine with soundness invariants, executed in CI
- **Pros:** Single authoritative model of legal states/transitions; invariants checked centrally and
  on every push; auditable; a foundation for bounded agentic actions
- **Cons:** Upfront modeling cost; the runtime must consult the model rather than improvise

## Decision

We chose **Option C**. The governance layer (`docs/governance/`) pairs written documents
(`CONSTITUTION.md`, `ONTOLOGY.md`, `PHILOSOPHY.md`) with a **TypeScript state machine** (~960 LOC):

- `state_schema.ts` (436 LOC) — the formal system state
- `transitions.ts` (256 LOC) — the only legal moves between states
- `soundness_check.ts` (117 LOC) — invariants that must always hold
- `current_state.test.ts` — the soundness checks, run in **GitHub Actions on every push**

Ethics are encoded as runtime invariants. A change that would let the system reach an illegal state
(e.g. a sellable Twin, an incomplete deletion, power granted for payment) fails CI.

## Consequences

### Positive
- The "constitutionally yours" claim has a code-level answer to "prove it"
- A formal substrate for Act-3 agentic actions (bounded, reversible, consent-gated per the ONTOLOGY)
- Audit/regulatory defensibility ahead of the curve — a differentiator in the JP/EU beachhead

### Negative / Trade-offs
- Modeling overhead; new powerful features must define their legal transitions, not just ship
- The state machine is a contract the rest of the system must respect — a discipline cost

### Risks
- **Model/implementation drift** — the state machine could lag what features actually do.
  Mitigation: soundness checks run in CI; the phase docs (`docs/governance/phases/`) gate
  capability expansion (Phase-A D4/D5/D6 enforcement).
- **Over-modeling** before product-market fit. Mitigation: scope the machine to the doctrines that
  are actually load-bearing for trust and agency; expand per phase, not speculatively.

## Implementation Notes

- Code: `docs/governance/state/` (`state_schema.ts`, `transitions.ts`, `soundness_check.ts`,
  `current_state.ts`, `current_state.test.ts`).
- Docs: `docs/governance/` (`CONSTITUTION.md`, `ONTOLOGY.md`, `PHILOSOPHY.md`), phases in
  `docs/governance/phases/`.
- Narrative: [Case Study §6](../../portfolio/CASE_STUDY.md#6-governance-as-architecture-the-unusual-part).
- Related: [ADR-003](./ADR-003-ai-gateway-edge-functions.md) (the AI gateway is where agentic actions
  will be checked against this model).
