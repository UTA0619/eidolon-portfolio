---
name: "✨ Feature / User Story"
about: Propose a capability, framed as business value → user story → acceptance criteria
title: "[feat] "
labels: ["type:feature", "needs-triage"]
---

> Delete the guidance lines as you fill this in. The goal is that anyone — engineer or stakeholder —
> can read this and understand *why it matters* before *how it's built*.

## Business context
_Why does this matter? What user pain or business outcome does it serve? Tie it to the
North-star (Twin Depth × Retained Users) or a named metric where possible._

## User story
> As a **<persona>**, I want **<capability>**, so that **<outcome>**.

## Acceptance criteria
- [ ] Given <context>, when <action>, then <observable result>
- [ ] …
- [ ] Analytics: the relevant event(s) fire and are verifiable
- [ ] Tests cover the happy path **and** the failure/degraded path

## Technical notes
_Affected feature(s) (`apps/mobile/lib/features/<name>/`), edge functions, data model changes,
and whether this needs a new [ADR](../../docs/DECISIONS/README.md) or a governance state-machine
change (`docs/governance/`)._

## Priority & sizing
- **Priority:** P0 / P1 / P2 / P3
- **Story points:** (1 / 2 / 3 / 5 / 8 / 13)
- **Expected business impact:** _e.g. "improves D7 retention" / "unblocks Act-2 reflection loop"_

## Definition of done
- [ ] `flutter analyze` clean · tests green · coverage not regressed
- [ ] If a significant decision was made, an ADR is added/updated
- [ ] Docs updated if behavior or architecture changed
