# Prompt 04 — Extract Business Logic

**Input:** `--repo <path>` (edge functions, AI prompt tables, background jobs, webhooks)
**Output:** `<out>/.scratch/business-logic.json`

## Purpose

Capture every decision that is NOT "render a component" or "CRUD a row". These are the parts a reimplementer is most likely to miss: AI prompts, workflow rules, external integrations, scheduled jobs.

## Output shape

```json
{
  "workflows": [
    {
      "id": "wf-generate-five-point-report",
      "trigger": "user clicks 'Generate Report' on /session/:id",
      "steps": [
        { "step": 1, "action": "validate prerequisites (discovery doc OR IM doc)", "evidence": ["supabase/functions/generate-five-point-n8n/index.ts:42"] },
        { "step": 2, "action": "upload doc to n8n webhook", "evidence": ["...:88"] },
        { "step": 3, "action": "persist returned report to sessions.generated_five_point_report", "evidence": ["...:156"] }
      ],
      "failure_modes": [
        { "condition": "no discovery doc and no IM doc", "behavior": "return 400 with message X" }
      ],
      "external_deps": ["n8n workflow URL", "Supabase storage"],
      "idempotency": "none|by-session-id"
    }
  ],
  "ai_prompts": [
    {
      "id": "prompt-discovery-summary",
      "source": "ai_prompts table (ai_prompt_versions)",
      "purpose": "Summarise a discovery call transcript",
      "model": "gpt-4o|claude-sonnet-4|...",
      "system_prompt_excerpt": "quoted prose, ≤500 chars",
      "temperature": 0.3,
      "reasoning_effort": "medium",
      "inputs": ["transcript text", "IM document text"],
      "outputs": "structured summary",
      "evidence": ["supabase/migrations/...:..", "chat: aimsg_01xxx"]
    }
  ],
  "integrations": [
    {
      "name": "n8n",
      "direction": "outbound",
      "auth": "webhook-signature|api-key|none",
      "purpose": "orchestrates five-point report generation",
      "endpoints_used": ["POST https://n8n.example/webhook/..."],
      "evidence": ["..."]
    }
  ],
  "scheduled_jobs": [],
  "webhooks_received": [],
  "feature_flags": [
    { "name": "enable_n8n_reports", "default": false, "evidence": ["src/hooks/useFeatureFlag.ts:..."] }
  ],
  "activity_logging": {
    "present": true,
    "captured_events": ["session_created", "report_generated"],
    "evidence": ["src/lib/activity-log.ts:..."]
  }
}
```

## How

- For each `supabase/functions/*/index.ts`: summarise the request → response flow in ≤5 steps.
- Read migration files that insert into `ai_prompts` / `ai_prompt_versions` to get seeded prompts.
- Check chat-insights for AI-prompt iteration history (which prompt versions changed, why).
- Capture external URLs (n8n, Stripe, etc.) but **redact any embedded tokens**.
- If a workflow's intent is unclear from code alone, cross-reference with chat intents.
- Do not transcribe full edge function bodies — just the flow.
