# ADR-003: AI Gateway in Supabase Edge Functions, with Tiered Model Routing + Fallback Chain

**Date:** 2026-06-02  
**Status:** Accepted (supersedes the Cloudflare-Worker LLM-proxy described in `docs/ARCHITECTURE.md` v0.1)  
**Deciders:** Founder / Principal Engineer

---

## Context

Every AI feature (story, dialogue, overnight simulation, weekly reflection, dungeon generation)
must call an LLM. Three hard constraints shape where and how:

1. **No model keys or billing on the client.** A leaked key or a runaway loop must be impossible
   from the app.
2. **Prompts must be grounded** in the player's real personality + recent memories — i.e. the call
   site needs cheap access to the you-graph (pgvector, see [ADR-002](./ADR-002-firebase-supabase-split.md)).
3. **The product must not hard-fail when a provider is down.** An AI companion that errors out
   breaks the daily ritual that the whole retention thesis depends on.

`docs/ARCHITECTURE.md` v0.1 placed the LLM proxy in a **Cloudflare Worker**. As the cognition
plane settled on Supabase + pgvector, that put the grounding data one network hop away from the
proxy. This ADR records the decision to consolidate the gateway into Supabase Edge Functions.

## Decision Drivers

- Co-locate the AI gateway with the grounding data (pgvector) — fewer hops, simpler auth
- Server-side enforcement of auth, rate-limit, and a hard **cost guard**
- **Tiered model routing**: cheap/fast model for classification, strong model for generation
- **Graceful degradation**: a fallback chain ending in a response that is honest about being degraded
- One runtime/deploy story for the cognition plane (Deno on Supabase)

## Options Considered

### Option A: Cloudflare Worker as LLM proxy (original doc)
- **Pros:** Global edge, cheap, fast cold starts
- **Cons:** Sits apart from pgvector → an extra hop (and extra auth) to fetch grounding data; a
  second deploy target and runtime to operate alongside Supabase

### Option B: Direct client → provider calls
- **Pros:** Simplest, lowest latency
- **Cons:** Disqualified — exposes keys and billing to the client; no central cost guard. Violates
  the CLAUDE.md "AI only via server" policy.

### Option C: Supabase Edge Functions as the AI gateway
- **Pros:** Lives next to pgvector → cheap grounding; one runtime (Deno) and one deploy with the
  rest of the cognition plane; natural place for auth + rate-limit + cost guard
- **Cons:** Not as globally distributed as Cloudflare's edge; Deno cold-start considerations

## Decision

We chose **Option C: Supabase Edge Functions**. The gateway runs as Deno functions
(`eidolon-respond`, `overnight-simulate`, `weekly-reflect`, `dungeon-generate`, `reality-sync`)
that authenticate the request, enforce a cost guard, **read grounding context from pgvector**,
route to the right model, and persist any new memory.

**Tiered routing:** Claude Haiku 4.5 for low-latency classification (tags, emotion); Claude Sonnet
4.5 for generation/reasoning. **Fallback chain:** Claude → GPT-4o-mini → **local template**. The
terminal template response carries a `degraded` flag so the UI can be honest with the user — this
behavior is locked by the "fallback honesty invariant" test (commit `b655b25`).

## Consequences

### Positive
- Grounding reads are a local query, not a cross-service call
- COGS is a *designed* variable: tiered routing + cost guard make per-user model spend controllable
  (see [Case Study §7](../../portfolio/CASE_STUDY.md))
- The product degrades instead of erroring; honesty about degradation is enforced by a test

### Negative / Trade-offs
- Less global edge distribution than a Cloudflare Worker; latency depends on Supabase region
- `docs/ARCHITECTURE.md` v0.1 now diverges from the tree until updated (tracked in
  [portfolio/ARCHITECTURE_DEEP_DIVE.md](../../portfolio/ARCHITECTURE_DEEP_DIVE.md) §6)

### Risks
- **Provider-wide outage** → users get template responses. Mitigation: that path is honest and
  tested, not a crash.
- **Cold starts** on infrequently hit functions. Mitigation: the daily-ritual functions are warm
  by usage pattern; revisit with provisioned concurrency if p95 regresses.

## Implementation Notes

- Functions: `backend/supabase/functions/` (`eidolon-respond`, `overnight-simulate`,
  `weekly-reflect`, `dungeon-generate`, `reality-sync`).
- Prompts: `backend/supabase/functions/_shared/` (`overnight_prompt.ts`, `reflection_prompt.ts`,
  with `__tests__/`) and `backend/functions/src/llm/prompts/`.
- Policy: [CLAUDE.md](../../CLAUDE.md) "LLM Prompt Policy" (Haiku=classify, Sonnet=generate,
  fallback chain).
- **Follow-up:** update `docs/ARCHITECTURE.md` v0.1 to replace the Cloudflare-Worker box with the
  Edge-Function gateway.
