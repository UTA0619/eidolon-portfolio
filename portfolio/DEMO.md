# Eidolon — Demo Walkthrough & Capture Plan

> Pre-launch, so this is a **capture plan + scripted walkthrough**, not a link to a live app. The
> app runs on the iOS Simulator today (Flutter 3.44.2, FVM-pinned). This document tells you exactly
> what to record, in what order, and which capability each shot proves — so the demo assets
> themselves become evidence, not decoration.
>
> Honesty note: the flows below exist in the tree (`apps/mobile/lib/features/`). Numbers shown in
> sample data are illustrative, labeled as such, never presented as real users.

---

## The one demo that matters: the morning-report viral loop

This single 30-second flow proves the most: a daily-habit hook, AI grounded in the user's state, and
an **instrumented viral-K funnel**. Lead with it.

### Storyboard (record as GIF #1 — "The Morning Ritual")

| # | Action on screen | What's happening underneath | Capability shown |
|---|---|---|---|
| 1 | App cold-opens to the home/Eidolon view | `app_opened` fires; time-of-day context (`morning`) attached | Instrumentation from first frame |
| 2 | Morning report card appears: "Here's what your Eidolon did overnight…" | `overnight-simulate` (Deno Edge Fn) generated it from the you-graph; `morning_report_viewed` fires | AI grounded in real user state |
| 3 | Stats + loot animate in (`_StatPill`, `_LootTile`) | Result rendered from persisted overnight memory | Async value loop (engagement without presence) |
| 4 | Tap **Share** → `ShareableMorningCard` renders | `morning_share_initiated` fires | The viral loop's entry point |
| 5 | OS share sheet → post to social/messaging | `morning_share_completed` fires | **viral-K = completed / viewed**, measurable |

> **Why this is the hero shot:** a VC can ask "what's your K?" and the events to compute it already
> exist (commit `62400cd`). The growth loop is engineered and instrumented, not hoped for.

### GIF #2 — "The Twin Notices You" (Act-2 proof)
Open the **weekly reflection** ("what I noticed about you"). Show the Twin surfacing a real pattern
from accumulated data ("you've opened the app every morning this week — your Eidolon is steadier
too"). Proves the wedge→Twin thesis is *shipping*, not just slide-ware.

### GIF #3 — "Send the Twin Adventuring" (async loop)
Trigger the on-demand overnight dispatch (`overnight_dispatch_tapped` → `overnight_dispatch_card`).
Proves engagement that accrues memory without demanding the user's presence.

### GIF #4 — "Graceful Degradation" (the engineering flex)
With the AI provider stubbed to fail, show the Twin still responding via the local-template
fallback **and visibly flagging that it's degraded** (the "fallback honesty invariant",
commit `b655b25`). Proves truthfulness is a tested system property. *Most portfolios never show
their failure mode — showing yours, working, is the differentiator.*

---

## Demo scenarios (for a live walkthrough / interview)

1. **The 30-second hook** — GIF #1 only. For recruiters/execs with no time.
2. **The thesis in 2 minutes** — GIF #1 → #2: "a fun daily game that is quietly building a personal
   AI." For product/founder audiences.
3. **The 5-minute engineering deep-dive** — pair GIF #4 with
   [ARCHITECTURE_DEEP_DIVE.md §4a](./ARCHITECTURE_DEEP_DIVE.md#4a-grounded-ai-response-with-graceful-degradation):
   talk through the sequence diagram while the fallback runs. For CTOs/architects.

---

## Sample data (illustrative — clearly labeled in-app as demo)

A seeded demo profile makes captures repeatable. Suggested seed (store under
`apps/mobile/lib/core/demo/`, which already exists):

```jsonc
// DEMO ONLY — not a real user
{
  "twin_name": "Nova",
  "big_five": { "O": 0.78, "C": 0.62, "E": 0.41, "A": 0.70, "N": 0.35 },
  "reality_days_logged": 28,        // for "Twin Depth" illustration
  "recent_memories": [
    "Player chose mercy over the cornered shade — third time this week.",
    "Slept poorly Tue–Thu; Nova's overnight pace was gentler in response."
  ],
  "overnight_result": {
    "summary": "Nova mapped the Whispering Stair and spared a lost familiar.",
    "loot": ["Quiet Lantern", "3× Memory Shard"],
    "mood": "steady"
  }
}
```

> Keep the seed honest: it illustrates *what a session looks like*, never implies a user base.

---

## Capture checklist (for whoever records this)

- [ ] iOS Simulator, light + dark mode (record both for GIF #1 — proves theming/`design_system`)
- [ ] Use the seeded demo profile above for repeatable shots
- [ ] GIF #1 (Morning Ritual) — the hero, ≤30s, loops cleanly
- [ ] GIF #2 (Weekly Reflection), #3 (Async Dispatch), #4 (Graceful Degradation)
- [ ] 1 full-length screen recording (2–3 min) with voiceover for the README/portfolio
- [ ] Export GIFs ≤ 5 MB each; drop into `portfolio/assets/` and embed in
      [README.md](./README.md) + [CASE_STUDY.md](./CASE_STUDY.md)
- [ ] For each asset, add a one-line caption naming the capability it proves

---

## Where these assets get embedded

| Asset | Lands in |
|---|---|
| GIF #1 (hero) | top of [portfolio/README.md](./README.md) and repo root README |
| GIF #2 | [Case Study §5](./CASE_STUDY.md#5-ai-product-sense--three-features-that-show-the-thesis-working) |
| GIF #4 | [Case Study §4](./CASE_STUDY.md#4-engineering-rigor-the-part-thats-verifiable) + Architecture §4a |
| Full recording | recruiter-guide.md (planned) |

> Until captured, these are tracked as TODOs in the [README expansion roadmap](./README.md#expansion-roadmap-depth-first).
> A demo asset is only added when it shows the real flow — a faked screenshot would undo the
> credibility the rest of this layer is built on.
