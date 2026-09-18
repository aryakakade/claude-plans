# Pre-registration — Phase 6: does the type-I IFN programme track MG-like TLS within patients?

**Written 2026-09-18, before any coupling statistic was computed. Frozen on commit.** Changes only
as dated addenda in §8. **Additive and deletable** (`phase6_coupling_*` files, its script, its run
log, this document); Phases 1–5 stand as reported. Sjögren's and SSc official files are read only.

## 1. Why

Cufi et al. 2014 (*J Autoimmun*, PMID 24393484) showed type-I interferon (IFN-β) drives CXCL13/
CCL21 and B-cell recruitment in the MG thymus — i.e. type-I IFN is upstream of MG's TLS. Our Phase
4 found a type-I IFN fibroblast programme in Sjögren's and SSc. If the IFN → TLS link generalizes,
**patients whose tissue carries more type-I IFN should have more MG-like TLS-organizing cells.**
This turns a literature bridge into something our own data can support or fail to support.

## 2. Hypothesis

**H1:** within each dataset, a patient's fibroblast type-I IFN composite is positively associated
with that patient's MG-map TLS score. **H0:** no positive association.

## 3. Data — existing per-patient values only, nothing recomputed

- **IFN (x):** `data/processed/phase4_ifn_composite.csv` — per-patient composite (Phase 4 primary,
  fibroblasts, z-scored within each dataset). Lines used: `sjogrens`, `ssc`, `pf`, `ra`,
  `uc_inflamed` (the UC non-inflamed supporting line is not used).
- **TLS (y):** the official Step 4 patient medians, full 25-gene signature, 4-donor-tonsil
  scoring: Sjögren's `step4_pss_vs_sicca_patient_level.csv` (PSS + SICCA); SSc
  `phase3_step4_ssc_patient_level.csv` + `phase3_step4_control_patient_level.csv`; PF
  `phase3_pf_step4_pf_patient_level.csv` + `phase3_pf_step4_control_patient_level.csv`; RA
  `phase2_step4_ra_patient_level.csv` + `phase2_step4_oa_patient_level.csv` (OA IDs matched by
  stripping the `OA_` prefix); UC `phase3_uc_step4_patient_medians.csv` groups `uc_inflamed` +
  `healthy_control`.
- **Unit:** a patient present in both files. Both arms (disease and comparator) are included —
  the hypothesis is about patients, not diagnoses.

## 4. Statistics

- **Within-dataset ranks:** x and y are each converted to percentile ranks within their own
  dataset (removes dataset-level scale and batch differences).
- **Primary:** Spearman correlation of the pooled within-dataset ranks; **stratified permutation
  test** (x shuffled within each dataset, 10,000 permutations, seed 2026), **one-sided,
  positive**. **H1 supported at p < 0.05.**
- **Secondary (pre-specified, not the pass criterion):** (a) the same test **excluding
  Sjögren's**, to ask whether the coupling exists beyond the discovery disease; (b) per-dataset
  Spearman ρ with its own one-sided permutation p, reported descriptively (small n each).
- **No other variants** — no other IFN or TLS definitions, no dropping patients or datasets, no
  covariate adjustment added afterwards, no two-sided switch.

## 5. Known limits, stated now

Small n per dataset (13–30). A patient-level "overall inflammation" confounder could raise both
x and y; this test cannot separate that from a specific IFN → TLS link. The TLS signature
contains IFN-inducible chemokines (CXCL9/10/11), so some shared induction is expected; x is
measured in fibroblasts and y in immune TLS-region cells, which limits but does not remove this.
**This test cannot confirm Aim 1 for any disease and cannot change any earlier outcome.**

## 6. Outcomes

- Primary p < 0.05: "patients' tissue IFN programme tracks MG-like TLS scores across datasets" —
  supports the Cufi-derived bridge as a hypothesis, with §5's limits.
- Primary p ≥ 0.05: "no patient-level coupling detected" — the bridge rests on literature only.
- Secondary (a) reported either way.

## 7. Outputs

`data/processed/phase6_coupling_patients.csv`, `phase6_coupling_results.csv`; script
`scripts/run_phase6_ifn_tls_coupling.py`; log `docs/phase6_coupling_runlog.md`.

## 8. Addenda (dated; the text above is frozen)

**Addendum 1 — 2026-09-18 15:14 IST, OUTCOME.** Run once as specified. 110 matched patients
(Sjögren's 13, SSc 22, PF 19, RA 27, UC 29). **Primary: ρ = +0.258, stratified one-sided
p = 0.0048 → coupling supported.** Secondary (a) excluding Sjögren's: ρ = +0.245, p = 0.0098.
Secondary (b) per dataset, all positive: RA +0.37 (p = 0.032), Sjögren's +0.36 (0.117), UC +0.26
(0.087), PF +0.22 (0.182), SSc +0.08 (0.358). Limits per §5 stand (correlation; possible global-
inflammation confound; IFN-inducible chemokines inside the TLS signature). Record:
`mg-thymus-map/docs/phase6_coupling_runlog.md`, `data/processed/phase6_coupling_*.csv`.
