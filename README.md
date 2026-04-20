# lovable-reverse-engineer-spec

A portable agent skill that turns a [Lovable.dev](https://lovable.dev) project into a structured, reimplementable specification — without locking you into React, Supabase, or shadcn.

Works with:
- **Claude Code** — via `SKILL.md`
- **Codex / Cursor / Aider / Cline** — via `AGENTS.md`
- **GitHub Copilot** — via `.github/copilot-instructions.md`

## What problem does this solve?

You prototyped on Lovable. It works. Now you want to:
- Rebuild it on a production stack (Next.js + Prisma, .NET + Postgres, whatever).
- Onboard teammates who weren't in the chat.
- Audit what the AI actually shipped vs. what you asked for.
- Feed a spec into a spec-driven framework (GSD, Spec-Kit, BMAD, Kiro).

Reading the raw chat transcript doesn't scale. The repo alone misses the intent, the failed attempts, and the business rules expressed in prose. This skill reads **both** and emits a versioned spec plus a GSD-compatible planning tree.

## Inputs

| Flag | Required | Purpose |
|---|---|---|
| `--repo <path>` | ✅ | Local clone of the Lovable GitHub repo |
| `--chat <path>` | ✅ | Export from [`loveable-chat-history-capture`](https://github.com/brendangooden/loveable-chat-history-capture) |
| `--out <path>` | optional | Output dir (default: `<repo>/.reverse-engineered/`) |

## Outputs

```
<out>/
├── spec/                          stack-agnostic, portable
│   ├── 00-overview.md
│   ├── 01-domain-model.md
│   ├── 02-features.md
│   ├── 03-user-flows.md
│   ├── 04-ui-design.md
│   ├── 05-data-model.md
│   ├── 06-integrations.md
│   ├── 07-business-logic.md
│   ├── 08-non-functional.md       ← populated via interactive Q&A
│   ├── 09-decisions.md            ← ADRs mined from chat
│   ├── 10-evolution.md            ← pivots + abandoned attempts
│   ├── 11-production-adjustments  ← POC shortcuts flagged for rework
│   └── implementation-hints/      ← Lovable-specific notes (optional)
├── .planning/                     GSD-ready
│   ├── PROJECT.md
│   ├── REQUIREMENTS.md
│   ├── ROADMAP.md
│   ├── BACKLOG.md                 ← items deferred past v1
│   └── phases/NN-slug/PLAN.md     only when project warrants breakdown
└── REVERSE-ENGINEER-REPORT.md     coverage matrix, confidence, TBDs
```

## Install

### Claude Code

```bash
# option A: clone into the skills dir
git clone https://github.com/brendangooden/lovable-reverse-engineer-spec \
  ~/.claude/skills/lovable-reverse-engineer-spec

# option B: symlink from anywhere
ln -s /path/to/lovable-reverse-engineer-spec ~/.claude/skills/lovable-reverse-engineer-spec
```

Then invoke in any Claude Code session:
> "Reverse-engineer the Lovable project at `C:\repos\my-lovable-app` using the chat export at `C:\chat-history\my-lovable-app`."

### Codex / Cursor / Aider / Cline

Copy `AGENTS.md` and `prompts/` into the root of your working project (or any parent your agent loads AGENTS.md from).

### GitHub Copilot

Copy this repo's `.github/copilot-instructions.md` + `AGENTS.md` + `prompts/` into your working project.

## Usage

1. Clone the Lovable repo locally.
2. Export chat history with [`loveable-chat-history-capture`](https://github.com/brendangooden/loveable-chat-history-capture) (`bun run export`).
3. Ask your agent to run this skill with the two local paths.
4. Answer the NFR elicitation questions (audience size, compliance, SLA, etc.).
5. Answer the POC-vs-production review (which shortcuts to carry forward vs adjust vs drop — the skill also auto-surfaces concerning patterns it found).
6. Review `REVERSE-ENGINEER-REPORT.md` for coverage and gaps.
7. Feed `.planning/ROADMAP.md` into GSD (`/gsd-new-project --from-roadmap`) or hand `/spec/` to any agent to rebuild.

## Design principles

- **Evidence-linked** — every claim cites a chat id or `path:line`. No hallucinated features.
- **Stack-free spec** — `/spec/` never names React, Supabase, shadcn. Those move to `implementation-hints/`.
- **Chat as primary intent source** — code shows what was built; chat shows what was meant. Conflicts are surfaced, not papered over.
- **Preserve failed attempts** — abandoned paths are ADR context, not noise.
- **POC ≠ production** — a Lovable prototype is ground truth for *what was built*, not *what should ship*. The skill asks up-front what to keep vs adjust vs drop and auto-surfaces concerning patterns (CORS wildcards, PII in logs, missing FKs, etc.) for per-item decisions.
- **Never modify inputs** — the repo and chat export are read-only.

## What qualifies as a "Lovable project"?

At least two of:
- `lovable-tagger` in `devDependencies`
- `componentTagger()` called in `vite.config.ts`
- `.lovable/` directory at repo root
- Auto-generated header in `src/integrations/supabase/client.ts`

If fewer than two match, the skill aborts. It's for Lovable specifically, not "any React app".

## Related

- [`loveable-chat-history-capture`](https://github.com/brendangooden/loveable-chat-history-capture) — required upstream tool
- [GSD (Get Stuff Done)](https://github.com/gsd-build/get-shit-done) — spec-driven framework this tool's `.planning/` output targets
- [GitHub Spec-Kit](https://github.com/github/spec-kit) — alternative destination for `/spec/` content
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) — skill-format inspiration

## License

MIT — see [LICENSE](./LICENSE).
