# Pre-registration — Ulcerative Colitis (UC), Aim 1 test

**Written 2026-09-15, before any UC data was downloaded or inspected. Frozen on commit.** Any
later change to this document must be a dated addendum below the line in §9, never an edit of the
text above it. Author: Claude Fable 5.1 with the user (Arya Kakade), session
`claude.ai/code/session_01BXjsokGjdHP7hTWKcme1sV`.

## 1. Why this document exists

After Sjögren's (confirmed), RA, SSc, and PF lung (all not confirmed), UC is the last disease this
project will test for Aim 1. The 2026-09-15 signature-subset diagnostic showed that choosing
analysis variants *after* seeing results can move p-values a lot (RA: full signature p=0.625 vs.
literature-prior half p=0.056). That is exactly the situation in which a borderline UC result
would tempt "one more variant." This document removes that option by fixing every choice now.

## 2. Hypothesis (Aim 1, unchanged from the project's original statement)

**H1**: TLS-organizing cells (GC-B-cell-like and Tfh-like) in UC colon score higher on the
MG-derived 25-gene TLS signature than the same cell states in healthy tonsil — i.e. UC builds its
TLS using the MG-anchored architecture, as Sjögren's does.

**H0**: they do not score higher.

Biological rationale, stated before testing: gut-associated lymphoid tissue has the strongest
independent literature support for organized tertiary lymphoid structures of any tissue this
project has considered — a fair test, not a favorable one.

## 3. Dataset

Smillie et al. 2019, *Cell* (PMID 31348891), Broad Single Cell Portal `SCP259`: 18 UC patients +
12 healthy donors, ~366K cells, deposited as three compartment matrices (Epithelial, Stromal
"Fib", Immune) plus `all.meta2.txt`. Downloaded by the user's father (ISEF Adult Sponsor) under
his own Terra/Google account; this session runs the bulk-download command he generates.

**The authors' compartment split is used only to load the data, never for cell typing.** Cell
types come from this project's own CellTypist + immune-mislabel-correction chain, identically to
every prior disease. Using the authors' labels would leak their curation into a test of ours.

## 4. Exactly what will be run (the primary test)

Identical to Sjögren's, RA, SSc, and PF — same scripts' logic, same functions, no UC-specific
statistics:

1. **Load**: merge the three compartment matrices (outer join on genes), attach `sample_id`,
   `patient_id`, `disease`, `tissue="colon"` from `all.meta2.txt`.
2. **Step 2**: `compute_qc_metrics` → `adaptive_mito_threshold` → `filter_cells(min_genes=200)`
   → `filter_genes(min_cells=3)` → `detect_doublets(batch_key="sample_id")` → `filter_doublets`
   → `normalize` → `annotate_cell_types` (CellTypist `Immune_All_High`, `majority_voting=True`)
   → `integrate_with_harmony(batch_key="sample_id")` → Leiden → `correct_immune_mislabels`.
3. **Step 4**: GC-B/Tfh calls via `call_tls_region_states` on `cell_type_corrected`; TLS-region =
   `is_gc_b_cell | is_tfh`; AUCell with the **full 25-gene signature**
   (`data/processed/tls_signature.csv`, unchanged); reference = **the 4-donor tonsil object**
   `tonsil_with_cell_states.h5ad`, unchanged (the 8-donor rebuild is NOT the primary reference).
4. **Statistic**: one median TLS-score per UC patient; exact cluster-permutation test, UC
   patients vs. 4 tonsil donors, **one-sided (UC > tonsil)**, exactly the Phase 1 method.
   **Pass = p < 0.05.** Also reported, same as SSc/PF: healthy-control patients vs. tonsil, and
   UC vs. healthy control (two-sided Mann-Whitney) — supporting, not the pass criterion.

**Group definitions, fixed now** (field names to be mapped once `all.meta2.txt` is inspected —
that inspection is format-only and changes nothing here):
- **UC (primary group)** = samples from UC patients labeled inflamed. Non-inflamed samples from UC
  patients are excluded from the primary group and reported separately as a secondary line.
- **Healthy control** = samples from healthy donors.
- **Patient unit** = the donor/subject identifier. Patients with multiple samples contribute ONE
  median (cells pooled across their samples) — the same rule that handled RA's and PF's
  multi-sample patients.

## 5. Secondary analysis (exploratory — cannot confirm Aim 1)

Exactly one: the same test with the **literature-prior 13-gene subset** instead of the full
signature (`provenance == literature_prior`). Reported as "approaches / does not approach
significance," as done for RA. **It cannot be used to declare Aim 1 met**, because the subset was
chosen after seeing other diseases' results.

## 6. What is NOT allowed, whatever the numbers say

- No other gene subsets. No MG-derived-only subset as a "rescue."
- No switching the primary reference to the 8-donor tonsil. (It may be reported as a sensitivity
  line only if the user later asks; it is not the primary.)
- No changing the tonsil-calibrated threshold, the GC-B/Tfh marker panels, the `n_std` outlier
  rule, or the compartment/patient definitions in §4 after data is loaded.
- No two-sided-to-one-sided or one-sided-to-two-sided switches after the fact.
- No dropping patients or samples for any reason other than the pre-specified QC chain.
- No running Steps 5–7 (stromal extraction / pharma) unless §7's pass condition is met.

## 7. Outcome rules — decided now

- **p < 0.05 (primary)**: Aim 1 confirmed in UC. Reported as the second disease in which the
  MG-anchored architecture transfers; project scoreboard becomes 2/5. Steps 5–7 then run for UC.
- **0.05 ≤ p < 0.10**: **not confirmed.** Reported as "not significant, borderline," with the
  number. No further tests will be run to move it. Scoreboard 1/5.
- **p ≥ 0.10**: not confirmed. Scoreboard 1/5, reported straight, consistent with RA/SSc/PF.
- The secondary (§5) result is reported alongside in every case, labeled exploratory.

None of these outcomes changes the project's title or Aim 1 as stated; they change only the
conclusion, which reports what the map found.

## 8. Time-box and known risks

- **Hard stop: 3 calendar days from the moment the data is on disk**, regardless of state. If the
  primary test has not run by then, UC is reported as "attempted, not completed" and the
  remaining time goes to the write-up. Real deadline context: true project cutoff ~2026-09-27/28.
- Known, pre-acknowledged risks that will be documented, not fixed mid-test: ~366K cells (3×
  PF's 109K — long CellTypist/Harmony runtimes; keep the machine awake); Harmony's unseeded
  internals (~15% run-to-run variation in corrected cell counts, seen on SSc); the pooled doublet
  threshold landing past the real-cell 99th percentile (seen on RA/SSc/PF at this scale).
- The three-compartment deposit is a new file format for this project — the loader gets a
  synthetic round-trip test before touching real data, as every prior loader did.

## 9. Addenda (dated; the text above is frozen)

*(none yet)*
