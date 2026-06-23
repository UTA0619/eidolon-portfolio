# Eidolon — Security & Threat Model

Version: 0.1 · 2026-06-23 · Owner: Founder / Principal Engineer

> Scope: the production system as described in [ARCHITECTURE.md](./ARCHITECTURE.md) and
> [ADR-002](./DECISIONS/ADR-002-firebase-supabase-split.md) /
> [ADR-003](./DECISIONS/ADR-003-ai-gateway-edge-functions.md). This document identifies assets,
> trust boundaries, and threats (STRIDE), with the mitigation in place or planned for each. Pre-launch:
> items marked *(planned)* are designed but not yet verified under production load.

---

## 1. What we're protecting (assets, ranked)

| # | Asset | Why it matters |
|---|---|---|
| **A1** | **The you-graph** — personality (Big Five) × episodic memory × reality timeline (steps/sleep/HRV) | The crown jewel. It is the moat *and* the most intimate data we hold. A breach here is both an existential trust event and a potential health-data exposure. |
| A2 | User identity & sessions | Account takeover → access to A1 |
| A3 | AI provider keys & spend | Leakage → financial loss (denial-of-wallet) |
| A4 | Governance guarantees (ownership, deletion, no-power-for-pay) | Violating these breaks the core brand promise |
| A5 | Game/economy integrity | Exploits damage fairness and monetization |

## 2. Trust boundaries

```
 ┌────────────┐   B1    ┌──────────────┐   B2    ┌──────────────────────┐   B3   ┌──────────────┐
 │  Client    │────────▶│  Firebase    │         │ Supabase Edge Fns +  │───────▶│ AI Providers │
 │ (untrusted)│────────▶│ (app plane)  │         │ Postgres/pgvector    │        │ (3rd party)  │
 └────────────┘   B2    └──────────────┘         │ (cognition plane)    │        └──────────────┘
        ▲                                          └──────────────────────┘
        │ the device and everything on it is outside our trust boundary
```

- **B1 — Client ↔ network:** the device is **untrusted**. Anything shipped to it (including any key)
  is considered public.
- **B2 — Client ↔ backend:** all requests authenticated; the backend never trusts client-supplied
  identity or authorization claims.
- **B3 — Backend ↔ AI providers:** outbound only, server-side, credentialed, and rate/cost-limited.

The defining decision: **no AI provider key or billing authority ever crosses B1.** All model access
is mediated server-side ([ADR-003](./DECISIONS/ADR-003-ai-gateway-edge-functions.md)).

## 3. STRIDE analysis

### Spoofing
- **Threat:** impersonating another user to read their you-graph (A1/A2).
- **Mitigations:** Firebase Auth for identity; Edge Functions verify the auth token before any
  pgvector access; the Firebase UID is the foreign key into the cognition plane
  ([ADR-002](./DECISIONS/ADR-002-firebase-supabase-split.md)).

### Tampering
- **Threat:** modifying requests to alter another user's data or the game economy (A1/A5).
- **Mitigations:** server-side validation of every mutation; **Row-Level Security (RLS)** at the
  database layer with `user_id` included in queries as a second line of defense; game-economy writes
  happen in functions, never trusted from the client.

### Repudiation
- **Threat:** denying an action (e.g. a deletion request, a purchase).
- **Mitigations:** the governance state machine models deletion/consent as explicit, logged
  transitions ([ADR-004](./DECISIONS/ADR-004-governance-as-code.md)); analytics + structured logs
  provide an audit trail. *(planned: formal audit-log retention policy.)*

### Information Disclosure — **the highest-priority category**
- **Threat:** exfiltration of A1 (the you-graph), including health-class data.
- **Mitigations:** RLS isolation per user; encryption in transit (HTTPS/WSS) and at rest (provider
  managed); the cognition plane is reachable only via authed Edge Functions, never directly from the
  client; least-privilege service roles. *(planned: field-level review for the most sensitive
  reality data; periodic access audits.)*
- **Threat:** **prompt injection** — content in a user's memories or inputs coercing the model into
  leaking system context or another user's data.
- **Mitigations:** prompts are assembled server-side from scoped, per-user data only (no cross-user
  context is ever in scope to leak); the model is given only that user's grounding. *(planned:
  output filtering + injection-pattern monitoring on the gateway.)*

### Denial of Service / Denial of Wallet
- **Threat:** driving up AI spend or exhausting capacity (A3).
- **Mitigations:** the gateway enforces **rate-limiting + a hard cost guard**; over-budget requests
  fall back to a local template rather than calling a paid model
  ([ADR-003](./DECISIONS/ADR-003-ai-gateway-edge-functions.md)); tiered routing keeps per-call cost
  bounded. *(planned: per-user spend ceilings + anomaly alerting as an SLO.)*

### Elevation of Privilege
- **Threat:** a normal user gaining admin/other-user capabilities, or **buying in-game power**
  (violating A4).
- **Mitigations:** authorization checks server-side; **"no power-for-pay" is encoded as a governance
  invariant** — a state where an entitlement grants power is illegal and fails CI
  ([ADR-004](./DECISIONS/ADR-004-governance-as-code.md)). Ethics here is an access-control property,
  not just a policy.

## 4. Governance-specific guarantees (beyond STRIDE)

Because trust *is* the product, three doctrines are treated as security properties:

| Doctrine | Enforcement |
|---|---|
| **Real deletion means real deletion** | A deletion request must reach a terminal, irreversible, complete state — a soundness invariant in the governance state machine. |
| **You own your Twin / it cannot be sold** | No legal transition produces a "sellable/ transferable-without-consent" state. |
| **No power-for-pay** | See Elevation of Privilege above. |

This is the unusual part of Eidolon's posture: the strongest privacy/ownership promises are not
policy paragraphs but **CI-enforced invariants** (see
[governance-as-code](../content/articles/governance-as-code.md)).

## 5. Residual risk & roadmap

| Risk | Status | Next step |
|---|---|---|
| Prompt-injection output leakage | Partially mitigated (scoped prompts) | Add gateway output filtering + monitoring |
| Denial-of-wallet at scale | Mitigated (cost guard) | Per-user ceilings + anomaly alerts (SLO) |
| Health-class reality data exposure | Mitigated (RLS, encryption) | Field-level sensitivity review + access audits |
| Production verification | **Pending — pre-launch** | Pen-test + security review before public launch |

> Honest status: the *design* is security-by-default; the *verification* (pen-test, load-tested cost
> guard, audit-log retention) lands as part of the Act-1 launch. This document will be versioned as
> those mitigations are confirmed in production.

---

*Reporting a vulnerability: see [SECURITY.md](../SECURITY.md). Related:
[ADR-003](./DECISIONS/ADR-003-ai-gateway-edge-functions.md),
[ADR-004](./DECISIONS/ADR-004-governance-as-code.md).*
