# Plan: order test (and conditional text-edit test) of the MedGemma step-4 adapter on 100 label-balanced MHIST test tiles

**Status: FROZEN 2026-11-08 (round 13); D1-D5 defaults applied per the owner's instruction. Nothing in this plan has been run on any test tile.**
Frozen in this order: the three scripts are final and `FROZEN_CODE.sha256` lists them; that file's sha256 is in section 2; the dev results are in
section 11; PLAN.md's sha256 is in `PLAN.md.sha256` and goes to the mentor by email as the outside timestamp. Any change afterwards is a dated
amendment made before the first test call, and none after any test output has been seen.
This file names no tile; the tile list is named by its sha256.

Owner: Aditya Kandukoori. Teammates: Rohan, Sid. Mentor: Kiran Nijjer.

## 1. Question and scope
Does the order effect seen with gemma-4-31b (explain first moves labels toward SSA) also appear in a fine-tuned MedGemma 1.5 4B that
classifies the task above chance? Scope, fixed by the owner: the **same 100 label-balanced test tiles** as the gemma order experiment
(`runs/ordering_controls`; 50 HP / 50 SSA, all test partition), **one greedy pass**, **local backend**, orders `co_p1`, `cte_p1`, `etc_p1`;
the text-edit test (T1/T2) only if section 6 says it is testable. The existing adapter pre-registration
(`PREREG_adapter_test_2026-10-31.md`, 977 tiles) and `heldout_test/PLAN.md` (977-tile draft) are not edited and not run.
Of the 100 tiles, **18** are also among the 20 pilot tiles on which the adapter's prompt `cte_p1` was written and piloted with
gemma-4-31b (counted: 18 of 100); section 5 repeats every test without them (82 tiles).

## 2. Frozen inputs (checked by sha256 at run time; `run_heldout.py` refuses a test run on any mismatch)
- Tile list: `d56ee265137924a185e257d76ff616e3e642f81769e112c2f8fe630ea24a04b8` (100 names, sorted, LF; local file, never tracked).
- Adapter `adapter_model.safetensors` `13b0dbb0a80767f698bbc2f851db6c583cd00ec5b8b6dbe719a1f9f18f854444`; `adapter_config.json`
  `cfc34f0360f0082518438b2f693cc0074a984086206c2ee9189e07d8f0e02b9f`; GGUF LoRA (llama.cpp's converter, f32) sha256 `02360acf...fb37`.
- Base `google/medgemma-1.5-4b-it` revision `91850547d9f0b2fdd21aa7c5f4f3d1a8a52c243b`; served from the Q8_0 GGUF with its f16 mmproj
  (llama.cpp with the macOS-13 Metal patch). **This is not the reference numerics** (the adapter was trained and dev-evaluated in
  4-bit nf4 through transformers/peft). **Decision rule (owner, 2026-11-08), met:** C1 (classify-only `co_p1`, local llama.cpp backend, 100 dev tiles vs Rohan's recorded tier-2 dev labels, a2397c4): label agreement 97.0%, dev accuracy 78.0%. Both clear the thresholds (agreement >= 90%, accuracy >= 75%), so the local llama.cpp backend is the **primary backend** and the results are not
  labelled exploratory for the backend (`LOCAL_BACKEND_PRIMARY = True` in `analyze_heldout.py`). The numerics still differ from the reference, which is a stated limitation.
- Prompts: `cte_p1` `eb0e4d85b580cebad7c1733b4d5e80bee6b28c322eb48032da169b7a7917654f`, `co_p1`
  `80c1ce121fc9b4ebdcbc8a4fff2b3c1c2c7ceb84e758505ce636d3c4d3a9a3a7`, `etc_p1`
  `02a6f2045dd2e8fcce1c95bfab4c13fdd549e9c123c278637860a0eac16a4f3b` (token-identical to the official processor; checked on every call).
- Render: the training bundle's (Menlo, 224 px, 4x4 grid); the tiles are rendered here and checked byte-for-byte against the bundle manifest.
- Code: `FROZEN_CODE.sha256` sha256 `b8b37948ec881569d148dd4616c091b9d66c26133bb247677621f7005f52d578`.
- Truth file (read only by `analyze`, after sealing): `annotations.csv` sha256 `231192b6e5fc46c8224e8850ef9882c917bf3eca58c7e211a58fb695bf002605`.

## 3. Conditions and decoding
Conditions `co_p1`, `cte_p1`, `etc_p1`, plus `ins_own` / `ins_own_edit` only if section 6 passes. 100 tiles x 3 = 300 calls (+ 200 if the edit test runs).
**Greedy** (temperature 0), `max_new_tokens` 1,500, the thinking opener (token 100) banned, no system prompt, no constrained grammar,
batch size 1, `K = 1` (a deterministic backend gives one answer per tile and condition; repeated calls were byte-identical in the dev
dry run, so the gemma noise-floor and permutation analyses have no counterpart here).
**Explain-first cap (C3, section 11):** **Decoding-only JSON-schema grammar** (`run_heldout.py --json-schema`, function `output_schema`): the answer must be the JSON object the prompt asks for, in that prompt's key order (`co_p1`, `cte_p1`: label, confidence, evidence; `etc_p1`: evidence, label, confidence), at most 8 evidence entries from the 8 allowed feature terms, `co_p1` with an empty evidence list. It only ends the repetition loops of explain-first answers; it does not choose the label. A stop string alone was tried first and was not enough (section 11). Because the grammar fixes key order per prompt, the manipulation check of section 5 is met by construction; the order test therefore rests on the prompt text, and the label is still free.. Whatever is chosen applies to all three conditions and does not change
the adapter, the prompts or their hashes.
No answer is repaired, retried or re-drawn; a transport fault is retried and never scored.

## 4. Scoring
Usable label: `comp_run.parse` returns HP or SSA (a complete JSON object with a `label`). Lenient label (sensitivity only): the first
`"label": "HP"|"SSA"` in the text. Truth (analysis only, after sealing): majority-vote label. Always-HP baseline on these 100 tiles: 50%.

## 5. Order test (mirrors the gemma analysis: `order_analysis.py`, exact McNemar on discordant tiles)
Direction as in ANALYSIS.md: HP->SSA means the first named condition said HP and the second said SSA.
- **Primary test (O3): `cte_p1` -> `etc_p1`** (gemma, same host replicate 1: 36 vs 1; Cerebras 41 vs 0). n = tiles usable in both.
  b = HP->SSA, c = SSA->HP. **Exact two-sided McNemar, alpha = 0.05.** Reported with it: flips, flip rate (Wilson 95%), net SSA shift
  (Newcombe method-10 paired interval, continuity-corrected phi; percentile tile bootstrap, 10,000 resamples, seed 20261031).
  Decision fixed now: **order effect appeared** if p < 0.05 and b > c; **reversed** if p < 0.05 and b < c; **did not appear** otherwise
  (read as "equivalent to no shift" only if the Newcombe 90% interval lies inside +/-10 points, else "inconclusive").
  Minimum data: fewer than 90 tiles usable in both -> "not testable" (numbers still given).
- **Manipulation check (gates the reading):** share of `etc_p1` answers with at least one evidence entry before the label (Wilson);
  below 90% -> a prompt effect, not an order effect.
- **Secondary:** O1 `co_p1` -> `etc_p1`, O2 `co_p1` -> `cte_p1` (same statistics, no multiplicity correction).
- **Accuracy per order** (all 100 tiles, unusable counted wrong, Wilson 95%, exact one-sided test against 50%; balanced accuracy,
  recall by class), and the **SSA-call rate** of each order with the net **SSA-rate shift** of `etc` and `cte` over `co`.
- **Without the 18 pilot-overlap tiles:** every test above (O1-O3, accuracy, SSA rate) repeated on the other 82 tiles
  (`excluding_pilot_overlap` in the sealed results).
- **Sensitivity:** lenient labels; a 3 x 3 table with "unusable" as a third outcome; `length` counts; new tokens.
- **Replication standard for the paper (primary backend per the C1 rule):** the effect "also appeared" in MedGemma if O3 is significant with b > c on the 100 tiles; the
  82-tile result is reported beside it and, if it disagrees, the disagreement is stated.

## 6. Text-edit test (included only if editable)
Decision rule: included only if at least half of the 20 dev explanations from the C2 dry run are editable by the frozen `edit_text.py` phrase
tables in the same way as in the gemma edit test (T2 differs from T1 and passes the acceptance rule). **Result (section 11): 4 of 20 dev explanations were editable (below half), so the text-edit test is NOT included for MedGemma.**
If it had been included: T1 = the adapter's own `etc_p1` explanation placed before the label, T2 = the same text minimally edited to argue the opposite
class; compared answer: the `co_p1` label; primary test: exact sign test on tiles whose label followed the edit (as `PREREG_order_edit.md`),
minimum 50 eligible tiles, else "not testable" with exclusions by reason. The paper says so with the dev counts (16 of 20 explanations contained no HP/SSA feature words the frozen edit tables could reverse).

## 7. Missing labels
An unusable answer counts as wrong for accuracy; a pair test uses tiles usable in both conditions and the drops are counted per
condition. No tile is excluded for any reason involving a true label or the outcome of a compared condition. If more than 10 tiles are
unusable in any one condition, that is reported first and the 3 x 3 table becomes a co-primary display.

## 8. Analysis order (results are sealed before any label is read)
Stage 1 order conditions (and stage 2/3 edit conditions if section 6 passes); `analyze_heldout.py seal` (label-free results, writes
`SEAL.json` with sha256 of every result file, PLAN.md and the scripts); the seal's sha256 is emailed to the mentor before step 3;
`analyze_heldout.py analyze --labels annotations.csv --seal-timestamp-ref "<where and when>"` refuses unless every sealed file is unchanged,
then adds the label-dependent results. Public output: aggregates only, no per-tile list, no tile name.

## 9. Owner decisions D1-D5 (recommended defaults, applied here)
- **D1 Amendment A2 (more than one step-4 configuration on test tiles):** approved for this plan; the existing `competence/PLAN.md` is not edited;
  the analyses are labelled exploratory under it.
- **D2 Overlap with the pre-registered H2:** this plan replaces H2 (H2 is not run); one runner, one backend, batch size 1.
- **D3 Where it runs:** this Mac only (llama.cpp, local backend). No test tile leaves this machine; not Kaggle, not Rohan's machine.
  Consequence: the llama.cpp numerics (section 2); primary under the C1 rule. If Rohan's runner becomes available it replaces the local backend, by amendment, before any test call.
- **D4 Rohan's confirmation of no prior test exposure:** not required to run (the audit found none in the repository, the Kaggle outputs or
  his branch); stated as a limitation, and requested from him in writing before the paper is submitted.
- **D5 Timestamp route:** email of PLAN.md and PLAN_nctcrc.md with both sha256 to the mentor (recommended and sufficient here); OSF
  registration if time allows, same files.

## 10. Known differences from the gemma tests
Greedy K = 1 instead of temperature 1.0 and K = 3; one deterministic answer per tile, so no resampling noise band; a different model
class (4B, fine-tuned for this task); backend is llama.cpp Q8 + LoRA (primary under the C1 rule, not the training numerics); 100 tiles (the McNemar power for a 36 vs 1 type effect is high, for a
10 vs 3 type effect it is low); 18 of 100 tiles were used to write the prompt (section 5, 82-tile repeat).

## 11. Dev checks done before freezing (2026-11-08, dev tiles only, local backend)
- **C1** (classify-only `co_p1`, local llama.cpp backend, 100 dev tiles vs Rohan's recorded tier-2 dev labels, a2397c4): label agreement 97/100 = 97.0% (Wilson 95% CI 91.5 to 99.0), dev accuracy 78/100 = 78.0% (68.9 to 85.0; Rohan's records 79/100 on the same tiles); the same numbers under the final grammar decoding. The rule is met, so the local backend is primary (section 2).
- **C2 (20 dev tiles, all three orders, final decoding):** finished (60 of 60 calls); 20 of 20 labels valid in every order.
- **C3 (explain-first cap):** with no stop string, 3 of 4 `etc_p1` answers had a usable label and all 4 ran to the 1,500-token cap (run-on repetition). With a stop string alone, 15 of 20 (75%) had a valid label: five answers repeated evidence entries 32-47 times until the cap. **With the grammar: 20 of 20 (100%) reach a valid label, all ended by themselves (mean 173 new tokens).** Adapter and prompt hashes unchanged.
- **C4 (editability, the 20 dev `etc_p1` explanations):** 4 of 20 eligible (20%); 16 of 20 excluded because the explanation held no HP/SSA criterion term the frozen edit tables can reverse (T1 rule reading "unclear"). Below half: edit test dropped.
- **Notes:** the C1 runs and the first dev runs used the system Python without the tokenizer check; the final-decoding runs above all passed the token-identity check (prompt token ids equal the official processor's, 120 of 120 calls).

## 12. Reporting commitment
The result is reported either way, in the paper and the report, with the primary test, the manipulation check, the accuracy per order and the
82-tile repeat. No condition, tile or analysis is dropped after seeing results. Anything outside this plan is a section labelled exploratory.

## 13. Projected run time (local backend, grammar decoding, dev20 timings)
Per tile (co + cte + etc): about 116 s; 100 test tiles: about 3.23 h; the text-edit test is not run. Timings: co_p1 24 s mean over 20 calls (0 cut at the cap); cte_p1 44 s mean over 20 calls (0 cut at the cap); etc_p1 48 s mean over 20 calls (0 cut at the cap).
