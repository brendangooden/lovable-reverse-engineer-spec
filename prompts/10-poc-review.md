# Prompt 10 — POC-vs-Production Review

**Inputs:** `.scratch/repo-inventory.json`, `.scratch/chat-insights.json`, `.scratch/business-logic.json`, `.scratch/data-model.json`, `.scratch/ui-inventory.json`
**Output:** `<out>/.scratch/poc-review.json`
**When to run:** after all 5 extractions, after NFR elicitation (`08`), **before** synthesis (`06`, `07`).

## Purpose

Lovable prototypes routinely ship shortcuts the user wants stripped in the production rebuild. The raw spec would copy those shortcuts as-is. This step asks the user what to keep, adjust, or drop.

Two parts run in sequence:

## Part A — Upfront trade-off checklist

Ask the user (Claude Code → `AskUserQuestion`; other hosts → inline Q&A) what was **deliberately** skipped/simplified during the POC that should NOT carry forward.

For each area below, offer 3 choices: **copy as-is** / **adjust for production** / **out of scope for v1**. Show a one-line inferred signal when possible.

1. **Authentication** — e.g. current: email/password via managed auth. Target may be SSO/Entra/OIDC.
2. **Authorisation model** — role granularity, cross-tenant role flex, deny-by-default, row-level isolation.
3. **Input validation & error handling** — are empty strings / missing fields rejected everywhere, or does POC trust happy path?
4. **Audit & activity logging** — PII in log bodies, retention, tamper resistance.
5. **Observability** — logs/metrics/tracing present? structured? correlated?
6. **Rate limiting & abuse controls** — absent in POC? required for target audience?
7. **Multi-tenancy / data isolation** — single-tenant shortcuts baked in?
8. **Secret handling** — hard-coded URLs/keys? env var hygiene? Key Vault integration target?
9. **Testing** — unit / integration / E2E coverage to carry forward or rewrite?
10. **Accessibility & i18n** — hard-coded strings, keyboard nav gaps?
11. **Error UX** — are failures silent? toasts only? retry surfaces?
12. **Performance** — N+1 queries, client-side polling loops, missing indexes?
13. **Data retention & lifecycle** — soft delete? hard delete? GDPR erasure?
14. **CI/CD & deployment** — POC had manual deploys; target pipeline?

Ask only the areas where the extraction surfaced signal or where NFRs suggest it matters. Skip obviously-fine areas.

## Part B — Auto-surfaced review items

Scan extraction outputs for concerning patterns and present each back to the user for a decision. Patterns to look for:

**From `business-logic.json`:**
- CORS wildcards on privileged endpoints
- PII written to logs or activity tables
- Silent empty prompts / missing-data fallbacks
- Non-transactional multi-row writes (user create+role, delete+cascade)
- Hard-coded external URLs / webhook hosts
- Missing rate limits on expensive workflows
- Error strings fed back into AI prompts

**From `data-model.json`:**
- Missing foreign keys (constraints declared in app only)
- Role migrations that left legacy policy names
- App-layer-enforced invariants that should be DB constraints
- Tables with RLS disabled or overly permissive policies
- Realtime publications that could leak sensitive columns

**From `repo-inventory.json` / `ui-inventory.json`:**
- God components (>400 LOC)
- Feature flags that affect security-relevant behaviour
- Hard-coded roles or tenant ids
- Client-side polling loops replacing proper async patterns
- Missing test coverage

**From `chat-insights.json`:**
- Decisions marked "ship it, fix later"
- Abandoned security or scale fixes
- User prose flagging a shortcut ("for now we'll…", "good enough for the demo")

For each item: present a summary + evidence, offer **copy as-is / adjust / out-of-scope for v1 / needs discussion**. Capture the user's choice and any notes.

## Output format

```json
{
  "tradeoffs": {
    "authentication": {
      "decision": "adjust",
      "current_state": "Managed auth email+password with admin-invited users",
      "target_state": "SSO-only via Entra, multi-tenant, roles from security groups",
      "rationale": "from NFRs + user input",
      "notes": "cross-tenant role flex needs design"
    },
    "rate_limiting": {
      "decision": "out-of-scope-v1",
      "notes": "acceptable for 100-1k internal users; revisit when client-facing"
    }
  },
  "review_items": [
    {
      "id": "RI-001",
      "category": "security",
      "title": "CORS wildcard on privileged service-role endpoints",
      "evidence": ["code: supabase/functions/create-user/index.ts:12"],
      "severity": "high",
      "decision": "adjust",
      "adjustment": "Lock CORS to known Entra-auth frontend origins; require authenticated Bearer token not service-role key",
      "user_notes": ""
    },
    {
      "id": "RI-002",
      "category": "privacy",
      "title": "Full chat message body persisted to activity log metadata",
      "evidence": ["code: src/lib/activity-log.ts:45", "chat: aimsg_01xxx"],
      "severity": "medium",
      "decision": "adjust",
      "adjustment": "Log event type + message id only; keep bodies in dedicated table with retention policy"
    }
  ],
  "unanswered_questions": [
    "How long should activity log bodies be retained?"
  ],
  "out_of_scope_for_v1": ["RI-005", "RI-008"]
}
```

## Consumption by downstream steps

`06-synthesize-spec.md` will:
- Produce `spec/11-production-adjustments.md` listing every `review_items[]` entry with decision + adjustment prescription.
- Tag related ADRs in `09-decisions.md` with `Status: POC-shortcut → production-recommend-<X>`.
- Use `tradeoffs[].target_state` to shape `spec/02-features.md` acceptance criteria and `spec/08-non-functional.md`.

`07-synthesize-gsd.md` will:
- Insert a late "Production Hardening" phase bundling all `adjust`-decision review items whose scope doesn't fit an earlier phase.
- Inline `adjust` items that belong to a specific feature area into that phase's PLAN.md as tasks tagged `[production-hardening]`.
- Drop `out-of-scope-v1` items into a separate `.planning/BACKLOG.md` with rationale.

## How

- Be concise. Don't re-explain a pattern — show the evidence and ask the decision.
- Batch related review items (e.g. all CORS issues) into one question with a multi-select.
- Skip questions where the extraction shows no signal — don't ask about rate limits if there's only one endpoint.
- For every `adjust` decision, draft a one-line `adjustment` prescription. The user can edit; don't leave it blank.
- If the user skips an area or says "not sure", record `decision: "needs-discussion"` and add to `unanswered_questions[]` — these surface as action items in the final report.
