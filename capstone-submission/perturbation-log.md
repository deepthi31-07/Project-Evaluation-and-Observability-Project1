# Perturbation Log: Evaluation and Observability Systems

## System 1 — Policy Extraction & Routing Pipeline
- **Change made:** Edited the routing configuration threshold in `policy_extractor/routing.py` from `0.85` to `0.99`.
- **Command run:** `pytest 01-policy-pipeline/tests/`
- **Observed result:** Assertion failure; records with high confidence (e.g., `0.93`) were incorrectly rejected or forced into routing exceptions due to the overly stringent threshold.
- **Unperturbed contrast:** With the baseline threshold of `0.85`, standard policy records correctly route to `auto_approve` or `spot_check` as expected.

## System 2 — Mortgage Extraction Pipeline
- **Change made:** Edited `fixtures/documents/income_sum_mismatch.txt` so that the stated `TOTAL MONTHLY EARNINGS` no longer matched the sum of line items.
- **Command run:** `mortgage-extract fixtures/documents/income_sum_mismatch.txt` (or via `pytest Project-Evaluation-and-Observability-Project1/capstone-submission/02-mortgage-extraction/tests/`)
- **Observed result:** `validation.consistent = false` with mismatch field `total_monthly_income`, calculated = `9642.17`, stated = `10892.17`, delta = `-1250.0`.
- **Unperturbed contrast:** The clean document `income_missing_bonus.txt` returns `validation.consistent = true` and correctly applies `null` for unstated income fields without triggering discrepancy flags.

## System 3 — Supply Chain Synthesis
- **Change made:** Enabled the source-timeout simulation for the logistics telemetry source (`--simulate-timeout`).
- **Command run:** `supply-chain-investigate meridian --offline --simulate-timeout`
- **Observed result:** Triggered explicit graceful degradation: `Sources unavailable: logistics unavailable (timeout)`, with `late_shipment_count` moved to the `Incomplete` category.
- **Unperturbed contrast:** The standard offline run successfully aggregates supplier audits and logistics telemetry, displaying a `Contested` status for `on_time_delivery_rate` with live divergence tracking.
