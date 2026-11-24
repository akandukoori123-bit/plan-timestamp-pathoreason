# Plan: meaning-preserving control edit for the text-edit test (gemma-4-31b, second host)

Status: written 2026-11-22, before any call of this experiment. Frozen when its sha256 is pushed to the public timestamp repo; changes after
that are dated amendments made before the first call. Results are reported whichever way they fall. No tile name appears in this file.

## Purpose
Editing a few feature words of the model's own explanation to argue the other class changed 83 of 100 labels (replicate 1). This asks whether
any small edit of the same size moves labels, or only edits that change what the explanation claims.

## Tiles and texts
- The 100 tiles of the text-edit test (round 4 `order_edit`), same host. T1 = the tile's own Friendli explain-first explanation placed before
  the label (the original T1 prompt file). T2 = the original opposite-class edit (unchanged, from round 4).
- T3 (control edit), built without any model call by `build_editcontrol.py` (sha256 90ab4583f6fd077932585923fa5f0c93ea3e4cc9e83b452893ba4e5c3795de49): from T1, exactly k words are replaced, where
  k is the number of changed words in that tile's T2 (median 4, range 1 to 11), using a fixed table of wording-only one-for-one synonyms
  ("characteristic"/"typical", "present"/"seen", "within"/"in" ...). Feature words, the frozen edit-table criterion words and class names are never
  changed, so every feature claim and the class it supports are kept; word count is unchanged. Positions are chosen with a fixed seed (20261122).
- **Eligibility rule (defined from word counts only, before any outcome).** A tile is eligible if and only if the number of non-feature, non-protected
  words in its T1 explanation that appear in the fixed synonym table (`SWAPS` in `build_editcontrol.py`) is at least k, the number of changed words
  in its T2. The rule uses only the T1 and T2 texts and the table. No true label, no model answer, no outcome and no earlier flip result is used,
  and none is available to the script. Result: **55 of 100 tiles eligible, 45 excluded.**
- Lists: eligible tiles `editcontrol_eligible.txt`, sha256 `192e00286b6fbbc9f76ee361f1e99677171021b1519fa6b34bc7eec8f9e09c17`; **excluded tiles `editcontrol_excluded.txt` (45 tiles),
  sha256 `56eb87419e9af6c83fd95767b6bc645293d2a3fc90d808c46754afc65dfbb928`**. The 45 excluded tiles will be listed in the results report (as a count by reason, and the list file
  kept locally), and no excluded tile is called under any condition.

## Calls
T3 on the 55 eligible tiles, 3 samples each (temperature 1.0, host-default sampling, Friendli pinned, the runner and parsing of the round-4 test):
165 calls. Concurrency check: T1 re-run on the same 55 tiles, 3 samples (165 calls), to measure drift since round 4. Optional secondary arm:
a same-class-direction edit with matched word count is not run in this plan (no fixed table exists for it; it would need its own plan).

## Analysis (pre-specified)
- Primary: on the 55 eligible tiles, replicate 1: tiles whose T3 label differs from the original T1 replicate-1 label (T3 flip rate, Wilson 95%),
  compared with the T2 flip rate on the same 55 tiles by an exact McNemar test on paired tiles (flip under T2 vs flip under T3).
- Secondary: the T3 flip rate against the 18.7% same-prompt noise of the second host (exact binomial, two-sided); replicates 2 and 3 and the pooled
  165 pairs; T3 vs the T1 re-run (same session).
- **Meaning check and validity rule.** `meaning_check_30.csv` (30 original/control pairs drawn at random from the 55 eligible pairs, A/B order randomised, no labels,
  no model answers) is rated blind by two teammates as same or different meaning. Reported: the share rated "same" by each rater, the share rated "same" by
  both raters, and **Cohen's kappa** between the two raters (with a note that kappa can be low when almost all pairs are rated "same"). **Validity rule: if fewer than
  80% of the 30 pairs (fewer than 24) are rated same-meaning by both raters, the control edit is reported as INVALID and is NOT used for the comparison with
  the 83/100 opposite-class edit** (the T3 flip rate is then reported descriptively only). The rule is applied after the ratings are in and before the primary
  analysis is read; the ratings are collected before any T3 call or, if collected later, the analysis stays unread until they are.
- **Deadline rule.** If the ratings of both raters are not received by **Wednesday 25 November 2026, 11:59 pm ET**, the control is reported as UNVALIDATED:
  its flip rate is given descriptively only (with its Wilson interval), and no comparison with the 83/100 opposite-class edit is claimed. A late rating
  received afterwards is reported as an addendum and does not change that report. If both ratings arrive in time, the validity rule above applies.
- Decision rule: if the control is valid (above), the control edit "moves labels like the opposite-class edit" if McNemar p >= 0.05; "moves fewer labels" if p < 0.05
  with fewer T3 flips; the T3 flip rate is called "within noise" if its Wilson interval contains 18.7%. Reported either way. If the control is invalid, none of
  these statements is made and the report says so. The same holds if the control is UNVALIDATED by the deadline rule.

## Cost stop
Estimate about $0.10; hard stop at $2.00 of API-reported cost.
