# Pre-registration — Phase 5: a second-generation, MG-vs-tonsil contrast signature

**Written 2026-09-18, before any derivation or test was run. Frozen on commit.** Changes only as
dated addenda in §10. Author: Claude Fable 5.1 with the user (Arya Kakade).

**Standing rules from the user:** (1) no existing result is touched — Phases 1–4 stand as
reported whatever this phase finds; (2) this phase is an additive, deletable extra: every file
it writes carries a `phase5_contrast_` prefix, and deleting those files, its script, its run log
and this document removes it completely. (The earlier "Phase 5 permutation test" idea in the
plan docs remains deferred and is unrelated.)

## 1. Why this document exists

The 2026-09-17 audit showed that the original 25-gene map's 12 MG-derived genes never transfer
to any disease (patient-level p = 0.80–0.99 everywhere, Sjögren's included) and that all of the
map's signal sits in its 13 literature chemokines, which behave as a property of lymphoid
tissue rather than of MG. Those 12 genes were ligand–receptor pairs merely *detected* between
MG's germinal-centre B and Tfh cells — generic T–B contact molecules, not what distinguishes MG.
A signature that is MG-anchored in the meaningful sense must be a **contrast**: genes that
separate MG's TLS-organizing cells from the same cell states in healthy lymphoid tissue. This
document fixes how that signature is derived and tested before either happens, so a positive
cannot be manufactured and a negative cannot be explained away.

## 2. Hypotheses

**H1 (derivation):** a stable set of genes exists that is higher in MG thymus GC-B/Tfh cells than
in healthy tonsil GC-B/Tfh cells.
**H2 (transfer, per disease):** that disease's GC-B/Tfh cells score higher on the contrast
signature than held-out healthy tonsil donors' GC-B/Tfh cells (as in Phase 1 Step 4), **and**
higher than the disease's own same-tissue comparator's GC-B/Tfh cells. Both are required for
"MG-like TLS" in that disease.
**H0:** neither holds.

## 3. Derivation — data and rules, fixed

- **Input, read-only:** MG `mg_with_cell_states.h5ad` (12 samples, 1,745 TLS-region cells =
  `is_gc_b_cell | is_tfh`, Phase 1 calls) and the **4-donor** tonsil `tonsil_with_cell_states.h5ad`
  (1,778 TLS-region cells). No disease dataset is used in derivation. Sjögren's is a *test*
  dataset here, never an input.
- **Unit:** pseudobulk per MG sample (12) and per tonsil donor (4): counts of that unit's
  TLS-region cells summed, counts-per-million, log1p.
- **Genes eligible:** present in both objects; detected (> 0) in ≥ 10% of MG TLS-region cells;
  not in the artifact families this project already excludes (mitochondrial, ribosomal,
  immunoglobulin, haemoglobin, sex-linked XIST/Y-genes, dissociation-stress IEGs — the same
  lists as Phase 2/3 scripts).
- **Selection, three conditions, all required:** (a) log2 fold change of mean pseudobulk CPM,
  MG over tonsil, ≥ 1; (b) Welch t-test on pseudobulk log-CPM, one-sided MG > tonsil, BH < 0.05
  across all eligible genes; (c) **stability**: resample cells with replacement within every
  unit 200 times and recompute (a)+(b); a gene is kept only if it passes in ≥ 95% of resamples.
- **Size:** if more than 25 genes qualify, keep the 25 with the largest log2 fold change (same
  size as the original map, for comparability). If **fewer than 10** qualify, the outcome is
  "no stable contrast signature derivable" and §5 is not run.
- **Reported alongside, not used for selection:** overlap with the original 25-gene map; how
  many selected genes are known interferon-stimulated genes (the Phase 4 panel) — because MG
  thymus is an interferon-rich tissue and a signature dominated by ISGs would be telling.

## 4. The confound, stated before derivation

MG (GSE233180, CD45+-sorted thymus) and the tonsil atlas are different studies, protocols and
sorting strategies; "MG vs tonsil" is perfectly confounded with "dataset vs dataset". Batch
genes can enter the signature, and no statistic on these two inputs can remove that. The
safeguards are therefore in the **test**, not the derivation: (i) the reference is **held-out
tonsil donors** the derivation never saw; (ii) **negative-control lines** — a signature that
also scores OA, healthy skin, healthy lung, healthy colon and SICCA above tonsil is measuring
"not tonsil", not "MG-like"; §7 turns that into a rule. A confounded signature that transfers
nowhere, or everywhere, is reported as such.

## 5. Transfer test — fixed (Phase 1 Step 4 design, with two additions)

- **Reference:** the four held-out donors of `tonsil_with_cell_states_8donors.h5ad` (BCLL-2-T,
  BCLL-8-T, BCLL-9-T, BCLL-13-T; 19 / 314 / 502 / 363 TLS-region cells), TLS-region cells only.
- **Score:** AUCell with the contrast signature, unchanged settings.
- **Per disease, TLS-region cells = `is_gc_b_cell | is_tfh`** on that phase's own cell-type
  labels (Sjögren's/RA/OA: `majority_voting`, Phase 1/2 calls; SSc/PF: `cell_type_corrected`
  from a fresh reprocess / raw-rebuild exactly as Phase 4 did; UC: the calls saved by Step 4 in
  `phase3_uc_step4_cell_scores.csv.gz`, reused, not recomputed).
- **Patient unit** as in every prior phase (Sjögren's `sample_id`; RA `individual`; OA
  `sample_id`; SSc `sample_id`; PF `sample_id`; UC `patient_id`, inflamed samples only for the
  primary UC line). A patient contributes with ≥ 1 TLS-region cell; an arm with < 5 contributing
  patients makes that line "underpowered".
- **Test A (vs tonsil):** one median per patient; exact cluster-permutation, disease > held-out
  tonsil, one-sided; p < 0.05.
- **Test B (vs own comparator, the addition that makes it disease-specific):** disease patients
  vs comparator patients of the same study (Sjögren's vs SICCA; SSc, PF, UC vs healthy; RA vs
  OA, cross-study, flagged), one-sided Mann-Whitney disease > comparator; p < 0.05.
- **Negative-control lines (not pass criteria, but they decide §7's batch verdict):** each
  comparator group vs held-out tonsil (SICCA, OA, healthy skin, healthy lung, healthy colon),
  one-sided exact permutation.
- **Sensitivity, reported only:** Test A repeated with the original 4-donor tonsil object (the
  derivation reference), to show how much circularity inflates it.

## 6. Not allowed, whatever the numbers say

No second derivation with different thresholds; no gene removed or added by hand; no other
reference; no switching one-sided/two-sided; no dropping a disease or a negative-control line;
no reinterpretation of Phases 1–4. If the signature is not derivable (§3) the phase ends there.

## 7. Outcome rules

- **Batch verdict first:** if ≥ 3 of the 5 negative-control lines are above held-out tonsil at
  p < 0.05, the signature is declared **batch-driven**; no disease is called MG-like, and the
  phase is reported as a negative about the derivation, not about any disease.
- Otherwise, per disease: **MG-like TLS** = Test A and Test B both p < 0.05; **partial** = one of
  the two; **not MG-like** = neither; **underpowered** by the § 5 floor.
- Scoreboard = count of MG-like diseases among the five. Reported as a count whatever it is.
  Sjögren's counts like any other disease. The original map's Aim 1 scoreboard (1 of 5) is not
  revised by this phase.

## 8. What each outcome would mean (written now so it cannot be re-spun)

- **Derivable, not batch-driven, MG-like in ≥ 1 disease:** a genuine MG-anchored program that
  transfers; reported as a second-generation result alongside — never instead of — the original.
- **Derivable, transfers nowhere:** MG's TLS program is MG-specific; strengthens the Phase 4
  reading that convergence lives in the stromal interferon program, not in TLS architecture.
- **Batch-driven:** the two-dataset derivation cannot separate disease from protocol; future work
  needs a same-study healthy thymus.
- **Not derivable:** MG TLS cells are not stably different from tonsil's at this power.

## 9. Time-box and outputs

Two working days from this commit. Outputs, all new: `data/processed/phase5_contrast_signature.csv`
(gene, log2FC, BH p, stability), `phase5_contrast_derivation_all_genes.csv`,
`phase5_contrast_step4_<disease>_*.csv`, `phase5_contrast_scoreboard.csv`; script
`scripts/run_phase5_contrast_signature.py`; log `docs/phase5_contrast_runlog.md`.

## 10. Addenda (dated; the text above is frozen)

**Addendum 1 — 2026-09-18 11:19 IST, OUTCOME.** Run once as specified (11:03:28 → 11:19:11).
Derivation: 2,806 eligible genes, 677 pass log2FC ≥ 1 + BH < 0.05, 383 stable; top 25 kept —
ten immediate-early/stress/hypoxia genes (NR4A2, NR4A3, DUSP1, DUSP4, PER1, RHOB, AREG, MT2A,
SLC2A3, ANXA1), three unannotated lncRNA IDs, zero overlap with the original map, zero ISGs.
Held-out tonsil donors score 0.0. Transfer: Test A p < 0.05 in all five diseases, **but Test B
fails everywhere** (no disease above its own comparator; RA underpowered) **and 4 of 5 healthy
comparator groups are also above tonsil (SICCA 0.043, healthy colon 0.012, healthy lung 0.026,
healthy skin 0.017)**. **Verdict per §7: BATCH-DRIVEN. MG-like TLS diseases: 0 of 5. No claim
for any disease.** Per §8: the two-dataset derivation cannot separate disease from protocol;
a second-generation signature needs a same-study healthy thymus. Phases 1–4 unchanged. Record:
`mg-thymus-map/docs/phase5_contrast_runlog.md`, `docs/logs/phase5_contrast_2026-09-18.log`,
`data/processed/phase5_contrast_*.csv`.
