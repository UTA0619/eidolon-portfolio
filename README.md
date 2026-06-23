# Eidolon — The Soul-Bound RPG

> "Your AI soul-twin keeps adventuring while you sleep."

Eidolon is the world's first RPG combining AI-generated dungeons, reality-linked gameplay,
AI companions, and asynchronous multiplayer — for iOS, Android, and Web (PWA).

> ### 👀 Reviewing this as a recruiter, architect, AI product leader, or investor?
> **Start with the [Portfolio / Proof Layer →](./portfolio/README.md)** — strategy, C4
> architecture, ADRs, governance-as-code, and a full product case study, grounded entirely in
> verifiable artifacts. Suggested order:
> [Executive Summary](./portfolio/EXECUTIVE_SUMMARY.md) (60 sec) →
> [Case Study](./portfolio/CASE_STUDY.md) →
> [Architecture Deep-Dive](./portfolio/ARCHITECTURE_DEEP_DIVE.md) →
> [Demo](./portfolio/DEMO.md).
>
> *Note: this repository is the **documentation & evidence layer**. Product source
> (app/backend/packages) is kept private; what's here is the strategy, architecture, decisions,
> and case study. Pre-launch — all revenue/retention figures are targets, never claimed traction.*

## Architecture

| Layer | Technology |
|-------|-----------|
| Mobile / Web | Flutter 3.27+ / Flame Engine 1.20+ |
| State | Riverpod 2.5+ |
| Backend | Firebase + Supabase + Cloudflare Workers |
| AI | Claude Sonnet 4.5 / Haiku 4.5 / GPT-4o-mini |
| Analytics | Mixpanel + GameAnalytics + AppsFlyer |
| Monitoring | Sentry + Crashlytics |

## Repository Structure

```
eidolon/
├── apps/mobile/          # Flutter app (iOS / Android / Web)
├── apps/admin/           # Internal admin dashboard
├── backend/functions/    # Firebase Cloud Functions (TypeScript)
├── backend/supabase/     # Supabase migrations + Edge Functions
├── packages/shared_types # Shared Dart/TS type definitions
├── packages/game_logic   # Core game engine logic
├── packages/design_system# UI component library
└── docs/                 # GDD, Architecture, API docs, ADRs
```

## Getting Started

### Prerequisites

- Flutter 3.27+ / Dart 3.6+
- Node.js 20+
- Firebase CLI
- Supabase CLI
- Melos (Dart monorepo tooling)

### Setup

```bash
# Install Melos globally
dart pub global activate melos

# Bootstrap all packages
melos bootstrap

# Run the mobile app
cd apps/mobile && flutter run
```

## Success Metrics

| KPI | Target |
|-----|--------|
| Monthly Revenue | ≥ $40,000 |
| MAU | ≥ 25,000 |
| D1 Retention | ≥ 55% |
| D7 Retention | ≥ 30% |
| D30 Retention | ≥ 18% |
| App Store Rating | ≥ 4.5★ |
| Crash-Free Sessions | ≥ 99.5% |

## License

Proprietary — All Rights Reserved. See [LICENSE](./LICENSE).
