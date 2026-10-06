# Orchestrate context

All figures below are user-reported or taken from earlier sessions; re-verify before reciting externally.

## What Orchestrate is
Recurring global AI-agent hackathon from HackerRank. Each edition drops a fresh problem statement with a tight window. Starter repo ships `AGENTS.md`, `problem_statement.md`, a dataset, and mandatory session logging to `log.txt`.

## Rubric (published)

| Artifact | Weight | Measures |
|---|---|---|
| Code zip | 30% | Real agent loop vs hardcoded pipeline, tool design, prompt quality, robustness |
| Output CSV | 30% | Correctness plus evidence-anchored justification per row |
| AI judge interview | 30% | Depth and self-awareness of limitations; 30 min, camera on, opens on submission, 12-hour window |
| AI chat transcript | 10% | Process: planning, constraints, debugging dialogue, deliberate iteration |

Published finding: no single metric predicts the leaderboard; balanced beats peaked.

## Results
- Aug 2026 edition: global #209.
- Sept 2026 "Buy or Wait?": global #51, 70.7/100 (transcript 9.8/10, interview 24.6/30, output CSV 13.5/30, code zip 22.8/30). Built with Claude Code.
- Oct 2026 edition: planned next entry.

## Toolkit
Community repo `NITISH-R-G/hackerrank-orchestrate-skills`: 35 skill folders (README says 34) plus the `orchestrate-kit` CLI. Full detail in `kit-repo.md`.
- `orchestrate evaluate <repo>` audits spec conformance, evidence quality, dataset coupling, determinism
- `orchestrate certify <repo>` stricter gate
- `orchestrate transcript analyze` scores the chat transcript
- `orchestrate interview` adaptive judge simulator
- `orchestrate mentor "<decision>"` pre-mortem against the rejection corpus (README says 40 entries, 9 measured rejections; earlier session notes said 45)
- `orchestrate score` / `orchestrate experiment start|finish|next|frontier` baseline → change → measure → accept/reject loop with local score history
- `orchestrate memory why-not "<idea>"` queryable rejection log
- `orchestrate release <repo>` final gate
- Core-flow skills in order: phase-gates, agent-architecture, robustness, justification-quality, ai-collaboration-transcript

Setup: `cp -r hackerrank-orchestrate-skills/skills/* .claude/skills/`, `pip install -e ./hackerrank-orchestrate-skills`, `orchestrate selftest`, `orchestrate memory seed`, then read `orchestrate memory list` before coding.

## Compliance trap
The starter `CLAUDE.md` is one line: `@AGENTS.md`. That import is what makes mandatory `log.txt` logging run, and the transcript is graded from that exact format. Never overwrite it. Append below, or paste instructions as the first message.

## Sept 2026 task and architecture
Task: for 250 requests decide `full_payment`, `partial_payment`, `installments`, `wait` or `not_recommended` using a 90-day forward cash-flow simulation that never drops below `minimum_balance_to_keep`.

- Deterministic core: state reconstruction, currency normalization (exact rate match, no guessing), 90-day forecast, `amount_safe_to_pay` by binary search over the simulation, `earliest_date_for_full_payment`, eligible-method enumeration with the 6-level tie-break, strict format enforcement.
- LLM layer (three narrow jobs): read messages and images for facts, write `decision_explanation`, stay robust to untrusted content.
- Tie-break order: complete-by-deadline, no-spending-changes, minimize-total-paid, start-earlier, fewer-payments, lowest `payment_option_id`. Prove it with a unit test where every criterion disagrees.

## Lessons from Sept 2026

1. **Rotating-vendor miss.** Recurrence detector grouped by (category, description, direction), so groceries/dining/transport with 5–8 vendor names never cleared the threshold and projected as zero. Historical-average restoration overshot ground truth 1.4x–11.6x and was rejected repeatedly across about ten rounds.
2. **Higher-scoring competitor fix.** Decide the key upfront: `(category, "*")` for volatile-but-real categories, `(category, description)` for fixed ones. Use the median, fitted on samples. Relax cadence to a bounded median gap (3–45 days) for those groups. Their per-row amount error stayed within 0–17% on all 25 samples.
3. **Commission income** was treated as stable salary (14/250 real rows); found and fixed.
4. **Debit-first same-date ordering** fix was built and verified internally, then contradicted a sample row and was fully reverted.
5. **Truthy gating.** Another team's `if args.get('uncertainty')` vetoed a correct decision when the model wrote "None - confirmed salary". Check any model-written free-text field used as a gate.
6. **`spending.py` attempted only full lump-sum** (32/250 rows affected, no sample evidence to fix against). The competitor's design enumerated plans and ran a bounded search over stoppable/reducible expenses in the base design.
7. **Stale numbers.** A live test count (198) disagreed with what docs cited (202); an earlier "110 tests" citation was also stale. Catch before the interview.
8. **Interview rehearsal** was deferred repeatedly; schedule it early and protect sleep.

## Oct 2026 checklist
1. Classify every category fixed-vendor vs volatile-but-real before any recurrence code; if finance-shaped again, redo the classification from scratch on the new dataset.
2. Pick the statistic per group type, fitted on samples.
3. Start the iteration protocol and change record from the first hour.
4. Compute ceilings before attacking gaps.
5. Enumerate plan candidates in the base design.
6. Keep the deterministic-core boundary; never let the model touch the final number.
7. Run Gate 7 early and often; rehearse the interview before the deadline.
