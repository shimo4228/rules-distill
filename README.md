# rules-distill

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/rules-distill)

An [Agent Skill](https://agentskills.io/specification) for Claude Code that scans your installed skills, finds principles that belong in the **always-loaded rules layer** (the rule files under `~/.claude/rules/` that Claude Code loads into every session), and drafts them as rules: appending to existing rule files, revising outdated content, or creating new ones. It never edits a rule until you confirm that candidate.

Because rules load in every session, what gets in matters more than how much. A candidate must be a fact, a wiring or a trap specific to your machine, harness or accounts; general advice the model already follows is turned away however many skills repeat it. It is for people who keep their own skills and rules; run it every so often, for example monthly or after installing new skills.

It is the Promote phase of the author's [Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle) (AKC), a human-gated cycle that turns a coding agent's repeated experience into skills and rules; this skill runs on its own without the rest of the cycle. The author's other work is listed under [More from the author](#more-from-the-author).

## Install

```bash
git clone https://github.com/shimo4228/rules-distill
mkdir -p ~/.claude/skills
cp -r rules-distill/skills/rules-distill ~/.claude/skills/rules-distill
```

Run it by typing `/rules-distill`. Claude does not start it on its own: the skill sets `disable-model-invocation: true`, so it stays out of every session's context until you call it.

It needs Claude Code with the **Glob**, **Read**, **Edit** and **Write** tools, plus **Bash** for the one `date -u` call that timestamps the ledger, and enough context to hold every skill and rule at once; the skill itself runs no scripts or subagents and needs no API keys. It reads skills from `~/.claude/skills/*/SKILL.md` and rules from `~/.claude/rules/`, and keeps a ledger of each candidate's verdict and whether you applied or skipped it at `~/.claude/skills/rules-distill/results.json`. The skill folder `skills/rules-distill/` holds only `SKILL.md`, so to update, copy that one file over (`cp rules-distill/skills/rules-distill/SKILL.md ~/.claude/skills/rules-distill/`) and the ledger stays.

If you want the whole cycle, install the [akc-cycle](https://github.com/shimo4228/akc-cycle) Claude Code plugin instead: it ships this same skill with the other skills of the cycle, called `/akc-cycle:rules-distill` there. Both copies come from one source in the author's harness; this repository is synced one way from it, so between syncs it can trail the plugin.

```
/plugin marketplace add shimo4228/akc-cycle
/plugin install akc-cycle@akc-cycle
```

## How It Works

The skill reads every skill and every rule into one context. Seeing the full rules text next to all skills is what makes the last test below ("not already in rules") reliable, so the design assumes your skills and rules fit in one context together; it does not split them into batches.

### Phase 1: Inventory (Glob, exhaustive)

Claude Code's Glob tool enumerates skill definition files (`~/.claude/skills/*/SKILL.md`) and the skill reads every rule file under `~/.claude/rules/` in full. It shows the counts before the analysis.

### Phase 2: Cross-read, Match & Verdict

**A principle is a candidate only if all of these hold:**
1. **Environment-specific**: a fact, a wiring or a trap particular to this machine, harness or set of accounts. A general engineering principle is not a candidate no matter how many skills repeat it.
2. **The model does not already do it**: if the current model handles it natively, a written rule freezes an older default and competes with the newer one.
3. **Not a procedure**: steps and workflows belong in a skill.
4. **Actionable behavior change**: expressible as "do X" / "don't do Y".
5. **Clear violation risk**: what goes wrong if it is ignored, in one sentence.
6. **Not already in rules**, including the same idea in different words.

How many skills repeat a principle (its recurrence) is reported as evidence, but it is not one of the six tests: a one-off environment trap can pass with a recurrence of 1.

### Phase 3: User Review & Execution

Candidates are presented in a summary table, then confirmed **one at a time**: each shows its evidence, violation risk, and draft text before asking `[y/n/skip]`. When a principle has a detailed procedure in a skill, the draft rule points back to that skill instead of copying the steps (as ``skill: `name` ``, the pointer form the author's rules use); the example below is a bare fact, so it has no pointer. Bulk approval is banned and the user can stop at any point. The skill **never modifies rules automatically**, because rules load every session, so a bad rule has outsized blast radius.

## Verdict Types

| Verdict | Meaning |
|---------|---------|
| **Append** | Add to an existing section of an existing rule file |
| **Revise** | Fix inaccurate or insufficient content in existing rules |
| **New Section** | Add a new section to an existing rule file |
| **New File** | Create a new rule file |
| **Already Covered** | Sufficiently covered in existing rules |
| **Too Specific** | Should remain at the skill level |

## Example Output

A run first shows how many skills and rule files it read, then a summary table (`# | Principle | Verdict | Target | Confidence`), then walks the candidates one at a time. Below is one candidate with the evidence for tests 1 to 3 and the violation risk: the worked example the skill itself carries, from the author's own account, not a run on a typical setup.

```
New Section in rules/common/debugging.md:
"If rate limits fire repeatedly during bulk writes to an external platform, treat them as a
policy signal, not a transient error, and stop the burst. Do not push through with backoff; report to a human."

Test 1 (environment-specific): an event that actually happened on this author's account. On 2026-07-16,
  continuing with backoff resulted in an indefinite account block + deletion of all created content. Not general
  HTTP 429 etiquette, but a stop condition specific to this operation
Test 2 (the model does not already do it): the model's default retries 429 as transient.
  The rule is needed to override that default
Test 3 (not a procedure): not a procedure but a declaration of fact: "repeated rate limits = policy signal"
Violation risk: pushing through loses the whole account (proven, unrecoverable)
Recurrence: 1 skill
```

## More from the author

- **[Not Reasoning, Not Tools — What If the Essence of AI Agents Is Memory?](https://dev.to/shimo4228/not-reasoning-not-tools-what-if-the-essence-of-ai-agents-is-memory-4k4n)** ([日本語](https://zenn.dev/shimo4228/articles/agent-essence-is-memory)): why the author built rules-distill to move principles from skills into rules, and how it closed a loop with learn-eval and the audit skills.
- **[Where to Put a Coding Agent's Knowledge — and How to Make It Stick](https://dev.to/shimo4228/where-to-put-a-coding-agents-knowledge-and-how-to-make-it-stick-161g)** ([日本語](https://zenn.dev/shimo4228/articles/coding-agent-memory-architecture)): which knowledge belongs in always-loaded rules, which in skills, and how often an agent actually follows each, as measured.
- **[akc-cycle](https://github.com/shimo4228/akc-cycle)**: installs this skill together with the rest of the cycle as one Claude Code plugin.
- **[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)**: the reasoning behind each phase of the cycle, Promote among them, recorded as dated design decisions.
- **[rules-stocktake](https://github.com/shimo4228/rules-stocktake)**: the reverse direction; it audits the rules you already load for what they cost in every session and proposes demoting or dissolving the ones that stopped earning their place.
- **[learn-eval](https://github.com/shimo4228/learn-eval)**: an earlier phase, which keeps what a session taught by saving it where a future session will find it, often in the skills this skill later reads.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with AKC next to the other long-running practice lines and their DOIs.

## License

MIT

<details>
<summary>For tools and AI assistants</summary>

rules-distill is an Agent Skill for Claude Code that reads all installed skills and all always-loaded rules in one context, finds principles that belong in the rules layer, and drafts them as rule edits, for people who maintain their own `~/.claude/rules/` and want the environment-specific principles in their skills promoted with evidence instead of by hand. It never writes a rule without a one-at-a-time confirmation.

It exists because rules load in every session, so what gets in matters more than how much. A rule earns that place only when it states a fact, a wiring or a trap specific to this environment that the current model would not handle by default; general advice repeated across many skills fails, because a written copy freezes an older default and competes with the newer one. How many skills repeat a principle is evidence, not a gate.

Canonical facts: MIT license; the skill payload (`skills/rules-distill/`) is a single `SKILL.md` with no scripts; maintained by one author (@shimo4228). Status: active, synced one way from the author's Claude Code harness by `scripts/sync-from-local.sh` (it never commits), and also shipped in the akc-cycle plugin, so this repository can trail the plugin between syncs. Requirements: Claude Code with Glob, Read, Edit and Write, plus Bash for the ledger's `date -u` timestamp, and enough context to hold every skill and rule at once; no keys. It runs only when called as `/rules-distill` (`disable-model-invocation: true`), edits files under `~/.claude/rules/` only after each confirmation, and writes a ledger to `~/.claude/skills/rules-distill/results.json` with each candidate's principle, verdict, target, evidence and status (applied or skipped).

Example: a run reports the counts of skills and rule files scanned, then a summary table (`# | Principle | Verdict | Target | Confidence`), then walks each candidate. Each gets one of six verdicts: Append, Revise, New Section, New File, Already Covered, or Too Specific. A candidate must pass six tests: environment-specific, not already done by the model, not a procedure, an actionable behavior change, a one-sentence violation risk, and not already in rules.

Links: [skills/rules-distill/SKILL.md](skills/rules-distill/SKILL.md) is the skill itself; [CHANGELOG.md](CHANGELOG.md) holds the release history; [llms.txt](llms.txt) and [llms-full.txt](llms-full.txt) are the machine-readable summary and reference. The skill implements the Promote phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle), concept DOI [10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726); cite AKC by that DOI. The installable form of the whole cycle is [akc-cycle](https://github.com/shimo4228/akc-cycle).

</details>
