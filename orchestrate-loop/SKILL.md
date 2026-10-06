---
name: orchestrate-loop
description: Playbook for HackerRank Orchestrate-style hackathons and agent builds — the loop patterns (agent loop, Plan→Build→Review, iteration/change-record loop, Claude→Claude Code prompt relay, verify-before-trust), Orchestrate rubric and scoring context, Aug/Sept 2026 lessons, and the Oct 2026 plan. Use whenever the user mentions Orchestrate, HackerRank challenges (Earn, Build Your Own Full-Stack), "looping with Claude", Claude Code kickoff or iteration prompts, AGENTS.md/log.txt, deterministic core vs LLM layer, or asks for a paste-ready prompt for Claude Code on a hackathon — even if they don't name the skill.
---

# Orchestrate Loop

Working context for HackerRank Orchestrate and similar timed agent builds. It exists so a fresh session starts from what was already learned instead of rediscovering it.

## Working style

- Terse, directive. Deliver complete, paste-ready output, not explanation. No scaffolding essays.
- Default role in Orchestrate work: **write the prompt for Claude Code** (the user runs Claude Code for implementation). Do not build the submission's code yourself unless asked. Over-building scaffold code instead of giving the prompt is a known failure.
- When the user pastes a Claude Code report, interpret it, check it against real artifacts, and answer with the next prompt.
- Never fabricate scores, percentages, rank probabilities, or test counts. Past sessions caught invented figures ("85/100", "8.7/10") in pasted reports. Say what is verified and what is not.

## The six loops

Read `references/loop-patterns.md` for full detail and templates. Summary:

1. **Agent loop ("looping with Claude")** — the model chooses tool calls, arguments, count and when to stop; code owns only the step cap and the fallback. Raw Messages API tool-use loop kept as one readable named function. Graders score "real agent loop vs hardcoded workflow".
2. **Plan → Build → Review** — state plan and reasoning before building, be specific while building, say out loud why a first draft is rejected. This is what the 10% transcript score measures.
3. **Iteration loop with change record** — compute the ceiling first, then CHANGE / REQUIREMENT / CLAIM / METRIC / BASELINE / RESULT / BLAST RADIUS / COUNTERFACTUAL / VERDICT (SHIP or REJECT). Diff the whole output, not just the target field.
4. **Prompt relay loop** — Claude.ai writes the prompt → the user pastes it into Claude Code → pastes the report back → Claude verifies and writes the next prompt.
5. **Verify-before-trust loop** — claims in reports are checked against repo state, files, hashes, live test counts. Stale docs (e.g. "202 tests" vs 198 live) are the common failure.
6. **Per-row safety loop** — validate schema, retry once, fall back to an honest escalation row, checkpoint and resume. One bad row never kills the run.

## Orchestrate context

Read `references/orchestrate-context.md` for rubric weights, results history, the kit CLI, the AGENTS.md/CLAUDE.md compliance trap, competitor findings and the Oct 2026 plan. Key points:

- Rubric: code zip 30%, output CSV 30%, AI judge interview 30%, chat transcript 10%. Balanced beats peaked.
- Aug 2026: global #209. Sept 2026 ("Buy or Wait?"): global #51, 70.7/100.
- Architecture that held up: **deterministic core / LLM language layer**. The model extracts evidence and writes explanations; code owns every number and decision.
- Oct 2026 plan: apply the grouping-key lesson (classify volatile vs fixed categories up front, robust statistic) and the iteration protocol from the first hour.

## The community skills repo (hackerrank-orchestrate-skills)

Read `references/kit-repo.md` for the CLI (including `orchestrate experiment` and `score` loop commands), the 20-rule playbook, judge prep with the CONSTANTS.md provenance drill, the 10 claims never to make, the release gate and the 24-hour time budget. Its method in one line: measure before shipping, measure blast radius, prove the counterfactual, attack your own measurement before trusting it. Its scores measure shape and process, never your real HackerRank score.

## Hard rules distilled from past rounds

- Do not overwrite the starter `CLAUDE.md` (it is `@AGENTS.md`, which drives mandatory `log.txt` logging). Append below it, or paste prompts as the first message instead.
- Never tune against sample rows until one number matches. That is dataset coupling and indefensible in the interview.
- Decide grouping keys and statistics before writing a detector; patches inherit what the grouping already discarded.
- Never gate a decision on truthiness of a model-written free-text field. Use booleans or enums.
- Revert a fix that contradicts any sample row, even if internally verified.
- Gate 7 (self-score: `orchestrate evaluate .` + `orchestrate mentor`) is never skipped.
- Do interview rehearsal before deadline pressure, not during.
- Keep a `CONSTANTS.md` with a provenance column (MEASURED / SPEC / STANDARD / BOUND / JUDGEMENT). Never call an untuned value "chosen after evaluation."
- Prove counterfactuals: disable the component and show the effect stops. A clean "0 rows changed" from an unverified harness is suspect.
- State the boundary of every guarantee ("deterministic offline, not with the hosted provider"). Name limitations before the judge asks.

## Output templates

When asked for a Claude Code prompt, use `references/prompt-templates.md` (kickoff, iteration protocol, targeted audit, final freeze).

## Keeping this skill current

After each Orchestrate edition, update `references/orchestrate-context.md` with results and the new rejection-log entries. Treat figures there as user-reported unless re-verified against a leaderboard or artifact.
