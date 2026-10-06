# The community skills repo: hackerrank-orchestrate-skills

Read from a fresh clone (latest commit: "Phase 2: experiment tracker + live score delta loop"). Not affiliated with HackerRank; built from their public writing plus one author's August 2026 submission. Treat its evidence tiers as the author's, not yours.

## Contents
1. What is in the repo
2. CLI commands (full list, including the loop commands)
3. The 20-rule playbook
4. The kit's own loop
5. Judge prep: what was probed, claims to never make
6. Release gate checklist (condensed)
7. Phase-gate time budget
8. Agent architecture guidance
9. Transcript guidance
10. Caveats

## 1. What is in the repo

- `skills/` — 35 skill folders (README says 34; `orchestrate-transcript-engineering` was added after). Auto-triggering `SKILL.md` files, about 2000 lines total.
- `orchestrate_kit/` — zero-dependency Python CLI: `memory`, `mentor`, `interview`, `evaluator`, `score`, `experiment`, `transcript`, `viz`, `plugin`, `selftest`.
- Docs: `PLAYBOOK.md` (20 rules), `TIMELINE.md` (F-1…F-48 defect history), `JUDGE-PREP.md`, `RELEASE-CHECKLIST.md`, `SCORING-HEURISTIC.md`, `RESEARCH.md`, `ARCHITECTURE_EVOLUTION.md` (about the kit's own memory system scaling, not relevant to a submission), ADRs in `docs/adr/`.
- A GitHub Action that runs `orchestrate evaluate` on push/PR.

Install: `pip install -e .` then `python -m orchestrate_kit memory seed`. `python -m orchestrate_kit` always works; the plain `orchestrate` command needs the install dir on PATH.

## 2. CLI commands

| Command | Use |
|---|---|
| `orchestrate evaluate <repo>` | Is it shippable? Exit 2 on any blocker. Blockers print above the score. |
| `orchestrate certify <repo>` | Stricter: any finding above INFO fails (exit 3). |
| `orchestrate release <repo>` | Submittable? Adds gates and manual items; last item is the authorship attestation, which only you can make. |
| `orchestrate selftest` | Injects defects and checks the audits catch them, plus benign cases that must stay quiet. |
| `orchestrate mentor "<proposal>"` | Pre-mortem before building: prior art, expected gain, risks, blast-radius method, evaluation plan. Prints UNKNOWN rather than guessing a gain. |
| `orchestrate memory why-not/recall/list/search/verify/add` | Rejection corpus. `add --status rejected` is refused without `--reconsider-if`. |
| `orchestrate interview --persona {architect,skeptic,security,practitioner} --difficulty {warmup,standard,hard,adversarial} [--panel] [--learn] [--save f.json]` | Adaptive judge simulator. Scores the shape of answers, not their truth. |
| `orchestrate transcript analyze <file>` | Lints a chat transcript against the published 4-dimension rubric. Not a predictor of the real score. |
| `orchestrate transcript compose "<goal>" --stage <understanding\|design\|planning\|implementation\|verification\|release>` | Fills a prompt blueprint with real inputs and memory hits. |
| `orchestrate score --repo . [--transcript f] [--interview-result f] [--what-if output +5]` | Estimated four-signal scoreboard with local history. `--official-score` is for calibration only. |
| `orchestrate experiment start "<title>" --target {code,output,transcript,interview} --hypothesis ...` | Captures a baseline score and opens an experiment. |
| `orchestrate experiment finish <id>` | Measures against baseline, records accept/reject to Engineering Memory (disable with `--no-memory`). |
| `orchestrate experiment list/show/compare/frontier/next/plan` | History, Pareto frontier, ladder-ordered next step, counterfactual-backed plan for a goal. |
| `orchestrate viz all --out diagrams/` | Nine generated Mermaid diagrams (generated, never hand-drawn). |

Every finding carries a confidence label: measured / observed / inferred / unknown. A skipped audit is UNKNOWN, never a pass.

## 3. The 20 rules (PLAYBOOK.md)

Evidence discipline: (1) a change is not an improvement until a number moves; (2) measure blast radius, not just the win; (3) prove the counterfactual (feature on shows effect, feature off removes it); (4) compute the ceiling before optimising; (5) rejection is a deliverable.
Auditing the audit: (6) attack the test itself with a negative control; (7) an audit never mutates what it audits; (8) measure at the right scale and inside the operating envelope.
Regression: (9) pin the output hash and keep a re-pin log with cause, effect, why-right, score; (10) hermetic tests (clear credentials, run each test file in isolation); (11) state the boundary of every guarantee.
Unseen data: (12) hunt coupling to the sample (rename IDs, reformat timestamps, shuffle rows, rename directories, detect file types by magic bytes, review lexicon terms that fire on exactly one row); (13) distinguish specified vs observed constants; (14) calibrate to the labeling policy, not a textbook ideal.
Release: (15) simulate the consumer from a fresh clone using only the README; (16) generated artifacts are stale until proven current; (17) documentation is production code.
Reporting: (18) separate VERIFIED / MEASURED / INFERENCE / UNKNOWN; (19) name the limitation before asked; (20) stop only when everything fixable is fixed or the remaining uncertainty is precisely nameable.

Their loop: state claim and metric first → compute ceiling → build the strongest version of the idea → measure gain, cost, blast radius → prove counterfactual → attack your own measurement → ship or reject and log the number → pin output and record cause.

## 4. The kit's own loop

`orchestrate experiment` is the tool form of the iteration loop in `loop-patterns.md` section 3: baseline → change → measure → accept/reject, with the result written to Engineering Memory. Pair it with `mentor` before the change and `memory why-not` to avoid re-building a known loser. Use it from the first hour, not at the end.

## 5. Judge prep (JUDGE-PREP.md, one first-hand Aug 2026 interview)

Quoted praise: measuring your own assumptions and rejecting changes that did not improve things. Quoted criticism: not knowing your own constants; owning every number is part of owning the decision.

Topics probed: high-level architecture, deterministic vs LLM reasoning, arbitration design, end-to-end execution flow, BM25 retrieval, confidence calibration, AI-assisted workflow, engineering tradeoffs, honesty about limitations.

**CONSTANTS.md with a provenance column** (do this first):
- MEASURED: an experiment chose it; name the losing alternative
- SPEC: fixed by the problem statement or labels; cite the line
- STANDARD: canonical value you did not tune; say exactly that
- BOUND: derived from an external standard; cite it
- JUDGEMENT: reasoned, not measured; say so

Never write "chosen after evaluation" for an untuned value; it dies to "show me the sweep."

Follow-up traps (weak → strong): "Why that number?" → provenance plus losing alternative. "Is it deterministic?" → boundary sentence naming where it stops. "Did you overfit?" → rejected ideas with numbers. "What's your score?" → name the set and its size. "Walk me through a row" → name each module in order with real field names. "More time?" → the specific uncertainty and the experiment that resolves it.

Ten claims never to make unless directly supported: incremental commits (read `git log` first), "I wrote every line myself", unqualified "deterministic", "vision/OCR improves my score", "I score 100%" without set and size, "retrieval is optimal", "no dataset-specific assumptions", "handles every scam type", "tests prove it generalises", "I know it will rank well."

Hour-before checklist: CONSTANTS.md read aloud; three rejections memorised with numbers; one row traced end to end; boundary sentence for every guarantee; `git log` read; two limitations chosen to volunteer; "I don't remember, but here is how it was chosen" rehearsed.

## 6. Release gate (RELEASE-CHECKLIST.md, condensed, cheapest checks first)

0. `git status --porcelain` clean; regenerate artifacts to a temp path and hash against committed; fresh clone to an empty directory following only the README; each test file passes alone.
1. Literal spec conformance: exact column names, order, separators, allowed values, one output per input in order.
2. Evidence for every change: metric named first, gain and cost, blast radius, counterfactual, ceiling, logged rejections.
3. Trust the tests: negative controls, audits do not write to ground truth, diff every artifact after the full suite.
4. Regression: golden hash pinned to the deterministic configuration; every fixed defect has a test that fails without the fix.
5. Unseen-data robustness (rule 12) and `CONSTANTS.md`.
6. Determinism across repeated runs, processes, hash seeds; boundary stated.
7. Security: injection, ReDoS, path traversal, failure paths degrade to valid output, no secrets.
8. Docs: every README number verified today; limitations stated first.
9. Stop condition. Verdict is exactly READY or NOT READY with the named blocker.

## 7. Time budget for a 24-hour event (phase-gates skill)

0–2h gates 0–2 (rubric, data, plan, agent loop design) · 2–4h gate 3 adversarial cases before handling code · 4–14h implement with tests · 14–17h review every row's justification · 17–19h self-score and fix weakest dimension · 19–21h interview prep, rehearse limitations · 21–23h final review and packaging · 23–24h buffer. 70% of the score is not the code. Always read the live challenge page; it overrides the repo.

## 8. Agent architecture guidance

- Loop is a named, readable function; step cap and termination explicit; fallback deliberate; prompts in named files or constants; config separate from logic; README states design decisions and tradeoffs.
- Hybrid is fine if you can defend the boundary: deterministic for loading, static retrieval, writing rows, step cap; agentic for judgment calls (is context sufficient, search again, is this adversarial, escalate and justify).
- Tool descriptions are prompt engineering; fewer well-chosen tools; informative errors that let the agent recover; tools unit-testable without the model.
- Prompts: specific role, stated decision criteria, constrained parseable output validated after, explicit permission to say "not sure, escalating".
- Single agent holding all evidence in context beat multi-agent summary hand-offs in one first-hand case study; default to single-agent-with-tools for cross-evidence reasoning unless you tested otherwise.

## 9. Transcript guidance

Scored on how you directed the AI, not what it produced. Structure as Plan → Build → Review: state the plan and alternatives before asking for code, state constraints explicitly, debug as dialogue (hypothesis first), and reject or revise output with stated reasons. Do not retroactively narrate or pad; say decisions out loud when you are already making them.

## 10. Caveats

- The rubric weights and signals come from HackerRank's May 2026 post-mortem and public material. The live challenge page wins on any conflict.
- Kit scores (`score`, `transcript analyze`, `interview`) measure shape and process, not correctness or your real HackerRank score. Do not quote them as predictions.
- The kit author's numbers (30/30 labeled, 81 tests, 48 defects, 9 rejections) are from a different challenge and dataset; they are examples of method, not benchmarks for yours.
