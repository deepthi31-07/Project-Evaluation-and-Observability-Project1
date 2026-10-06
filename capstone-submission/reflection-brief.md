# Reflection Brief: Evaluation and Observability Project

## 1. Policy Routing & Calibration
- **Passing test count:** 48 passed (from policy pipeline test runs).
- **Selected record:** POL-2025-002 (spot_check).
- **Analysis:** High confidence does not equal semantic correctness. Calibration slicing revealed hidden failures, particularly around the umbrella exclusions slice ($n=2$, confidence $0.93$, accuracy $0.00$, Brier $0.865$), demonstrating that deterministic routing and independent review guardrails are necessary when model-expressed confidence fails to track true error rates.

## 2. Schema-Enforced Two-Pass Extraction
- **Passing test count:** 25 passed in 0.86s (from `02-mortgage-extraction/evidence/mortgage_extraction_output.txt`).
- **Document run:** `income_missing_bonus.txt`.
- **Classified type:** `income_verification`.
- **Missing-field output:** `bonus_monthly=null` and `bonus_ytd=null` (from `document-runs.txt`).
- **Analysis:** The schema and prompt allow `null` for unstated fields, so the extractor preserves document silence instead of fabricating zero or hallucinating missing values. Validation checks explicitly flagged discrepancies (e.g., total monthly income discrepancies with calculated vs stated deltas).

## 3. Multi-Source Synthesis & Observability
- **Conflict identified:** `on_time_delivery_rate` = `95.0 percent` from `supplier_audit` versus `78.0 percent` from `logistics` (from `investigation-runs.txt`).
- **Timeout handling:** Sources unavailable: `logistics unavailable (timeout)`.
- **Analysis:** The briefing preserves both contradictory values with proper attribution and degrades gracefully when logistics sources time out, rather than failing silently or assuming absence equates to zero risk.

## 4. Reliability Principle Application
Across all three systems, the core reliability principle applied is **explicit attribution of uncertainty and source degradation**. In production agentic systems, silent failures or hallucinated defaults are more dangerous than explicit missing-source annotations or circuit-breaker timeouts.
