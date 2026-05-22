# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

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
| [`claude-skill-search-first`](https://github.com/shimo4228/claude-skill-search-first) | Research before coding | First phase of the lifecycle |
| [`claude-skill-stocktake`](https://github.com/shimo4228/claude-skill-stocktake) | Quality audit of existing skills / commands | Second phase — what to keep / retire before distilling further |
| [`claude-skill-learn-eval`](https://github.com/shimo4228/claude-skill-learn-eval) | Per-session pattern extraction with quality gate | Third phase — feeds candidate patterns into this skill |

This skill implements the **Promote** phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) — a Zenodo-citable six-phase bidirectional growth loop for sustaining intent alignment between an AI agent and its operator over time.
