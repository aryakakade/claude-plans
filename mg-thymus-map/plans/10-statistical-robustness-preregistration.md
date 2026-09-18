# Pre-registration — Phase 7: statistical robustness of the headline results

**Written 2026-09-18, before any robustness statistic was computed. Frozen on commit.** Changes only
as dated addenda in §7. **Read-only on every earlier result; additive and deletable**
(`phase7_robustness_*` files, its script, its run log, this document). No earlier number, file or
outcome is changed by anything here — including if a headline turns out fragile. The user's
standing rule holds: the official Sjögren's and SSc results are read, never rewritten.

## 1. Why

The user (2026-09-18): "we haven't cross validated our work … rely heavily on p-values, confidence
intervals, or FDR corrections." Gene-level tests are already BH-corrected within their families,
but three things are missing: (a) a correction across the project's family of *primary* tests,
(b) patient-level confidence intervals and effect sizes for the headline tests, and (c) resampling
checks that no single patient or donor drives a result. The machine-learning sense of
cross-validation does not apply (no predictive model is claimed); for hypothesis tests the
equivalents are leave-one-out resampling, bootstrap intervals, and replication on held-out samples.

## 2. The primary-test family (fixed now)

Ten tests — every primary test the project has reported, whatever its outcome:
Aim 1 (patients vs 4-donor tonsil, one-sided exact permutation) for Sjögren's (p = 0.003), RA
(0.625), SSc (0.676), PF (0.241), UC (1.000); Phase 4 composite interferon (one-sided
Mann-Whitney) for Sjögren's (0.007), SSc (0.002), PF (0.086), UC-inflamed (0.781) — RA was
underpowered and has no p; Phase 6 coupling primary (0.0048). **Correction: Benjamini–Hochberg
(FDR q < 0.05) and, as the stricter check, Bonferroni (α = 0.05/10) across all ten.**

## 3. Robustness checks for the four headline results

Headlines: **H-A** Sjögren's Aim 1; **H-B** Sjögren's Phase 4; **H-C** SSc Phase 4; **H-D** Phase 6.

1. **Leave-one-out (LOO).** Re-run each headline test dropping each unit once: every disease
   patient, every comparator patient, and (H-A) every tonsil donor. For H-D, drop each patient
   (110 re-runs) and each whole dataset (5 re-runs). Report the fraction of LOO runs with p < 0.05
   and the largest LOO p. Permutation counts: exact where feasible, else 10,000 (seed 2026).
2. **Bootstrap 95% confidence intervals, patient level** (10,000 resamples, seed 2026, percentile
   intervals): H-A — difference in medians of per-patient scores (Sjögren's minus tonsil donors),
   resampling patients and donors separately; H-B, H-C — difference in medians of the composite
   interferon score (disease minus comparator) and the rank-biserial effect size; H-D — Spearman ρ,
   resampling patients within each dataset and recomputing within-dataset ranks.
3. **Held-out replication of H-A.** Score the TLS-region cells of the four tonsil donors that were
   never part of the reference (8-donor object: BCLL-2-T, BCLL-8-T, BCLL-9-T, BCLL-13-T) with the
   unchanged 25-gene map; re-run the Sjögren's Aim 1 test against them (one-sided exact
   permutation). The same test is reported for the other four diseases, for completeness.

## 4. Robustness verdicts (fixed now)

- **Survives multiplicity:** BH q < 0.05 across §2's family (Bonferroni reported alongside).
- **LOO-robust:** p < 0.05 in every LOO run; **mostly robust:** in ≥ 80%; otherwise **fragile**.
- **CI:** interval excludes zero (or ρ interval excludes 0).
- **H-A replicates** if p < 0.05 against the held-out donors.
Each headline gets all four verdicts reported, favourable or not.

## 5. Not allowed

No other test family, no dropping a test from §2, no other LOO scheme, no other held-out donors,
no reinterpretation of any earlier outcome. A fragile verdict is reported as fragile.

## 6. Outputs

`data/processed/phase7_robustness_{fdr,loo,bootstrap,heldout}.csv`; script
`scripts/run_phase7_robustness.py`; log `docs/phase7_robustness_runlog.md`.

## 7. Addenda (dated; the text above is frozen)

**Addendum 1 — 2026-09-18 23:10 IST, OUTCOME and two findings made while running.**
(a) *Statistic.* Phase 1's Sjögren's Aim 1 test used the pooled-cell median (p = 0.00303); later
diseases used the median of per-patient medians (Sjögren's on that statistic: p = 0.0121). Both
are patient-level permutation tests; H-A is reported under both. (b) *Scoring pitfall in the new
scripts.* decoupler's AUCell drops genes empty across the cells it is given, so scores depend on
scoring the whole object; the first run of §3.3 (held-out) scored a TLS-only subset. Whole-object
scoring reproduces Phase 1 exactly (every patient median to 0.0, p = 0.00303). Corrected and re-run.
**Outcome:** all four headlines survive BH FDR across the ten primary tests (q 0.015–0.018; 0.021–
0.030 if Sjögren's Aim 1 uses p = 0.012); SSc Phase 4, Phase 6 and Sjögren's Aim 1 (pooled-cell)
also survive Bonferroni; Sjögren's Phase 4 does not (0.070). Every headline is LOO-robust (largest
LOO p 0.0038–0.033). Every bootstrap CI excludes zero (H-A barely: +0.0001 to +0.044). H-A
replicates against held-out tonsil donors (p = 0.012 pooled-cell; 0.046 per-patient-median).
Full record: `mg-thymus-map/docs/phase7_robustness_runlog.md`.
