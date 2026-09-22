# Pre-registration — Phase 9: does the interferon ↔ TLS coupling survive adjustment for depth and overall inflammation?

**Written 2026-09-22, before any partial correlation was computed. Frozen on commit.** Changes
only as dated addenda in §7. Additive and deletable (`phase9_*` files, its script, its run log,
this document). Read-only on every earlier result; no earlier number or outcome changes.

## 1. Why

Phase 6 (ρ = +0.26, p = 0.0048) and Phase 8 (without the map's interferon-inducible genes,
ρ = +0.30, p = 0.0010) are correlations. Both run logs state the same untested caveat: a patient's
**overall inflammation** could raise the interferon score and the TLS score together, and
**sequencing depth** could raise both through better detection. This phase tests that caveat
instead of only declaring it.

**This is a confounder check, not a new finding, and it remains correlational. No causal claim
is made or implied whatever the result.**

## 2. Hypothesis

**H1:** the positive association between a patient's fibroblast type-I interferon score and their
MG-map TLS score persists after adjusting for sequencing depth and overall inflammation.
**H0:** it does not.

## 3. Variables (all per patient, from files already on disk)

- **x — interferon:** `phase4_ifn_composite.csv` composite, unchanged (Phase 6's x).
- **y — TLS:** the official Step 4 patient-median MG-map score, unchanged (Phase 6's y).
- **z₁ — depth:** log₁₀ of the median total UMI counts across **all** of that patient's cells.
  Taken across all cells, not TLS-region cells, because the saved objects for RA, OA, SSc and PF
  do not all carry TLS calls; this is a proxy, stated as such.
- **z₂ — overall inflammation:** the fraction of that patient's cells labelled with an immune
  type (B cells, T cells, Plasma cells, Macrophages, Monocytes, DC, pDC, Mast cells, ILC,
  Myelocytes, Megakaryocytes/platelets) by that dataset's own cell-type column
  (`cell_type_corrected` where present, else `majority_voting`).

Sources: `sjogrens_with_cell_states.h5ad`, `uc_processed.h5ad`, `ra_with_cell_states.h5ad` +
`oa_with_cell_states.h5ad`, `pf_harmonized.h5ad`, `ssc_harmonized.h5ad`. Patients are those
matched in Phase 6.

## 4. Statistic

1. Convert x, y, z₁, z₂ to **percentile ranks within each dataset** (as Phase 6 did).
2. **Residualise:** regress ranked x on [z₁, z₂] by ordinary least squares (intercept included)
   and keep the residuals; do the same for ranked y.
3. **Partial correlation:** Pearson correlation of the two residual vectors — the rank-based
   partial correlation of x and y given z₁ and z₂.
4. **Test:** stratified permutation — shuffle x's residuals within each dataset, 10,000
   permutations, seed 2026, **one-sided positive**. **H1 supported at p < 0.05.**

**Reported alongside:** the unadjusted coupling on the same patients; each control's own
correlation with x and with y (so a reader can see whether the confounders relate to either at
all); the same analysis adjusting for each control separately.

**Multiplicity:** this becomes test 12 in the project family; BH q is reported across all 12.

## 5. Not allowed

No other control variables; no switching to a different x or y; no dropping datasets or patients;
no two-sided switch; no reporting the adjusted result only if it is more favourable than the
unadjusted one — both are reported whatever they show.

## 6. Outcomes

- p < 0.05: the coupling is not explained by depth or overall inflammation; the Phase 6/8 caveat
  is tested and survives.
- p ≥ 0.05: the coupling is at least partly carried by those factors; Findings 3 is reported with
  that qualification, and the cross-phase documents are updated to say so.

## 7. Addenda (dated; the text above is frozen)

**Addendum 1 — 2026-09-22 09:08 IST, OUTCOME.** Run once as specified, 110 patients, none
dropped. **Primary (adjusted for depth + immune fraction): ρ = +0.254, one-sided p = 0.0045 →
H1 supported; the coupling survives adjustment.** Unadjusted on the same patients ρ = +0.257
(p = 0.0049); depth only +0.246 (0.0049); immune fraction only +0.267 (0.0025). Why it barely
moves: the controls correlate modestly with the interferon score (depth +0.19, immune fraction
+0.28) but essentially not with the TLS score (+0.09, +0.00), and a confounder must move both.
Added as test 12: BH q = 0.0115; Bonferroni 0.054. Claim remains correlational; unmeasured
confounders are not excluded. Record: `mg-thymus-map/docs/phase9_partial_correlation_runlog.md`.
