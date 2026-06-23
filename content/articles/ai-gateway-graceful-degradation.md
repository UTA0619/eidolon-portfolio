# Designing an AI Gateway That Degrades Honestly

*A technical deep-dive from the Eidolon project. Companion to
[ADR-003](../../docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md).*

---

## Three constraints that shaped everything

Eidolon is an AI companion. Every meaningful interaction — story beats, dialogue, the overnight
simulation, the weekly reflection — is an LLM call. Three constraints made "just call the API from
the app" a non-starter:

1. **No keys or billing on the client.** A leaked key or a runaway retry loop on a million phones is
   an existential cost event. The blast radius has to be server-side and capped.
2. **Responses must be grounded.** A generic LLM reply is worthless here; the Twin has to answer
   *as someone who knows you* — which means every prompt is built from the user's Big Five
   personality and recent episodic memories, read out of `pgvector`.
3. **It must never hard-fail.** The entire retention thesis is a daily ritual. An AI companion that
   throws an error instead of speaking doesn't just degrade UX — it breaks the habit the company is
   built on.

These three forces — cost containment, grounding, and resilience — point at the same conclusion: a
**server-side AI gateway** that sits next to the data and owns the failure modes.

## Where the gateway lives, and why it moved

The original architecture put the LLM proxy in a Cloudflare Worker. As the cognition plane settled
on Supabase + pgvector (the "you-graph"), that placement meant the proxy was one network hop away
from the grounding data it needed on *every* call. So the gateway was consolidated into **Supabase
Edge Functions (Deno)**, co-located with pgvector. (This is exactly the kind of reversal worth
recording — see [ADR-003](../../docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md), which
supersedes the earlier doc.)

The lesson: **put the gateway where the grounding data is.** A few milliseconds per call matters
less than the simplicity of one auth domain and one round-trip to build the prompt.

## The request path

```
client ──authed──▶ Edge Function (eidolon-respond)
                     │
                     ├─ 1. auth + rate-limit
                     ├─ 2. cost guard: under budget?  ──no──▶ local template (degraded=true)
                     ├─ 3. load grounding from pgvector (Big Five + recent memories)
                     ├─ 4. build prompt
                     ├─ 5. Claude Sonnet 4.5  ──fail/timeout──▶ GPT-4o-mini ──fail──▶ local template
                     ├─ 6. persist new memory → pgvector
                     └─ 7. return response (+ degraded flag)
```

Two design choices in that path do most of the work.

### Tiered model routing — cost as a designed variable

Not every call deserves the same model. Classification work (tagging, emotion detection) goes to
**Claude Haiku** for low latency and low cost; generation and reasoning go to **Claude Sonnet** for
quality. The result is that **cost-per-user is a variable you design, not a number you discover on
the invoice.** Combined with the cost guard, this is what gives an AI product a credible path to
margin instead of being a venture-subsidized demo.

### The fallback chain — and the part most teams skip

The chain is `Claude → GPT-4o-mini → local template`. The first two are unremarkable. The last one
is the interesting part: when every provider is unreachable, the gateway returns a **locally
generated template response** so the ritual survives an outage.

But a fallback that *pretends* to be the real Twin is a lie that erodes trust the moment a user
notices. So the terminal response carries a **`degraded` flag** that flows back to the UI, and the
product is honest: it tells the user it's running in a reduced mode rather than passing off a
canned line as the Twin's own thought.

This honesty is not left to discipline — it's locked by a test, the **"fallback honesty invariant"**
(commit `b655b25`), which asserts that a degraded response is *marked* degraded. Truthfulness is
treated as a system property with a regression test, the same way you'd protect any other invariant.

## Why "honest degradation" is the whole point

It's easy to demo the happy path. What separates a real AI product from a prototype is how it
behaves when the model is slow, down, or over budget. Eidolon's answer:

- **It still responds** (the ritual holds).
- **It tells the truth about the response** (trust holds).
- **It can't accidentally start lying** (the invariant is tested).

Most portfolios never show their failure mode. Showing yours — *working, and honest* — is the
differentiator.

## The trade-offs (honestly)

- **Less global edge distribution** than a Cloudflare Worker; latency now depends on the Supabase
  region. Accepted in exchange for grounding-locality and one auth domain.
- **Cold starts** on infrequently hit functions. The daily-ritual functions stay warm by usage; if
  p95 regresses, provisioned concurrency is the lever.
- **Template fallback is a floor, not a feature.** It keeps the lights on; it is not a substitute
  for provider reliability, and the `degraded` rate should be monitored as an SLO.

## Takeaways for other teams

- **Never let the client hold model keys or unbounded spend.** Put a gateway in front, server-side.
- **Co-locate the gateway with your grounding data.** Round-trips to build the prompt add up.
- **Route by job, not by default.** Tiered models turn COGS into a design parameter.
- **Plan the failure path, then make it honest.** A graceful fallback that lies is worse than an
  error. Mark degraded responses, and *test* that you do.

---

*See also: [ADR-003](../../docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md),
[Architecture Deep-Dive §4a](../../portfolio/ARCHITECTURE_DEEP_DIVE.md#4a-grounded-ai-response-with-graceful-degradation),
and [Case Study §4](../../portfolio/CASE_STUDY.md#4-engineering-rigor-the-part-thats-verifiable).*
