# Pre-registration — Phase 4: type-I-interferon stromal program across five diseases

**Written 2026-09-17, before any new analysis was run. Frozen on commit.** Changes only as dated
addenda in §9. Author: Claude Fable 5.1 with the user (Arya Kakade).

**Standing rule from the user (2026-09-17): no existing result is touched.** Nothing from Phases
1–3 is re-run into its original output files; every file this phase writes carries a new
`phase4_ifn_*` prefix; no document reporting an earlier result is edited to change a number.

## 1. Why this document exists

Two diseases already carry a patient-level-confirmed type-I-interferon-stimulated-gene (ISG)
program in their disease-specific stromal residual: Sjögren's (Phase 1 Step 5: IFI6, IFI44L,
XAF1, discovered without a seed list) and SSc (Phase 3 Step 7: 10 of 12 a-priori ISGs, discovery
genes IFITM2/3 excluded). Through Phase 1's upstream pathway query (IFNAR1/2, JAK1/2, TYK2,
STAT1/2, inhibitory direction) both reach the same approved drug set: anifrolumab plus JAK/TYK2
inhibitors. The question is how far that convergence extends. This document fixes the test before
PF, UC and RA are looked at for these genes, so a five-disease scoreboard can be reported without
"one more variant".

## 2. Hypothesis

**H1 (per disease):** in the fibroblast compartment, disease patients express the canonical
type-I ISG program more strongly than that disease's own comparator patients.
**H0:** they do not.

**Convergence claim (project level):** the number of diseases in which H1 holds. It is reported
as a count, whatever it is (0 to 5). No count is "success" or "failure".

## 3. The ISG panel — fixed, identical for every disease

IFI6, IFI44L, XAF1, ISG15, MX1, IFI27, IFI44, OAS1, IFIT1, IFIT3, RSAD2, STAT1 — the same 12 genes
SSc's Step 7 used (Phase 1's three Sjögren's residual ISGs plus the standard IFN-signature core).

**Per-disease exclusions, to keep every test non-circular:**
- Sjögren's: IFI6, IFI44L, XAF1 are its own discovery genes → tested on the remaining **9**.
- SSc: IFITM2/IFITM3 were its discovery genes and are not in the panel; its 12-gene result
  **already exists** (`data/processed/phase3_step7_isg_convergence_patient_level_bh.csv`) and is
  reported from that file, not re-run. Its composite (§5) needs a fibroblast object; SSc is
  reprocessed fresh into new `phase4_*` outputs only.
- PF, UC, RA: no ISG was a discovery gene for them → full **12**.

## 4. Datasets, compartments, comparators, patient units — fixed

| disease | object (existing, read-only) | cell-type column | fibroblast cells | comparator | patient unit |
|---|---|---|---|---|---|
| Sjögren's | `sjogrens_with_cell_states.h5ad` | `majority_voting` (correction did not exist in Phase 1) | 4,760 PSS / 7,503 SICCA | non-Sjögren's SICCA (same study) | `sample_id` (7 vs 6) |
| SSc | fresh reprocess (Phase 3 chain) | `cell_type_corrected` | ~21K | healthy control (same study) | `sample_id` (12 vs 10) |
| PF | raw `GSE135893` restricted to `pf_harmonized.h5ad`'s QC-passed cells; labels copied from that object | `cell_type_corrected` | 4,178 PF / 484 control | healthy control (same study) | `sample_id` = `Sample_Name` (20 vs 10) |
| UC | `uc_processed.h5ad` | `cell_type_corrected` | 5,265 inflamed / 8,948 non-inflamed / 5,609 healthy | healthy control (same study) | `patient_id` (18 vs 12) |
| RA | `ra_with_cell_states.h5ad` + `oa_with_cell_states.h5ad` | `majority_voting` | 30,959 RA / 10,413 OA | OA (**cross-study**) | RA `individual` (23), OA `sample_id` (4) |

- **Compartment: fibroblasts only**, everywhere (SSc's Step 7 design). The label column is the
  one that phase's own pipeline produced; no re-annotation.
- **UC primary group = inflamed samples pooled per patient** (as in the UC Step 4
  pre-registration); non-inflamed samples per patient are a separate supporting line.
- **RA is the weak line by construction and is labelled as such in every output**: cross-study
  comparator, n = 4 OA, and 10 of 25 "RA" individuals are not RA (Phase 2's documented
  limitation). Before testing, `sc.pp.combat(key="dataset")` is applied to the RA+OA fibroblast
  expression matrix (Phase 2 Step 5's own batch fix, unchanged). RA's outcome counts in the
  scoreboard but is flagged.
- **Patient contribution floor: a patient contributes only with ≥ 10 fibroblasts** (a mean over
  fewer cells is noise). The number of contributing patients per arm is reported. A disease's
  test is **"underpowered — not interpretable"** if either arm has < 5 contributing patients.

## 5. Statistics — fixed

**Primary, one test per disease — composite ISG score.** For each contributing patient, the
mean log-normalized expression of each panel gene over that patient's fibroblasts; each gene is
then standardized (z-score across all contributing patients of both arms of that disease) so no
single highly expressed gene dominates; the composite is the mean of the standardized genes.
One-sided Mann-Whitney, disease > comparator. **H1 holds at p < 0.05.** 0.05 ≤ p < 0.10 is
reported as "not significant, borderline" and does not hold. No BH across diseases: each disease
is its own pre-stated hypothesis and is reported individually.

**Secondary, for comparability with SSc's Step 7 — gene by gene.** `patient_level_test_batch`
(per-patient means, one-sided Mann-Whitney UP, BH within that disease's panel family). Reported
as "n of N genes clear BH; n of N point up". Descriptive; it does not decide H1.

**Exploratory only, one line per disease:** the same composite on Phase 1's broader stromal
compartment (Fibroblasts + Epithelial cells + Endothelial cells). Cannot decide H1.

## 6. Not allowed, whatever the numbers say

No other gene panels; no removing genes from the panel after seeing results; no other primary
compartment; no switching one-sided/two-sided; no dropping a dataset; no changing the ≥ 10-cell
floor; no swapping UC's primary group to all-UC or non-inflamed; no re-running Sjögren's or SSc
into their original files. RA is reported even if its result is inconvenient in either direction.

## 7. Outcome rules

Per disease: **present** (p < 0.05) / **not present** (p ≥ 0.05, borderline noted if < 0.10) /
**underpowered**. Scoreboard = count of "present" among the five, with RA's flag attached. The
drug mapping (anifrolumab + JAK/TYK2 inhibitors) is fixed a priori by the pathway gene set and
carries no evidential weight; a disease is listed under that drug class only if its ISG program
is "present" here. Known prior literature is stated per disease in the write-up (type-I IFN
signature in inflamed UC mucosa is well documented; in IPF fibroblasts it is debated; in RA it
marks a patient subset) so that no "present" is oversold as novel.

## 8. Time-box and outputs

One working day from this commit. Outputs, all new: `data/processed/phase4_ifn_composite.csv`
(one row per disease × patient), `phase4_ifn_scoreboard.csv` (one row per disease),
`phase4_ifn_genes_<disease>.csv` (gene-level), `phase4_ifn_exploratory_stromal.csv`; script
`scripts/run_phase4_ifn_convergence.py`; log appended to `docs/phase3_uc_runlog.md`'s successor,
`docs/phase4_ifn_runlog.md`.

## 9. Addenda (dated; the text above is frozen)
