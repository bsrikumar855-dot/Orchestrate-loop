# Orchestrate Loop

A Claude skill for HackerRank Orchestrate-style hackathons: the loop patterns that held up under a 24-hour build, the scoring context behind them, and paste-ready prompts for Claude Code.

It gives a fresh Claude session the working memory of a previous run, so it starts from lessons already paid for instead of rediscovering them.

## What it covers

| Area | What you get |
|---|---|
| **Loop patterns** | Six loops used end to end: the agent loop, Plan → Build → Review, the iteration loop with a change record, the Claude → Claude Code prompt relay, verify-before-trust, and per-row failure handling |
| **Orchestrate context** | Rubric weights, how the four graded artifacts interact, the `AGENTS.md` / `log.txt` compliance trap, results history, lessons from two editions, a checklist for the next one |
| **Prompt templates** | Kickoff, iteration protocol, targeted audit and final-freeze prompts for Claude Code |
| **Toolkit digest** | A condensed read of the community [`hackerrank-orchestrate-skills`](https://github.com/NITISH-R-G/hackerrank-orchestrate-skills) kit: its CLI, 20-rule playbook, judge prep and release gate |

## The six loops

1. **Agent loop** — the model chooses tools, arguments and when to stop; code supplies only a step cap, a fallback and a final-answer contract. Graders look for a real loop versus a hardcoded workflow.
2. **Plan → Build → Review** — state the plan and reasoning first, build with specifics, reject your own drafts out loud. This is what the chat-transcript score reads.
3. **Iteration loop** — compute the ceiling first, then record every change as `CHANGE / REQUIREMENT / CLAIM / METRIC / BASELINE / RESULT / BLAST RADIUS / COUNTERFACTUAL / VERDICT`. Diff the whole output, not the target field.
4. **Prompt relay** — Claude writes the prompt, Claude Code builds, the report comes back, Claude verifies it and writes the next prompt.
5. **Verify-before-trust** — check claims against real files, hashes and live test counts. Docs drift under heavy iteration.
6. **Per-row safety** — validate schema, retry once, fall back to an honest escalation row, checkpoint and resume. One bad row never kills the run.

The outer loop is the phase-gate sequence (rubric → plan → architecture → robustness → build → justifications → transcript → self-score → interview → release). Self-scoring is the one gate never skipped.

## Core idea

> Deterministic core, LLM language layer. Code owns every number and decision; the model extracts evidence and writes explanations.

Around it, a few rules that kept paying off:

- Never tune against sample rows until one number matches. That is dataset coupling.
- Decide grouping keys and statistics before writing a detector. Patches inherit what the grouping already discarded.
- Never gate a decision on the truthiness of a model-written free-text field. Use booleans or enums.
- Prove counterfactuals: disable a component and show its effect stops.
- Keep a constants file with a provenance column (measured / spec / standard / bound / judgement).
- State the boundary of every guarantee and name limitations before a judge asks.
- Do not overwrite the starter `CLAUDE.md` if it is `@AGENTS.md`; that import drives the mandatory session logging.

## Repository layout

```
orchestrate-loop/
├── SKILL.md                        # entry point: trigger description, working style, loop summary, hard rules
└── references/
    ├── loop-patterns.md            # the six loops in detail, change-record template, phase gates
    ├── orchestrate-context.md      # rubric, results, lessons, next-edition checklist
    ├── prompt-templates.md         # paste-ready Claude Code prompts
    └── kit-repo.md                 # digest of the community toolkit
```

`SKILL.md` stays short and points to the reference files, which Claude reads only when needed.

## Install

### Claude Code

Copy the skill folder into your project (or into your personal skills directory):

```bash
mkdir -p .claude/skills
cp -r orchestrate-loop .claude/skills/
```

Claude Code picks it up automatically. It triggers from the description, so no slash command is needed.

### Claude.ai

Zip the `orchestrate-loop` folder (with `SKILL.md` at the top level of the folder) and upload it as a custom skill in your Claude skill settings.

## How it triggers

The description fires on things like:

- "Orchestrate", "HackerRank challenge", "looping with Claude"
- "give me the Claude Code kickoff prompt" or "next iteration prompt"
- `AGENTS.md`, `log.txt`, deterministic core vs LLM layer
- reviewing a pasted Claude Code report during a hackathon

Example prompts:

```
Kickoff prompt for Claude Code. Problem statement is pasted below.
```

```
Here is the Claude Code report from the last round. Verify it and give me the next prompt.
```

```
Compare this competitor repo against ours and tell me what to take for the next edition.
```

## Using it with the community toolkit

This skill is a digest and a set of working patterns. It does not replace the toolkit, which has per-gate auto-triggering skills and a CLI that audits your actual repo (`evaluate`, `release`, `selftest`, `mentor`, `interview`, `experiment`, `score`).

```bash
git clone https://github.com/NITISH-R-G/hackerrank-orchestrate-skills.git
cd hackerrank-orchestrate-skills
pip install -e .
python -m orchestrate_kit memory seed
cp -r skills/* /path/to/your-project/.claude/skills/
```

Use both: the toolkit to check the work, this skill to run the loop around it.

## Keeping it current

After each edition, update `references/orchestrate-context.md` with the new results, the rejected ideas with the numbers that killed them, and the real problem-statement details (schema, tie-breaks, track). Until a problem statement exists, the skill carries no task-specific spec.

## Status and caveats

- Not formally evaluated. Frontmatter and packaging validate; trigger accuracy and output quality have not been measured against a test set.
- Scores, ranks and results in the context file are self-reported and unverified against any leaderboard.
- Rubric weights come from HackerRank's public material. The live challenge page wins on any conflict.
- Toolkit scores (`score`, `transcript analyze`, `interview`) measure the shape of an answer or process, not correctness and not a real HackerRank score.
- The toolkit's own numbers come from a different challenge and dataset; treat them as method examples, not benchmarks.
- Not affiliated with or endorsed by HackerRank or the toolkit's author.

## Credits

The toolkit digest in `references/kit-repo.md` summarizes [NITISH-R-G/hackerrank-orchestrate-skills](https://github.com/NITISH-R-G/hackerrank-orchestrate-skills). Credit for that method and its case studies belongs to its author.
