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

**Addendum 1 — 2026-09-15, still before any UC data was downloaded.** Two gaps found on a
skeptical re-read, closed here rather than left to be decided after seeing results.

**(a) Minimum contributing sample size — when the test is "underpowered" rather than "not
confirmed."** A patient contributes to the primary test only if at least one of their cells is
called TLS-region (`is_gc_b_cell | is_tfh`); PF lung showed this can silently remove patients
(3 of 20 had none). With only 4 tonsil donors, the exact permutation test's smallest achievable
one-sided p is 1 / C(n_UC + 4, 4): for n_UC = 2 that is 0.067 — the test cannot pass regardless
of the biology. Rule, fixed now: **the primary test is reported as "underpowered — not
interpretable" (a third outcome, distinct from §7's "not confirmed") if fewer than 8 of the 18 UC
patients contribute a median.** At n_UC = 8 the floor is 1 / C(12, 4) = 0.002, comfortably below
0.05. The same floor (8 of 12) applies to the healthy-control-vs-tonsil supporting line. The number
of contributing patients is reported in every case.

**(b) Pooling versus the inflamed-only primary group.** §4's two rules interact. Resolution:
for the **primary** group, a UC patient's median is computed from that patient's **inflamed
samples only**; non-inflamed samples from UC patients are pooled per patient into a separate
**secondary "non-inflamed UC" line**, reported alongside, not part of the pass criterion. If
`all.meta2.txt` carries no inflamed/non-inflamed distinction at all, the primary group is all
samples from UC patients, pooled per patient, and this is stated in the results. Mapping the real
field names onto these definitions is format inspection, not analysis, and is permitted.

**(c) One clarification, not a change.** §4's supporting comparisons (healthy control vs. tonsil;
UC vs. healthy control, two-sided) and Step 4's descriptive outputs (proportion of TLS-region
cells above the tonsil threshold; bootstrap CI) are reported for continuity with Sjögren's/RA/SSc/
PF. **None of them is a pass criterion.** Only §4 step 4's one-sided exact permutation test, at
p < 0.05 with n_UC ≥ 8, confirms Aim 1.

**Addendum 2 — 2026-09-16 00:27 IST, OUTCOME (data on disk 2026-09-15 23:37 IST; primary result
50 minutes later, inside the 3-day box).** Primary test run exactly as specified above, no
variants: 18 of 18 UC patients contributed a median (Addendum 1a floor met, test interpretable);
UC (inflamed samples) vs. 4 tonsil donors, exact permutation (7,315), one-sided, observed pooled
median difference −0.0347, **p = 1.000**. **Outcome per §7: NOT CONFIRMED (p ≥ 0.10).** UC's
TLS-region cells score below tonsil, not above. Supporting lines (not pass criteria): non-inflamed
UC vs. tonsil p = 0.997 (18/18); healthy control vs. tonsil p = 1.000 (12/12); UC (inflamed) vs.
healthy control two-sided p = 0.169. Sole secondary (§5, exploratory): literature-prior 13-gene
subset UC vs. tonsil p = 0.0012 (UC above tonsil) — **but healthy colon vs. tonsil on the same
subset p = 0.0033**, so it is not UC-specific and cannot confirm Aim 1, as §5 already states.
Steps 5–7 not run (§6). Project Aim 1 scoreboard: 1 of 5 (Sjögren's only). Full timestamped
record: `mg-thymus-map/docs/phase3_uc_runlog.md`; stage logs `docs/logs/phase3_uc_step*.log`;
result tables `data/processed/phase3_uc_step4_*.csv`; code on branch
`worktree-ssc-findings-prose`.

**Addendum 3 — 2026-09-18, EXPLORATORY and post-hoc (does not change Addendum 2's outcome).**
Two read-only robustness checks, asked for by the user (`mg-thymus-map/scripts/explore_uc_robustness.py`,
outputs `data/processed/explore_uc_robustness_*.csv`, commit `59bd952`).
(a) **Depth — correction to Addendum 2's interpretation.** UC TLS-region cells carry a median
1,490 UMIs vs tonsil's 3,378. With tonsil counts binomially thinned to UC depth (×0.44), UC
(inflamed) is **indistinguishable** from tonsil (diff −0.005, one-sided p = 0.74), not below it;
healthy colon likewise (p = 0.38). The "every patient below every donor" shortfall was largely a
depth artifact. The outcome stays **not confirmed** (UC is not above tonsil); the cleanest
evidence against H1 is same-study, same-depth: UC (inflamed) not above healthy colon (p = 0.17).
(b) **Drug mechanism.** Type-I IFN is up in no UC compartment (fibroblasts, epithelium,
endothelium, T, B, plasma, macrophages), so anifrolumab (anti-IFNAR1) has no rationale in UC.
Up in inflamed UC instead: the OSM/IL-11 inflammation-associated-fibroblast programme
(fibroblasts p = 0.006; epithelium, T cells, macrophages) and IFN-γ (epithelium p = 0.010,
macrophages p = 0.015) — matching Smillie et al. 2019 (PMID 31348891) and West et al. 2017
(PMID 28368383). All three cytokines signal through JAK1/JAK2, and JAK inhibitors are already
approved for UC (tofacitinib, Sandborn et al. 2017 NEJM, PMID 28467869; upadacitinib, e.g.
Panaccione et al. 2025, PMID 40347957). **Reading: same drug class, different mechanism** —
Sjögren's and SSc reach JAK inhibitors through type-I interferon (which also makes anifrolumab
relevant); UC reaches them through IFN-γ and OSM/IL-11, which matches their clinical use.
