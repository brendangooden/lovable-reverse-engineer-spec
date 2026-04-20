---
name: lovable-reverse-engineer-spec
description: Reverse-engineer a Lovable.dev project (GitHub repo + exported chat history) into a structured, stack-agnostic specification plus a GSD-ready planning tree. Use when the user asks to "extract a spec from a Lovable app", "document a Lovable project", "rebuild a Lovable prototype on a different stack", or provides a local path to a Lovable repo and a chat-history export and wants a re-implementable spec.
---

# Lovable Reverse-Engineer Spec

Turn a Lovable.dev project into documentation that another AI coding agent can rebuild from — on any stack.

## Inputs (local paths only)

- `--repo <path>` — clone of the Lovable GitHub repo.
- `--chat <path>` — export produced by [`loveable-chat-history-capture`](https://github.com/brendangooden/loveable-chat-history-capture) (contains `raw/*.json`, `edits/*.json`, `attachments/`, `timeline.md`, `index.json`).
- `--teams-chat <path>` *(optional)* — path to a Teams/Slack/email transcript (`CHAT.md` or similar) where stakeholders discussed the project. Adds strategic, cross-app, and external context that's missing from Lovable chat and code.
- `--out <path>` — where to write the spec (defaults to `<repo>/.reverse-engineered/`).

If args missing, ask the user before proceeding. Never guess paths.

## Output (produced in `<out>/`)

```
spec/                               # agent-agnostic, stack-free
  00-overview.md
  01-domain-model.md
  02-features.md
  03-user-flows.md
  04-ui-design.md
  05-data-model.md
  06-integrations.md
  07-business-logic.md
  08-non-functional.md
  09-decisions.md
  10-evolution.md
  11-production-adjustments.md      # POC shortcuts flagged for rework
  implementation-hints/             # optional Lovable-specific notes
.planning/                          # GSD-compatible
  PROJECT.md
  REQUIREMENTS.md
  ROADMAP.md
  phases/NN-slug/PLAN.md            # only when project warrants breakdown
REVERSE-ENGINEER-REPORT.md          # coverage matrix, confidence, TBDs
```

## Orchestration (what you do when invoked)

Execute these steps in order. Each `prompts/NN-*.md` file is a self-contained instruction block — load it when you reach that step.

1. **Preflight** — follow `prompts/01-survey-repo.md` preflight block. Detect Lovable fingerprints (`lovable-tagger` in `package.json` devDependencies, `componentTagger()` call in `vite.config.ts`, `.lovable/` directory, auto-generated header in `src/integrations/supabase/client.ts`). If none match, stop and explain. Validate chat export has `index.json` and `timeline.md`.

2. **Parallel extraction** — fan out subagents (general-purpose or Explore) running these prompts concurrently. Each writes structured JSON under `<out>/.scratch/`:
   - `prompts/01-survey-repo.md` → `.scratch/repo-inventory.json`
   - `prompts/02-mine-chat.md` → `.scratch/chat-insights.json`
   - `prompts/03-extract-data-model.md` → `.scratch/data-model.json`
   - `prompts/04-extract-business-logic.md` → `.scratch/business-logic.json`
   - `prompts/05-extract-ui.md` → `.scratch/ui-inventory.json`
   - `prompts/11-mine-teams-chat.md` → `.scratch/teams-insights.json` *(skip if `--teams-chat` not provided)*

3. **NFR elicitation** — run `prompts/08-elicit-nfrs.md`. Ask the user for audience size, concurrency, data sensitivity, compliance regime (GDPR/HIPAA/SOC2/none), SLA targets, perf budgets, hosting constraints, budget ceiling. Record to `.scratch/nfrs.json`. Use AskUserQuestion in Claude Code; in AGENTS.md hosts use inline Q&A.

4. **POC-vs-production review** — run `prompts/10-poc-review.md`. Ask the user which POC shortcuts to carry forward and which to adjust for production (auth, validation, logging, observability, rate limits, multi-tenancy, testing, etc.). Auto-surface concerning patterns from extraction outputs (CORS wildcards, PII in logs, missing FKs, god components, etc.) and request per-item decisions. Record to `.scratch/poc-review.json`. **Critical:** a Lovable POC is ground truth for *what was built*, not necessarily *what should ship*.

5. **Sizing** — compute complexity score = `routes + tables + edge_fns + distinct_feature_areas`. If ≤ 12 emit a single-milestone roadmap with one phase per feature area; if > 12 break into milestones aligned with feature boundaries (auth, data capture, reporting, admin, etc.).

6. **Synthesize** — run `prompts/06-synthesize-spec.md` then `prompts/07-synthesize-gsd.md`. Every claim must cite evidence: `(chat: <message_id>)` or `(code: <path>:<line>)`. No unsourced statements. Production adjustments land in `spec/11-production-adjustments.md` and tagged ADRs in `09-decisions.md`.

7. **Verify** — run `prompts/09-verify-coverage.md`. Produce `REVERSE-ENGINEER-REPORT.md` with a coverage matrix that proves every route, every DB table, every edge function, and every documented Lovable plan appears in at least one spec document. Flag low-confidence sections and every TBD.

## Hard rules

- **Evidence-linked only** — no claim without a chat id or `path:line` citation.
- **Stack-agnostic in `spec/`** — capture WHAT the product is and does. Move Lovable-specific "HOW" details (React, shadcn, Supabase) to `spec/implementation-hints/`.
- **Never invent NFRs** — if the user skips a question, write `[TBD — awaiting stakeholder input]` in `08-non-functional.md`.
- **Treat chat as primary source of intent** — code shows what was built; chat shows what was *meant*. Conflicts go in `09-decisions.md` with both sides.
- **Preserve failed attempts** — dead ends in chat history are valuable ADR context. Capture them in `10-evolution.md`.
- **Never modify the input repo or chat export.** Only write under `<out>/`.

## When to break work across multiple GSD phases

- Small POC (≤12 complexity): 1 milestone, 1 phase per feature area.
- Medium (13-30): 1 milestone, multiple phases; phase boundaries = feature area.
- Large (>30): multiple milestones aligned with user-facing product surfaces.

Follow GSD roadmap conventions from [`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done). Each phase PLAN.md lists tasks with Depends-on fields and acceptance criteria derived from `spec/02-features.md`.

## Failure modes to watch for

- **Not a Lovable repo** — preflight fails; stop and recommend alternative skills.
- **Chat export stale vs repo HEAD** — warn but proceed; record delta in report.
- **Sensitive content in chat** — refuse to include API keys, tokens, or passwords that appear in messages. Redact and flag.
- **Subagent returns vague output** — re-prompt with narrower scope; do not paper over gaps.
