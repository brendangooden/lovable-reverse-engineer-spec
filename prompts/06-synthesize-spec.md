# Prompt 06 — Synthesize `/spec/`

**Inputs:** `.scratch/repo-inventory.json`, `.scratch/chat-insights.json`, `.scratch/data-model.json`, `.scratch/business-logic.json`, `.scratch/ui-inventory.json`, `.scratch/nfrs.json`
**Output:** `<out>/spec/*.md`

## Hard rules

- **Every claim must cite evidence.** Use `(chat: umsg_01xxx)` for chat ids and `(code: src/path/file.tsx:42)` for code. No unsourced statements except the one-liner in `00-overview.md`.
- **Stack-free** — `spec/*.md` must not name React, Supabase, shadcn, Tailwind, Vite, Deno. Move all those to `spec/implementation-hints/`.
- **Prose budget** — overview + features + flows are the high-value docs. Keep the others tight.

## Docs to produce

### `00-overview.md`
One-liner + problem statement + target users + top-3 outcomes. Mine from `chat-insights.intents` + `user_personas`. ≤ 400 words.

### `01-domain-model.md`
Entities and relationships in domain language (not table names). Derive from `data-model.json.tables` but rename with business vocabulary. Include an entity diagram in Mermaid.

### `02-features.md`
For each feature (mine from `chat-insights.intents` where `outcome=shipped`):
```
## F-NN — <feature name>
**Users:** <persona>
**Intent:** <why>
**Behaviour:**
- Given …
- When …
- Then …
**Acceptance criteria:**
- [ ] observable outcome 1
- [ ] observable outcome 2
**Evidence:** (chat: …) (code: …)
```
Features are ordered by dependency, not chronology.

### `03-user-flows.md`
Per route in `ui-inventory.pages`: entry → actions → outcomes → exits. Include state transitions. Mermaid sequence diagrams for multi-step flows (report generation, upload, auth).

### `04-ui-design.md`
Design tokens, layout primitives, interaction patterns. Reference design tokens abstractly (e.g., "primary accent colour") — put exact HSL values in `implementation-hints/frontend.md`.

### `05-data-model.md`
Tables/entities from `data-model.json` but with:
- Domain name (not DB name)
- Columns with business meaning
- Authorization rules in prose (not SQL RLS)
- Relationships
Strip Supabase-specific defaults into `portability_notes`.

### `06-integrations.md`
External systems. Each integration = direction, auth model, data exchanged, failure behaviour, SLA assumption. No API keys or URLs with tokens.

### `07-business-logic.md`
From `business-logic.workflows` and `ai_prompts`:
- Workflow name
- Trigger
- Preconditions
- Steps (language-neutral)
- Postconditions
- Failure modes

For AI prompts: purpose, inputs, outputs, model tier needed, temperature/reasoning characteristics — **not** the exact prompt text (that's implementation).

### `08-non-functional.md`
From `nfrs.json`. Sections: Audience & Scale, Performance, Availability, Security & Privacy, Compliance, Observability, Cost, Accessibility. Each section has bullets with targets. `[TBD]` where user skipped.

### `09-decisions.md`
From `chat-insights.decisions` + `chat-insights.business_rules`. ADR format:
```
## ADR-NN — <title>
**Status:** accepted|superseded-by-ADR-MM
**Context:** …
**Decision:** …
**Consequences:** …
**Evidence:** (chat: …)
```

### `10-evolution.md`
Chronological narrative of pivots + abandoned attempts from `chat-insights.pivots` + `chat-insights.abandoned`. Each entry cites message ids.

### `implementation-hints/` (optional, only when useful)
- `frontend.md` — Lovable used React + shadcn + Tailwind with these tokens. A rebuilder on Flutter should …
- `backend.md` — Lovable used Supabase (auth + Postgres + edge functions + storage). On a non-Supabase stack you need …
- `testing.md` — patterns observed.

## How

- Write each file independently; don't cross-contaminate sections.
- Use headings that are grep-friendly (`## F-01 —`, `## ADR-03 —`).
- Keep prose sentences short. Sacrifice grammar for precision.
- When data conflicts (chat says X, code says Y), document both and flag in `09-decisions.md`.
