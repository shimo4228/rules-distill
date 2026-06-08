# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Changed

- Layout normalized to nested: `SKILL.md` and `scripts/` moved into `skills/rules-distill/` so the repo matches the other ten `claude-skill-*` repositories. Install instructions in `README.md`, `llms.txt` link targets, and the `llms-full.txt` Layout line updated accordingly. History preserved via `git mv`
- Install instructions simplified: stale `commands/rules-distill.md` copy step removed (the skill is discovered via `SKILL.md` frontmatter `user-invocable: true`, not via a separate `commands/` file)

### Planned

- Initial public release.

### What it does

A Claude Code Agent Skill that scans all installed skills, identifies cross-cutting principles that appear in multiple skills, and distills them into rules — either appending to existing rule files, revising outdated content, or creating new rule files when a principle has no home yet. The skill is the final phase of the knowledge lifecycle: `search-first → skill-stocktake → learn-eval → rules-distill` — research → quality audit → pattern extraction → principle promotion.

### Components

- `SKILL.md` — the skill body. Scans `~/.claude/skills/` and `.claude/skills/`, finds principles that recur across multiple skills, and proposes consolidation into `~/.claude/rules/` or `.claude/rules/` files.
- `scripts/` — helper scripts for scanning and parsing skill frontmatter.

### Scope

The skill assumes Claude Code's `skills/` + `rules/` separation and works on Markdown frontmatter conventions. Adopters using a different organizing principle (e.g., a single skill folder without rules) substitute the equivalent destination; the cross-skill principle detection logic itself is harness-neutral.

### Requirements

- `bash` 4 or later (for `scripts/`)
- Claude Code with Agent tool support (for the principle-promotion decision phase)
- No external runtime dependencies

### Relationship to companion skills

| Skill | Role | When |
|---|---|---|
| [`search-first`](https://github.com/shimo4228/search-first) | Research before coding | First phase of the lifecycle |
| [`skill-stocktake`](https://github.com/shimo4228/skill-stocktake) | Quality audit of existing skills / commands | Second phase — what to keep / retire before distilling further |
| [`learn-eval`](https://github.com/shimo4228/learn-eval) | Per-session pattern extraction with quality gate | Third phase — feeds candidate patterns into this skill |

This skill implements the **Promote** phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) — a Zenodo-citable six-phase bidirectional growth loop for sustaining intent alignment between an AI agent and its operator over time.
