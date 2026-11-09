# Plan: order test on 100 label-balanced NCT-CRC-HE tiles (gemma-4-31b and GPT-4.1)

**Status: written 2026-11-08 (round 13), before any call of this experiment.** Nothing in it has been run. It is frozen when its
sha256 is taken, written to `PLAN_nctcrc.md.sha256` and emailed to the mentor (the outside timestamp). Any change after that is a
dated amendment made before the first call. No amendment after any result has been seen. No MHIST image is used or sent.

Owner: Aditya Kandukoori. Teammates: Rohan, Sid. Mentor: Kiran Nijjer.

## 1. Question
Does explaining before classifying move labels in one direction on a second tissue dataset, as it did on MHIST? This tests
whether the MHIST order effect depends on that dataset (3.7 of the paper). Both orders are compared as in the MHIST second-host
experiment (`runs/ordering_controls`, host Friendli, K = 3, temperature 1.0).

## 2. Data and inputs (frozen)
- **Dataset:** CRC-VAL-HE-7K, the validation set of NCT-CRC-HE-100K (Zenodo record 1214456, CC BY 4.0), file `CRC-VAL-HE-7K.zip`,
  md5 `2fd1651b4f94ebd818ebf90ad2b6ce06` (checked). 224 x 224 px, 0.5 um/px. Classes NORM (normal colon mucosa) and TUM
  (colorectal adenocarcinoma epithelium).
- **Tiles:** 50 NORM + 50 TUM, drawn with `random.Random(f"20261110:{CLASS}").sample` over the zip's member names (sorted), after
  removing the 100 tiles of the round-10 classify-only pilot (`pilot_manifest.json`, seed 20261108). Pools after exclusion: NORM 703,
  TUM 1,171. The list is `inputs/tiles.json`, sha256 `0d44bcdd7130361db27234ceedbaab2280222e6c016888aee5f11d74bd996dbe`.
  Chance accuracy is 50% by construction.
- **Prompts:** the MHIST rendered prompts `co_p1`, `cte_p1`, `etc_p1` with only the class text replaced (`build_nct_inputs.py`):
  "tile from a colorectal polyp" -> "tile of colorectal tissue"; the two class lines -> `NORM  normal colon mucosa` and
  `TUM   colorectal adenocarcinoma epithelium`; "HP or SSA" -> "NORM or TUM"; the label field. The 4 x 4 grid paragraph, the feature
  list (including MHIST terms such as serration) and the JSON schema are unchanged. sha256: `co_p1` `3d90aed1...1837`,
  `cte_p1` `cff7323d...bdb3`, `etc_p1` `9235122d...ae33` (full values in `inputs/INPUTS.sha256`).
- **Image:** the tile with the same 4 x 4 green grid and cell labels (`grid.draw_grid`, as the MHIST runs).
- **Models (pinned, fallbacks off):** `google/gemma-4-31b-it` through OpenRouter provider Friendli (the MHIST second host) and
  `openai/gpt-4.1` through provider OpenAI. If GPT-4.1 is unavailable at the start of the run, `qwen/qwen3-vl-235b-a22b-instruct`
  (provider DeepInfra fp8) replaces it for the whole experiment and every report says so.
- **Sampling:** temperature 1.0, host-default sampling otherwise, `max_tokens` 1500, no system prompt, no thinking budget;
  a received but unparseable answer is re-asked up to 3 times, then counted invalid (wrong for accuracy; the tile drops out of
  a pair that needs it).

## 3. Conditions and calls
Orders `co_p1` (classify only), `cte_p1` (classify then explain), `etc_p1` (explain then classify). 3 repeats per tile and order.
100 tiles x 3 orders x 3 repeats = 900 calls per model, 1,800 in all, interleaved at random. Cost estimate: `cost_estimate.json`
(about $3.6; worst case $46, so the run has a hard stop at $5.00 of API-reported cost, under the $25 round cap).

## 4. Pre-specified analysis (`analyze_nct_order.py`, self-tested)
A label is the parsed `label`. The direction "toward tumour" is NORM -> TUM (the analogue of HP -> SSA), "toward normal" TUM -> NORM.
1. **Primary test, per model:** exact McNemar (two-sided binomial on discordant tiles) of `etc_p1` vs `cte_p1` at **replicate 1**
   (the comparison reported for the MHIST second host, 36 vs 1). Holm correction across the two models, alpha = 0.05.
2. **Secondary:** the same test at replicates 2 and 3, and on the per-tile majority of the 3 repeats; the TUM-rate shift of `etc`
   and `cte` over `co` (mean of per-tile rates of the 3 repeats, tile bootstrap 95% interval, 5,000 resamples, seed 20261110).
3. **Noise band:** from the 3 classify-only repeats: the mean pairwise flip rate, and the 95% band of the mean per-tile difference
   between two repeats; a shift outside the band is "beyond noise".
4. **Accuracy per order against 50% chance:** replicate-1 tile accuracy with Wilson 95% interval and exact binomial p vs 0.5, also
   the majority-of-3 tile accuracy and the pooled-call accuracy (calls share tiles, so only the tile rows are tests).
5. **Replication rule (stated now):** the order effect "also appeared" for a model if the primary McNemar has Holm-adjusted p < 0.05
   with more flips toward tumour than toward normal (the MHIST direction); "did not appear" if it is not significant; "reversed"
   if significant toward normal. The result is reported for each model whatever it is. A model with accuracy not above 50% (lower
   Wilson bound <= 0.5) in `cte_p1` is reported as "not a working classifier on this task", and its order result is labelled so.
6. Anything not above is exploratory and labelled so.

## 5. Commitments
- The result is reported either way, for both models, with the numbers above.
- All records are kept (`results/*.jsonl`, public-dataset images only). No result is dropped, re-drawn or re-run, except resuming an
  interrupted run with the same command (units already holding a valid answer are not called again).
- If the cap or the credit stops the run, the report states how many calls were made and no test is run on a partial set unless
  every order of a tile has all 3 repeats (the analysis uses tiles with complete data and says how many).

## 6. Known differences from the MHIST experiment
Different tissue and classes (carcinoma vs normal is an easier task than HP vs SSA); the feature vocabulary was written for HP/SSA
and is kept; only two of the MHIST controls (no filler, checklist, copy or edit texts); a different model for the second dataset if
GPT-4.1 is replaced.
