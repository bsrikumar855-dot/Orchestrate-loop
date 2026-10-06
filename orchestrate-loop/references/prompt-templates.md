# Prompt templates for Claude Code

Write these as complete, paste-ready blocks. Fill the bracketed parts from the real `problem_statement.md` and `AGENTS.md`; never invent a schema, column list or track.

## 1. Kickoff (first message in the session)

```
You already read AGENTS.md and are logging to log.txt per its rules. Keep doing that
every turn, unprompted. This message adds strategy; it does not replace AGENTS.md.

Load the skills in .claude/skills/ and apply them throughout, not just when named.

Deadline: [date/time + timezone]. Work Plan -> Build -> Review: state the plan and
reasoning before building, be specific while building, and say out loud why you reject
your own first draft. Tell me which phase gate you are in at the start of each cycle.

Task in one paragraph: [from problem_statement.md]. Output: one row per [unit] in
output.csv with the exact columns/order in AGENTS.md [section].

Architecture: deterministic core (plain code, unit-tested, zero model calls) +
thin agentic layer. Deterministic: [state reconstruction, normalization, simulation,
ranking, format enforcement]. LLM only: [evidence extraction, explanation writing,
robustness to untrusted content]. The model never touches a final number.

Agent loop: Claude chooses tools and when to stop; code supplies only the step cap and
the fallback. Per-row failures retry once, then fall back to an honest escalation row.

Before any grouping/recurrence logic: classify every category as fixed-vendor or
volatile-but-real, and pick the statistic per group, fitted on samples.
```

## 2. Iteration protocol (next message after build)

```
From here every change follows this loop.
1. Compute the ceiling: how many sample rows does this gap touch, how many are wrong
   because of it vs unrelated causes?
2. Change record for every non-trivial change:
   CHANGE / REQUIREMENT / CLAIM / METRIC / BASELINE / RESULT / BLAST RADIUS /
   COUNTERFACTUAL / VERDICT (SHIP or REJECT)
3. After any core or extraction change, diff the FULL sample output against the
   previous run.
4. Reject large blast radius with small gain. Log rejections via orchestrate memory.
5. Never tune until one sample number matches.
Report each change as its change record. Run orchestrate mentor before nontrivial
design decisions.
```

## 3. Targeted audit (one bounded check)

```
One targeted check, unrelated to [closed investigation].
Pattern to look for: [exact code pattern + why it bites].
1. Find every place it occurs.
2. For each, can the model plausibly produce a value that defeats it?
   Structured booleans/enums are already safe; confirm and move on.
3. If found, size it from real cached evidence, not in the abstract.
If nothing is found, say so plainly and stop; do not hunt for a weaker version.
```

## 4. Final freeze

```
Evidence-gated final audit on the frozen files. No scores, percentages or rank
predictions; describe expected outcome qualitatively. Re-derive every number cited in
docs from the live repo (test counts especially). Confirm code zip, output.csv and
log.txt tell the same story. End with TECHNICAL FREEZE and the three remaining
non-code actions ordered by value (likely interview rehearsal).
```
