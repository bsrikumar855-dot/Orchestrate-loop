# Winning approaches

Patterns distilled from public top-finisher material for the May, June and August 2026 Orchestrate editions: one 1st-place support-triage repo, one 3rd-place and one 37th-place project with full design docs, the organizer's own published analysis, and the public leaderboards for those three months. Everything is paraphrased and no author is named.

**Evidence limits, read first**
- Only a few repos were reachable. Verified: May 1st place (code and write-up), a June top-3 project, an August top-40 project. The August and June 1st-place repos were not found. The September leaderboard was not found.
- These are patterns that top finishers used, not proof that each pattern caused the rank. The organizer's own conclusion is that behavior (tradeoff reasoning, evidence, ownership) drove results more than any tool or stack.
- Leaderboard figures below were computed from the public JSON for each edition. Treat the percentages as calibration, not targets.

## 1. What the leaderboards say

Per-stage average score as a percent of that stage's maximum, for the top 1 / top 10 / top 50 of each edition (stage maxima: transcript 10, interview 30, output CSV 30, code zip 30).

| Edition (field size) | Cutoff #1 / #10 / #50 total | Transcript | Interview | Output CSV | Code zip |
|---|---|---|---|---|---|
| May (1,349) | 76.0 / 72.3 / 65.4 | 82 / 78 / 63 | 62 / 70 / 64 | 76 / 69 / 69 | 88 / 80 / 76 |
| June (1,773) | 72.5 / 67.5 / 63.3 | 65 / 87 / 86 | 84 / 75 / 72 | 45 / 48 / 46 | 91 / 79 / 74 |
| August (1,983) | 84.2 / 80.8 / 74.0 | 89 / 95 / 89 | 87 / 82 / 80 | 77 / 70 / 68 | 87 / 89 / 82 |

Field medians for context: transcript about 1.4 to 3.8 out of 10, interview 11 to 15 out of 30, output CSV 8 to 15 out of 30, code zip 16 to 19 out of 30.

Readings:
- **Balance wins.** In every edition most of the top 50 sat in the top quartile on all four stages: 37 of 50 in May, 33 in June, 41 in August. The organizer reports the same: every top-10 finisher was top quartile across all four.
- **Output CSV is the usual weak stage for top finishers.** In June the output stage was the weakest for 49 of the top 50. In August it was the weakest for 34 of 50. Strong code and strong interview with a mediocre CSV is the common top-50 profile, so a team that also nails label agreement separates from the pack.
- **Transcript and interview are cheap points that most of the field leaves on the table.** Median transcript is a few points out of 10, while top-50 averages are 63 to 89 percent. This is process quality, not luck.
- **Rank 51 and rank 209 are close to rank 50.** Cutoffs: top 50 needs roughly 63 to 74 total depending on edition; rank 209 needs roughly 56 to 67. The marginal point is cheap near the cutoff and expensive near the top.
- **Architecture seen at the top (organizer analysis).** A single agent with retrieval plus guardrails dominated the top. The best performer used it; multi-agent workflows held the next two spots. The organizer also found the most common coding tool was over-represented among winners (about 44 percent of the top 50 versus about 14 percent overall) but says tool and behavior were correlated and behavior was the driver.
- **What interviewers rewarded.** Specific technical decisions with tradeoffs; concrete test evidence and regressions; safety mechanisms grounded in what the code really does; visible ownership instead of delegating decisions to the tool. **What they penalized.** Generic explanations, unsubstantiated safety claims, answers that contradict the implementation.

## 2. The master pattern: the model describes, code decides

Used by both projects with full design docs, and echoed in the organizer's summary of winning themes.

- The model returns a **structured record of observations** (a proposed label, ranked candidates, yes/no claims about the content). It does not return the binding answer.
- Plain, pure functions turn that record into the final columns through an ordered **rule table**. The rule that won, and every rule that fired and lost, is written on the row.
- Benefits to claim in the interview, each of which is testable:
  1. **Repeatable.** Cache the model's record, then re-run the decision layer offline with zero API calls. A flag like `--from-cache` is enough.
  2. **Auditable.** Every column traces to a rule id plus a logged observation.
  3. **Injection-resistant by structure.** Hostile text inside a message or image can at worst corrupt one observation; it cannot reach code that never takes it as input.
  4. **Constraints are structural.** "History may add caution but must never flip the verdict" is enforced by leaving history out of the record the decision code reads, not by a prompt sentence.
- **Rules declare the inputs they read.** Each rule carries a `reads` set. A rule like "mute a business that is impersonating a brand" reads verification state and domain state only, so the message text is not in scope.
- **Two tests defend the seam from both sides.** (a) A threat-model test proves injected text cannot reach the rules that do not read it. (b) An ablation test degrades each declared input and requires the rule to stop firing; a rule that keeps firing with its input removed was passing for the wrong reason, and a null result there is a defect.
- **Separate error classes.** Class A: your code or rules (deterministic, fix and re-grade for free from the cache). Class B: the model misperceived (needs a paid re-run, so batch them and run rarely). Most wasted budget is re-rolling Class B hoping to fix Class A.
- **State the boundary of the guarantee.** "Deterministic given the perception record," never "deterministic" alone.
- **Honest limit seen in practice.** This pattern earned code and interview points but did not fix label disagreement when the model's default proposal was wrong near a class boundary. Rules were never wrong; the proposal they defaulted to was. Budget time to attack the model's boundary calls (resolved definitions in the prompt, majority vote on borderline reads) as well as the rule layer.

## 3. Support-triage agent patterns (May edition, 1st place)

Single agent, one retrieval tool, thin deterministic wrapper around it.

- **One tool with three modes: search, read, grep.** The agent searches, then must read the article, then answers. A short fixed flow in the prompt, a hard cap of roughly 8 tool calls, and a graph recursion limit.
- **Hybrid retrieval.** BM25 plus dense embeddings fused with reciprocal rank fusion, then a cross-encoder rerank. Deterministic chunking with stored character offsets so a snippet can be traced to its source span. A typed filter object (company, product area, breadcrumb substring) that is translated to a safe query and silently drops unknown values instead of erroring.
- **Pre-LLM short-circuit rules, applied in a fixed order.** Empty ticket, pleasantry or thanks, clearly out of scope, prompt injection, too vague to act on. Each answers with a canned, correct response without spending a model call. Injection screening is regex based and multilingual (English plus French, Spanish, German variants at least).
- **Cheap model for tagging, stronger model only where needed.** Language normalization and company hinting run on the small model; the answer comes from the agent.
- **Structured decision object.** Pydantic model with literal enums for status and request type, a slugified product area, a response, a one-to-three sentence justification, and a list of cited paths. A validator normalizes slugs; a coercion step clamps anything off-enum to a safe value.
- **Citation guard (the standout idea).** After the agent returns, compare cited paths with the paths the agent actually *read* this turn. Rules:
  1. Drop any cited path that was not read, and record what was dropped on the trace.
  2. If the reply cites nothing verified **and** the agent never read anything, downgrade the row to an escalation with a canned message and say so in the justification.
  3. If it read evidence but cited the wrong path, keep the reply and flag a citation mismatch for review.
  4. Exempt pleasantries and invalid requests.
  Net effect: the agent can never ship a confident answer that has no read evidence behind it. This costs nothing at inference time and is easy to defend in the interview.
- **Escalation contract in the prompt.** Escalate when the request is high risk, sensitive, an override demand, identity theft or fraud, or the evidence is thin. Escalations get a short, fixed-style acknowledgement with BAD and GOOD examples in the prompt. After the model answers, a code step trims or replaces any escalation text above a word cap (about 35 words).
- **Failure ladder.** Any exception, recursion-limit hit, or missing structured output becomes an honest escalation row, never a crash and never a guess. Every ticket writes a per-row JSON trace (tool calls, reads, dropped citations, flags).
- **Contrastive examples for taxonomy fields.** The prompt lists close product-area pairs with when to pick each, which beat a bare list of allowed values.
- **Evaluation hygiene.** A sample-evaluation runner scores the labeled tickets, and the README states what the numbers do and do not show.

## 4. Multimodal verification patterns (June edition, top 3)

- **Perception is a tool loop with a zoom.** The vision model can crop or inspect a named region (coarse quadrants plus object-specific aliases, or explicit coordinates) before reporting. An invalid region returns a center crop plus a note rather than an error. Cap tool rounds (about 6) then force a final answer.
- **Typed seam.** One object is the only bridge from the model to the decision code. Enum lists are defined once in a schema module and the tool schema and output validator are generated from it. A unit test asserts those literals equal the problem statement's lists.
- **Grounding tests.** Blank-drop (replace the image with a blank and the verdict must collapse to "not enough information") and swap (change the image and the output must change). If a vision agent approves a claim with a blank image, it was matching text, not pixels.
- **Gate "not enough information" on evidence sufficiency, not on a soft authenticity flag.** The design review found the prescribed rule conflicted with a ground-truth row, adopted the safer gate, and wrote the deviation down as an open decision for the reviewer instead of deciding silently.
- **History as an additive overlay only.** Claimant history can add review flags but cannot change the verdict. Enforced by never passing history to the verdict function. Numeric thresholds are few, documented and bounded.
- **Fingerprint reused media.** Hash each image so the same photo reused across claims is detectable. Cache by content hash, never by path.
- **Majority vote only on borderline reads.** Repeating the model three times on borderline cases raised cost per row modestly. The author reports the expensive error class (missed contradictions) moving from 60 to a steady 80 percent across two separate runs, and presents the repeat run as the evidence it was not a fluke. The causal link to the vote is the author's claim, not something verified here. Measure variance before claiming a gain.
- **Cost as a first-class number.** Per-row cost was tracked and reported at each stage. Prompt-cache the stable prefix and verify the cache-read token count is non-zero. Downsample context images, keep the claimed region full resolution.
- **Free text is generated from logged facts.** Justifications come from the cue, the branch and the rule. If the result is empty or ungrounded, a deterministic template is used.
- **Safe-default row guarantee.** One valid row per input row, always. Per-row checkpointing so a crash resumes with only missing rows. Byte-exact echo of input columns by carrying them as opaque strings through the stdlib CSV module, all-quoted UTF-8, and a round-trip test.
- **Design review before coding.** A skeptical pass on the written design with findings tagged by severity, each with a resolution or a question for the reviewer, plus a complexity audit: every component must name the concrete failure it prevents. Tools that fail that test are cut or demoted to code (history became a code overlay instead of a model tool).
- **Cautionary tale in the write-up.** An earlier 15-stage pipeline finished mid-pack because the author could not crisply measure why each piece helped. The next edition's smaller design with measured parts did far better on code and transcript. Prefer fewer components you can explain.

## 5. Adversarial routing patterns (August edition, top 40)

- **Fence untrusted content.** All message text, image descriptions and transcripts are wrapped in a block with an unguessable nonce. If attacker text contains the fence token, it is escaped before insertion and an event is logged with a hash, never the content.
- **Two-stage isolation for media.** The extraction call (describe image, transcribe audio) has no routing vocabulary at all. Its output is then fenced as data before the routing call. Injected text in an image arrives demoted to a quoted description.
- **A rule for text that addresses the router.** A flag set by perception when the content tries to direct the system fires a hard override (mute as scam). It applies whether or not the injected claim is true.
- **Ordered rule tiers.** Safety overrides, then payment authority, then guards, then caps and floors, then the model's proposal as the default. **Caps beat floors**, with one documented guard that can escape a cap. The overridden floor is recorded as overridden, not lost.
- **Explicit failure dispositions.** Every failure is assigned exactly one of ABORT (output would be silently wrong), DEGRADE (row proceeds with a named degraded state that is counted) or FAIL ROW (row recorded as failed, run continues, no output written). A partial output file is never written because it looks like a submission. Distinct exit codes for each class. Budget exhaustion is ordered: every completed call is already cached, so a rerun resumes and pays only for the remainder.
- **Never trust file extensions.** Detect container format from magic bytes. In one corpus more than half of the media had a wrong extension (audio and image formats mislabeled, one video named as an image).
- **Content-addressed cache shipped with the submission.** Keyed by hash of file bytes and prompt version. A reviewer can reproduce the exact output with no API key, and iteration on rules is free. Pair it with an `assert-identical` gate that diffs two runs.
- **Measure the noise floor of the model once.** Re-deriving the entire cache from cold against the live API moved the action column by 3 rows of 110 (about 97 percent stable), moved the labeled score by one row, and made the confidence column the most volatile because it is a sum with no thresholding. Consequence: a change worth one labeled row is not evidence, and never swap one artifact for another on a one-row difference.
- **Evidence is selected, never generated.** The model never writes an evidence id. A tiered ranking over the receiver's own history picks them, from a shortlist, with owner and strict-past invariants asserted. Ungrounded citations by construction: zero.
- **Reason text from a clause grammar** with a register check (length bounds, one sentence, no identifiers). Violations fail the row instead of being truncated, because a silently trimmed reason is a wrong explanation.
- **Confidence computed in code** from named features and written to the audit record. Report calibration but never optimize it.
- **Prompts get resolved definitions, not thresholds.** When a class boundary was off, the fix that worked was giving the model resolved definitions of the boundary in the prompt, not tuning a numeric threshold against 30 rows. Stop after the boundary is moved a small, justified amount.

## 6. Evaluation and anti-overfitting discipline

- **Generalization policy.** The dataset is evidence about the domain, not the specification. Every rule must be justifiable from the problem statement or from how the domain works generally. The data may motivate or falsify a rule, never be its sole justification. No rule keyed on an id, a string from the data, or an enumerated combination whose cardinality comes from sample size. A test greps the code for corpus literals.
- **Findings, principles, refused artifacts.** For each striking pattern found in the data, write three lines: the finding, the general principle taken, and the artifact-fitted rule that was refused. This is a very strong interview answer.
- **Review questions per constant** (citation, literal, cardinality, numeric provenance, unseen input, growth). A constant whose only source is a distribution read off the sample is rejected.
- **Anti-overfitting protocol.**
  1. No threshold adjusted to fix a specific sample row.
  2. State why the rule should exist, write it, then look at what happened. Do not sweep and keep the best.
  3. Reading a wrong row to understand why is essential; turning it into a fix needs a stated class of cases larger than the sample row.
  4. The labeled set is read after the change, not before to decide what to build.
  5. The tell: if the honest justification is "because it improves sample accuracy," drop the change.
- **Three-tier merge gates.**
  - Hard gates (no acceptable nonzero value): output contract valid, ungrounded citations 0, hard parse failures 0, reasons outside register 0, unexplained action changes on an offline rerun 0, architecture tests pass (import graph, no corpus literals, no network in the decision layer).
  - Review gates (written justification needed): labeled regression of two or more rows, high churn for tiny net gain, a safety rule that stops firing, action distribution collapsing onto one answer.
  - Diagnostic only, never gating: token counts, per-class F1 where support is under 5, calibration, confidence histogram shape.
- **When label-free and labeled disagree, trust label-free.** Label-free checks are assertions about your own system and are true or false. A small labeled set is an estimate with a wide interval. Measure how the labeled set differs from the target set and write the asymmetries down; if the labeled set under-represents the adversarial or structural cases, a safety rule scoring zero on it is expected, not a defect.
- **Resolution table.** Label-free clean and labeled down at most 1 row: noise, keep on reasoning. Label-free clean and labeled down 2 or more: investigate the rows, keep only if the stated reasoning explains them. Label-free regressed and labeled up: revert. Both regressed: revert.
- **Honest limits at n of 20 to 30.** One row is 3 to 5 points. Do not call anything under two rows an improvement. Say so in the README and in the interview.
- **Attack your own work with a repro requirement.** Point a model at the code to hunt for bugs, but every suspected bug must be reproduced by actually running it. In one review that filter turned 24 candidate bugs into 8 real ones.

## 7. Design-doc suite worth copying

Written before implementation, short, cross-checked for one source of truth:

1. Problem analysis (with a decision record, including places the spec contradicts itself)
2. Dataset analysis (what the data contains; facts only)
3. Generalization policy (above)
4. Threat model (what is attacker-controlled, which code can read it)
5. Failure modes (table: failure, how it shows, where caught, what happens instead, what the report shows)
6. System design and decision engine (rule table with `reads` sets)
7. Evaluation strategy (gates, anti-overfitting, noise floor)
8. Design review (severity-tagged findings, complexity audit, open questions)
9. Implementation plan with phases
10. Interview prep (failure modes first, tradeoffs not features, what did not work)

Cost: roughly one to two hours at the start. Payoff: the interview becomes reading your own notes back, and the transcript shows visible planning.

## 8. Do and do not

| Do | Do not |
|---|---|
| Keep the model as a perceiver; put binding decisions in tested pure code | Let the model write the final label, confidence, or evidence ids |
| Cache model records, content-addressed; ship the cache | Re-roll the model hoping for a better number |
| Guard citations against what was actually read | Let a reply cite documents that were only seen in snippets |
| Prove the input matters (blank-drop, swap, ablation) | Claim a rule reads something without a test that degrades it |
| Escalate honestly when evidence is thin or the run fails | Fabricate a confident answer or crash the batch |
| Write the failure disposition for every failure up front | Write a partial output file |
| Measure the model's own noise floor once | Celebrate or reject a one-row change |
| Name the stage you lost and why | Present a flattering number without its denominator |
| Cut any component you cannot tie to a failure it prevents | Build a many-stage pipeline you cannot measure part by part |
| Spend early hours on the output CSV's label agreement as well as on code quality | Treat the output stage as solved because the code stage is strong |

## 9. How to use this in a new edition

1. Hour 0: read the problem statement, list attacker-controlled inputs, write the generalization policy and failure-mode table.
2. Build the typed seam and the pure decision layer with tests before the first paid model call.
3. Cache perception; iterate on rules offline; run the paid re-derivation rarely and log the diff.
4. Add the guards that fit the task: citation guard for retrieval, grounding tests for vision, fencing and rule `reads` sets for adversarial text.
5. Before freezing: measure the noise floor, run the ablations, run the self-score gate, and rehearse the interview from the decision log.
6. Reserve explicit time late in the build for the output stage: look at where the model's default proposal sits relative to class boundaries, since that is the usual place top finishers lose points.
