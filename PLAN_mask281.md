# Plan: cited-cell masking on 281 label-balanced MHIST test tiles (gemma-4-31b, second host)

Status: written 2026-11-22, before any call of this experiment. Frozen when its sha256 is pushed to the public timestamp repo. Any change after
that is a dated amendment made before the first call. Results are reported whichever way they fall. No tile name appears in this file.

## Purpose
The cited-vs-control masking effect appeared on the 319- and 427-tile sets (102 vs 49, 130 vs 79) and not on the earlier 185-tile label-balanced
set (54 vs 47). This run asks whether it appears on a new label-balanced set drawn from the whole test split, with the 427 replication's method.

## Tiles
- 281 MHIST test tiles: 140 HP and 141 SSA by majority label (annotations.csv, sha256 231192b6...), drawn by `random.Random("20261122:<class>").shuffle`
  over the sorted test-partition names of each class; the first 140 HP and 141 SSA are primary, the rest of each class is an ordered reserve.
- Tile list file (local, never tracked): `mask281_tiles.tsv` (name, class, primary/reserve, rank), sha256 `4df42cd40a81ea0733d0bb128f3e46ca7703b499591bc3ff6d477488108051c7`;
  primary names only: `mask281_primary.txt`, sha256 `3e7be933a41ad944ee60a3a665a6119cce1b827209f8c0266c6d5c8533720283`.
- Overlap rule: every test tile was drawn in some earlier sweep, so overlap is allowed and reported, never used to select. Of the 281 primary tiles,
  89 are in the 319 set, 134 in the 427 set and 52 in the 185 set. Every tile gets a fresh baseline answer, fresh citations and fresh masks
  in this run; no earlier answer, citation or mask is reused.
- Unusable tiles (no feasible control under the mask.py rules, or no usable baseline): replaced by the next reserve tile of the same class,
  in rank order, until 140/141 usable tiles exist or the reserve is used up. Replacements are counted and reported.

## Method (identical to the 427-tile replication, `run_ext2.sh`)
Host: OpenRouter `google/gemma-4-31b-it`, provider Friendli pinned, fallbacks off, host-default sampling, temperature 1.0. Baseline: `cte_p1`,
one call per tile. Masks: `mask.py --subset 3 --per-tile-seed` (the 3 cited cells covering the most tissue; area-matched control cells disjoint
from them, tissue within 5 points), then `finalize_mask_ext.py`. Masked calls: `run_masked.py` with arms cited and tissue_matched and the three
occlusions mean fill, Gaussian blur, black. Per tile: 1 + 6 calls (about 1,970 calls for 281 tiles plus replacements).

## Analysis (pre-specified)
- Primary: per tile, label changes over the 3 occlusions under the cited mask vs under the control mask; tiles favouring cited vs favouring
  control, ties excluded; exact two-sided sign test, alpha 0.05. Reported with the share of non-tied tiles favouring cited (Wilson 95%).
- Sensitivity: the same test excluding the tiles that are in the 185-tile set.
- Secondary: the same sign test per occlusion (mean, blur, black), Holm across the three.
- Descriptive: by true label and agreement group (cells under 20 tiles marked small).
- Decision rule: "appeared" if the primary p < 0.05 with more tiles favouring cited; "reversed" if p < 0.05 favouring control; "did not appear"
  otherwise. The result is reported in the paper either way, beside the 319, 427 and 185 results, with the post hoc tile-set analysis labelled exploratory.

## Cost stop
Estimate about $0.45 at Friendli prices (measured MHIST second-host token counts); hard stop at $3.00 of API-reported cost. If the stop is hit,
the analysis uses tiles with all 7 calls and says how many.
