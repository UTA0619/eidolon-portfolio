# ADR-005: Per-Feature Clean Architecture with Riverpod and `Result<T,E>`

**Date:** 2026-05-26  
**Status:** Accepted  
**Deciders:** Founder / Principal Engineer

---

## Context

Eidolon is built solo but must scale to a team and to many features without turning into a
big ball of mud. There are already 13 features (`auth`, `bond`, `dungeon`, `eidolon`,
`morning_report`, `weekly_reflection`, `away_report`, `reality_sync`, `gacha`, `daily_reward`,
`home`, `onboarding`, `settings`). We need a single structural contract every feature obeys so any
contributor can navigate any feature on sight, and so business logic stays testable independent of
UI and IO.

## Decision Drivers

- **Uniformity:** one folder shape across all features → fast onboarding, parallel work
- **Testability:** domain logic must be unit-testable with no Flutter/IO dependencies
- **Predictable error handling:** failures crossing layer boundaries must be explicit, not thrown
- **State without footguns:** no scattered `setState`; one state/DI mechanism
- The quality gate of **≥80% coverage** ([CLAUDE.md](../../CLAUDE.md)) must be reachable

## Options Considered

### Option A: Layered-by-type (all pages together, all models together)
- **Pros:** Familiar
- **Cons:** Cross-cutting changes touch many folders; poor feature isolation; doesn't scale to teams

### Option B: MVC/MVVM with `setState` + service locator
- **Pros:** Low ceremony
- **Cons:** Business logic leaks into widgets; hard to test; implicit dependencies

### Option C: Per-feature Clean Architecture (`data`/`domain`/`presentation`) + Riverpod + `Result<T,E>`
- **Pros:** Each feature self-contained with the dependency rule pointing inward; domain pure and
  testable; Riverpod gives compile-checked DI + state; typed errors make failure explicit
- **Cons:** More boilerplate per feature; a learning curve for the layering discipline

## Decision

We chose **Option C**. Every feature follows the same contract:

```
features/<name>/
├── data/         (datasources, models, repositories impl)
├── domain/       (entities, repository interfaces, usecases)  ← depends on nothing
└── presentation/ (pages, widgets, providers)
```

- **Riverpod** is the single DI + state container — **never `setState`** (CLAUDE.md).
- The **domain layer returns `Result<T,E>`**; errors are values that cross boundaries explicitly,
  not exceptions that unwind through the UI.
- Widgets stay small (no widget > 300 lines; split into sub-widgets).

`morning_report` is the reference implementation: `data/datasources` → `data/repositories` →
`domain/entities` + `domain/repositories` → `presentation/providers` + `presentation/widgets`.

## Consequences

### Positive
- Any feature is navigable on sight; new contributors are productive fast
- Domain logic is unit-testable in isolation — the ≥80% coverage gate is achievable (46 test files
  across the app today)
- `Result<T,E>` makes error paths visible and forces handling (the fallback-honesty behavior in
  ADR-003 surfaces cleanly through this)

### Negative / Trade-offs
- Boilerplate: each feature carries data/domain/presentation scaffolding even when small
- Mitigated by code generation (`*.g.dart`, `*.freezed.dart`) for providers/models

### Risks
- **Discipline erosion** under deadline pressure (logic creeping into widgets). Mitigation:
  `flutter analyze` zero-warnings gate + the 300-line widget rule + the uniform folder contract make
  violations visible in review.

## Implementation Notes

- Contract: [CLAUDE.md](../../CLAUDE.md) "Feature Folder Structure (Clean Architecture)".
- Reference feature: `apps/mobile/lib/features/morning_report/`.
- Component view: [portfolio/ARCHITECTURE_DEEP_DIVE.md](../../portfolio/ARCHITECTURE_DEEP_DIVE.md) §3a.
