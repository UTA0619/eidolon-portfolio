# Architecture Decision Records (ADRs)

Each ADR captures one significant, hard-to-reverse decision: the context, the options weighed, the
choice, and the consequences (including the trade-offs we accepted). They are written so a new
engineer — or a reviewing CTO — can understand *why* the system is shaped the way it is, not just
*what* it does.

Format: [ADR-000-template.md](./ADR-000-template.md).

| # | Decision | Status | Date |
|---|---|---|---|
| [001](./ADR-001-flutter-monorepo.md) | Flutter + Melos monorepo for iOS/Android/Web | Accepted | 2026-05-22 |
| [002](./ADR-002-firebase-supabase-split.md) | Dual backend: Firebase (app plane) + Supabase/pgvector (you-graph) | Accepted | 2026-05-24 |
| [003](./ADR-003-ai-gateway-edge-functions.md) | AI gateway in Supabase Edge Functions; tiered routing + fallback chain | Accepted (supersedes ARCHITECTURE.md v0.1 CF-Worker) | 2026-06-02 |
| [004](./ADR-004-governance-as-code.md) | Governance as a typed state machine under CI | Accepted | 2026-06-08 |
| [005](./ADR-005-clean-architecture-riverpod-result.md) | Per-feature Clean Architecture + Riverpod + `Result<T,E>` | Accepted | 2026-05-26 |

> ADRs 002–005 were backfilled on 2026-06-23 to record decisions already implemented in the tree.
> Backfilling is deliberate: the decisions were real and load-bearing; documenting them closes the
> gap between what the code does and what the record explains.
