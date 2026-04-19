# Lovable Reverse-Engineer Spec (AGENTS.md)

This file mirrors `SKILL.md` without the Claude-specific frontmatter. Codex, Cursor, Aider, Cline, and any AGENTS.md-aware agent should load this file.

## Purpose

Turn a Lovable.dev project (GitHub repo + exported chat history) into a structured, stack-agnostic specification plus a GSD-ready planning tree. The output lets any agent on any stack rebuild the product.

## Inputs (local paths only)

- `--repo <path>` — clone of the Lovable GitHub repo.
- `--chat <path>` — export from [`loveable-chat-history-capture`](https://github.com/brendangooden/loveable-chat-history-capture) (contains `raw/*.json`, `edits/*.json`, `attachments/`, `timeline.md`, `index.json`).
- `--out <path>` — where to write the spec (defaults to `<repo>/.reverse-engineered/`).

If args are missing, ask the user. Never guess.

## Output (produced in `<out>/`)

```
spec/
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
  implementation-hints/
.planning/
  PROJECT.md
  REQUIREMENTS.md
  ROADMAP.md
  phases/NN-slug/PLAN.md
REVERSE-ENGINEER-REPORT.md
```

## Orchestration

Load and follow `prompts/NN-*.md` in order:

1. **Preflight** (`prompts/01-survey-repo.md` preflight) — detect Lovable fingerprints: `lovable-tagger` dep, `componentTagger()` in vite config, `.lovable/` dir, auto-gen header in `src/integrations/supabase/client.ts`. Stop if none match.
2. **Parallel extraction** — run `01-survey-repo.md`, `02-mine-chat.md`, `03-extract-data-model.md`, `04-extract-business-logic.md`, `05-extract-ui.md`. Each writes `.scratch/*.json`.
3. **NFR elicitation** — `prompts/08-elicit-nfrs.md`. Ask user about audience size, concurrency, data sensitivity, compliance, SLAs, perf, hosting, budget.
4. **Sizing** — complexity = `routes + tables + edge_fns + feature_areas`. ≤12 → single milestone; >12 → multiple.
5. **Synthesize** — `prompts/06-synthesize-spec.md` then `prompts/07-synthesize-gsd.md`. Every claim cites `(chat: <id>)` or `(code: <path>:<line>)`.
6. **Verify** — `prompts/09-verify-coverage.md` → `REVERSE-ENGINEER-REPORT.md`.

## Hard rules

- Evidence-linked statements only.
- `spec/` is stack-free; Lovable-specific choices go in `spec/implementation-hints/`.
- TBD for missing NFRs — never invent.
- Redact secrets found in chat.
- Never modify input repo or chat export.

See `SKILL.md` for full detail.
