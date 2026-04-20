# Changelog

All notable changes to this skill are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning: [SemVer](https://semver.org/).

## [Unreleased]

## [0.3.0] — 2026-04-20

### Added
- `prompts/11-mine-teams-chat.md` — optional extractor for external chat transcripts (Teams, Slack, email). Runs in parallel with the other 5 extractors when `--teams-chat <path>` is provided. Filters multi-project content to only advisory/cross-cutting, then captures stakeholder intents, strategic decisions, cross-app context, external resources (SharePoint etc.), stakeholders, risks/concerns, timeline anchors, and unanswered questions.
- New evidence citation formats: `(teams: YYYY-MM-DD HH:MM @Speaker)`, `(slack: …)`, `(email: …)` — accepted by the evidence-check step.
- Synthesis now merges three signal sources: repo (what was built), Lovable chat (dev-level intent), external chat (stakeholder-level intent). Strategic ADRs get `[Stakeholder]` prefix. External knowledge resources land in `06-integrations.md`. Teams-raised risks cross-check NFRs.
- Coverage matrix tracks stakeholder intents, strategic decisions, external resources, and teams-raised risks.

### Changed
- `SKILL.md` / `AGENTS.md` inputs table + orchestration step 2 list the new extractor.
- README "Three signal sources" added to design principles.

## [0.2.0] — 2026-04-20

### Added
- `prompts/10-poc-review.md` — POC-vs-production trade-off elicitation. Runs between extraction and synthesis. Two parts:
  - **Part A:** upfront checklist asking the user which areas were deliberately simplified for the POC and should be adjusted/dropped/kept (auth, validation, logging, observability, rate limits, multi-tenancy, secrets, testing, accessibility, performance, retention, CI/CD).
  - **Part B:** auto-surfaces concerning patterns from extraction outputs (CORS wildcards, PII in logs, missing FKs, god components, silent empty prompts, non-transactional writes, etc.) and asks per-item decision: `copy-as-is` / `adjust` / `out-of-scope-v1` / `needs-discussion`.
- New spec doc `spec/11-production-adjustments.md` — trade-off decisions (TO-NN) + review items (RI-NN) with severity, evidence, decision, and adjustment prescription.
- New planning artefact `.planning/BACKLOG.md` — items deferred past v1, with revisit triggers.
- ADR status variant `POC-shortcut → production-recommend-RI-NN` in `09-decisions.md`.
- `09-verify-coverage.md` now checks POC-review completeness (every review item surfaces, every `adjust` becomes a tagged task or backlog entry, every `needs-discussion` lands in action items).

### Changed
- `SKILL.md` and `AGENTS.md` orchestration: inserted step 4 "POC-vs-production review" between NFR elicitation and synthesis.
- `spec-frontmatter.schema.json` enum now accepts `11-production-adjustments`.
- `README.md` Outputs tree + Usage flow updated.

## [0.1.0] — 2026-04-18

### Added
- Initial skill package.
- `SKILL.md` for Claude Code with frontmatter + orchestration.
- `AGENTS.md` mirror for Codex, Cursor, Aider, Cline.
- `.github/copilot-instructions.md` shim.
- `prompts/01..09-*.md` — nine subprompts covering preflight, extraction, elicitation, synthesis, verification.
- `schemas/chat-message.schema.json` — mirrors `loveable-chat-history-capture` output format.
- `schemas/spec-frontmatter.schema.json` — optional machine-readable spec headers.
- `examples/ubt-advisory-portal/` — worked example against a real Lovable project.
- Dual output: agent-agnostic `/spec/` + GSD-native `.planning/`.
- Interactive NFR elicitation with 10-question checklist.
- `REVERSE-ENGINEER-REPORT.md` coverage matrix, evidence check, confidence scores.
