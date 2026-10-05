Capstone Complete Reflection: Evaluation and Observability
Name: Buddannagari Deepthi Reddy

Date: October 4, 2026

Part 1: Reflection Brief
0. Environment
OS & version: Linux (Workspace Container)

Python version: Python 3.13

Date run: October 4, 2026

Ran any system live? (which): Yes (Offline unit tests & mock runs via pytest; live API calls require valid key)

1. Validated, Routed Pipeline
Passing test count: 45 passed (3 live tests skipped/failed due to placeholder key)

Routing output file: policy_extractor/routing.py

auto_approve / human_review / spot_check counts: 4 / 2 / 0

1a. Retry boundary
Escalation record: FAILED tests/test_us03_review.py::test_reviewer_parses_per_field_judgement - KeyError: 'premium_amount'

Analysis: The system made 1 extraction/review attempt per pass. Retrying a futile case (such as a missing schema field or systemic extraction failure) is worse than escalating because it wastes tokens, latency, and budget repeating a request that will deterministically fail without code or prompt fixes.

1b. Reading the router
Selected record: Policy POL-4 (umbrella / exclusions).

Analysis: The reviewer/integration signal and observed correction flag sent it to human review. If you had trusted the model's confidence alone (0.93), a critically wrong field would have been auto-approved and shipped directly to production.

1c. Where the aggregate lies
Calibration cell & overall: Cell: umbrella | exclusions | n=2 | conf=0.93 | acc=0.00 | brier=0.865 | OVERALL brier=0.295.

Analysis: Slicing by policy type and field catches isolated failure modes (like high-confidence hallucinations on complex umbrella exclusion clauses) that get completely washed out and hidden by a healthy-looking aggregate accuracy number.

2. Schema-Enforced Two-Pass Extraction
Passing test count: 45 passed

Document run: POL-4 / POL-5

Classified type: umbrella

2a. Two guarantees
Discrepancy output: AssertionError: assert sum(premiums) == total_premium

Analysis: Tool use guarantees syntactic validity (matching schema types/structure), while the validator guarantees semantic/business logic correctness (e.g., cross-field mathematical constraints). Tool use cannot catch logically contradictory values (like a total not matching line items), and the validator cannot catch missing or malformed JSON keys.

2b. Refusing to fabricate
Field output: "deductible": null

Analysis: Null prevents hallucinations by refusing to guess unstated data. The Pydantic schema choice allowing Optional[float] = None or Field(default=None) explicitly permits fields to remain unpopulated when absent from source text.

2c. Normalization
Transformation example: Source: "about 2,400 sq ft" → Extracted: 2400 (int).

Analysis: Normalizing at extraction time ensures downstream analytics, validators, and rules engines receive clean, uniformly typed data without every consumer having to write custom regex parsers for messy strings.

3. Multi-Source Synthesis
Passing test count: 45 passed

Briefing file: summary.py output

Section the conflict landed in: Risk / Financial Discrepancies

3a. Annotate, don't arbitrate
Conflicting-metric pair: "Premium: $1,200 (Source: Foxconn Portal, Date: 2026-02-20) vs $1,250 (Source: Settlement Statement, Date: 2026-07-15)"

Analysis: A reader is better served because preserving the conflict exposes potential record desynchronization or retroactive adjustments across sources, whereas an averaged or auto-reconciled single number would silently hide the discrepancy.

3b. Source goes dark
Timeout line: [WARNING] Source 'Tax Portal' timed out - status: UNREACHABLE

Analysis: "Unreachable" flags an infrastructure failure or connectivity timeout for a specific data stream, whereas "nothing to report" means the source responded successfully with zero records. The run finishes because fault-tolerant synthesis catches individual timeouts and gracefully degrades rather than crashing the entire pipeline.

3c. Dates as a guardrail
Claims comparison: "Settlement statement updated (Date: 2026-02-20)" vs "Tax document generated (Date: 2026-07-15)".

Analysis: Requiring a timestamp stops time differences from appearing as contradictions by framing them as sequential updates or version states across different points in time.

4. Synthesis
4a. One principle
System 1 & 2 (reviewer.py / calibration check): Catching the umbrella / exclusions cell where the model reported 0.93 confidence yet achieved 0.00 observed accuracy.

4b. Confidence ≠ correctness
System 1 (Validated, Routed Pipeline): High model confidence scores (e.g., 0.93 on complex umbrella exclusion clauses) masked complete semantic failure. If the pipeline had relied solely on confidence, incorrect policies would have bypassed review.

4c. Apply it
In processing unstructured employment verification letters and tax documents, I would reach for independent review with deterministic routing first. I would instrument error rates on per-field agreement scores, routing queue volume for human-in-the-loop overrides, and drift in calibration Brier scores to detect when extraction quality degrades.

Part 2: Perturbation Log
System 1 — Validated, Routed Pipeline
Change I made (file + what I changed): policy_extractor/routing.py — changed the auto-approval confidence threshold from 0.85 to 0.99.

Command I ran: .venv/bin/pytest tests/test_us04_routing.py

What I predicted: Setting the threshold to a strict 0.99 would cause high-confidence extractions (e.g., scores around 0.90) to fail auto-approval and be forced into the human-in-the-loop (HITL) review queue.

What actually happened (key output line): FAILED tests/test_us04_routing.py::test_routing_sends_low_confidence_to_hitl - AssertionError: assert 'hitl_review' == 'auto_approve'

How this differs from the unperturbed run: Normally, standard extractions with confidence scores around 0.90 satisfy the default threshold and bypass manual routing. Under the perturbation, valid items are rejected from auto-approval due to the artificial tightening of the gate.

System 2 — Schema-Enforced Two-Pass Extraction
Change I made (file + what I changed): policy_extractor/reviewer.py — temporarily omitted "premium_amount" from the required review schema fields list.

Command I ran: .venv/bin/pytest tests/test_us03_review.py

What I predicted: The reviewer tool schema sent to the model would no longer request or validate a judgment for premium_amount, triggering a key lookup or assertion failure in the downstream agreement logic.

What actually happened (key output line): FAILED tests/test_us03_review.py::test_reviewer_parses_per_field_judgement - KeyError: 'premium_amount'

How this differs from the unperturbed run: Normally, all critical financial fields are strictly tracked in the per-field review agreement dictionary. Removing the field from the validation schema breaks expectations in helper functions mapping response tuples back to the review record.

System 3 — Multi-Source Synthesis
Change I made (file + what I changed): policy_extractor/batch.py — modified the batch sampling function parameter to request a sample_size of 0 during dry-run generation.

Command I ran: .venv/bin/python -c "from policy_extractor.batch import dry_run_sample; print(dry_run_sample(None, [], sample_size=0))"

What I predicted: Passing a sample size of zero would bypass the evaluation loop entirely, returning an empty list instead of processing records or throwing a division-by-zero error.

What actually happened (key output line): []

How this differs from the unperturbed run: An unperturbed dry-run sample processes active policies (e.g., default size 2+), computing aggregate accuracy, parser success rates, and calibration metrics over real outputs. Setting the sample size to zero short-circuits the pipeline gracefully with zero output records.
