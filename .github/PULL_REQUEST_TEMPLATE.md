## What & why
_What does this change, and what user/business outcome does it serve? Link the issue it closes._

Closes #

## How
_Key implementation decisions. If a significant or hard-to-reverse choice was made, link or add an
[ADR](../docs/DECISIONS/README.md)._

## Type
- [ ] Feature
- [ ] Fix
- [ ] Refactor (no behavior change)
- [ ] Docs / chore

## Checklist
- [ ] `flutter analyze` clean · `dart format` applied
- [ ] Tests added/updated — happy path **and** failure/degraded path
- [ ] Coverage not regressed (gate: ≥ 80%)
- [ ] Analytics events added/verified if user-facing
- [ ] Strings localized (no hardcoded UI text — i18n from day one)
- [ ] AI calls go through an edge function (never direct from client)
- [ ] If a decision was made → ADR added/updated
- [ ] If a governed capability changed → `docs/governance/` state machine + soundness checks updated

## Screenshots / evidence
_Before/after, a recording, or the relevant logs/events._

## Risk & rollback
_Blast radius and how to revert if this misbehaves in production._
