# Perturbation Log

## 1. Policy Pipeline: Low-Confidence Routing Fallback
- **Target System**: Insurance Policy Extraction Pipeline (`01-policy-pipeline`)
- **Perturbation**: Injected an ambiguous extraction result where field confidence dropped below the threshold or reviewer disagreed on coverage limits.
- **Expected Behavior**: Deterministic routing catches the discrepancy or low confidence and rejects auto-approval.
- **Observed Behavior**: Routed to `HUMAN_REVIEW` queue (`test_ac_04_02_low_confidence_routes_to_human_review` & `test_ac_04_06_high_confidence_plus_reviewer_disagreement_still_routes_to_human_review` PASSED). Decision logged into routing JSON.

## 2. Mortgage Extraction: Mathematical Total Discrepancy
- **Target System**: Mortgage Document Extraction (`02-mortgage-extraction`)
- **Perturbation**: Tested `fixtures/documents/income_sum_mismatch.txt` where individual income lines ($5,416.67 + $1,250.00 + $2,140.00 + $385.50 + $450.00) sum to $9,642.17, while the document stated total is $10,892.17.
- **Expected Behavior**: Mathematical consistency validator flags the mismatch rather than silently accepting stated totals.
- **Observed Behavior**: Extraction marked `consistent: false`. Discrepancy logged for `total_monthly_income` with `calculated: 9642.17`, `stated: 10892.17`, and `delta: -1250.0` (as verified in `discrepancy-run.txt`).

## 3. Supply Chain Investigation: Logistics Source Timeout
- **Target System**: Multi-Source Supply Chain Synthesis (`03-supply-chain`)
- **Perturbation**: Executed investigation using `--simulate-timeout` on supplier `meridian` to simulate logistics API failure.
- **Expected Behavior**: Coordinator handles source timeout gracefully without failing the entire synthesis.
- **Observed Behavior**: Logistics source marked as `unavailable (timeout)`. Synthesis degraded gracefully to single-source metrics (`average_lead_time_days`) and retained corroborated findings from `supplier_audit` and `internal_quality` (as verified in `timeout-run.txt`).
