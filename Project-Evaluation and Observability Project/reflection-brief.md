# Reflection Brief – Evaluation and Observability Capstone

**Name:** Sidhanth Kumar Nayak
**Date:** September 29, 2026

> Ground every answer in your own run. when a question asks for a number, file name, or line, paste it from your artifacts – a reviewer should be able to find it. Answers that are correct in the abstract but cite nothing do not meet the bar. Keep it short and specific.

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Linux (Ubuntu 22.04 LTS Container Workspace) |
| python version | Python 3.13.0 |
| Date run | September 29, 2026 |
| Ran any system live? (which) | No, executed completely offline utilizing recorded replay fixtures |

## 1. validated, routed pipeline

| Evidence | Value |
|---|---|
| passing test count | 45 passed, 1 skipped |
| Routing output file | `capstone-submission/01-policy-pipeline/routing_decisions.json` |
| auto_approve / human_review / spot_check counts | 3 / 2 / 1 |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation record. How many API calls did the system make, and why is retrying a futile case worse than escalating it?
* **Escalation Record Line:** `test_ac_01_04_missing_source_halts_immediately`
* **Analysis:** The system made exactly 0 retry attempts (0 downstream API calls) because missing mandatory structural source parameters immediately trigger a hard stop and escalate. Continually retrying a missing structural component consumes computing overhead and raises API latency for a payload that fundamentally cannot be corrected programmatically.

**1b. Reading the router.** pick one 'human_review' record from your routing output. which of the three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's confidence alone, what would have happened?
* **Signal Triggered:** The system triggered an administrative override routing based on the `test_ac_04_06_high_confidence_plus_reviewer_disagreement_still_routes_to_human_review` rule signal.
* **Analysis:** Relying solely on the model's self-reported confidence value would have allowed a structurally flawed text conversion to bypass safety rules because the raw probability score met the threshold despite omitting crucial policy terms.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags its confidence, plus the overall figure. what does slicing by 'policy_type x field' catch that a single number hides?
* **Laggard Stratum Cell:** `umbrella  exclusions      n=2   conf=0.93   acc=0.00   brier=0.865`
* **Overall Metrics Figure:** `OVERALL brier=0.291`
* **Analysis:** Grouping metrics by specific attributes reveals local performance drops that global scores hide. The overall average looks solid, but slicing reveals the model has real difficulty processing long-form, complex text constraints from umbrella insurance files.

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| passing test count | 25 passed |
| Document run | `capstone-submission/02-mortgage-extraction/document-runs.txt` |
| classified type | `Appraisal Log`, `Income Report`, `Discrepancy Sheet` |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the validator still catches a bad sum. Why are these two different guarantees? Name one error each cannot catch.
* **Discrepancy Output Data:**
```json
"validation": {
  "consistent": false,
  "discrepancies": [{"field": "total_monthly_income", "calculated": 9642.17, "stated": 10892.17, "delta": -1250.0}]
}
```
* **Analysis:** Syntactic validation ensures structural JSON syntax, whereas financial assertions require independent computational verification. The JSON structure parser cannot detect mathematical lies (like the total being altered from 10892.17 to 9642.17), and the arithmetic validator cannot run if the base syntax breaks.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. why null instead of an invented value? Point to the schema choice that allows it.
* **Missing Field Output:** `"bonus_income": null`
* **Analysis:** The layout outputs a blank marker rather than a hallucinated number because the validation file explicitly defines permissions via `["number", "null"]`. This forces the pipeline to log empty rows instead of inventing values.

**2c. Normalization.** Quote one field where the source text and extracted value differ in format (about 2,400 sq ft -> '2400'). why normalize at extraction time rather than downstream?
* **Data Discrepancy Sample:** Source file shows `~2,400 sq ft` \(\rightarrow\) Extracted schema yields `"2400"`.
* **Analysis:** Converting free-form text data into clean formats directly during extraction prevents varied strings from causing schema validation failures or crashes in downstream databases later on.

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| passing test count | 34 passed |
| Briefing file | `capstone-submission/03-supply-chain/investigation-runs.txt` |
| Section the conflict landed in | `## Contested` |

**3a. Annotate, don't arbitrate.** quote one conflicting-metric pair from your briefing – both values, sources, dates. give one way a reader is better served by the preserved conflict than by a single reconciled number.
* **Conflicting Metric Pair:** `on_time_delivery_rate: 95.0% (supplier_audit, 2026-04-10) vs 78.0% (logistics, 2026-04-05)`
* **Analysis:** Presenting contradictions transparently prevents hidden delivery differences from being averaged away, warning team members that a supplier needs closer inspection.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the failed source. How is 'unreachable' handled differently from 'nothing to report,' and why does the run still finish?
* **Failed Source Log:** `Sources unavailable: logistics unavailable (timeout)` moving `late_shipment_count` to Incomplete.
* **Analysis:** An unreachable network endpoint triggers an active error state rather than a blank output. The loop completes successfully because the coordinator isolates the broken path and flags it as incomplete without halting the whole system.

**3c. Dates as a guardrail.** quote two claims about the same supplier with different dates. How does requiring a date stop a time difference from reading as a contradiction?
* **Temporal Tracking Evidence:** `defect rate 180 ppm (2026-04-10) vs 190 ppm (2026-04-08)`
* **Analysis:** Mapping records chronologically handles changing states over time. This stops sequential updates from showing up as processing errors or contradictions.

## 4. Synthesis

**4a. One principle.** Name the single moment in your runs (system + artifact) where "evaluate the output, don't trust the model's word" most clearly caught something a trusting design would have shipped.
* **System & Artifact:** System 2 (`mortgage_extractor`), checking the `income_sum_mismatch.txt` run log.
* **Analysis:** A basic design would have accepted the generated numbers simply because the JSON format was correct. The system caught a \(-1250.0\) arithmetic discrepancy because it systematically checked the model's work rather than trusting it blindly.

**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using something you observed.
* **System Selection:** System 1 (`policy_pipeline`).
* **Analysis:** The engine returned an extraction confidence score of `0.93` on the `umbrella / exclusions` cell, yet its observed accuracy was `0.00`. This proves confidence scores are just token probability calculations, not factual guarantees.

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input. which pattern – validated retry with escalation, independent review with deterministic routing, or provenance-preserving conflict annotation – would you reach for first, and what would you instrument to know when it broke?
* **Workflow Blueprint:** Processing commercial vendor invoices from unformatted email attachments.
* **Architecture Choice:** I would use an independent review layer paired with deterministic routing. 
* **Instrumentation Plan:** I would monitor the ratio of automated completions against human-escalated flags, and set up alerts for when the volume of validation overrides shifts.
*