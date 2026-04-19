# Prompt 07 — Synthesize `.planning/` (GSD-native)

**Inputs:** everything under `.scratch/` + the `/spec/*.md` files just written
**Output:** `<out>/.planning/{PROJECT.md, REQUIREMENTS.md, ROADMAP.md, phases/NN-slug/PLAN.md}`

## Sizing

Complexity score = `len(routes) + len(tables) + len(edge_functions) + len(distinct_feature_areas)`.

- **≤12** → one milestone, one phase per feature area (aim for 3-5 phases total).
- **13-30** → one milestone, more phases (aim for 6-10). Group tightly related features into single phases.
- **>30** → multiple milestones. Each milestone delivers a coherent user-facing surface (e.g., M1 = auth + session CRUD, M2 = document upload + AI reports, M3 = admin + analytics).

## `PROJECT.md`

```
# <Project Name>

## Vision
<one-liner from spec/00-overview.md>

## Target Users
- <persona 1>
- <persona 2>

## Success Metrics
- <from spec/00-overview.md and chat-insights>

## Current Milestone
v0.1 — reimplementation from spec

## Tech Stack
*Reimplementer's choice.* The source was Lovable (see `spec/implementation-hints/`).
```

## `REQUIREMENTS.md`

One requirement per feature in `spec/02-features.md`. Format:
```
## R-NN — <feature title>
**Source:** spec/02-features.md#F-NN
**Must:** <observable criterion>
**Should:** <…>
**Evidence:** (chat: …) (code: …)
```
Plus cross-cutting requirements from `spec/08-non-functional.md` prefixed `R-NFR-NN`.

## `ROADMAP.md`

GSD roadmap format:
```
# Roadmap

## Milestone: v0.1 — Reimplementation

### Phase 01 — Foundation
**Goal:** Establish stack, auth, base layout.
**Covers requirements:** R-01, R-NFR-01, R-NFR-02
**Depends on:** (none)
**Success:** user can sign in, land on a home route.

### Phase 02 — <feature area>
**Goal:** …
**Covers requirements:** R-03, R-04, R-05
**Depends on:** Phase 01
**Success:** …
```

Rules:
- Every requirement from `REQUIREMENTS.md` appears under exactly one phase's "Covers requirements".
- Phase ordering respects data-model dependencies (can't build reporting before sessions exist).
- Each phase has a single observable goal, not a list of tasks.

## `phases/NN-slug/PLAN.md` (only when breakdown warranted)

Emit PLAN.md files **only** when a phase has >3 tasks or non-trivial dependencies. Small phases can stay as the ROADMAP entry alone.

PLAN.md format:
```
# Phase NN — <name>

## Goal
<copied from ROADMAP>

## Tasks
### T-NN.01 — <task>
- **Depends on:** (none | T-NN.00)
- **Acceptance:** <observable>
- **Wave:** 1

### T-NN.02 — <task>
- **Depends on:** T-NN.01
- **Acceptance:** <observable>
- **Wave:** 2

## Verification
- [ ] R-XX acceptance criterion met
- [ ] R-YY acceptance criterion met
```

Waves = independent tasks in the same wave can run in parallel.

## How

- Don't invent requirements. Every R-NN must trace to a feature/NFR in `/spec/`.
- Don't specify the new stack. The reimplementer picks React/Next/Flutter/whatever.
- Tasks in PLAN.md should be stack-agnostic ("implement session CRUD endpoints and UI") — let the executor choose concrete tech.
- For multi-milestone projects, emit `milestones/MN-slug/ROADMAP.md` and keep top-level `ROADMAP.md` as the milestone index.
