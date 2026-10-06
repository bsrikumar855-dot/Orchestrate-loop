# Loop patterns

## Contents
1. Agent loop
2. Plan → Build → Review
3. Iteration loop with change record
4. Prompt relay loop
5. Verify-before-trust loop
6. Per-row safety loop
7. Phase gates (the outer loop)
8. HackerRank full-stack build loop

---

## 1. Agent loop ("looping with Claude")

Strategy used for Sept 2026: Claude decides which tool to call, with what arguments, how many times, and when it is done. Control flow is not fixed in advance. Code only supplies:

- a **step cap** (`ORCHESTRATE_MAX_STEPS`, default 10 in the scaffold)
- a **fallback** when the cap is hit or no final answer arrives
- a **final-answer contract** (last text block parsed as JSON; confirm this matches how the model signals "done")

Design choices that scored:
- Raw Anthropic Messages API tool-use loop, not an SDK abstraction, so the loop is a named, readable function someone can find in ten seconds (matters for code review and the interview).
- Tool descriptions say what a tool returns on no match (empty list, never an error) so the model checks explicitly.
- Untrusted content (messages, receipts) is fenced in delimiters and treated as data; two dataset messages were disguised scam payloads.
- Deterministic core is called as tools; the model never does arithmetic.

Failure to avoid: a "loop" that is really a hardcoded decision tree. The code-zip grader looks for the difference.

## 2. Plan → Build → Review

Per cycle: state the plan and reasoning, build with specific instructions, review including rejecting your own first draft out loud. Say which phase gate you are in at the start of each cycle. This is scored as process in the 10% transcript signal, so it must happen from message 1 and cannot be retrofitted.

## 3. Iteration loop with change record

Before touching anything, compute the **ceiling**: how many sample rows does this gap touch, and how many are wrong because of this gap versus unrelated causes? Two rows maybe means a low ceiling; do not spend an hour.

Then every non-trivial change gets:

```
CHANGE:
REQUIREMENT: <exact line in problem_statement.md or AGENTS.md>
CLAIM:
METRIC: <field-level accuracy on the samples, not a vibe>
BASELINE:
RESULT:
BLAST RADIUS: <rows changed; right direction; wrong direction>
COUNTERFACTUAL: <disable the change; does the effect disappear?>
VERDICT: SHIP / REJECT
```

Rules:
- Reject when blast radius is large and metric gain small.
- Diff the full sample output against the previous run after any core or extraction change.
- Log rejections (`orchestrate memory` conventions). A rejected change is real interview material.
- Never iterate until one sample number matches (rejected in Sept: request_05-style reverse engineering).
- If a 5–25% miss is directionally right across many rows, suspect one systematic bug in the simulation (pending-debit ordering, essential-spending conservatism, boundary-date off-by-one), not many independent errors.

Tool form: `orchestrate experiment start "<title>" --target {code,output,transcript,interview}` captures a baseline, `orchestrate experiment finish <id>` measures against it and writes the accept/reject to Engineering Memory, `experiment next` suggests the next step. Run `orchestrate mentor` before and `memory why-not` to avoid re-building a known loser. The kit's own 8-step version: claim and metric first → ceiling → strongest version of the idea → measure gain/cost/blast radius → counterfactual → attack your own measurement → ship or reject and log → pin output and record cause. See `kit-repo.md`.

Attack-your-own-measurement examples worth remembering: an ablation reported "0 of 110 rows changed" because both arms ran the same backend (bound at import), real answer 5; a benchmark overwrote the submission artifact (diff every artifact after the full suite); a clean result from a broken audit is the most dangerous output.

## 4. Prompt relay loop

1. Claude.ai produces a prompt file for Claude Code.
2. The user pastes it as the next message (first message for kickoff).
3. The user pastes the Claude Code report back, plus competitor repos or uploaded artifacts when relevant.
4. Claude interprets, flags anything unverified, and writes the next prompt.

Prompts reference the standing files by name (AGENTS.md sections, `.claude/skills/`, `log.txt`) rather than restating them.

## 5. Verify-before-trust loop

- Inspect uploaded files directly (code.zip, output.csv, log.txt): test counts, hashes, content.
- Check the zip and output/log tell the same story (same test count, same cited scores) before upload.
- Re-run the install and test suite from a fresh venv on the exact artifact.
- Skepticism cuts both ways: when verification confirms a suspicious-looking result, say so and drop the doubt.
- Documents drift under heavy iteration (stale test counts across rounds). Re-derive numbers from the live repo before reciting them in an interview.

## 6. Per-row safety loop

`process_row_safely`: run, validate against schema, on schema violation retry once, on any other exception log full traceback, then fall back to an honest escalation row and continue. Also: checkpoint/resume, exact output column names and order, schema validation before write, cost tracking.

## 7. Phase gates (the outer loop)

```
Gate 0 Understand the rubric
Gate 1 Plan and decompose
Gate 2 Agent architecture
Gate 3 Robustness and adversarial (before writing input handling)
Gate 4 Implement
Gate 5 Justification quality
Gate 6 Transcript hygiene (continuous)
Gate 7 Self-score: orchestrate evaluate . + orchestrate mentor  (never skip)
Gate 8 Interview readiness
Gate 9 Final submission review: orchestrate release .
```

Skip depth inside a gate when behind, never the gate.

## 8. HackerRank full-stack build loop (Earn / Build Your Own)

Context: a full-stack submission (React + Node + MongoDB) was rejected once. Required fixes before resubmission, which double as a pre-submission checklist for any similar build:

- server-enforced step-by-step status lifecycle on a core record
- two or more seeded accounts with different roles, and more seed data
- role-based authorization beyond simple ownership
- automated tests run against a real database

Loop: get the idea approved by the HackerRank-side reviewer, follow their guidelines doc while building, name the repo to the required convention, submit through the Discord thread, then handle reviewer feedback as a new iteration with the change record above.
