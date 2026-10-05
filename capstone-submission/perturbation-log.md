Perturbation Log
System 1 — Validated, Routed Pipeline
Change made (file + what I changed): policy_extractor/routing.py — changed the auto-approval confidence threshold from 0.85 to 0.99.

Command run: .venv/bin/pytest tests/test_us04_routing.py

What I predicted: Setting the threshold to a strict 0.99 would cause high-confidence extractions (e.g., scores around 0.90) to fail auto-approval and be forced into the human-in-the-loop (HITL) review queue.

What actually happened (key output line): FAILED tests/test_us04_routing.py::test_routing_sends_low_confidence_to_hitl - AssertionError: assert 'hitl_review' == 'auto_approve'

How this differs from the unperturbed run: Normally, standard extractions with confidence scores around 0.90 satisfy the default threshold and bypass manual routing. Under the perturbation, valid items are rejected from auto-approval due to the artificial tightening of the gate.

System 2 — Schema-Enforced Two-Pass Extraction
Change made (file + what I changed): policy_extractor/reviewer.py — temporarily omitted "premium_amount" from the required review schema fields list.

Command run: .venv/bin/pytest tests/test_us03_review.py

What I predicted: The reviewer tool schema sent to the model would no longer request or validate a judgment for premium_amount, triggering a key lookup or assertion failure in the downstream agreement logic.

What actually happened (key output line): FAILED tests/test_us03_review.py::test_reviewer_parses_per_field_judgement - KeyError: 'premium_amount'

How this differs from the unperturbed run: Normally, all critical financial fields are strictly tracked in the per-field review agreement dictionary. Removing the field from the validation schema breaks expectations in helper functions mapping response tuples back to the review record.

System 3 — Multi-Source Synthesis
Change made (file + what I changed): policy_extractor/batch.py — modified the batch sampling function parameter to request a sample_size of 0 during dry-run generation.

Command run: .venv/bin/python -c "from policy_extractor.batch import dry_run_sample; print(dry_run_sample(None, [], sample_size=0))"

What I predicted: Passing a sample size of zero would bypass the evaluation loop entirely, returning an empty list instead of processing records or throwing a division-by-zero error.

What actually happened (key output line): []

How this differs from the unperturbed run: An unperturbed dry-run sample processes active policies (e.g., default size 2+), computing aggregate accuracy, parser success rates, and calibration metrics over real outputs. Setting the sample size to zero short-circuits the pipeline gracefully with zero output records.