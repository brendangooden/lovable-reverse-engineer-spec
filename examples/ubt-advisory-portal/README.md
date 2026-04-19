# Example — UBT Advisory Portal

A worked example of running this skill against a real Lovable project.

## Prerequisites

Two sibling repos cloned locally:

```
C:\Users\<you>\Source\Repos\
  ├── ubt-advisory-portal-poc          ← the Lovable project
  └── loveable-chat-history-capture    ← the chat exporter
```

## Step 1 — Capture chat history

```bash
cd C:\Users\<you>\Source\Repos\loveable-chat-history-capture
# configure .env with Firebase creds (see that repo's README)
bun run export -- --project ubt-advisory-portal --output ../ubt-advisory-chat-history
```

You should see:

```
ubt-advisory-chat-history/
├── raw/            # ~400 JSON files
├── edits/          # edit metadata
├── attachments/    # images uploaded to Lovable
├── timeline.md
└── index.json
```

## Step 2 — Run the skill

From any Claude Code / Codex / Copilot session with this skill installed:

> Reverse-engineer the Lovable project at `C:\Users\<you>\Source\Repos\ubt-advisory-portal-poc` using the chat export at `C:\Users\<you>\Source\Repos\ubt-advisory-chat-history`. Write output to `./ubt-advisory-spec/`.

### Expected preflight output

```
✔ lovable-tagger found in devDependencies
✔ componentTagger() in vite.config.ts
✔ .lovable/plan.md present
✔ auto-gen header in src/integrations/supabase/client.ts
Lovable fingerprint: 4/4 — proceeding.
```

### Expected extraction summary

```
repo-inventory:     4 routes, 19 feature components, 8 edge functions, 17 migrations
chat-insights:      412 messages, 47 shipped intents, 12 decisions, 5 pivots
data-model:         7 tables, 1 enum, 2 storage buckets
business-logic:     4 workflows, 3 AI prompts, 1 external integration (n8n)
ui-inventory:       design tokens in src/index.css, shadcn/ui, mobile-responsive
complexity score:   7 + 4 + 8 + 4 = 23 → medium (single milestone, multiple phases)
```

### NFR elicitation sample

The skill will pause and ask:

```
[1/10] Audience size — expected total users in first 12 months?
       [inferred from chat: "internal tool for UBT consultants" → <100, please confirm]
```

## Step 3 — Expected output

```
ubt-advisory-spec/
├── spec/
│   ├── 00-overview.md                  (~300 words)
│   ├── 01-domain-model.md              (Session, Document, Report, User, Role entities)
│   ├── 02-features.md                  (F-01..F-15)
│   ├── 03-user-flows.md                (4 routes, 2 sequence diagrams)
│   ├── 04-ui-design.md                 (shadcn-style tokens, abstracted)
│   ├── 05-data-model.md                (7 entities with RLS rules in prose)
│   ├── 06-integrations.md              (Supabase-auth, n8n, OpenAI)
│   ├── 07-business-logic.md            (4 workflows, 3 AI prompts)
│   ├── 08-non-functional.md            (from your elicitation answers)
│   ├── 09-decisions.md                 (~12 ADRs)
│   ├── 10-evolution.md                 (5 pivots documented)
│   └── implementation-hints/
│       ├── frontend.md
│       └── backend.md
├── .planning/
│   ├── PROJECT.md
│   ├── REQUIREMENTS.md                 (~18 R-NN + 9 R-NFR-NN)
│   ├── ROADMAP.md                      (1 milestone, ~5 phases)
│   └── phases/
│       ├── 01-foundation/PLAN.md
│       ├── 02-session-management/PLAN.md
│       ├── 03-document-upload/PLAN.md
│       ├── 04-ai-reports/PLAN.md
│       └── 05-admin-activity/PLAN.md
└── REVERSE-ENGINEER-REPORT.md          (coverage matrix + confidence scores)
```

## Step 4 — Use the spec

Feed into GSD:
```bash
cd <new-empty-repo>
cp -r ubt-advisory-spec/.planning .
claude "/gsd-progress"   # GSD reads ROADMAP.md and suggests next phase
```

Or hand `/spec/` to any agent and say "build this on Next.js + Prisma + Clerk — ignore `/spec/implementation-hints/` and pick your own stack".

## Known caveats in this example

- The Lovable project's `generate-five-point-n8n` edge function delegates to an external n8n workflow whose internals are outside this repo — spec documents the contract (input → output) but not the workflow logic.
- `.lovable/plan.md` on disk only shows the *latest* Lovable-authored plan. Historical plans are recovered via git log on that file.
- Some pages (`NotFound.tsx`) are trivial and appear only once in `03-user-flows.md` as a catch-all.
