# Prompt 08 — Elicit NFRs

**Output:** `<out>/.scratch/nfrs.json`

## Purpose

Non-functional requirements rarely live in code or chat. Ask the user. Never invent.

## How to ask

**In Claude Code:** use `AskUserQuestion` with the checklist below, one question at a time or grouped (4 per call max).

**In Codex / Copilot / plain AGENTS.md:** prompt the user inline with a numbered checklist and wait for a response.

## Checklist (ask ALL of these)

Before asking, skim `chat-insights.json` for any hints (e.g., user mentioned "internal tool", "HIPAA", "10k users") and **pre-fill** those with `[inferred: …, please confirm]`.

1. **Audience size**
   - Expected total users in first 12 months? (<100 / 100-1k / 1k-10k / 10k-100k / 100k+ / unsure)
   - Geographic reach? (single region / multi-region / global)
   - Language support? (english-only / specific locales / i18n-from-day-1)

2. **Concurrency**
   - Peak concurrent users? (estimate; "unsure" is fine)
   - Peak concurrent operations of expensive workflows (e.g., AI report gen)?

3. **Data sensitivity & compliance**
   - Data classification: (public / internal / confidential / restricted / regulated)
   - Applicable regimes: (none / GDPR / HIPAA / SOC2 / ISO27001 / PCI / other)
   - Data residency constraints? (any region / EU-only / specific country)
   - Retention requirements?

4. **Availability & SLA**
   - Target uptime: (best-effort / 99% / 99.5% / 99.9% / 99.99%)
   - Acceptable planned maintenance windows?
   - Disaster recovery expectations (RPO/RTO)?

5. **Performance targets**
   - Page-load budget (p95)? (any / <2s / <1s / <500ms)
   - API latency budget (p95)? (any / <500ms / <200ms / <100ms)
   - AI/workflow latency tolerance? (seconds / tens-of-seconds / minutes OK / async-ok)

6. **Security model**
   - Auth requirements: (email-password / SSO / MFA required / passwordless)
   - Role model: (single role / RBAC / ABAC / custom)
   - Audit log requirements?
   - Secret-handling constraints?

7. **Hosting & budget**
   - Hosting preferences/constraints? (any cloud / AWS-only / Azure-only / on-prem / air-gapped)
   - Monthly infra budget ceiling? (none / <$100 / <$500 / <$5k / <$50k / enterprise)
   - Self-hostable requirement?

8. **Observability**
   - Required: logs / metrics / traces / error-tracking / product-analytics? (pick all)
   - Alerting needed? channels?

9. **Accessibility**
   - Target WCAG level: (none-stated / A / AA / AAA)
   - Screen-reader support required?

10. **Other constraints**
    - Offline support?
    - Mobile-native requirement (not just responsive web)?
    - Integration must-haves (SSO provider, payment, CRM, etc.)?
    - Anything else that would make this project fail if missed?

## Output

Write `.scratch/nfrs.json`:
```json
{
  "audience": { "users_12mo": "1k-10k", "geo": "global", "languages": ["en"], "inferred": false },
  "concurrency": { ... },
  "compliance": { "classification": "confidential", "regimes": ["SOC2"], "residency": "any", "retention": "[TBD]" },
  "availability": { ... },
  "performance": { ... },
  "security": { ... },
  "hosting": { ... },
  "observability": { ... },
  "accessibility": { ... },
  "other": { ... },
  "skipped_questions": ["retention"]
}
```

Any question the user skips → leave value as `"[TBD]"` and add the key to `skipped_questions`. Do **not** fabricate defaults.
