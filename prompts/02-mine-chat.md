# Prompt 02 — Mine Chat History

**Input:** `--chat <path>` (output of `loveable-chat-history-capture`)
**Output:** `<out>/.scratch/chat-insights.json`

## Source structure (what you're reading)

- `<chat>/index.json` — manifest; map of message id → updateTime/createTime.
- `<chat>/raw/<id>.json` — one file per message. Schema in `schemas/chat-message.schema.json`.
- `<chat>/edits/<edit_id>.json` — file-edit metadata (path, action, commit SHA).
- `<chat>/attachments/` — binary uploads.
- `<chat>/timeline.md` — human-readable transcript.

Message `fields.role` = `user` or `ai`. Relevant fields per message:
- `content` — prose
- `images` — attachments
- `edit_id` — link to a file edit
- `current_page` — page user was on when sending
- `tool_diffs` — AI tool calls (file changes, commands)
- `cost_credits` — proxies "effort"

## What to extract

Produce `chat-insights.json`:

```json
{
  "meta": {
    "first_message": "ISO timestamp",
    "last_message": "ISO timestamp",
    "message_count": 412,
    "user_message_count": 180,
    "ai_message_count": 232,
    "total_credits": 640,
    "unique_pages_visited": ["/dashboard", "/session/:id"]
  },
  "intents": [
    {
      "id": "intent-001",
      "kind": "feature_request|bug_fix|refactor|styling|infra|question",
      "summary": "Add IM document fallback for advisor feedback",
      "first_mention": "umsg_01xxx",
      "related_messages": ["umsg_01xxx", "aimsg_01xxx"],
      "related_edits": ["edit_01xxx"],
      "outcome": "shipped|abandoned|superseded",
      "page_context": "/session/:id"
    }
  ],
  "decisions": [
    {
      "id": "adr-001",
      "title": "Use Supabase edge functions instead of a separate backend",
      "rationale_excerpt": "quoted prose from message",
      "evidence_message_ids": ["umsg_01xxx"],
      "date": "ISO"
    }
  ],
  "pivots": [
    {
      "from": "initial approach",
      "to": "revised approach",
      "trigger_message_id": "umsg_01xxx",
      "reason": "quoted user prose"
    }
  ],
  "abandoned": [
    {
      "intent_summary": "original attempt at X",
      "why_dropped": "…",
      "message_ids": [...]
    }
  ],
  "business_rules": [
    {
      "rule": "Five-point report requires a discovery document or IM document fallback",
      "evidence_message_ids": [...],
      "evidence_code_paths": []
    }
  ],
  "user_personas": [
    {
      "role": "consultant",
      "inferred_from": ["umsg_01xxx"],
      "goals": ["manage client sessions", "generate briefings"]
    }
  ],
  "constraints": [
    {
      "constraint": "Must work for offline usage / must support PDF upload / etc",
      "evidence_message_ids": [...]
    }
  ]
}
```

## How

- Don't load every `raw/*.json` into context. Stream: list filenames, batch-read 20-50 at a time, extract structured claims, discard.
- Use `timeline.md` for fast sequencing; use `raw/*.json` when you need `tool_diffs`, `current_page`, or `edit_id` correlation.
- Correlate edits → messages via `edit_id` to enrich intents with actual code changes.
- When a user message contains a declarative statement ("We need to…", "Must never…", "Always…") treat as a business_rule or constraint candidate.
- Collapse repeated back-and-forth about the same change into a single intent.
- Quote user prose verbatim for rationale (short excerpts, ≤200 chars each).
- Flag any API keys / tokens / passwords in chat → redact to `[REDACTED]` in outputs.

Scope: keep `chat-insights.json` under ~500KB. Truncate `related_messages` arrays to the 5 most informative ids; put total count in `_full_count`.
