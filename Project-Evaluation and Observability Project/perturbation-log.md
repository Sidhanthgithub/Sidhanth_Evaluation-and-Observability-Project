# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run it, and record what actually happened. See the starters in the Instructions, or design your own (your own experiment earns more credit).

## System 1 - validated, routed pipeline

* **Change I made (file + what I changed):** Open a sample policy text document inside `data/policies/` and manually deleted the mandatory structural policy header and core declarations block entirely.
* **Command I ran:** `.venv/bin/policy-extractor pipeline data/policies/ --routing-out routing_decisions.json`
* **What I predicted:** The pipeline would immediately trigger a hard stop validation error and escalate the file without wasting computing overhead on retry loops.
* **What actually happened (paste the key output line):** `test_ac_01_04_missing_source_halts_immediately` (The system logged a critical data failure and made exactly 0 retry attempts / 0 downstream API calls).
* **How this differs from the unperturbed run:** An unperturbed run completes smoothly on the very first pass, matching metadata parameters successfully without invoking immediate escalation routing.

---

## System 2 - schema-enforced two-pass extraction

* **Change I made (file + what I changed):** Created a modified copy of the mortgage text document inside `fixtures/documents/` and manually edited a declared monthly income total string value, altering the number from `10892.17` to `9642.17`.
* **Command I ran:** `.venv/bin/mortgage-extract fixtures/documents/income_sum_mismatch.txt --mode replay`
* **What I predicted:** The schema extractor would output valid structural JSON, but the secondary validation pass would catch the math difference and explicitly flag a consistency failure.
* **What actually happened (paste the key output line):** 
```json
"validation": {
  "consistent": false,
  "discrepancies": [
    {
      "field": "total_monthly_income",
      "calculated": 9642.17,
      "stated": 10892.17,
      "delta": -1250.0
    }
  ]
}
```
* **How this differs from the unperturbed run:** Clean documents with matching math records (like `appraisal_informal_sqft.txt`) output `"consistent": true` with a completely empty discrepancies array.

---

## System 3 - multi-source synthesis

* **Change I made (file + what I changed):** Simulated an API dependency failure on the `meridian` company dataset by executing the network fault-injection timeout flag.
* **Command I ran:** `.venv/bin/supply-chain-investigate meridian --offline --simulate-timeout`
* **What I predicted:** The coordinator system would log a graceful partial degradation state, mapping the missing node explicitly while letting the rest of the run finish.
* **What actually happened (paste the key output line):** 
```text
Sources unavailable: logistics unavailable (timeout)
```
(This moved `late_shipment_count` directly into the `## Incomplete` section, completely removing the conflicting `95% vs 78%` delivery rate timeline from the briefing report).
* **How this differs from the unperturbed run:** The standard baseline run completes with the timeline data intact, displaying the competing metrics side-by-side with distinct source attributions inside the `## Contested` block.
