# Recruiter Guide — How to Read This Project in 5 Minutes

> For a technical recruiter or sourcer deciding whether to advance this candidate and how to pitch
> them to a hiring manager. No engineering background needed.

---

## The 30-second pitch (steal this for your hiring manager)

> "Solo-built a pre-launch AI product with the rigor of a funded startup — strategy, cloud
> architecture, documented decisions, and AI engineering — and is honest about what's done vs.
> not. The repo is the proof: clone it and the claims hold up."

---

## What roles this fits

| Role | Fit | Why |
|---|---|---|
| AI / IT Consultant | Strong | Business cases, ROI thinking, structured problem framing, executive-readable docs |
| Enterprise Architect | Strong | C4 diagrams, ADRs, security/cost boundaries, governance-as-code |
| AI Product Manager / Lead | Strong | Features mapped to metrics, a North-star tree, instrumented growth loop |
| Founder / early-stage / CTO-track | Strong | Owns strategy, architecture, and execution end-to-end |
| Pure backend/frontend IC (narrow) | Partial | Range is the story; for a narrow IC role, point to the engineering rigor section |

**Seniority signal:** mid-to-senior. The standout traits are *judgment* (pivoted the whole
strategy on evidence), *honesty* (a section that lists his own gaps), and *breadth* (VC narrative
down to typed error handling) — not raw years.

## Your 5-minute skim path

1. [Executive Summary](./EXECUTIVE_SUMMARY.md) — one page, the whole story (60 sec).
2. [Case Study §8 "60 seconds to a hiring manager"](./CASE_STUDY.md#8-what-id-tell-a-hiring-manager-in-60-seconds) — pre-written talking points.
3. [Case Study §9 "Honest Gaps"](./CASE_STUDY.md#9-honest-gaps-the-most-credible-section) — proves he self-assesses. This is the section that separates real from inflated.
4. Glance at the [diagrams](./ARCHITECTURE_DEEP_DIVE.md) — even if you don't read them, that they exist and render is the signal.

## Green flags to highlight to the hiring manager

- **It's real and verifiable** — 13 features, 245 source / 46 test files, 98 commits. Not slides.
- **Pivoted his own thesis** (RPG → personal AI) on an unsentimental review — rare self-correction.
- **Honest about pre-launch status** — never claims users or revenue he doesn't have.
- **Documents decisions** (5 ADRs) — works the way senior engineers work.
- **AI done responsibly** — tested the AI for *honesty about failure*, not just happy-path demos.

## Likely questions you'll get, and the answer

- *"Is this just AI-generated fluff?"* → No — every number traces to the codebase; clone and check.
  The honest-gaps section is something fluff never includes.
- *"Does he have production experience?"* → This is pre-launch and solo — that's stated plainly. The
  *practices* (architecture, testing, ADRs, observability) are production-grade; the *traction* is
  ahead. Screen for it directly in interview.
- *"Team experience?"* → Solo project. The collaboration practices are demonstrated (branch
  workflow, PR-based change, ADRs); real team-scale proof is a fair interview topic.

## How to position him

Lead with **range + judgment**, not "X years of Y." This is a candidate who can sit in a room with
executives *and* write the code — valuable for consulting, founding-engineer, and architect tracks.
Pair this guide with the [Hiring Manager Guide](./hiring-manager-guide.md) for the technical deep read.
