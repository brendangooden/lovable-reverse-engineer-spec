# Changelog

All notable changes to this skill are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning: [SemVer](https://semver.org/).

## [Unreleased]

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
