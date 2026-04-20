# Prompt 11 — Mine External Chat (Teams / Slack / email threads)

**Input:** `--teams-chat <path-to-CHAT.md>` (optional flag)
**Output:** `<out>/.scratch/teams-insights.json`
**When to run:** in parallel with the other 5 extractors. Skip entirely if no `--teams-chat` arg provided.

## Purpose

Lovable chat captures the developer–AI loop. The repo captures what was built. Neither captures **stakeholder-level signal** — strategic pivots, cross-app context, client names, who owns what, external deadlines, risk conversations. External chat transcripts (Teams, Slack, email) fill that gap.

Critical: external chats almost always span **multiple projects**. You must filter to content relevant to the project under reverse-engineering.

## Source format

`CHAT.md` is a plain-text markdown transcript. Expect:
- Header block with conversation topic, participant list, date range, message count
- Date-grouped sections (`## YYYY-MM-DD`)
- Messages in the form `### <Speaker Name> — <YYYY-MM-DD HH:MM>` followed by prose
- System events (member added, topic set, calls) as italicised single lines
- External links (SharePoint, docs, image URLs)
- No message ids — evidence citation uses `(teams: YYYY-MM-DD HH:MM @speaker)` format

Some exports use different formats (Slack JSON, email .eml, plain text). If the file doesn't match the Teams-markdown format, degrade gracefully: extract what you can by pattern matching, and note the format in `meta.source_format`.

## Extraction strategy

1. **Scan the whole file once** to build a participants list and identify which projects are discussed.
2. **Tag each substantive message** with `project: advisory | other-project | cross-cutting | off-topic` based on context. Only `advisory` + `cross-cutting` land in insights.
3. **Extract structured claims** — batch process, don't accumulate raw text.
4. **Redact** any credentials, tokens, or client-confidential data (prices, contracts). Names of employees + public project names are context, not secrets.

## Output shape

```json
{
  "meta": {
    "source_format": "teams-markdown",
    "source_path": "<path>",
    "conversation_topic": "UBT AI Squad",
    "date_range": ["2026-03-26", "2026-04-20"],
    "message_count_total": 553,
    "message_count_advisory_or_crosscutting": 142,
    "participants": [
      { "name": "Denver Connell", "role_inferred": "Product owner" },
      { "name": "Bryan Tunley", "role_inferred": "Leadership / AI enablement" }
    ],
    "projects_mentioned": ["Advisory Portal", "OBC Events", "Seminar chat"]
  },
  "stakeholder_intents": [
    {
      "id": "si-001",
      "summary": "Replace advisor spreadsheet workflow with guided briefing tool",
      "raised_by": "Denver Connell",
      "evidence": ["teams: 2026-03-27 03:19 @Denver Connell"],
      "project_context": "advisory",
      "priority_signal": "high|medium|low"
    }
  ],
  "strategic_decisions": [
    {
      "id": "sd-001",
      "title": "Pause advisory feature X to prioritise seminar app launch",
      "rationale_excerpt": "short verbatim quote ≤200 chars",
      "decided_by": "Bryan Tunley",
      "evidence": ["teams: 2026-04-05 09:15 @Bryan Tunley"],
      "date": "2026-04-05"
    }
  ],
  "cross_app_context": [
    {
      "note": "Shared AI prompt patterns reused from OBC seminar chat",
      "related_project": "OBC seminar chat",
      "implication": "Backend service layer should be prompt-agnostic",
      "evidence": ["teams: 2026-03-30 14:00 @Denver Connell"]
    }
  ],
  "external_resources": [
    {
      "url": "https://ubt365.sharepoint.com/sites/...",
      "description": "AI Squad project folder with background docs",
      "owner": "Denver Connell",
      "evidence": ["teams: 2026-03-26 23:27 @Denver Connell"]
    }
  ],
  "stakeholders": [
    {
      "name": "Denver Connell",
      "inferred_role": "Project lead / Advisory PM",
      "project_involvement": "advisory + cross-cutting"
    }
  ],
  "risks_and_concerns": [
    {
      "concern": "Volunteer advisor onboarding friction if auth is complex",
      "raised_by": "Brendan Gooden",
      "status": "open|addressed-in-code|deferred",
      "evidence": ["teams: 2026-04-10 11:00 @Brendan Gooden"]
    }
  ],
  "timeline_anchors": [
    {
      "date": "2026-03-26",
      "event": "Teams group 'UBT AI Squad' created",
      "significance": "project kick-off"
    }
  ],
  "unanswered_questions_raised_in_chat": [
    {
      "question": "What is the plan for external volunteer advisor access outside UBT tenancy?",
      "raised_by": "Bryan Tunley",
      "status": "unresolved",
      "evidence": ["teams: 2026-04-15 16:20 @Bryan Tunley"]
    }
  ],
  "off_topic_project_signals": [
    "Mentions of 'OBC Events' project discussed 2026-04-02 — not advisory, skipped"
  ],
  "safety_flags": [
    "No credentials found. Client names mentioned: [REDACTED list in output — n/a]"
  ]
}
```

## Evidence format

Teams/external messages have no stable ids. Use `teams: YYYY-MM-DD HH:MM @<Speaker Name>` as the citation. If the speaker is a system event, use `teams: YYYY-MM-DD HH:MM [system]`.

If the source isn't Teams (e.g. Slack), use `slack:` or `email:` prefix. Keep prefixes consistent for later grep.

## Consumption by synthesis (`06-synthesize-spec.md`)

Teams insights flow into:
- **`00-overview.md`** — stakeholder-level framing and business context (who asked for this, why, what it replaces).
- **`02-features.md`** — scope hints from stakeholder intents; some features will have dual evidence `(chat: …) (teams: …)`.
- **`08-non-functional.md`** — audience & compliance hints (e.g. "external volunteer advisors" raised in Teams but absent from Lovable chat).
- **`09-decisions.md`** — strategic ADRs raised at stakeholder level, not in Lovable chat. Prefix ADR title with `[Stakeholder]` if origin is teams-only.
- **`10-evolution.md`** — timeline anchors + strategic pivots.
- **`11-production-adjustments.md`** — concerns raised in Teams (onboarding friction, compliance worries) that inform trade-off decisions.
- **Potential new section in `06-integrations.md`** — external resource links (SharePoint, shared docs) as inputs the reimplementer will need.

Risks and unanswered questions also feed `REVERSE-ENGINEER-REPORT.md` action items.

## How

- Don't dump the full transcript into insights — extract claims only.
- Cap quotes at ≤200 chars.
- If a message references an image URL with no context, note the timestamp + speaker but don't fabricate what the image showed.
- If filtering is ambiguous, err on the side of `cross-cutting` so reviewers see it.
- Size target: keep the output JSON under 400KB.
