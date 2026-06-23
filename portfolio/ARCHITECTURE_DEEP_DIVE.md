# Eidolon — Architecture Deep-Dive (C4 + Sequences)

> Companion to the [Case Study](./CASE_STUDY.md). This document is for the reader who wants to
> verify the system is *designed*, not assembled — a CTO deciding if they'd trust the design, and
> a VC checking the moat is real infrastructure, not a slide.
>
> Every box below maps to something in the tree (`apps/mobile/lib/`, `backend/supabase/functions/`,
> `backend/functions/src/`). Where the running system diverges from
> [`docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md), this document describes **what is actually
> built** and says so — see [§6 Divergences](#6-divergences-doc-vs-tree).

---

## 1. C4 Level 1 — System Context

```mermaid
C4Context
  title Eidolon — System Context

  Person(player, "Player", "Opens the game daily; feeds the Twin personality, memory, and reality data")

  System(eidolon, "Eidolon", "AI-native RPG that bootstraps a personal AI ('Twin')")

  System_Ext(anthropic, "Anthropic Claude", "Sonnet 4.5 (story/critical), Haiku 4.5 (events/classify)")
  System_Ext(openai, "OpenAI", "GPT-4o-mini — fallback model")
  System_Ext(media, "Media AI", "Replicate SDXL (assets), ElevenLabs (voice)")
  System_Ext(reality, "Reality Data", "HealthKit / Google Fit — steps, sleep, HRV")
  System_Ext(stores, "App Stores + RevenueCat", "Distribution + subscription billing")

  Rel(player, eidolon, "Plays daily, shares morning report", "iOS / Android / Web PWA")
  Rel(eidolon, anthropic, "Generates story, dialogue, reflections", "HTTPS")
  Rel(eidolon, openai, "Falls back when Claude unavailable", "HTTPS")
  Rel(eidolon, media, "Generates assets & voice", "HTTPS")
  Rel(eidolon, reality, "Pulls reality signals (consented)", "OS APIs")
  Rel(eidolon, stores, "Distribution & entitlements", "SDK")

  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="2")
```

**Read this for:** the boundary. The player only ever talks to Eidolon. No AI provider, key, or bill
is reachable from the client — that boundary is enforced one level down.

---

## 2. C4 Level 2 — Containers

```mermaid
C4Container
  title Eidolon — Containers

  Person(player, "Player", "")

  System_Boundary(eidolon, "Eidolon") {
    Container(app, "Flutter App", "Flutter / Flame / Riverpod / GoRouter", "13 features in Clean Architecture; the only thing the player touches")

    Container(firebase, "Firebase", "Auth / Firestore / Storage / Crashlytics", "Identity, realtime game state, crash telemetry")

    ContainerDb(pg, "Supabase Postgres + pgvector", "PostgreSQL", "The 'you-graph': personality vectors x episodic memory x reality timeline")

    Container(edge, "Supabase Edge Functions", "Deno", "AI gateway + game logic: eidolon-respond, overnight-simulate, weekly-reflect, dungeon-generate, reality-sync")

    Container(fns, "Firebase Functions", "TypeScript", "Monetization, social, reality ingestion, prompt library")
  }

  System_Ext(ai, "AI Providers", "Claude / GPT-4o-mini / SDXL / ElevenLabs")

  Rel(player, app, "Uses", "HTTPS/WSS")
  Rel(app, firebase, "Auth, game state", "SDK")
  Rel(app, edge, "Requests AI generation", "HTTPS (authed)")
  Rel(edge, pg, "Reads personality+memory, writes new memories", "SQL / pgvector")
  Rel(edge, ai, "Calls models with fallback chain + cost guard", "HTTPS")
  Rel(fns, pg, "Reality + monetization writes", "SQL")
  Rel(app, fns, "Billing, social", "HTTPS")

  UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

**The two decisions an architect should notice:**

1. **Supabase Edge Functions are the AI gateway.** Every model call is server-side, authed, and
   wraps the fallback chain + cost guard. The Flutter client holds no model keys. (The original doc
   placed this in a Cloudflare Worker; the tree implements it in Edge Functions — §6.)
2. **pgvector is a first-class container, not a cache.** The moat ("you-graph") is queryable
   infrastructure: every Eidolon prompt is grounded by reading the player's personality + recent
   memories out of Postgres before the model is called.

---

## 3. C4 Level 3 — Components

### 3a. Flutter feature (Clean Architecture) — `morning_report`

Representative of all 13 features; the dependency rule points inward (presentation → domain ← data).

```mermaid
C4Component
  title Component — morning_report feature (Flutter)

  Container_Boundary(feat, "features/morning_report") {
    Component(page, "MorningReportPage", "Widget", "Renders the report; entry point")
    Component(card, "ShareableMorningCard", "Widget", "Render target for the viral share image")
    Component(prov, "MorningReportProvider", "Riverpod", "State + side-effects; emits analytics events")
    Component(uc, "Domain (entities/repo iface)", "Dart", "MorningReport entity; repository contract")
    Component(repoimpl, "RepositoryImpl", "Dart", "Maps model<->entity; returns Result<T,E>")
    Component(ds, "DataSource", "Dart", "Calls Supabase Edge Function / reads cache")
  }

  ContainerDb_Ext(edge, "Edge: overnight-simulate", "Deno", "")
  Component_Ext(analytics, "Analytics", "Mixpanel", "morning_report_viewed, morning_share_*")

  Rel(page, prov, "watches")
  Rel(card, prov, "renders state")
  Rel(prov, uc, "invokes use case")
  Rel(uc, repoimpl, "implemented by")
  Rel(repoimpl, ds, "delegates")
  Rel(ds, edge, "fetches overnight result", "HTTPS")
  Rel(prov, analytics, "tracks funnel events")

  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

**Why it matters:** the dependency rule is enforced — `domain` depends on nothing; `data` and
`presentation` depend on `domain`. Errors cross boundaries as typed `Result<T,E>`, not exceptions.
This is the contract repeated across all 13 features (CLAUDE.md), which is *why* the codebase scales.

### 3b. AI Gateway components — `eidolon-respond` Edge Function

```mermaid
flowchart TB
  req([Authed request from client]) --> auth[Auth + rate-limit check]
  auth --> guard{Cost guard<br/>under budget?}
  guard -- no --> tmpl[Local template response<br/>flagged as degraded]
  guard -- yes --> ctx[Load context from pgvector:<br/>Big Five + recent memories]
  ctx --> prompt[Build prompt<br/>backend/.../llm/prompts]
  prompt --> sonnet{Claude Sonnet 4.5}
  sonnet -- ok --> persist[Persist new memory -> pgvector]
  sonnet -- fail/timeout --> mini{GPT-4o-mini}
  mini -- ok --> persist
  mini -- fail --> tmpl
  persist --> resp([Grounded response])
  tmpl --> resp
```

**Why it matters:** the fallback chain is a *component-level* design, and the terminal fallback is a
local template that **declares itself degraded** — covered by the "fallback honesty invariant" test
(`b655b25`). Truthfulness is treated as a system property, not a vibe.

---

## 4. Key Sequences

### 4a. Grounded AI response with graceful degradation

```mermaid
sequenceDiagram
  autonumber
  participant C as Flutter Client
  participant E as Edge: eidolon-respond
  participant V as Postgres/pgvector
  participant S as Claude Sonnet 4.5
  participant G as GPT-4o-mini
  participant T as Local Template

  C->>E: request(player_id, context) [authed]
  E->>E: rate-limit + cost-guard
  E->>V: fetch Big Five + recent memories
  V-->>E: personality + memory embeddings
  E->>S: prompt(grounded in user state)
  alt Sonnet OK
    S-->>E: response
  else Sonnet fails/timeout
    E->>G: same prompt
    alt GPT-4o-mini OK
      G-->>E: response
    else all providers down
      E->>T: build local template
      T-->>E: response (degraded=true)
    end
  end
  E->>V: persist new memory (if model-generated)
  E-->>C: response (+ degraded flag)
```

The **`degraded` flag** flows back to the client so the UI can be honest with the user. That is the
behavior the honesty-invariant test protects.

### 4b. The morning-report viral loop (the instrumented growth engine)

```mermaid
sequenceDiagram
  autonumber
  participant Cron as Scheduler
  participant O as Edge: overnight-simulate
  participant V as pgvector
  participant C as Client
  participant A as Analytics
  participant Soc as Social / OS Share

  Cron->>O: nightly: simulate Twin's overnight adventure
  O->>V: read state -> generate report -> write memory
  Note over C: Player opens app in the morning
  C->>A: morning_report_viewed
  C->>C: render ShareableMorningCard
  C->>A: morning_share_initiated
  C->>Soc: share image to social/messaging
  Soc-->>C: shared
  C->>A: morning_share_completed
  Note over A: viral-K = share_completed / morning_report_viewed
```

**Why it matters:** the growth loop isn't hoped for — it's a measured funnel
(`morning_report_viewed → morning_share_initiated → morning_share_completed`), instrumented in commit
`62400cd`. A VC can ask "what's your K?" and the events to answer already exist.

---

## 5. Deployment

```mermaid
flowchart LR
  subgraph Client
    ios[iOS]:::dev
    and[Android]:::dev
    web[Web PWA]:::dev
  end
  subgraph CICD[CI/CD]
    gha[GitHub Actions]
    cm[Codemagic + Fastlane]
  end
  subgraph Cloud
    fb[(Firebase<br/>Auth/Firestore/Storage)]
    sb[(Supabase<br/>Postgres+pgvector<br/>Edge Functions/Deno)]
    ai[AI Providers<br/>Claude/OpenAI/SDXL/ElevenLabs]
  end
  obs[Sentry + Crashlytics<br/>+ Mixpanel/GameAnalytics]

  ios & and & web --> fb
  ios & and & web --> sb
  sb --> ai
  gha --> sb
  cm --> ios & and
  ios & and & web -.telemetry.-> obs
  classDef dev fill:#1f2937,stroke:#7B6CF6,color:#fff;
```

CI-checked governance invariants (`docs/governance/state/current_state.test.ts`) run in GitHub
Actions — the constitution is enforced on every push, not just documented.

---

## 6. Divergences (doc vs. tree)

Stating these *is* the senior move — it shows the diagrams were drawn from the code, not the wishlist.

| [`docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md) says | The tree actually has | Why / plan |
|---|---|---|
| Cloudflare Worker as the LLM proxy | Supabase **Edge Functions (Deno)** as the AI gateway | Edge Functions co-locate with pgvector (the grounding data), removing a network hop. Recorded in [ADR-003](../docs/DECISIONS/ADR-003-ai-gateway-edge-functions.md); `docs/ARCHITECTURE.md` v0.1 still needs the box updated. |
| "Flame Engine" front-and-center | Flame present; most shipped value is in feature UIs + AI loops | The wedge→Twin pivot ([STRATEGY.md](../docs/strategy/STRATEGY.md)) shifted effort from combat to the bond/reflection loop — diagrams reflect that. |

> These diagrams are anchored to the repository as it stands. They will be kept in sync as the
> Act-1 launch lands; divergences are tracked here rather than hidden.
