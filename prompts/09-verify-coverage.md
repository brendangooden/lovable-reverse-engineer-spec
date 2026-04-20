# Prompt 09 — Verify Coverage

**Inputs:** all of `<out>/spec/`, `<out>/.planning/`, and `.scratch/*.json`
**Output:** `<out>/REVERSE-ENGINEER-REPORT.md`

## Purpose

Prove the spec is complete and every claim is evidenced. This is the skill's quality gate.

## Checks to run

### 1. Coverage matrix
For each of these inventory items, confirm it appears in at least one spec doc:

| Item type | Source | Must appear in |
|---|---|---|
| Route | `repo-inventory.routes[]` | `spec/03-user-flows.md` |
| Table | `data-model.tables[]` | `spec/05-data-model.md` + referenced in ≥1 feature in `spec/02-features.md` |
| Edge function | `repo-inventory.edge_functions[]` | `spec/07-business-logic.md` |
| Feature component | `ui-inventory.components[role=feature]` | `spec/03-user-flows.md` or `spec/02-features.md` |
| AI prompt | `business-logic.ai_prompts[]` | `spec/07-business-logic.md` |
| Chat ADR | `chat-insights.decisions[]` | `spec/09-decisions.md` |
| Shipped intent | `chat-insights.intents[outcome=shipped]` | `spec/02-features.md` |
| `.lovable/plan.md` history | `repo-inventory.lovable_artifacts.plan_md_history` | `spec/09-decisions.md` or `10-evolution.md` |
| Requirement | `.planning/REQUIREMENTS.md R-NN` | exactly one phase in `.planning/ROADMAP.md` |
| POC review item | `poc-review.json.review_items[]` | `spec/11-production-adjustments.md` + (if `adjust`) a tagged task in `.planning/phases/*/PLAN.md` or `.planning/BACKLOG.md` |
| POC tradeoff | `poc-review.json.tradeoffs[]` | `spec/11-production-adjustments.md` + (if `adjust`) shaped into `spec/02-features.md` acceptance criteria or `spec/08-non-functional.md` |

### 2. Evidence check
Grep each `spec/*.md` for unsourced claims:
- Every bullet or sentence stating a fact should contain `(chat: …)`, `(code: …)`, or `(nfr: …)`.
- Exceptions: `spec/00-overview.md` first paragraph; section headers; Mermaid diagrams.

### 3. Stack-agnostic check
Grep `spec/*.md` (excluding `implementation-hints/`) for banned terms:
`React`, `Supabase`, `shadcn`, `Tailwind`, `Vite`, `Deno`, `Edge Function`, `Postgres`, `Radix`, `Zod`, `TanStack`.
Any hit → flag as violation; move content to `implementation-hints/`.

### 4. Secret leak check
Grep all outputs for:
- Strings matching `[A-Za-z0-9]{32,}` that aren't commit SHAs
- Keywords `api_key`, `secret`, `password`, `token`, `bearer`
Flag and recommend redaction.

### 5. NFR completeness
Check `nfrs.json` for `skipped_questions[]`. Each skipped item should appear in `spec/08-non-functional.md` as `[TBD]`.

### 6. Chat-only vs code-only divergence
- Claims in chat without code backing → list as "aspirational/unbuilt".
- Code without chat rationale → list as "undocumented decisions".

### 7. POC-review completeness
- Every `review_items[]` in `poc-review.json` appears in `spec/11-production-adjustments.md`.
- Every `review_items[]` with `decision: adjust` is either a tagged task in `.planning/phases/*/PLAN.md` OR in a "Production Hardening" phase OR (if deferred) in `.planning/BACKLOG.md`.
- Every `decision: out-of-scope-v1` item is in `.planning/BACKLOG.md`.
- Every `decision: needs-discussion` item appears in the report's action items list.
- ADRs in `spec/09-decisions.md` corresponding to flagged POC shortcuts have `Status: POC-shortcut → production-recommend-RI-NN`.

## Output format

```markdown
# Reverse-Engineer Report

**Generated:** <ISO timestamp>
**Source repo:** <path>
**Source chat:** <path> (N messages)
**Complexity score:** <int>
**Milestone plan:** single | multi

## Coverage matrix
| Item | Count | Covered | Missing |
|---|---|---|---|
| Routes | 4 | 4 | — |
| Tables | 7 | 7 | — |
| Edge functions | 8 | 8 | — |
| Feature components | 15 | 14 | `PromptVersionHistory` |
| AI prompts | 3 | 3 | — |
| Chat ADRs | 12 | 11 | adr-009 |
| Shipped intents | 47 | 46 | intent-038 |
| Requirements → Phases | 23 | 23 | — |
| POC review items | 14 | 14 | — |
| POC tradeoffs | 9 | 9 | — |

**Coverage: 97.8%**

## Evidence-citation check
- Total claims scanned: 312
- Unsourced: 4
- Violations:
  - `spec/02-features.md:88` — "Reports auto-refresh every 30s" no citation.

## Stack-agnostic check
- Banned-term violations: 2
  - `spec/07-business-logic.md:45` — mentions "Supabase storage" — move to implementation-hints.

## Secret leak check
- 0 potential secrets detected.

## NFR completeness
- 9/10 NFR sections answered.
- Skipped: `retention` — marked TBD in spec/08-non-functional.md.

## Chat vs code divergence
### Aspirational (in chat, not in code)
- "Export to PDF" mentioned in umsg_01xxx but no implementation.
### Undocumented (in code, not in chat)
- Table `audit_log` has no chat rationale — added silently in migration 20240201.

## Confidence scores
| Section | Confidence | Why |
|---|---|---|
| 00-overview | High | 18 chat messages back the one-liner |
| 01-domain-model | High | Schema + chat agree |
| 02-features | High | All mined from shipped intents |
| 03-user-flows | Medium | 2 flows have only code evidence |
| 04-ui-design | Medium | No screenshots for 3 pages |
| 05-data-model | High | — |
| 06-integrations | Medium | n8n workflow URL is opaque |
| 07-business-logic | High | — |
| 08-non-functional | Medium | 1 TBD |
| 09-decisions | High | — |
| 10-evolution | Medium | Some pivots lack final outcome |
| 11-production-adjustments | High | — |

## Action items
1. Fill `[TBD: retention]` in spec/08-non-functional.md.
2. Add citation for reports auto-refresh claim.
3. Review chat-only "Export to PDF" — scope into future milestone?
```

## How

- Keep the report deterministic; if you ran twice on the same input, output should match.
- Fail loudly: if coverage < 90% or any secret leak suspected, put a **BLOCKER** header at the top.
