# claude-skill-rules-distill

An [Agent Skill](https://agentskills.io/specification) that scans your installed skills, extracts cross-cutting principles appearing in multiple skills, and distills them into rules — appending to existing rule files, revising outdated content, or creating new ones.

The final piece of the knowledge lifecycle:

```
search-first → skill-stocktake → learn-eval → rules-distill
(research)     (quality audit)   (extraction)  (principle promotion)
```

## Install

### Claude Code

```bash
# Copy skill + scripts + command
cp SKILL.md ~/.claude/skills/rules-distill/SKILL.md
cp -r scripts/ ~/.claude/skills/rules-distill/scripts/
cp commands/rules-distill.md ~/.claude/commands/rules-distill.md
```

## How It Works

### Phase 1: Inventory (Deterministic Collection)

Scripts enumerate all installed skills and rule files exhaustively:

| Script | Purpose |
|--------|---------|
| `scripts/scan-skills.sh` | Enumerate skill files with frontmatter and usage stats |
| `scripts/scan-rules.sh` | Enumerate rule files with H2 heading index |

### Phase 2: Cross-read, Match & Verdict (LLM Judgment)

A subagent cross-reads skills and full rules text in a single pass. No grep pre-filtering — rules are small enough (~800 lines) that the LLM reads everything for accurate semantic matching.

**Extraction criteria** (all must be true):
1. Appears in 2+ skills
2. Actionable behavior change ("do X" / "don't do Y")
3. Clear violation risk (1 sentence)
4. Not already in rules (even if worded differently)

### Phase 3: User Review & Execution

Candidates are presented with verdicts. User approves, modifies, or skips each one.

**Never modifies rules automatically.**

## Verdict Types

| Verdict | Meaning |
|---------|---------|
| **Append** | Add to an existing section of an existing rule file |
| **Revise** | Fix inaccurate or insufficient content in existing rules |
| **New Section** | Add a new section to an existing rule file |
| **New File** | Create a new rule file |
| **Already Covered** | Sufficiently covered in existing rules |
| **Too Specific** | Should remain at the skill level |

## Design Principles

- **What, not How**: Extract principles (rules) only. Code examples stay in skills.
- **Link back**: Drafts include `See skill: [name]` references.
- **Deterministic collection, LLM judgment**: Scripts guarantee exhaustiveness; the LLM guarantees contextual understanding.
- **Anti-abstraction safeguard**: 3-layer filter (2+ skills, actionable behavior test, violation risk) prevents overly abstract principles from entering rules.

## Example Output

```
Rules Distillation Report

Skills scanned: 56 | Rules: 22 files | Candidates: 4

| # | Principle                                          | Verdict     | Target          |
|---|----------------------------------------------------|-------------|-----------------|
| 1 | LLM output: normalize, type-check before reuse     | New Section | coding-style.md |
| 2 | Define explicit stop conditions for iteration loops | New Section | coding-style.md |
| 3 | Compact context at phase boundaries, not mid-task   | Append      | performance.md  |
| 4 | Separate business logic from I/O framework types    | New Section | patterns.md     |
```

## Related Skills

| Skill | Role in Knowledge Lifecycle |
|-------|-----------------------------|
| [search-first](https://github.com/shimo4228/claude-skill-search-first) | Research before coding |
| [skill-stocktake](https://github.com/shimo4228/claude-skill-stocktake) | Quality audit of skills |
| [learn-eval](https://github.com/shimo4228/claude-skill-learn-eval) | Quality-gated knowledge extraction |
| **rules-distill** | Principle promotion from skills to rules |

Together, these form a complete self-improvement loop for AI agents:

```
Experience (skills) → Extraction (learn-eval) → Curation (skill-stocktake) → Principle promotion (rules-distill) → Behavior change → New experience → ...
```

## Requirements

- `jq` (JSON processing)
- `bash` 4+ (macOS: uses `while IFS= read -r` for compatibility)
- Claude Code with Agent tool support

## License

MIT
