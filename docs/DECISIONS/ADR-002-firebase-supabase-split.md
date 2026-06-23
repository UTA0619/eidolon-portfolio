# ADR-002: Dual Backend — Firebase for App Plane, Supabase for the You-Graph

**Date:** 2026-05-24  
**Status:** Accepted  
**Deciders:** Founder / Principal Engineer

---

## Context

Eidolon needs two very different backend workloads:

1. **App plane** — auth, realtime game state, file/asset storage, crash telemetry, remote
   config. Commodity needs where time-to-ship and a mature mobile SDK matter most.
2. **Cognition plane** — the "you-graph": personality vectors (Big Five), episodic memory
   embeddings, and a reality timeline (steps/sleep/HRV). This is the moat (see
   [STRATEGY.md](../strategy/STRATEGY.md) §5) and requires relational queries + vector similarity
   search in the same store.

Forcing both onto one backend means either weak vector search (Firestore) or rebuilding mobile
auth/realtime/storage primitives from scratch (Supabase-only).

## Decision Drivers

- First-class **vector search** for memory retrieval (grounding every AI prompt in real user state)
- Relational integrity for the personality × memory × reality model
- Mature, batteries-included **mobile** auth / realtime / storage / crash reporting
- Minimize bespoke infrastructure pre-launch (solo team)
- Keep the moat data in a portable, SQL-standard store (no lock-in on the asset that matters most)

## Options Considered

### Option A: Firebase only
- **Pros:** One SDK, fastest mobile setup, generous free tier
- **Cons:** Firestore vector search is immature; document model is a poor fit for the relational
  you-graph; the moat data ends up in a proprietary store

### Option B: Supabase only
- **Pros:** Postgres + pgvector is ideal for the you-graph; one store; SQL portability
- **Cons:** Have to assemble mobile auth/realtime/storage/crash that Firebase ships out of the box;
  slower to first playable

### Option C: Firebase (app plane) + Supabase (cognition plane) — dual backend
- **Pros:** Each plane uses the best-fit tool; pgvector holds the moat in portable Postgres;
  Firebase accelerates the commodity mobile needs
- **Cons:** Two backends to operate; a data-ownership boundary to keep clean; potential identity
  reconciliation between Firebase Auth and Supabase rows

## Decision

We chose **Option C: dual backend**. Firebase owns the app plane (Auth, Firestore for live game
state, Storage, Crashlytics, Remote Config). Supabase owns the cognition plane (Postgres + pgvector
+ Edge Functions). The deciding factor is that **the moat is the you-graph**, and pgvector in
portable Postgres is the right home for it — while Firebase removes weeks of undifferentiated mobile
plumbing.

## Consequences

### Positive
- Vector similarity search is native; memory retrieval grounds every AI call (see ADR-003)
- Moat data lives in standard Postgres — portable, inspectable, no proprietary lock-in
- Fast path to a playable build via Firebase's mature SDKs

### Negative / Trade-offs
- Two backends to monitor, secure, and reason about
- A clear ownership boundary must be enforced: app/session state → Firebase; identity-graph →
  Supabase. Blurring it would create dual-write bugs.

### Risks
- **Identity reconciliation:** Firebase Auth UID must map deterministically to Supabase rows.
  Mitigation: UID is the foreign key; Edge Functions verify the Firebase token before touching
  the you-graph.
- **Dual-write drift** if game state and cognition data overlap. Mitigation: they are kept
  disjoint by design — no entity is authoritative in both stores.

## Implementation Notes

- App plane: Firebase (see [CLAUDE.md](../../CLAUDE.md) stack).
- Cognition plane: `backend/supabase/` — `migrations/`, `functions/` (Deno Edge Functions).
- Related: [ADR-003](./ADR-003-ai-gateway-edge-functions.md) (why the AI gateway lives in Supabase
  Edge Functions, next to the you-graph).
