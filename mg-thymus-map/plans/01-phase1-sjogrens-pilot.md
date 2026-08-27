# Phase 1 Plan — MG (Thymus) vs. Sjögren's Syndrome Pilot

Status: **In progress.** Phase 0 (repo scaffold) and Phase 1 Steps 1–2 (data
assembly, QC/normalization/annotation/integration) are implemented, tested, and
verified against real downloaded data as of 2026-08-27 — see the "Implementation
notes" under each step below for what was actually built and found. Steps 3–7 are
still at the planning stage, not yet started.
Parent plan: [`00-overview.md`](00-overview.md)

## 1. Why Sjögren's Syndrome first

The source document lists five diseases (SLE, RA, Sjögren's, MG, Graves') plus a healthy tonsil
control, but building and validating the whole pipeline against all five before knowing it works is
risky. Sjögren's is a reasonable first target to pair with MG because:

- Like MG (thymus), Sjögren's is anchored in a **single, well-defined target organ** (the salivary/
  lacrimal gland), which keeps Phase 1's data assembly scoped to two organs + one control, not
  five.
- Sjögren's TLS biology is comparatively well studied (ectopic germinal centers in salivary gland
  are a textbook TLS example), so there's published biology to sanity-check pipeline output against
  — e.g. known autoantigens (Ro/SSA, La/SSB), known TLS-associated markers (CXCL13, BAFF/
  `TNFSF13B`), and known druggable targets (BAFF inhibitors like belimumab) that a working
  pipeline should be able to rediscover.
- Public scRNA-seq datasets for Sjögren's salivary gland exist and are a similar scale to thymus MG
  datasets, keeping the pilot's compute footprint modest.

**Confirmed** — see question 2 in section 7 (Resolved).

## 2. Objectives

1. Prove the full 5-stage pipeline (assembly → TLS isolation → projection → stromal extraction →
   pharmacogenomic mapping) runs end-to-end on real public data for one disease pair.
2. Produce a first-pass answer to: *does Sjögren's syndrome build its TLS using transcriptional
   architecture shared with MG's thymic TLS, and if so, how much (quantified), and what's left
   over (stromal/organ-specific)?*
3. Establish the reusable shape (interfaces, config schema, test fixtures) that Phases 2–4 will
   replay per-disease with minimal code changes.

## 3. Definition of done for Phase 1

- [x] MG thymus, Sjögren's salivary gland, and healthy tonsil scRNA-seq datasets are downloaded,
      documented (source, accession, license), and load cleanly into `anndata` objects. **Done
      2026-08-26/27** — see Step 1 implementation notes.
- [x] QC + normalization + cell-type annotation produces a labeled MG dataset and a labeled
      Sjögren's dataset with a shared, consistent cell-type vocabulary. **Done 2026-08-27** — see
      Step 2 implementation notes. Extended beyond the original wording: all *three* datasets
      are QC'd/normalized/annotated, and a batch-effect check + Harmony integration was added —
      not originally itemized here, but necessary to trust the shared vocabulary across datasets.
      (Doublet detection was briefly a gap the same day — closed before end of day; see Step 2.)
- [ ] An MG-derived TLS gene/cell-state signature exists, with a written rationale for how it was
      derived. **Not started.**
- [ ] That signature has been scored (AUCell) against the Sjögren's dataset, with a quantified
      "proportion shared" metric, benchmarked against the healthy tonsil control. **Not started.**
- [ ] Stromal populations in Sjögren's have been isolated after subtracting the shared signal, with a
      ranked marker-gene list. **Not started.**
- [ ] Those markers have been queried against DGIdb/ChEMBL, producing a ranked candidate
      target/drug list with provenance (which database, which gene, which compound). **Not
      started.**
- [~] Every stage above has unit tests (synthetic fixtures) and the full pipeline has one integration
      test (tiny fixture data) — all passing via `make test`. **Steps 1–2 fully covered (56 tests,
      100% coverage on all Step 1–2 modules, `make test` green); Steps 3–7 not yet built.**
- [x] `README.md` documents how to set up the environment and run the Phase 1 pipeline
      end-to-end via `make phase1` (or equivalent). **Done in Phase 0**; will need a further update
      once Step 3+ stages are wired into `pipeline.py` (currently only loads/validates config).
- [ ] A short written summary of Phase 1 findings exists (even if preliminary/negative) — a
      pipeline that runs but finds no shared signal is still a valid, useful Phase 1 result. **Not
      started** — depends on Steps 3–7.

## 4. Step-by-step plan

### Step 1 — Data assembly — Status: ✅ Complete (2026-08-26/27)

**Goal:** get MG (thymus), Sjögren's (salivary gland), and healthy tonsil scRNA-seq into local,
documented, loadable form.

**Implementation notes (what was actually built and found):**
- Code: `src/mg_thymus_map/data/{mg_thymus,sjogrens,healthy_tonsil,_common,schema}.py` in the
  `mg-thymus-map` repo. 22 tests (unit + one real integration test against a self-built synthetic
  10x sample), all passing, no dependency on the real downloaded data (which is gitignored).
- **Real file formats, confirmed by inspection (not assumed from the GEO/Zenodo page text):**
  - **MG (GSE233180):** the `.h5` files are **CellBender-denoised** output (ambient-RNA-corrected
    counts), not raw CellRanger — confirmed via the internal HDF5 structure (`latent_cell_probability`,
    `contamination_fraction_params`, `ambient_expression`, `training_elbo_per_epoch` are CellBender
    signatures). `scanpy.read_10x_h5` reads it directly with no special handling needed. **Correction
    to the dataset description**: GEO's "24 samples" is 12 patients' scRNA-seq `.h5` files
    (MG1–MG12) **plus 12 separate BCR repertoire-seq files** (V(D)J immune repertoire — a
    different assay, not gene expression) — not 24 scRNA-seq samples. Only the 12 `.h5` files are
    used; the BCR data is a candidate bonus dataset for a later phase, not loaded here.
  - **Sjögren's (GSE272409):** GSM-prefixed flat 10x mtx triplets (e.g.
    `GSM8401511_PSS_A_{barcodes,features,matrix}...`). `scanpy.read_10x_mtx`'s `prefix` parameter
    handles this directly — no custom parser needed.
  - **Tonsil (Zenodo 10373041):** nested per-donor CellRanger output,
    `scRNA-seq/<subproject>/<gem_id>/filtered_feature_bc_matrix/`, joined against the atlas's own
    `scRNA-seq/metadata/cellranger_metadata.csv` (columns: `subproject, gem_id, library_id,
    library_name, type, donor_id`) to map each folder to a donor. Only `type == "not_hashed"`
    libraries are used — each maps to exactly one donor unambiguously; `hashed_cdna`/`hashed_hto`
    libraries pool multiple donors per 10x GEM well and would need HTO demultiplexing to resolve
    per-cell donor identity, out of scope for Phase 1. `max_donors` parameter subsamples the atlas
    (used `max_donors=4` for Phase 1, per the plan's subsampling call).
- **Real cell counts loaded:** MG 52,269 cells (36,387 genes); Sjögren's 82,867 cells (13 samples: 7
  PSS + 6 SICCA, matching the earlier accession-resolution numbers); tonsil 30,103 cells (4 donors,
  36,601 genes) — **superseded**: this donor selection turned out to be accidentally all-male (a real
  bug found during Step 2's audit — see Step 2's third bug). Corrected tonsil count is 42,718 cells
  (still 4 donors, now sex-balanced).
- **Two files found in the Downloads folder that are explicitly NOT part of this project**, confirmed
  with the user: `GSE228033_RAW.tar` (an unrelated MG-thymoma scRNA-seq dataset) and an
  `ISEF_MG_PROJECT/mg_env` Python virtual environment (a separate, unrelated setup) — both
  ignored, not incorporated.

- Confirmed accessions (see question 1's resolution): **GSE233180** (MG thymus), **GSE272409**
  (Sjögren's salivary gland), and the **Human Tonsil Atlas** / ArrayExpress **E-MTAB-13687**
  (healthy tonsil control).
- For every dataset, record accession, organism, tissue, disease status, cell count, sequencing
  platform, and license/access terms in a small `data/DATASETS.md` manifest (or
  `configs/phase1_mg_sjogrens.yaml`).
- Prefer datasets that already include raw or lightly-processed counts (not just pre-computed
  embeddings), since we need to run our own QC/normalization consistently across datasets. Two
  format notes to handle here: GSE233180 is CD45+-sorted (immune-cell-enriched) counts, not
  whole-tissue (see the caveat under Step 2); the Human Tonsil Atlas is distributed via
  ArrayExpress/Zenodo/`HCATonsilData` (Bioconductor) rather than a plain GEO `GSE` download, so
  its loader needs to pull processed matrices from Zenodo (or convert from the Bioconductor
  object) into `AnnData`, not follow the same code path as the two GEO-hosted datasets.
- Write a download script per dataset (idempotent — skips if already present, verifies checksum)
  rather than manual downloading, so Phase 1 is reproducible from a clean checkout.
- Validate on load: expected `AnnData` shape, gene symbols present, `obs` has the metadata
  columns the rest of the pipeline needs (disease, tissue, donor/sample ID).

**Tests: done.** Per-loader unit tests (filename parsing, sample-ID/disease-status mapping,
donor-metadata selection logic — all pure functions, no real data needed) plus a schema validator
(`SchemaError`/`validate_dataset`, required `obs` columns: `sample_id`, `disease`, `tissue`) with its
own tests, plus one real integration test that builds a tiny synthetic 10x mtx sample on the fly and
runs `load_sjogrens` against it end-to-end. No `DATASETS.md` manifest file was created separately
— the same information (accession, source, description, provenance) lives directly in
`configs/phase1_mg_sjogrens.yaml`'s `datasets:` block, which is what `pipeline.py` actually reads.

### Step 2 — QC, normalization, cell-type annotation — Status: ✅ Complete (gap closed 2026-08-27)

**Doublet detection — added 2026-08-27, closing the gap flagged earlier the same day.** Built
`src/mg_thymus_map/qc/doublets.py` (`detect_doublets`/`filter_doublets`), wrapping scanpy's
Scrublet integration with `batch_key='sample_id'` so doublet *simulation* respects per-sample
boundaries rather than pooling across samples. Motivation beyond generic hygiene: Step 3's planned
cell–cell-communication analysis could misread a doublet's trivial co-expression of two cell types'
genes as evidence of a real biological interaction — doublet contamination is a plausible source of
a false positive for exactly the kind of finding this project is trying to make. **Real bug found
while wiring this up:** running Scrublet on completely unfiltered data crashes — its internal
per-batch processing silently drops near-zero-count cells (the same root cause as the earlier NaN
mito bug), then fails writing results back for cells it dropped. Fixed by running doublet detection
*after* `filter_cells`/`filter_genes`, not before.

**First-pass results were misleadingly low, and the cause was found and fixed the same day.**
Initial run gave suspiciously low flag rates (MG 0.05%, Sjögren's 0.23%, tonsil 0.04% — vs. the
~5% often quoted for droplet-based scRNA-seq). Investigated rather than accepted at face value:
checked the actual per-sample *threshold* Scrublet's automatic detection computed for each of MG's
12 samples, and found it ranging **0.35–0.55** — a huge spread — with **6 of the 12 samples
getting a threshold above every real cell's score in that sample**, flagging zero doublets not
because those patients are doublet-free, but because Scrublet's automatic bimodal-threshold fit
failed on samples too small (2,700–5,800 cells each) to show a clean split. **Fix:** added
`compute_pooled_doublet_threshold` (`src/mg_thymus_map/qc/doublets.py`) — pools every sample's
*simulated* doublet scores (tens of thousands of points instead of a few thousand per sample) and
finds the real valley between the two modes via kernel density estimation, same logic Scrublet uses
per-sample, just with far more statistical power. Applies one threshold per dataset uniformly,
instead of Scrublet's fragile per-sample ones. (Also fixed a real bug found while building this:
naive valley-finding can mistake a tiny KDE numerical ripple in a sparse tail for a genuine second
peak — confirmed on synthetic unimodal test data, an artifact peak >400x smaller in density than
the real one. Fixed with a minimum relative-prominence check.)

**Corrected results after the pooled-threshold fix** (MG and Sjögren's numbers below are final;
tonsil's were superseded by a second, unrelated fix — see below):

| Dataset | Pooled threshold | Doublets flagged (before fix → after) |
|---|---|---|
| MG | 0.2990 | 23/45,686 (0.05%) → **299/45,686 (0.65%)** |
| Sjögren's | 0.2769 | 151/66,484 (0.23%) → **546/66,484 (0.82%)** |
| Tonsil (old donor set) | 0.3005 | 11/26,972 (0.04%) → 235/26,972 (0.87%) — *superseded, see below* |

Now consistent across all three datasets rather than wildly different, and no longer suspiciously
low — still below the ~5% ballpark sometimes quoted, but that figure is itself a rough field-wide
average, not a target to hit.

---

**Second real bug found the same day, while auditing this work at the user's request: CellTypist's
`majority_voting=False` setting was the wrong choice.** `annotate_cell_types` originally used
`majority_voting=False` (each cell classified independently, no neighborhood context) rather than
`majority_voting=True` (a second automated pass where each cell's neighbors vote on its final
label). Checked what this actually cost, rather than assumed it was fine: CellTypist's own
`conf_score` showed a real low-confidence tail on all three datasets — MG 8.7% of cells below 0.5
confidence, **Sjögren's 22.2%**, tonsil 3.4%. Ran both modes on real data and compared:

| Dataset | Overall cells relabeled by majority voting | Of low-confidence (<0.5) cells, % relabeled |
|---|---|---|
| MG | 6.2% | 39.2% |
| Sjögren's | 17.0% | 53.4% |
| Tonsil | 3.5% | 30.9% |

`majority_voting=True` is still fully automated (no manual curation) — same category as question
4's resolution, just the better option within it, not a different one. Fixed to `majority_voting=True`
(`src/mg_thymus_map/annotate/celltypist_annotate.py`); `majority_voting` (not `predicted_labels`)
is now the column used downstream, documented as such.

---

**Third real bug found the same audit pass: tonsil's `max_donors` subsampling was accidentally
all-male.** `load_healthy_tonsil`'s donor selection took the first N donors in whatever order they
appeared in `cellranger_metadata.csv`. Checked what `max_donors=4` had actually picked: **BCLL-2-T,
BCLL-8-T, BCLL-9-T, BCLL-10-T — all four male**, out of a pool that's 4 male / 4 female among the
available `not_hashed` donors (not a coincidence of this run; the file happens to be ordered so all
male donors' subprojects come first). A real, previously-uncorrected confound: MG is 10
female/2 male (per GEO) and Sjögren's is a clinically female-predominant autoimmune disease —
comparing both against an all-male healthy control isn't a fair baseline, given well-documented
sex-based differences in immune biology. **Fix:** added `load_donor_demographics` and
`_select_balanced_donors` (`src/mg_thymus_map/data/healthy_tonsil.py`), which round-robins the
selection across sex groups instead of taking a naive first-N. Now selects **BCLL-10-T (M, 14,274
cells), BCLL-11-T (F, 6,847), BCLL-12-T (F, 9,245), BCLL-14-T (M, 12,352)** — 2/2 balanced, one
sample (`not_hashed` gem_id) per donor. (Correcting an inaccurate claim from an earlier check: each
donor maps to exactly *one* `not_hashed` sample, not multiple capture runs — the initial check that
suggested otherwise hadn't filtered to `not_hashed` first, so it was counting `hashed_cdna`/
`hashed_hto` rows too.) The raw cell-count jump (30,103 → 42,718) is simple arithmetic: the old,
accidentally-all-male selection happened to include the two *smallest* donors in the whole
8-donor pool (BCLL-2-T at 1,918 cells and BCLL-8-T at 5,693 — the smallest and third-smallest of
all 8), which the sex-balanced fix swapped out for meaningfully larger ones (BCLL-11-T and
BCLL-14-T). BCLL-10-T, the single largest of all 8 donors (14,274 cells), stayed in both sets by
coincidence.

This changed tonsil's underlying data (different donors, more total cells), which cascades through
QC and doublet detection — both had to be re-run.

**Consolidated final numbers, all three fixes applied together:**

| Dataset | Mito threshold | Cells after QC | Doublet threshold | Doublets flagged | Final cells |
|---|---|---|---|---|---|
| MG | 9.5% | 45,686 | 0.2990 | 299 (0.65%) | 45,387 |
| Sjögren's | 38.1% | 66,484 | 0.2769 | 546 (0.82%) | 65,938 |
| Tonsil | **25.3%** (was 21.0%) | **38,273** (was 26,972) | 0.3300 | 201 (0.53%) | **38,072** (was 26,961) |
| **Combined** | — | — | — | — | **149,397** (was 138,957 with the old donor set) |

Annotation (with `majority_voting=True`) and Harmony integration re-run on this final data; same
overall biological picture as every prior run (same dominant cell types per tissue, same broad
mixing pattern) — if anything, mixing looks *slightly* better in a few clusters now (e.g. the
fibroblast cluster picked up a real tonsil contribution it didn't have before, since the new donor
set apparently has more of them) — the corrections changed the underlying numbers substantially
without changing the qualitative conclusions, which is itself reassuring evidence of robustness.

**Goal:** each dataset becomes a clean, annotated `AnnData` on a shared cell-type vocabulary.

**Implementation notes (what was actually built, decided, and found):**

*QC thresholds actually used:* `min_genes_per_cell=200`, `min_cells_per_gene=3` (standard
field defaults) plus a **per-dataset adaptive mitochondrial-% cutoff** (median + 5×MAD, capped
at 50%) rather than one fixed number — computed values: MG 9.5%, Sjögren's 38.1%, tonsil
**25.3%** (recomputed after the donor-selection fix below; was 21.0% with the old, accidentally
all-male donor set). Rationale: germinal-center B cells and plasma cells (the exact populations this
project cares about) naturally run metabolically "hot" and have legitimately elevated mitochondrial
content — a single fixed cutoff risks systematically stripping them out. This was tested, not just
argued: a stricter fixed 5% cutoff on MG would remove 39.0% of plasma cells and 50% of cycling
cells (vs. 16.4% of T cells) — direct evidence a fixed cutoff disproportionately damages exactly the
cells this project is about. A stronger alternative (`min_genes=500`, `min_cells=10`) was also
tested and rejected: it would cost ~23% of all cells overall, hitting Sjögren's hardest (whole-tissue,
stromal cells naturally express fewer genes than immune cells) — risking exactly the stromal
populations Step 5 needs. Generic AI-sourced "rule of thumb" mito cutoffs (5% for immune-only,
10–15% for Sjögren's, 8–10% for tonsil) were checked against real per-cell-type retention and
rejected too: in every one of the three datasets, the suggested cutoffs would have stripped the
*dominant* cell-type population far harder than a bulk population (e.g. Sjögren's at "lenient"
10% would lose 57% of epithelial cells and 77.8% of B cells). (Note: the `500/10` and generic-advice
comparisons above were run against tonsil's original — later found to be accidentally all-male —
donor set; not re-verified against the corrected donor set, though the methodology conclusion,
adaptive beats fixed, isn't expected to depend on which specific donors were used.) **Retention at
the chosen thresholds:** MG 87.4% cells kept (52,269→45,686), Sjögren's 80.2% (82,867→66,484),
tonsil 89.6% (42,718→38,273, with the corrected donor set).

*Two real bugs found and fixed while running this against real data (not hypothetical):*
1. `adaptive_mito_threshold` used `np.median`, which propagates `NaN`. MG's CellBender output has
   genuine zero-total-count cells (0/0 = NaN mito%), which silently poisoned the threshold for the
   *entire* dataset. Fixed with `np.nanmedian`.
2. The MAD-based approach can degrade toward the plain median if >50% of cells share
   near-identical mito% (documented as a known caveat in the code — not hit by the real data, but a
   real edge case).

*Normalization:* `log1p` with `target_sum=1e4` (the field-standard convention — Seurat's
LogNormalize, scanpy tutorials, and critically CellTypist's own training convention all assume
this). **Bug found and fixed:** the first version let `scanpy.pp.normalize_total` use its
per-dataset-median default instead of the fixed 1e4, which would have broken comparability
between datasets and specifically broken compatibility with CellTypist annotation (next).

*Annotation:* [CellTypist](https://www.celltypist.org/) v1.7.1, model **`Immune_All_High.pkl`**
(one of 61 available models, pulled fresh from celltypist.org), used **consistently across all three
datasets on purpose** — dataset-specific alternatives exist (`Cells_Human_Tonsil.pkl`,
`Developing_Human_Thymus.pkl`) and are likely marginally more accurate in isolation, but each has
its own private label vocabulary, which would break the cross-organ comparability Steps 3–4
depend on. Runs with `majority_voting=True` (see the "second real bug" note above for why —
initially `False`, fixed during the same audit pass as the doublet-threshold and tonsil-donor bugs).
**Validation, not just plausibility-checking:** without ever telling the classifier which tissue was
which, the results matched well-established biology for each: MG's top populations were T cells
plus double-positive/double-negative thymocytes and an ETP population (textbook thymus T-cell
developmental stages); Sjögren's largest population was plasma cells (matches Sjögren's
well-known plasma-cell-rich pathology); tonsil's largest population was B cells (tonsils are
textbook B-cell-rich secondary lymphoid organs). Held up after the `majority_voting` fix and the
tonsil donor-selection fix too — same qualitative picture on every re-run. This is strong evidence
the QC/normalization upstream didn't corrupt the biology.

*Batch/dataset-effect check — found a real, strong effect:* first run, combining all three
post-QC/annotated datasets (139,142 cells, before the doublet/majority-voting/tonsil-donor fixes
below) and clustering (PCA on batch-aware HVGs → neighbors → Leiden), showed nearly every
cluster was 95–100% cells from a single dataset — cells were grouping by which study they came
from, not by cell type, exactly the failure mode this check exists to catch. Re-confirmed on the
final, fully-corrected 149,397-cell dataset after all three fixes — same conclusion (batch effect is
real, Harmony integration is needed).

*Integration — Harmony, with a real library-compatibility bug found and fixed:*
`scanpy.external.pp.harmony_integrate` (the standard way to call Harmony from scanpy) raises a
shape-mismatch error against **every currently-installable `harmonypy` version** (tried 0.2.0 and
2.0.0 — both fail identically). Root cause confirmed by direct inspection: `harmonypy`'s `Z_corr`
now returns already shaped `(n_cells, n_pcs)`, but scanpy's wrapper was written against an older
API where it was `(n_pcs, n_cells)` and needed an external `.T` — the wrapper's extra transpose
now breaks. Fixed by calling `harmonypy.run_harmony()` directly instead of going through scanpy's
wrapper (`src/mg_thymus_map/qc/integration.py`). (Also documented, not fixed since it's not our
bug: `harmonypy` itself crashes on very small inputs, <~75 cells, where its auto-picked cluster
count comes out to exactly 1.) **After integration (first run, illustrative):** real, substantial
mixing improvement for several clusters (e.g. one went from ~0% mixed to 34.5% MG / 15.1%
Sjögren's / 50.4% tonsil). Many clusters remained dataset-pure even after integration — checked cell-type-by-cell-type rather
than assumed to be a failure, and traced to two legitimate, non-alarming causes: (a) cell types that
structurally cannot exist in one of the datasets (MG is CD45+-sorted, so it has zero stromal cells
and zero non-thymus-specific developmental states; those clusters staying MG-only or
Sjögren's/tonsil-only is *correct*, not under-correction), and (b) large population-size differences
between datasets (Sjögren's has ~100x more plasma cells than MG; tonsil has far more B cells than
the other two) causing numerically-dominated but still-genuinely-mixed clusters. A small number of
clusters (a few hundred to a few thousand cells) remain genuinely ambiguous and are noted as such,
not explained away.

*Final combined+integrated object:* **149,397 cells** (after all three bug fixes below: doublet
pooled-threshold, `majority_voting=True`, tonsil sex-balanced donors), saved locally as
`data/interim/harmonized.h5ad` (regenerated each time a fix changed the underlying data —
gitignored, not committed to git, reproducible from raw data + this pipeline).

*Vocabulary:* since one consistent CellTypist model was used, the cell-type labels are already a
shared vocabulary by construction (no separate remapping step was needed, contrary to what this
step originally anticipated). What CellTypist's `Immune_All_High` does *not* cover: follicular
dendritic cells (FDC) and high endothelial venules (HEV) specifically — these are subtypes within
its generic "Fibroblasts"/"Endothelial cells" categories, not broken out. Per the original plan, this
level of TLS-specific marker-based refinement is Step 3's job (deriving the MG signature), not
Step 2's.

- Per-dataset QC: min genes/cell, min cells/gene, mitochondrial % filtering, doublet detection.
  Thresholds should be config values, not hardcoded, since different datasets/platforms need
  different cutoffs.
- Normalization: log1p + a size-factor method (CPM-style or `scanpy`'s default), consistently
  across all three datasets so downstream scores are comparable.
- Cell-type annotation: assign each cell a type from a shared vocabulary that spans thymus,
  salivary gland, and tonsil (e.g. B cell, plasma cell, T cell subsets, follicular dendritic cell, HEV
  endothelial cell, epithelial subtypes, fibroblast). This vocabulary is the join key that lets Step 3's
  signature transfer meaningfully to Step 4's projection.
  **Caveat:** GSE233180 was generated from CD45+-sorted (hematopoietic-enriched) thymic cell
  suspensions, so it will not contain non-immune stromal/epithelial cell types (follicular dendritic
  cells, HEV endothelial cells, thymic epithelial cells, fibroblasts) — the vocabulary's stromal
  entries simply won't be populated for the MG side. This is workable for Step 3, which only needs
  the immune-cell architecture; the stromal/niche context that GSE233180 lacks is exactly what the
  Step 3 spatial cross-check (whole-tissue Visium, not sorted) is there to supply instead. GSE272409
  (Sjögren's) is whole-tissue and does include non-immune states, which Step 5 needs.
- Batch/dataset-effect check: confirm cells cluster by biology (cell type) rather than purely by
  dataset of origin before trusting cross-dataset comparisons; apply integration (Harmony/scVI) if
  needed.

**Tests: done.** `compute_qc_metrics`/`filter_cells`/`filter_genes`/`adaptive_mito_threshold`/
`normalize` all have unit tests against synthetic data with known outliers planted in (100% coverage
on these modules); `combine_datasets`/`compute_cluster_composition` have unit tests;
`integrate_with_harmony` and `detect_doublets` each have a real (not mocked) integration test since
both harmonypy and Scrublet need no downloaded model and are fast enough to run for real on tiny
synthetic data — this is what caught two of the real bugs described above; `filter_doublets` has
pure unit tests; `annotate_cell_types` has a mocked unit test (CellTypist itself needs a downloaded
model, exercised manually against real data instead, not in the automated suite). **Not done:** no
test that annotation output stays within an approved label vocabulary list — moot for now since
CellTypist's own fixed label set is being used directly as the vocabulary, not a hand-maintained
approved list.

**Open sub-question:** see section 7, question 4 (automated vs. manual annotation, and who
reviews biological calls).

### Step 3 — TLS isolation (from MG data only)

**Goal:** derive "the MG immune structure" — the shared architecture to project onto Sjögren's.

- **Seed with the established 12-chemokine TLS signature** (per question 3's resolution), then
  refine using the MG data rather than deriving fully de novo.
- Identify the cell populations and interactions specifically associated with TLS formation in the
  MG thymus data: germinal-center-like B cells, T follicular helper cells, follicular dendritic
  cells, high endothelial venules, relevant chemokine expression (e.g. CXCL13, CCL21).
- Use cell–cell communication analysis (e.g. `liana-py`/`squidpy`) between these populations to
  confirm they form a coherent interacting structure, not just co-occurring cell types.
- Output: a defined **gene signature** (and/or cell-state definition) representing this structure,
  saved as a versioned artifact (e.g. a gene list with weights) that Step 4 consumes — not
  recomputed ad hoc each time. Tag each gene with its provenance (literature prior vs.
  MG-derived refinement) per question 3's resolution.
- Write down the rationale/method used, since this is the crux of the whole project's first aim.
- **Spatial validation (added per question 6's resolution):** after deriving the scRNA-seq-based
  signature, check that it is enriched specifically in the spatially-defined germinal-center/medulla
  niche regions reported by [Spatial transcriptomics elucidates medulla niche supporting germinal
  center response in myasthenia gravis-associated thymoma (Cell Reports,
  2024)](https://www.cell.com/cell-reports/fulltext/S2211-1247(24)01028-3) — Visium data on
  thymoma **and thymic hyperplasia** samples (the latter matches the ~80%-of-MG hyperplasia
  pathology this project is about), plus normal fetal/pediatric thymus controls. This is the main
  defense against the "circular / trivially shares generic immune genes" risk in section 6: it checks
  the signature against real spatial organization instead of relying solely on inferred (non-spatial)
  cell–cell communication. Scope this as reprocessing the paper's deposited Visium data with our
  own AUCell scoring if the data proves easy to obtain and reasonably sized; fall back to comparing
  gene-list overlap with the paper's reported niche markers if not — either way it's a secondary
  check, not a blocking requirement for Step 3's core output.

**Tests to add later:** unit test that signature-derivation is deterministic given fixed input +
seed; a test that the output signature format matches what Step 4 expects (schema contract); a
test that every gene in the output signature carries a provenance tag (literature-prior or
MG-derived).

### Step 4 — Cross-disease projection (AUCell)

**Goal:** quantify how much of Sjögren's TLS architecture is "explained by" the MG signature.

- Score every cell in the Sjögren's dataset (and the healthy tonsil control) against the MG
  signature using AUCell (via `decoupler-py` or `pyscenic.aucell`).
- Define what "high" means: a threshold or distribution comparison, not just eyeballing — e.g.
  compare the AUCell score distribution in Sjögren's TLS-region cells vs. healthy tonsil, using an
  explicit statistical test with multiple-testing correction if scoring many gene sets.
- Compute the headline Phase 1 metric: proportion of Sjögren's (putative TLS) cells whose score
  exceeds the shared-architecture threshold, benchmarked against the tonsil control's own score
  distribution (tonsil is *already* a lymphoid structure, so it's the "shared architecture is normal
  here" reference point, not a naive zero baseline).

**Tests to add later:** unit test of the AUCell wrapper against a tiny synthetic expression matrix
with a known planted signal (score should come out high for cells with the signature "turned on");
a test of the thresholding/statistics function against known distributions.

### Step 5 — Stromal extraction

**Goal:** isolate the non-immune, Sjögren's-specific damage signal left over after removing the
shared immune architecture.

**Scoping note:** the source doc's Stromal Extraction step names "thymic epithelial cells" as its
first example target (alongside "synovial fibroblasts" for RA) — i.e. the full project vision runs
this step on MG's own thymus too, not just on the peripheral diseases. Phase 1 as planned only
runs it on Sjögren's; MG-side extraction is deliberately deferred, not forgotten — see question 7.
It would need a different MG dataset than GSE233180, since that one is CD45+-sorted and
excludes thymic epithelial cells entirely (see Step 2's caveat). A disease-relevant candidate exists
for when this is picked up: [Myasthenia gravis-specific aberrant neuromuscular gene expression by
medullary thymic epithelial cells in thymoma](https://www.nature.com/articles/s41467-022-31951-8)
(Nature Communications, 2022), which specifically characterizes "neuromuscular mTECs" —
medullary thymic epithelial cells aberrantly expressing AChR-mimicking self-antigens, the
published mechanistic driver of thymic MG autoimmunity.

- For cells/regions where the shared signature is "explained", regress out or condition on that
  score, then look at the non-immune stromal compartment (salivary gland epithelial cells) for
  differential expression between Sjögren's and healthy tonsil/control that is *not* attributable to
  the shared signature.
- Output: a ranked list of stromal marker genes specific to Sjögren's tissue damage, distinct from
  the general shared TLS program.
- **Ground-truth check (added per question 5/6's resolution):** compare the derived stromal
  marker list against the fibroblast and pericyte/mural cell findings from [Molecular and spatial
  analysis of tertiary lymphoid structures in Sjogren's syndrome (Nature Communications, Jan
  2025)](https://www.nature.com/articles/s41467-024-54686-0) — the closest thing to matched
  ground truth this pilot has access to. **Correction (verified against the paper's actual Data
  Availability statement):** the paper's scRNA-seq is GEO **GSE272409** (confirmed
  scRNA-seq-only; a separate **GSE272410** holds unrelated bulk RNA-seq data). Its spatial
  component is **Nanostring GeoMx spatial proteomics + multiplex immunofluorescence
  imaging** — not Visium, and not deposited in any repository; the paper states only "Source data
  are provided with this paper" (i.e. files attached to the article itself, not a queryable dataset).
  So this ground-truth check is a comparison against the paper's **reported findings and any
  Source Data tables**, not a reprocessing of a proper spatial dataset the way Step 3 does for MG.
  **Concrete targets (found while verifying the above):** a Source Data file exists and is directly
  downloadable — `41467_2024_54686_MOESM7_ESM.xlsx` (9.2 MB), linked from the article page.
  The paper also reports specific marker genes for its two key stromal states, usable as the
  comparison list even before that file is pulled:
  - **Immunofibroblast state** (CCL19⁺/TNFSF13B⁺ fibroblasts): `CD34, CCL19, TNFSF13B,
    ICAM1, VCAM1, CD82, CXCL9` (+ `RELB, NFKB2` for non-canonical NFκB signaling;
    `IFNGR1, IFNGR2, TNFRSF1A, IRF1, SOCS1`).
  - **Pericyte/mural cell state** (the paper's headline "undescribed" finding): core
    `RGS5, ACTA2, MCAM`; defining `CCL21, CCL19, TNC`; plus `CCL8, VCAM1, CCL2` — notably
    *lacking* `TNFSF13B, CD82, PDGFRA`, which is what distinguishes it from the immunofibroblast
    state above.
  Same logic as the BAFF/CXCL13 positive control in Step 7: check whether our own
  independently-derived stromal marker list recovers these genes (or close pathway relatives).

**Tests to add later:** unit test of the subtraction/regression function against synthetic data
where the "shared" and "unique" signals are known/planted, confirming the function recovers the
right split.

### Step 6 — Pharmacogenomic mapping

**Goal:** turn the stromal marker gene list into candidate druggable targets.

- Query DGIdb (GraphQL API) and ChEMBL (REST API) for each top-ranked stromal marker gene.
- Deduplicate/rank results (e.g. by interaction confidence, existing approved-drug status, or
  number of independent sources).
- Output a ranked table with full provenance (gene → source database → compound/drug →
  interaction type) so results are auditable, not just a bare gene list.

**Tests to add later:** client tests against recorded API-response fixtures (no live network calls
in CI); a test that the ranking/dedup logic behaves correctly on synthetic overlapping results.

### Step 7 — Validation & reporting

**Goal:** sanity-check the whole pipeline against known Sjögren's biology, and write up findings.

- Positive-control check: does the pipeline surface expected genes/targets (e.g. BAFF/
  `TNFSF13B`, CXCL13) among its outputs? If not, that's a signal to revisit earlier steps before
  trusting Phase 2+.
- Spatial/ground-truth cross-check: summarize how the Step 3 comparison (against the MG
  thymoma/hyperplasia Visium paper, a real reprocessable spatial dataset) and the Step 5
  comparison (against the Sjögren's GSE272409 paper's reported findings — a literature/marker
  comparison, not a reprocessed spatial dataset, per that correction) came out — agreement
  strengthens confidence in the pipeline; disagreement is a useful, reportable finding in its own
  right, not a failure.
- Write a short Phase 1 report (in `docs/` or a notebook) covering: what was run, the headline
  shared-architecture metric, the stromal target list, and whether the positive controls held up.
- **Limitations section (required, not optional):** explicitly state what this pilot does and doesn't
  show — e.g. GSE233180's CD45+ sorting means the MG signature is immune-cell-only; a single
  disease pairing (n=1) can't yet claim the "shared architecture" generalizes to RA/SLE/Graves';
  sample sizes are modest; the literature-seeded signature (question 3) means part of the
  "shared architecture" finding is anchored to prior publications, not purely novel; **ambient-RNA
  correction is inconsistent across datasets** — MG's `.h5` files were already processed with
  CellBender (ambient-RNA-corrected) by the original authors before upload, but Sjögren's
  (GSE272409) and the tonsil atlas (Zenodo 10373041) only provide already-filtered matrices, not
  the raw/unfiltered droplets CellBender needs as input (confirmed 2026-08-27: Sjögren's
  per-sample cell counts are filtered-matrix scale, 3,000–8,600; tonsil's per-donor folders contain
  only `filtered_feature_bc_matrix`, no `raw_feature_bc_matrix` sibling) — so this can't be corrected
  to match without sourcing different files, and wasn't pursued given the likely-modest practical
  impact (strongly-expressed marker genes sit well above the ambient noise floor) against the fair
  deadline. Judges at a fair (per question 4's resolution) reward intellectual honesty about a
  project's boundaries — this section is a rigor signal, not a hedge to bury.

## 5. Deliverables checklist

- [~] `data/DATASETS.md` (or config) — dataset manifest with provenance. **Done via config, not a
      separate file**: `configs/phase1_mg_sjogrens.yaml`'s `datasets:` block.
- [x] `src/mg_thymus_map/data/` — download + load + validate. **Done** (loaders only; the
      download itself was done manually by the user, not scripted — see Step 1's plan-vs-actual).
- [x] `src/mg_thymus_map/qc/` — QC + normalization + annotation. **Done**, with the doublet-detection
      gap noted in Step 2. Also includes `annotate/` (CellTypist wrapper, a separate subpackage) and
      batch-effect/integration code (`qc/integration.py`) that wasn't originally itemized here.
- [ ] `src/mg_thymus_map/signature/` — MG TLS signature derivation. **Not started (Step 3).**
- [ ] `src/mg_thymus_map/scoring/` — AUCell projection + statistics. **Not started (Step 4).**
- [ ] `src/mg_thymus_map/stromal/` — subtraction / stromal extraction. **Not started (Step 5).**
- [ ] `src/mg_thymus_map/pharma/` — DGIdb/ChEMBL clients + ranking. **Not started (Step 6).**
- [x] `tests/unit/`, `tests/integration/`, `tests/fixtures/` covering all of the above. **Done for
      Steps 1–2** (56 tests, 100% coverage on those modules); Steps 3–7 have none yet.
- [x] `README.md` — updated with Phase 1 setup/run instructions. **Done in Phase 0**; needs a
      revisit once `pipeline.py` actually orchestrates Step 3+ (currently config-loading only).
- [x] `Makefile` — `make setup`, `make test`, `make phase1`. **Done in Phase 0.**
- [ ] A short Phase 1 findings write-up, including the spatial/ground-truth cross-check (Steps 3, 5, 7 —
      reprocessed Visium data for MG's thymoma/hyperplasia paper; reported-findings comparison
      only for Sjögren's GSE272409, since it has no reprocessable spatial deposit), the ambient-RNA
      correction inconsistency (see Step 7's Limitations note), and a required Limitations section
      (Step 7). **Not started** — depends on Steps 3–7. Worth including in that section: doublet
      rates (settled at 0.53–0.87% across all three datasets), the `majority_voting` annotation fix,
      and the tonsil donor sex-balance fix — all three found during a deliberate audit pass
      (2026-08-27, user asked to "recheck everything"), not left as open observations.

## 6. Risks & mitigations

| Risk | Mitigation |
|---|---|
| Public datasets don't have compatible metadata/consistent cell annotations | Build the shared cell-type vocabulary explicitly (Step 2) rather than relying on each dataset's own labels |
| "TLS isolation" signature is circular / trivially matches because MG and Sjögren's share generic immune genes (any B/T cell genes), not TLS-specific ones | Compare against a generic-immune-cell baseline signature, not just presence/absence, so the metric reflects TLS-specific architecture, not "both have B cells" |
| AUCell threshold is arbitrary and drives the headline result | Use the tonsil control's distribution as the reference rather than a hand-picked cutoff; report sensitivity to threshold choice |
| Small pilot datasets → underpowered stats | Be explicit in the write-up about sample size and treat Phase 1 conclusions as provisional pending Phase 2–5 replication |
| Real data is large / slow to iterate on | Build and test the pipeline against small synthetic/downsampled fixtures first; only run on full real data once the fixture-based tests pass |

## 7. Open questions for Phase 1

1. **Dataset accessions** — Status: **Resolved**
   Do you already have specific dataset accessions in mind for MG thymus, Sjögren's salivary
   gland, and healthy tonsil? Or should this phase start with a short dataset-research task that
   proposes candidates for your review before any downloading happens?
   **Resolution:** All three settled.
   - **MG thymus → GSE233180** ([Single-Cell Transcriptomics Identifies a Prominent Role for the
     MIF-CD74 Axis in Myasthenia Gravis Thymus](https://pmc.ncbi.nlm.nih.gov/articles/PMC11978437/)).
     Thymic cell suspensions from 12 immunotherapy-naïve AChR-Ab+ early-onset MG (EOMG)
     patients (10 female, 2 male). **Correction (verified against the actual downloaded files):**
     the GEO page's "24 samples" is 12 scRNA-seq `.h5` files (one per patient, MG1–MG12) plus
     12 separate BCR repertoire-seq files (V(D)J immune repertoire, a different assay — not used
     for this project, but a candidate bonus dataset for B-cell clonality analysis in a later phase),
     not 24 independent scRNA-seq samples — the thymic-hyperplasia-associated subtype
     that matches the "~80% of MG begins in the thymus" framing in the source doc (as opposed to
     thymoma-associated MG). CD45+-sorted, 29,688 cells across 15 annotated populations, with
     reported germinal-center B-cell trajectories — directly relevant to TLS biology. See the
     CD45+-sorting caveat noted under Step 2. GEO series title: "Thymic B lineage cell landscape
     in Myasthenia gravis"; supplementary data is one `GSE233180_RAW.tar` (836.8 MB, per-sample
     H5/CSV count matrices).
   - **Sjögren's salivary gland → GSE272409** — 13 samples (7 primary Sjögren's Syndrome/PSS
     patients confirmed by focal lymphocytic sialadenitis histology, 6 non-Sjögren's sicca syndrome
     patients — **not** healthy controls; the project's healthy baseline is the separate tonsil
     dataset). Whole-tissue (dissociation + dead-cell removal only, no cell-type sorting), so it also
     covers the non-immune stromal side Step 5 needs. Confirmed **scRNA-seq only** via the
     paper's actual Data Availability statement — see question 6's resolution for the correction on
     what this accession does and doesn't include. Supplementary data: `GSE272409_RAW.tar`
     (367.9 MB, per-sample MTX/TSV count matrices).
   - **Healthy tonsil → Human Tonsil Atlas** ([An atlas of cells in the human tonsil, Massoni-Badosa
     et al., *Immunity* 2024](https://www.cell.com/immunity/fulltext/S1074-7613(24)00031-1)).
     17 donors, >556,000 cells across 5 modalities, 121 annotated cell types/states, healthy tonsil —
     purpose-built as a reference atlas rather than a side comparison in an unrelated study.
     **Not a single GEO accession, and not ArrayExpress either for our purposes** (verified against
     the actual ArrayExpress page): **E-MTAB-13687** is *raw FASTQ* across all 5 modalities — using
     it would mean running our own Cell Ranger/Space Ranger, well out of scope. The actual source
     to pull from is Zenodo: **`https://zenodo.org/records/10373041`**, file `scRNA-seq.zip`
     (4.4 GB, per-sample CellRanger outputs — the same tier of raw/lightly-processed counts as
     GSE233180/GSE272409, so we still run our own QC/annotation) plus
     `tonsil_atlas_donor_metadata.csv` for provenance. Also distributed as the `HCATonsilData`
     Bioconductor package and as Seurat objects (a separate Zenodo record, `8373756`) if ever
     needed for cross-checking annotations against the atlas's own published cell-type calls.
     Bonus, and this one *is* confirmed accurate (unlike the Sjögren's spatial claim — see question
     6's correction): this same Zenodo record's `spatial_transcriptomics.zip` has real Visium data
     (2 slides) — not used in Phase 1, but genuinely available if a later phase wants a tonsil spatial
     cross-check, unlike the Sjögren's side.
     (Considered and set aside: GSE139324 — an HNSCC immune-profiling study that includes only
     5 non-cancer tonsil samples as a side comparison among a much larger cancer cohort; usable
     as a fallback/independent check, but not purpose-built for this project.)

2. **Sjögren's as the first pairing** — Status: **Resolved**
   Is Sjögren's actually the right first pairing, or is there a reason (e.g. a mentor's suggestion, or
   data availability you already know about) to start with a different disease instead?
   **Resolution:** Confirmed — Sjögren's, out of the other four (RA, SLE, Graves', and the already
   MG-anchored thymus itself), is the best-compatible first pairing to try. Consistent with §1's
   rationale (single well-defined target organ, well-studied TLS biology, known positive controls)
   and with the confirmed dataset fit (GSE272409 — question 1).

3. **TLS signature prior** — Status: **Resolved**
   Should the MG TLS signature (Step 3) be seeded with an established literature signature (e.g.
   the commonly used 12-chemokine TLS signature) and then refined, or derived fully de novo from
   your MG data alone? A literature prior gives an early sanity check but also biases the
   "shared architecture" finding toward what's already published.
   **Resolution:** Seed with the established 12-chemokine TLS signature, then refine using the MG
   data. Step 3 should document, when reporting results, which genes came from the literature
   prior vs. which were added/reweighted from the MG-specific refinement — so the eventual
   "shared architecture" finding can be read honestly as partly literature-anchored, not purely
   novel, and the refinement step's contribution is auditable on its own.

4. **Annotation rigor & biological review** — Status: **Resolved**
   Automated reference-based cell-type annotation (fast, consistent, but sometimes miscalls rare
   TLS-specific states) vs. manual marker-gene curation (slower, but sanity-checkable)? And who
   reviews the biological plausibility of intermediate outputs (cell-type calls, the TLS signature,
   the stromal gene list) — you, a mentor/advisor with immunology expertise, or should the plan
   assume it's just automated checks against known marker genes?
   **Resolution:** Automated reference-based annotation, checked against known marker genes —
   no manual marker-gene curation pass. Biological review happens by sharing intermediate
   outputs (cell-type calls, the TLS signature, the stromal gene list) with the mentor at each stage,
   rather than a dedicated in-pipeline manual-curation step. Practically: each stage's output should
   be saved in a mentor-shareable form (a short summary table/plot, not just a raw `.h5ad`) so
   handoff doesn't require extra work later.

5. **Ground truth for comparison** — Status: **Resolved**
   Is any ground-truth or previously published Sjögren's scRNA-seq analysis available to compare
   our stromal target list against, beyond the general positive-control genes mentioned in Step 7?
   **Resolution:** Yes. [Molecular and spatial analysis of tertiary lymphoid structures in Sjogren's
   syndrome](https://www.nature.com/articles/s41467-024-54686-0) (Nature Communications, Jan
   2025) combines scRNA-seq (GEO **GSE272409**), spatial proteomics/imaging, and other
   modalities of minor salivary glands specifically to map TLS formation in Sjögren's, and reports
   fibroblast and pericyte/mural cell states with immunological function — directly usable as a
   comparison point for Step 5's stromal target list. Discovered incidentally while researching
   question 6. **Correction (verified against the actual GEO page and the paper's Data
   Availability statement):** GSE272409 itself is scRNA-seq only; the paper's spatial component
   (Nanostring GeoMx + multiplex immunofluorescence, not Visium) has no repository accession at
   all, only "Source data provided with this paper." So this ground truth is the paper's *reported
   findings* to compare against, not a dataset to reprocess — see question 6's correction for detail.

6. **Spatial data in scope?** — Status: **Resolved**
   Is this scRNA-seq only, or is any spatial transcriptomics data in scope? TLS are inherently a
   spatial phenomenon (organized cellular niches); scRNA-seq alone can capture cell states and
   inferred interactions but not physical organization. This affects Step 3 ("TLS Isolation")
   significantly, so worth resolving before that step starts.
   **Resolution:** In scope as a **validation layer, not a required core pipeline input** — with an
   important asymmetry between the two organs, discovered after this was first resolved (verified
   against the actual GEO pages and the Sjögren's paper's Data Availability statement):
   - **MG side: a real, reprocessable spatial dataset exists.** [Spatial transcriptomics elucidates
     medulla niche supporting germinal center response in myasthenia gravis-associated thymoma
     (Cell Reports, 2024)](https://www.cell.com/cell-reports/fulltext/S2211-1247(24)01028-3) is
     genuine 10x Visium data on thymoma and thymic hyperplasia samples, matching the
     hyperplasia pathology behind ~80% of MG. Step 3's spatial validation can reprocess this with
     our own AUCell scoring, as planned.
   - **Sjögren's side: no reprocessable spatial dataset exists.** GEO **GSE272409** is
     scRNA-seq only. The Sjögren's paper's spatial component is Nanostring GeoMx spatial
     proteomics + multiplex immunofluorescence imaging (not Visium), and it has **no repository
     accession at all** — the paper's Data Availability statement covers only the scRNA-seq
     (GSE272409) and an unrelated bulk RNA-seq series (GSE272410), and states just "Source data
     are provided with this paper" for everything else. So Step 5's Sjögren's ground-truth check
     is necessarily a comparison against the paper's *reported findings* (and its Source Data
     files, if easily obtainable from the article page), not a reprocessed spatial dataset — this is
     now the only option for that side, not a time-driven fallback choice.

   Rather than making raw spatial processing (spot deconvolution, niche/neighborhood detection, a
   spatial data model beyond plain `AnnData`) a required Phase 1 pipeline stage — disproportionate
   engineering scope for a pilot whose main job is proving the
   assembly→projection→subtraction→mapping shape works — Phase 1 uses whatever's actually
   available per organ (MG: reprocessed Visium; Sjögren's: reported findings) as an independent
   validation check (see the notes in Steps 3, 5, and 7). Building spatial processing as a first-class
   pipeline stage is deferred as a candidate for a later phase.

7. **MG-side stromal extraction (thymic epithelial cells) — when, not if** — Status: **Resolved**
   The source doc's Stromal Extraction step names "thymic epithelial cells" as its first example
   target, and Aim 2 calls for isolating therapeutic targets "for each organ" — this is a required
   part of the project's stated scope, not optional, so it must land somewhere before the project is
   considered complete. The only real question was timing.
   **Resolution:** Not required for Phase 1's definition of done (Phase 1 stays scoped to proving the
   pipeline once, end-to-end, via the Sjögren's pairing — MG only serves as the immune-signature
   donor there, for which the CD45+-sorted GSE233180 is sufficient). Phase 1 may attempt MG-side
   extraction as a stretch goal after the Sjögren's pairing works, since the marginal cost is mostly
   dataset-sourcing, not new code (Step 5's subtraction logic is already generic). If not completed
   during Phase 1, it becomes an explicit required deliverable of **Phase 5** (full integration —
   see [`00-overview.md`](00-overview.md) §4), structurally the same as running the pipeline on RA
   or SLE: MG treated as a target organ, not just a source. It must not be silently dropped. A
   candidate dataset for when this is picked up is noted in Step 5 (the nmTEC paper); its GEO
   accession still needs to be found and vetted.

## 8. Fair-deadline schedule (added 2026-08-26, corrected 2026-08-26)

Deadline: submission to the **IRIS National Fair (India) by 2026-09-30** — ~34 days from when
this was added (corrected from an initially-stated 2026-09-15, which was a self-imposed buffer,
not the real deadline). With the extra time, **Phase 1 proceeds at full rigor exactly as detailed in
§§3–7** — no lazy cuts. Concretely, that means: proper spatial validation on the MG side
(reprocessing the Cell Reports paper's deposited Visium data with our own AUCell scoring, not a
gene-overlap shortcut) and the best available ground-truth comparison on the Sjögren's side
(there's no reprocessable spatial dataset there — see question 6's correction — so the extra time
buys thoroughness in comparing against the paper's reported findings and Source Data, not
reprocessing something that doesn't exist), full test coverage (unit + integration + API-client
fixture tests, not just a handful of core unit tests), and standard Phase 0 infra (a real repo
scaffold, `pytest` with coverage, a proper `Makefile`) — not a stripped-down MVP version of any
of it.

What's still explicitly *not* in scope for this submission — not because of time pressure, but
because it's genuinely separate work by design:
- **MG-side stromal extraction** (§7 question 7) — deferred to Phase 5, as already resolved on its
  own merits (it's structurally a "run the pipeline on another organ" task, the same shape as
  Phases 2–4, not part of proving the pipeline once).
- **Phases 2–7** (RA/SLE/Graves', full 5-disease integration, cross-disease pharmacogenomics,
  packaging/dissemination) — Phase 1 alone, done properly, is the submission.

### Week-by-week plan (2026-08-26 → 2026-09-30, ~34 days)

**Progress vs. plan:** running ahead of schedule. Phase 0 (Week 1's work) and all of Steps 1–2
(originally Week 2's work, planned through Sep 8) were both actually completed by **2026-08-27**
— i.e. Week 2's target was hit about 12 days early, including doublet detection (briefly a gap the
same day, closed before end of day — see Step 2).

| Week | Dates | Work | Est. hours | Avg/day (~6 active days) |
|---|---|---|---|---|
| 1 | Aug 26 – Sep 1 | ✅ **Actually done Aug 26.** Phase 0: proper repo scaffold (`uv` + `pyproject.toml`, `src/mg_thymus_map/` layout, `configs/`, `tests/{unit,integration,fixtures}`), `pytest` + coverage wired up, `Makefile` (`setup`/`test`/`phase1`). Kick off downloads for all 3 datasets (GSE233180, GSE272409, Human Tonsil Atlas) — start early since the tonsil atlas's Zenodo/Bioconductor distribution needs its own loader path. | ~15–20 | ~2.5–3.5 hrs |
| 2 | Sep 2 – Sep 8 | ✅ **Actually done Aug 26–27, ~11 days early.** Steps 1–2 in full: load, QC, normalize, and annotate all 3 datasets onto the shared cell-type vocabulary; batch/dataset-effect check and integration (Harmony/scVI) if needed. Unit tests for QC filters and vocabulary validation written alongside, not bolted on after. This is historically where real-data pipelines lose the most time to surprises (mismatched gene symbols, unexpected metadata gaps) — the extra week of buffer here is deliberate. **Actual surprises hit:** real bugs (not gene-symbol mismatches, but a NaN-propagation bug, a normalization-target bug, and a harmonypy/scanpy version-compatibility bug), all found and fixed; doublet detection didn't get built. | ~30–40 | ~5–6.5 hrs |
| 3 | Sep 9 – Sep 15 | Step 3 in full: TLS signature seeded with the 12-chemokine prior and refined on MG data, with provenance tagging; cell–cell communication analysis (`liana-py`/`squidpy`); **full spatial validation** — reprocess the MG Cell Reports Visium data with our own AUCell scoring against the derived signature, not the gene-overlap fallback. Unit tests for signature determinism and schema. | ~25–35 | ~4–5.5 hrs |
| 4 | Sep 16 – Sep 22 | Step 4: AUCell cross-disease projection onto Sjögren's + tonsil, with proper statistics (distribution comparison, multiple-testing correction, threshold-sensitivity reporting). Step 5: stromal extraction on Sjögren's, plus the ground-truth comparison against the GSE272409 paper's reported fibroblast/pericyte findings and Source Data (there is no reprocessable spatial dataset on the Sjögren's side — see question 6's correction). Tests for the AUCell wrapper, thresholding stats, and the subtraction/regression logic against synthetic planted-signal fixtures. | ~30–38 | ~5–6 hrs |
| 5 | Sep 23 – Sep 29 | Step 6: DGIdb/ChEMBL pharmacogenomic mapping, with client tests against recorded API-response fixtures (no live network calls in CI), proper dedup/ranking with full provenance. Step 7: validation against positive controls (BAFF, CXCL13, plus the spatial cross-checks from weeks 3–4), full findings write-up, README finalized, full test suite pass. | ~22–31 | ~3.5–5 hrs |
| — | Sep 30 | Buffer day / final review / submission. | ~4–6 | — |

**Total: ~130–170 hours over 34 days, averaging ~4–5 hrs/day** — confirmed as realistic against
the stated availability (3–5+ hrs/day, most days), though weeks 2 and 4 (data wrangling across
three differently-formatted datasets; stromal extraction plus the full ground-truth comparison) run
toward the upper end and are the likeliest places to need longer sessions. These are estimates, not
a contract — if actual pace diverges meaningfully, revisit this schedule explicitly (extend a week's
work into the buffer, or re-open this section to decide what to trim) rather than silently cutting
corners.
