# Phase 1 Plan — MG (Thymus) vs. Sjögren's Syndrome Pilot

Status: **In progress.** Phase 0 (repo scaffold) and Phase 1 Steps 1–3 (data
assembly, QC/normalization/annotation/integration, and TLS signature
derivation including spatial validation) are implemented, tested, and
verified against real downloaded data as of 2026-09-01 — see the "Implementation
notes" under each step below for what was actually built and found. Steps 4–7 are
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
- [x] An MG-derived TLS gene/cell-state signature exists, with a written rationale for how it was
      derived. **Done 2026-08-28/2026-09-01** — 25-gene signature (13 literature_prior + 12
      mg_derived), plus spatial validation confirming it scores significantly higher in
      germinal-center/medulla Visium niches than cortex — see Step 3 implementation notes.
- [ ] That signature has been scored (AUCell) against the Sjögren's dataset, with a quantified
      "proportion shared" metric, benchmarked against the healthy tonsil control. **Not started.**
- [ ] Stromal populations in Sjögren's have been isolated after subtracting the shared signal, with a
      ranked marker-gene list. **Not started.**
- [ ] Those markers have been queried against DGIdb/ChEMBL, producing a ranked candidate
      target/drug list with provenance (which database, which gene, which compound). **Not
      started.**
- [~] Every stage above has unit tests (synthetic fixtures) and the full pipeline has one integration
      test (tiny fixture data) — all passing via `make test`. **Steps 1–3 fully covered (cell-state
      calling, communication analysis, signature-merge, spatial loader, and spatial validation all
      covered) — 103 tests total, `make test` green;
      Steps 4–7 not yet built.**
- [x] `README.md` documents how to set up the environment and run the Phase 1 pipeline
      end-to-end via `make phase1` (or equivalent). **Done in Phase 0**; will need a further update
      once Step 3+ stages are wired into `pipeline.py` (currently only loads/validates config).
- [ ] A short written summary of Phase 1 findings exists (even if preliminary/negative) — a
      pipeline that runs but finds no shared signal is still a valid, useful Phase 1 result. **Not
      started** — depends on Steps 3–7.

## 3a. Data-source compliance (ISEF Human Participants / Tissue rules) — checked 2026-08-31

Checked directly against Society for Science's published ISEF International Rules (not assumed) whether
any of this project's four datasets require IRB approval or human-participants paperwork. All four are
secondary analysis of already-published, already-public data — no participant interaction, no physical
tissue in hand, no new data collection — which two separate documented exemptions cover:

- **Human Participants rules** ([societyforscience.org/isef/international-rules/human-participants](https://www.societyforscience.org/isef/international-rules/human-participants/)),
  "Exempt Studies (Do Not Require IRB Preapproval or Human Participants Paperwork)": *"Data/record
  review studies... in which the data are taken from preexisting data sets that are publicly available
  and/or published and do not involve any interaction with humans or the collection of any data from a
  human participant for the purpose of the student's research project."*
- **Tissue & Body Fluid rules** ([societyforscience.org/isef/international-rules/tissue-and-body-fluid](https://www.societyforscience.org/isef/international-rules/tissue-and-body-fluid/)),
  "Exempt Studies (No SRC Pre-Approval Required)": *"Projects utilizing only data or images are exempt
  from IACUC pre-approval ONLY if the originating study is published in a peer-reviewed journal or the
  data is available in a publicly available database. In this case, the student must provide a
  reference to the original study OR link to the database."*

| Dataset | What's actually used | Public/published basis |
|---|---|---|
| MG thymus | GEO `GSE233180` | Public GEO accession, published paper |
| Sjögren's | GEO `GSE272409` | Public GEO accession, published paper |
| Healthy tonsil | Zenodo record for `E-MTAB-13687` | Public Zenodo record, Massoni-Badosa et al., *Immunity* 2024 |
| MG spatial (Visium) | **figshare `10.6084/m9.figshare.25052546`** | Public figshare deposit (CC BY 4.0, no access request), Yasumizu et al., *Cell Reports* 43(9):114677, 2024; original sample collection reviewed/approved by Osaka University's Research Ethics Committee, protocol ID 10038-13 |

**Important distinction to keep straight, checked explicitly because it's easy to conflate:** the
Yasumizu et al. paper's *raw* sequence data is deposited separately at Japan's JGA (Japanese
Genotype-phenotype Archive) under accession `JGAS000672` — that one is **controlled-access**, requiring
a formal Data Access Committee application, and is **not** what this project uses or should cite. What
this project uses is the same paper's *separate*, fully public **figshare** deposit of the processed
Visium output (`res.cxg.h5ad`). Any compliance paperwork or citation for the spatial dataset should
reference the figshare DOI, not `JGAS000672` — citing the JGA accession would incorrectly imply access
to the controlled-access raw data.

**Caveat, stated plainly rather than overclaimed:** this is Society for Science's published rule text,
checked directly against the primary source rather than assumed — not an official ISEF/SRC approval.
The actual compliance determination for this project is made by its affiliated fair's Scientific Review
Committee (SRC), not by this document. Worth a direct confirmation from a mentor/SRC before treating
this as final, even though the rule text maps onto this project's situation cleanly on every point
checked.

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

**Honest caveat on the sex-balance fix, checked at the cell level not just the donor level:**
donor-level representation is perfectly balanced (2 male, 2 female), but *cell-level* representation
still skews male, because the two male donors individually contribute more cells than the two
female donors: on the final post-QC/doublet-removed data, **22,818 male cells (59.9%) vs. 15,254
female cells (40.1%)**. A real, substantial improvement over the original bug (100% male / 0%
female), but not a perfect 50/50 by cell count — worth stating plainly in the Limitations section
rather than letting "2/2 donors" imply a cleaner balance than what's actually there. Not pursued
further (e.g. subsampling cells per donor to force exact cell-level parity) given the improvement is
already substantial and the fair deadline.

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

### Step 3 — TLS isolation (from MG data only) — Status: ✅ Complete (2026-09-01, spatial validation closes out the step)

**Goal:** derive "the MG immune structure" — the shared architecture to project onto Sjögren's.

**Fine-grained cell-state calling — done 2026-08-27.** CellTypist's broad labels ('B cells', 'T
cells') don't distinguish GC B cells or Tfh cells, so added a separate scoring pass:
`src/mg_thymus_map/signature/markers.py` (literature-cited marker panels for GC B cells, Tfh, FDC,
HEV — FDC/HEV included for completeness/reuse on Sjögren's and tonsil, but expected to find nothing
against MG, which is CD45+-sorted and has no stromal cells) and
`src/mg_thymus_map/signature/cell_states.py` (`score_cell_state` wraps scanpy's `score_genes`;
`call_cell_state` flags a cell as the fine-grained type if its score is an outlier — mean + 2·std —
*within* its own broad CellTypist category, the same adaptive-threshold pattern used for mito% and
doublet-score cutoffs elsewhere in this project, rather than a hardcoded score cutoff). Run against
real MG data (45,387 post-QC cells): **406/7,022 B cells (5.8%) called GC B cell-like**,
**1,339/27,348 T cells (4.9%) called Tfh-like**. FDC check against MG's 0 fibroblasts correctly
returned 0, as expected.

**Cell–cell communication analysis — done 2026-08-27, investigated carefully rather than taken at
face value.** Built `src/mg_thymus_map/signature/communication.py`
(`build_tls_cell_group`/`run_communication_analysis`/`filter_between`/`find_pair`), wrapping
`liana-py`'s consensus `rank_aggregate` (`use_raw=False`, since this project's saved AnnData objects
carry log1p data in `.X` with no `.raw` set). Ran on real MG data (`n_perms=1000`) between the
GC_B_cell (406 cells) and Tfh (1,339 cells) groups: 94 candidate interaction rows.

Checked explicitly for four textbook GC/Tfh biology pairs before drawing any conclusion: **only
CD40LG→CD40 (Tfh→GC_B_cell) was found**, ranked among the very top hits (specificity_rank 0.045,
tied with the top tier) — LTB→CD40 (lymphotoxin-beta, itself a well-known TLS-organizing cytokine)
and the CD28/CD80/CD86/CTLA4 costimulatory axis also appeared near the top. CXCL13-CXCR5,
ICOS-ICOSLG, and IL21-IL21R were **not** found — not just filtered from this specific pairing, but
absent from all 6,253 rows of the full all-groups output. Rather than treat this as either "liana
confirmed everything" or "something's broken," checked raw per-group expression proportions
directly against liana's `expr_prop=0.1` filter (minimum fraction of cells expressing a gene within
each group, below which liana excludes the pair before scoring):

| gene | role | GC_B_cell expr. | Tfh expr. | clears 10% floor? |
|---|---|---|---|---|
| CXCL13 | ligand | 1.0% | 5.2% | no, either group |
| CXCR5 | receptor | 32.8% | 22.1% | yes |
| ICOSLG | receptor | 0.2% | 0.0% | no, either group |
| ICOS | ligand (Tfh side) | 4.2% | 91.6% | yes in Tfh |
| IL21 | ligand | 0.2% | 5.0% | no, either group |
| IL21R | receptor | 19.7% | 15.8% | yes |
| CD40LG | ligand | 1.0% | 12.6% | **yes, barely, in Tfh** |
| CD40 | receptor | 34.5% | 0.6% | yes |

Every missing pair traces to its *ligand* failing the detection floor in both groups, and the one
ligand that does clear it (CD40LG, at 12.6%) is exactly the one liana reports. This is the known
scRNA-seq dropout problem for low-abundance, rapidly-secreted signaling molecules (CXCL13, IL21,
ICOSLG are textbook examples — made, translated, and secreted fast enough that steady-state mRNA is
barely captured) rather than a bug in the analysis or evidence the interactions aren't real.
CXCL13 specifically is also expected to be weak here regardless, since it's predominantly an
FDC/stromal product and MG has no stromal cells (CD45+-sorted).

**Consequence for the signature/provenance design:** absence from liana's communication evidence
must never be used to demote or remove a gene from the literature-anchored seed signature — it
reflects a detection ceiling on secreted cytokines, not biology. Communication evidence should only
ever *add* corroborated `mg_derived` entries on top of the `literature_prior` seed list, never gate
it.

**Signature-merge step — done 2026-08-28: the actual final Step 3 output.** Built
`src/mg_thymus_map/signature/merge.py` (`add_communication_evidence`/`save_signature`/
`load_signature`), combining the 13-gene literature seed with communication evidence into a single
provenance-tagged gene list — this is the concrete artifact Step 4 consumes.

**Real bug caught while building this, in the analysis method itself, not the code:** the initial
design filtered communication rows on `specificity_rank` alone (liana's own tutorial does the same,
e.g. `specificity_rank <= 0.01`). Checking the real 94-row GC_B_cell↔Tfh result directly found this
unreliable here — `specificity_rank` has a hard floor in this data, where **13 of the 94 rows tie at
the exact same value** (0.045454...), almost certainly because liana's rank-aggregation method
assigns an identical minimal p-value to every pair ranked #1 across all its component methods; it
can't further distinguish among simultaneous #1s. This makes a specificity-only threshold fragile:
at liana's own stricter tutorial value (0.01), it would have **excluded CD40LG-CD40** — the one
pairing with independent textbook support (see above) — purely because of where the threshold line
fell through the tied block, while keeping less-established pairs. Fix: require **both**
`specificity_rank` and `magnitude_rank` to clear thresholds together. `magnitude_rank` doesn't have
the tie-floor problem — it spreads continuously from 0.043 to 1.0 on the same rows, with a genuine,
fairly wide gap (0.527 to 0.771, the single biggest gap in the sorted list) separating real signal
from generic costimulation noise. This two-statistic requirement is this project's own reasoned
response to an empirical finding, not a citation of established practice — documented as such in
`merge.py` rather than presented as a field-standard default. (A first draft of this same
docstring/reasoning also miscounted how many pairs clear the gap — said "five," the code's own
threshold actually keeps seven; caught by re-deriving the real output end-to-end and checking the
gene count against the description, not trusting the description.)

**Real final signature (25 genes) from actual MG data**, saved to `data/processed/tls_signature.csv`
(gitignored, regenerated by script — same convention as the interim `.h5ad` files, not committed):

| Provenance | Genes |
|---|---|
| `literature_prior` (13) | CCL2, CCL3, CCL4, CCL5, CCL8, CCL18, CCL19, CCL21, CXCL9, CXCL10, CXCL11, CXCL13, TNFSF13B |
| `mg_derived` (12) | CD40LG, CD40, LTB, CD22, PTPRC, FAM3C, CLEC2D, ADAM28, ITGA4, LAMP1, CD28, CD86 |

Of the 12 `mg_derived` genes: CD40LG-CD40 is textbook germinal-center biology; LTB-CD40 is
biologically non-canonical (lymphotoxin-beta signals through LTBR, not CD40 — likely a resource
database or co-expression artifact, included and flagged rather than silently dropped); CD28-CD86 is
real T-B costimulation biology but not TLS-specific on its own; the rest (CD22, PTPRC, FAM3C,
CLEC2D, ADAM28, ITGA4, LAMP1) are included under the same mechanical rule without individual
mechanistic review — a caveat worth remembering if this signature's `mg_derived` half is scrutinized
later, e.g. during Step 7's validation.

**Literature check on all 7 mg_derived pairs — done 2026-08-31.** The above paragraph flagged that
most of the 12 `mg_derived` genes hadn't had individual mechanistic review. Closed that gap with a
dedicated literature search per pair, run precisely because the `mg_derived` tag only certifies
"found in this project's own MG communication evidence," not "novel to science" — a distinction easy
to blur if left unchecked. Findings:

| Pair | Status | Evidence |
|---|---|---|
| CD40LG→CD40 | **Well established** | Textbook Tfh–B cell help signaling, decades of literature |
| CD28→CD86 | **Well established** | Direct receptor-ligand binding, documented specifically in T–B interactions ([Int Immunol 9(5):637, 1997](https://academic.oup.com/intimm/article-pdf/9/5/637/18324139/090637.pdf)) |
| CD22→PTPRC (CD45) | **Well established** | CD22 recognizes α2,6-sialylated glycans on CD45 as a direct cis-ligand, documented since the 1990s ([Doody et al. 1995, PMID 7537381](https://pubmed.ncbi.nlm.nih.gov/7537381/); [PMC11407986](https://pmc.ncbi.nlm.nih.gov/articles/PMC11407986/)) |
| ADAM28→ITGA4 | **Well established** | ADAM28's disintegrin domain is an activation-dependent ligand for α4β1 integrin on lymphocytes, shown experimentally in multiple dedicated papers ([Bridges 2002, PMID 11724793](https://pubmed.ncbi.nlm.nih.gov/11724793/); [Bridges 2003, PMID 12667064](https://pubmed.ncbi.nlm.nih.gov/12667064/?dopt=Abstract); [McGinn 2011, doi:10.1042/CBI20100885](https://onlinelibrary.wiley.com/doi/10.1042/CBI20100885)) |
| LTB→CD40 | **Not supported** | No literature found showing LTB binds CD40 directly — LTB's actual receptor is LTβR, structurally and functionally distinct ([PMID 14517219](https://pubmed.ncbi.nlm.nih.gov/14517219/)). Confirms the artifact suspicion above. |
| FAM3C→LAMP1 | **Weak** | Only evidence is one entry in a generic high-throughput affinity-capture-MS interactome screen ([BioPlex, Huttlin et al. 2017, *Nature*, PMID 28514442](https://thebiogrid.org/interaction/2267326/lamp1-fam3c.html)), run in HEK293T cells (not immune cells), never characterized as a functional signaling pair |
| FAM3C→CLEC2D | **No evidence found** | No dedicated study or database record of this specific pairing. FAM3C's known receptors are FGFRs (established for its paralog FAM3B/PANDER); CLEC2D's established ligand is KLRB1 (CD161). Looks like a computational/predicted database entry rather than a shown interaction. |

**Net result:** 9 of the 12 `mg_derived` genes (CD40LG, CD40, CD28, CD86, CD22, PTPRC, ADAM28, ITGA4)
sit on interactions with real, independent literature support predating this project — they weren't
in the TLS-specific seed list, but they are not new to science. Only LTB (via LTB-CD40) and FAM3C
(via both its pairs) lack solid independent support. **No schema change made**: the `mg_derived`
provenance tag is left as-is (it correctly means "corroborated by this project's own MG communication
evidence," a distinct and still-useful claim from `literature_prior`'s "came from the TLS-specific
literature search"). Whether to split `mg_derived` into literature-confirmed vs. -unconfirmed
sub-tags is a deliberate future decision — it would touch `add_communication_evidence`'s output
schema, its tests, and Step 4's consumption of the signature — not something to fold in silently
alongside a literature audit.

**Threshold honesty note, same date — are the specificity_rank ≤ 0.05 / magnitude_rank ≤ 0.6 cutoffs
"ideal"? No, and that shouldn't be implied.** The two-statistic rule fixes one specific, concretely
observed failure (specificity_rank's tie floor excluding CD40LG-CD40 at liana's own stricter tutorial
cutoff of 0.01) — it is not validated as a generally correct or optimal rule:

1. `magnitude_threshold=0.6` was picked by eyeballing the single largest gap in a sorted list of only
   15 values (the rows that already passed `specificity_threshold=0.05`) — a "natural breaks"
   heuristic, not a statistically principled cut. No permutation test or bootstrap stability check
   (resampling cells and re-running `liana` to see if the same gap reappears) was done.
2. `specificity_threshold=0.05` was itself picked to sit just above the 0.045454 tie floor — i.e., to
   admit that whole tied block — rather than derived independently the way magnitude's gap was.
3. The combination was judged "successful" mainly because it kept the one pair (CD40LG-CD40) already
   known to be real from independent textbook biology. That is tuning against a single labeled
   positive, not validation against an independent holdout set. Of the four textbook pairs checked
   going in, three (CXCL13-CXCR5, ICOS-ICOSLG, IL21-IL21R) were absent from liana's output entirely
   due to scRNA-seq dropout and never actually entered the threshold decision.
4. No sensitivity sweep was run (e.g. `magnitude_threshold` ∈ {0.5, 0.6, 0.7} or
   `specificity_threshold` ∈ {0.03, 0.05, 0.1}) to check whether the resulting 7-pair/12-gene list is
   robust to nearby threshold choices, and no cross-method check (e.g. rerunning with `squidpy` or
   `CellPhoneDB` alone on the same cells) was done.

Bottom line: this is a defensible, documented fix for one real problem found in this project's own
data, not a statistically certified or field-validated cutoff. The literature check above (mostly
independently corroborating the resulting gene list) is reassuring, but it corroborates the *output*
after the fact — it doesn't retroactively make the threshold-selection process itself rigorous.

**Sensitivity sweep — done 2026-08-31, closing item 4 above.** Re-ran the real `liana` analysis
against `data/interim/mg_with_cell_states.h5ad` (same `n_perms=1000`, `seed=1337` — reproduced the
documented 6,253 total rows / 94 `GC_B_cell`↔`Tfh` rows exactly, confirming this is the same data and
method, not a re-derivation) and swept `specificity_threshold` × `magnitude_threshold` over a grid.
This revises the assessment above in **both** directions, not just confirming it was bad:

| | Verdict before the sweep | Verdict after running it |
|---|---|---|
| `magnitude_threshold=0.6` | "eyeballed a gap" — implied weak | **More defensible than implied.** At `specificity_threshold=0.05`, the gene set is *identical* (12 genes) for every `magnitude_threshold` from 0.55 to 0.75 — a real, verified stability plateau, not just a visually large gap in 15 numbers. Below 0.55 the set shrinks (loses CD28, CD86, LAMP1 at 0.50); above 0.75 it grows (gains CD53, CD80 at 0.80). |
| `specificity_threshold=0.05` | "sits just above the tie floor" — implied arbitrary but stable | **Less defensible than implied.** The result doesn't just lose CD40LG-CD40 below 0.05 — it collapses to 0 genes entirely at 0.01, 0.02, and 0.03, then jumps straight to 12 genes at 0.05, 15 at 0.07, 21 at 0.10. This is a cliff at the 0.045454 tie floor, not a gradient. `0.05` is the smallest round number that clears the tied block of 13 rows — not validated as "correct" in any stronger sense. An equally defensible round number like `0.07` would have added CD5, CD72, and ST6GAL1 to the signature instead. |

**Net conclusion:** the 12-gene result is robust to `magnitude_threshold` choice within a genuine
~0.2-wide band, but fragile to `specificity_threshold` choice in a way that isn't fixable by simply
picking a "better" constant — the tie floor itself is the problem. The more durable fix is likely
either (a) dropping `specificity_rank` as a hard cutoff and relying on `magnitude_rank`'s validated
plateau alone, or (b) checking whether `liana`'s underlying `cellphone_pvals` column (present in the
same output, not rank-transformed, so it shouldn't share the tie-floor artifact) gives a usable
alternative. Neither has been implemented — see the validation backlog immediately below.

## 4b. Validation backlog — cutoffs needing further work (added 2026-08-31)

Prioritized, concrete next steps for the ad hoc/adaptive cutoffs flagged as not-yet-validated above.
Added because flagging fragility without a plan to close it isn't sufficient — Step 4 will build a
headline cross-disease finding on top of this signature, so an unresolved threshold problem here
propagates forward if left alone.

- [x] **Fix or replace `specificity_rank` as a hard cutoff — checked 2026-08-31, negative result.**
      Checked whether `liana`'s raw `cellphone_pvals` column avoids `specificity_rank`'s tie-floor
      artifact. It does not — it's dramatically worse: **61 of the 94 rows (65%) are tied at the exact
      floor value 0.000**, plus 4 more at 1.000 and 3 at 0.005 (only 29 unique values across all 94
      rows). Root cause: `liana`'s own documented CellPhoneDB p-value formula is
      `count(permuted ≥ observed) / n_perms`, and with `n_perms=1000` the smallest representable
      non-zero p-value is 0.001 — any pair no permutation ever matched floors at exactly 0.000. This
      is a resolution artifact of permutation testing at this depth, not fixable by a different cutoff
      on the same statistic, so `cellphone_pvals` is ruled out as a replacement. `specificity_rank`
      (aggregated across all of `liana`'s component methods, not a single permutation p-value) remains
      the better-behaved option despite its own tie floor. **Remaining options, not yet done:** (a)
      drop `specificity_rank` as a hard gate and rely on `magnitude_rank`'s validated 0.55–0.75 plateau
      alone, or (b) stop hunting for a better p-value-like statistic on the same underlying permutation
      test and add a genuinely orthogonal evidence line instead — which is also what the literature
      recommends, see below.

      **Permutation-depth check — done 2026-08-31, real but bounded improvement.** [Phipson & Smyth
      2010](https://www.degruyterbrill.com/document/doi/10.2202/1544-6115.1585/html) ("Permutation
      P-values Should Never Be Zero") establish that an exact `p=0.000` from a finite permutation test
      is a known resolution artifact, not evidence the biology is weak — and predicts more permutations
      should improve resolution. Tested directly: reran the same analysis at `n_perms` = 1,000 →
      10,000 → 100,000 (same seed, same data). The top `specificity_rank` tie cluster shrank
      **13 → 9 → 7 pairs** across the three depths (runtime 20s → 77s → 708s) — a real, diminishing-
      but-genuine improvement, consistent with a true resolution limit rather than a bug. **The final
      12-gene signature is unaffected at every depth** — recomputing the output at the project's actual
      thresholds (`specificity_threshold=0.05`, `magnitude_threshold=0.6`) against all three runs gives
      the identical 12 genes each time, because every pair that ever appears in the reshuffling tie
      block stays well under 0.05 regardless of resolution. Also reassuring: CD80-CTLA4 and CD86-CTLA4
      remain tied for "most significant" at every depth but never enter the signature, because they
      fail `magnitude_rank` (1.000, 0.912 — both over the 0.6 cutoff) — the two-statistic design
      catches them where `specificity_rank` alone cannot. **Recommendation:** raise `n_perms` (e.g. to
      10,000) as a general-quality improvement for future signature work, both for resolution and to
      move away from the uncorrected `b/m` p-value formula — but this is not required to trust the
      current 12-gene result, which is now empirically shown stable across a 100× range of permutation
      depth rather than merely assumed to be.
- [ ] **Cross-method check.** Rerun the same `GC_B_cell`↔`Tfh` comparison through a single
      non-consensus method (`squidpy`'s ligrec, or one `liana` component method alone, e.g.
      CellPhoneDB) on the same cells, and check how much the resulting gene set overlaps with the
      12-gene `rank_aggregate` result. Tests whether the finding is a consensus-method artifact or
      reproduces under an independent method.
- [ ] **Expand the validation benchmark past 4 pairs.** The "does this method recover known biology"
      check so far is only CD40LG-CD40, CXCL13-CXCR5, ICOS-ICOSLG, IL21-IL21R (from `liana`'s own
      tutorial framing). A larger curated set of known GC/Tfh interactions, pulled from actual
      immunology literature rather than one tutorial's examples, would make this a real benchmark
      instead of an anecdote.
- [ ] **Test `min_genes_per_cell=200`/`min_cells_per_gene=3` the same way the mito-% cutoff was
      tested.** Row 3 in the cutoffs table (§4a) was checked against real per-cell-type retention and
      found to fail for generic conventions; rows 1–2 haven't been given that same empirical check on
      MG/Sjögren's/tonsil data specifically, despite tracing to the same kind of borrowed-tutorial
      source (§4a's honest caveat on rows 1–2).
- [ ] **Build the AUCell threshold (Step 4) as adaptive-against-tonsil, not fixed.** Highest-priority
      item on this list — the plan already states this cutoff "drives the headline result," and it
      doesn't exist yet. The stated intent (benchmark against the tonsil control's own score
      distribution) is the right approach in principle; make sure it's actually built that way rather
      than defaulting to a fixed AUCell score under time pressure.

**What the field recommends before trusting CCC results — researched 2026-08-31, before starting
spatial validation, to check whether anything cheaper/complementary should happen first.** Both
`sc-best-practices.org`'s cell–cell-communication chapter and `liana`'s own validation methodology
(Dimitrov et al. 2022, *Nat Commun*, doi:10.1038/s41467-022-30755-0; Dimitrov et al. 2024's LIANA+,
*Nat Cell Biol*, doi:10.1038/s41556-024-01469-w) converge on the same answer: CCC inference from
scRNA-seq alone is a hypothesis-generation step, and predictions should be corroborated against
**orthogonal modalities** before being trusted — specifically **spatial co-localization**, **protein
abundance**, or **downstream-signalling activity**. `liana`'s own papers benchmark their predictions
against exactly these three. This is a genuine external confirmation, not just self-justification,
that **spatial validation (already planned below) is the field-standard next move, not an
idiosyncratic choice for this project.**

Of the three, protein abundance isn't available (no CITE-seq/proteomics in this project's data) and
spatial co-localization is the larger effort already scoped below. The one that's cheap and doable
*before* that larger lift, using data already in hand: **a downstream-signalling concordance check**
— e.g. using `liana`'s/`decoupler`'s pathway or TF-target-activity tools to check whether a target
gene set plausibly downstream of a given receptor (e.g. NF-κB target genes downstream of CD40
signalling) is actually elevated in the receptor-expressing population (GC_B_cell for CD40). This
wouldn't replace spatial validation, but would add a second, cheap, already-in-hand line of evidence
specifically for the pairs currently resting on the least support — LTB-CD40 (flagged non-canonical)
and the two FAM3C pairs (no independent literature support found). **Not yet implemented** — flagged
here as a candidate item, not started, pending a decision on whether it's worth the scope before
moving to spatial validation.

Left to close out Step 3: the spatial validation sub-step below.

- **Seed with the established 12-chemokine TLS signature plus TNFSF13B/BAFF** (13 genes; per
  question 3's resolution and its 2026-08-28 follow-up below), then refine using the MG data rather
  than deriving fully de novo.
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

  **Status check — 2026-08-31: not started, no spatial data downloaded yet.** Confirmed by listing
  `data/raw/` directly: only the three scRNA-seq datasets (`mg_thymus`, `sjogrens_salivary_gland`,
  `healthy_tonsil`) exist. No Visium/spatial data has been acquired. Next concrete step: locate the
  Cell Reports paper's actual data-deposit accession (GEO/Zenodo/other) for its Visium samples before
  anything can be downloaded or reprocessed.

  **Done — 2026-09-01.** The figshare deposit (`res.cxg.h5ad`, 3.5 GB) was downloaded manually by the
  user and dropped in `data/raw/mg_spatial/`. Built `src/mg_thymus_map/data/mg_spatial.py`
  (`load_mg_spatial`) and `src/mg_thymus_map/signature/spatial_validation.py`
  (`score_signature_aucell`/`compare_niche_enrichment`), 17 new unit tests (103 total, `make test`
  green), config entry added to `configs/phase1_mg_sjogrens.yaml`.

  **Real finding, checked before use, not assumed: the h5ad bundles three separate studies.**
  `obs['project']` has `Yasumizu_et_al` (36,838 spots, `condition` thymoma/hyperplasia -- the actual
  MG Cell Reports data this project is cleared to use per section 3a), `Helmi_et_al` (18,329 spots)
  and `Suo_et_al` (4,629 spots), both entirely `condition == "normal"` -- other public reference
  thymus atlases the depositors integrated for context, matching this section's own expectation of
  "normal fetal/pediatric thymus controls" bundled alongside the MG data. `load_mg_spatial` defaults
  to `Yasumizu_et_al` only (an explicit opt-in is required for the other two), since their own
  license/access terms haven't been independently checked the way section 3a checked Yasumizu's.
  **Worth a mentor/SRC confirmation before treating this scoping decision as final**, same caveat
  already stated for section 3a generally.

  **"Repeat the cleaning methods" -- what carried over and what couldn't, checked rather than
  assumed identical.** The deposit is Yasumizu et al.'s own fully-processed atlas object: QC metrics
  (`n_genes_by_counts`, `total_counts`, `pct_counts_mt`), scVI/Harmony embeddings, Leiden clusters,
  and spatial niche labels are already computed by the depositors, and `.X` is already
  `normalize_total`+`log1p`'d with **no raw-counts layer retained** (confirmed by inspection:
  `expm1(.X).sum(axis=1)` is constant at ~7053 across every spot checked, i.e. scanpy's
  per-dataset-median `normalize_total` default -- not this project's own `target_sum=1e4`
  convention -- and `.raw`/`.layers` are both empty). Consequences, decided explicitly rather than
  left implicit:
  - **Filtering: reused the exact same functions** (`qc.filters.filter_cells`,
    `adaptive_mito_threshold`, `filter_genes`) unmodified, on the depositors' own precomputed QC
    columns, at the same `min_genes_per_cell=200` project convention. Found the **same NaN-mito edge
    case as GSE233180** (1 zero-count spot, `pct_counts_mt` = 0/0 = NaN) -- `adaptive_mito_threshold`'s
    existing `nanmedian` fix handled it correctly without any code change, a real (if small) piece of
    evidence the earlier fix generalizes.
  - **Normalization: could NOT be re-run** -- there are no raw counts to normalize with
    `target_sum=1e4`. Documented as a limitation rather than silently skipped. Mitigation: AUCell
    scoring is rank-based within each spot, and `normalize_total` followed by `log1p` is a
    monotonic transform of each spot's raw counts regardless of the target sum used -- so gene
    ranks within a spot, and therefore the AUCell score, are unaffected by which target sum the
    depositors used. This is a real property of the method, not just a hopeful assumption, but it's
    still worth a mentor's sanity check given it wasn't independently re-derived here.
  - **Doublet detection: deliberately skipped**, not an oversight -- doublets are a per-droplet
    single-cell concept; Visium spots legitimately contain multiple cells by design.
  - **Cell-type annotation: not re-run** -- the depositors already ship per-spot deconvolution
    scores for ~60 cell states (`B_GC`, `CD4_Tfh`, `cTEC`, etc. in `obs`), a finer-grained and more
    directly comparable source than re-running CellTypist on spot-level (multi-cell) expression
    would produce.

  **QC results (Yasumizu_et_al, 36,838 spots -> filtered):** adaptive mito threshold **5.26%**
  (`min_genes=200`, `min_cells_per_gene=3`) -- **36,403/36,838 spots kept (98.8%)**, 13,206/13,207
  genes kept. Retention is high and even across the 10 samples (94.8-100%) and across
  thymoma/hyperplasia (98.8%/98.6%), as expected for an already-curated Visium deposit. One real,
  investigated-not-assumed exception: the `junction` niche (cortico-medullary boundary) retained only
  **38.6%** of its spots (123/319) -- checked directly rather than accepted at face value: junction
  spots have a median of only 160 detected genes (vs. thousands in cortex/medulla), so most fail the
  `min_genes=200` floor on their own QC metrics, not from any filtering bug. Plausibly a real
  boundary-zone sparsity effect (thin transitional tissue, fewer captured transcripts) rather than a
  processing artifact, but not independently confirmed against the source paper. Saved to
  `data/interim/mg_spatial_qc.h5ad` (36,403 x 13,206; gitignored, regenerated from raw + this
  pipeline, same convention as the other interim files).

  **Spatial validation result -- the actual Step 3 close-out, real numbers:** all **25/25** signature
  genes are present in the Visium gene panel (no coverage gap). AUCell-scored every cleaned spot
  against the full 25-gene signature (`decoupler.mt.aucell`, `tmin=5`), then ran one-sided
  Mann-Whitney U tests (BH-corrected across niches) comparing each niche's score against `cortex`:

  | Niche | n spots | Median score | Cortex median | p-value (BH-adj.) |
  |---|---|---|---|---|
  | `medulla_GC` (germinal center) | 117 | **0.148** | 0.050 | 9.1e-70 |
  | `medulla` | 3,711 | **0.115** | 0.050 | ~0 |
  | pooled medulla/medulla_GC/medulla_FN1 | 4,178 | **0.110** | 0.050 | ~0 |
  | `medulla_FN1` | 350 | 0.055 | 0.050 | 0.248 (not significant) |
  | `stroma` | 3,141 | 0.039 | 0.050 | 1.000 (not significant) |
  | `junction` | 123 | 0.033 | 0.050 | 1.000 (not significant) |

  **Update, 2026-09-01 (see "Limitations (2),(3),(4) closed" below for the full analysis): the
  normal-thymus baseline check found broad `medulla` enrichment also occurs in normal (non-MG)
  thymus, so `medulla_GC` specifically -- not pooled `medulla` -- is the disease/GC-specific
  result and should be read as this validation's headline finding.**

  **The signature is significantly and substantially enriched specifically in medulla and
  medulla_GC (germinal-center) spots** -- medulla_GC has the single highest median score of any
  niche in the tissue, matching the biological hypothesis this check exists to test (Step 3's
  signature was derived from GC B cell / Tfh communication evidence; it should, and does, score
  highest in the spatially-defined germinal-center niche). Held up when split by condition: the
  pooled medulla comparison stays significant (p~0) in both thymoma (n=3,018) and hyperplasia
  (n=1,160) subsets separately, not just pooled. **Real specificity, not a blanket "everything is
  high" artifact**: `stroma` and `junction` are NOT enriched (medians at or below cortex), and
  `medulla_FN1` (a distinct fibronectin-rich medullary subregion, per the original niche_annot_v2
  scheme) does not reach significance either -- the signature discriminates between spatial regions
  rather than trivially scoring high everywhere, which is exactly the "not circular / not just
  generic immune genes" defense this check was added for (see question 6). Full results table saved
  to `data/processed/spatial_validation_results.csv` (gitignored, regenerated by script).

  **AUCell-rank-invariance claim -- empirically confirmed, 2026-09-01, closing limitation (1)
  below.** The argument that AUCell scores don't depend on which `target_sum` was used to normalize
  (since `normalize_total` rescales every gene in a given cell/spot by the same factor, and `log1p`
  is monotonic, so within-cell gene ranking is unchanged regardless of target_sum) was tested
  directly rather than left as reasoning alone: took real MG scRNA data
  (`data/interim/mg_with_cell_states.h5ad`, which -- unlike the spatial deposit -- has raw counts in
  `layers['counts']`), normalized the *same* raw counts twice (`target_sum=1e4`, this project's own
  convention, vs. `target_sum=7053`, matching the constant row-sum found in the MG spatial deposit's
  already-normalized `.X`), and AUCell-scored both against the real 25-gene signature across all
  45,387 cells. Result: **scores were numerically identical between the two normalizations -- max
  absolute difference 0.0000000000, Pearson correlation 1.0000000000, 0/45,387 cells with any
  rank-order difference.** The invariance claim is now empirically confirmed on real data, not just
  argued mathematically.

  **Limitations (2), (3), (4) below -- all closed 2026-09-01**, by reading the actual Yasumizu et al.
  paper (obtained via its Osaka University institutional-repository open-access PDF, since the
  ScienceDirect/Cell Reports page 403s to automated fetches) and running two follow-up analyses.
  `bootstrap_median_diff` (`spatial_validation.py`) was added as a small, tested, reusable function
  for limitation (2)'s check; the normal-baseline comparison for (3) was run ad hoc (not saved as a
  permanent module, consistent with how this project's other one-off statistical checks --
  sensitivity sweeps, permutation-depth checks -- are documented as findings here rather than built
  into the pipeline).

  **(4) Cross-check against the paper's own reported findings -- done, and it corroborates the
  result.** The paper explicitly reports **CXCL13** as "a key chemokine for the maintenance of the
  GC," stating it "was present both inside and around GCs in the medulla" -- CXCL13 is in this
  project's signature (`literature_prior`). It also reports **CCL19** as specific to the medulla
  ("CCL19-CCR7 interactions were specific to ... the medulla"; medulla-resident mTECs "expressed
  both CCL25 and CCL19"), and **CXCL10** as expressed by migDCs (a medulla-enriched population).
  CCL19 and CXCL10 are also both in the signature (`literature_prior`). **3 of the 25 signature
  genes are thus directly, independently corroborated by the original paper's own narrative as
  medulla/GC-associated** -- a real external check, not just internal consistency between our
  signature and the depositors' own spatial cluster labels. Honestly, the paper's focus is a
  specific chemokine-receptor axis analysis (CCR9-CCL25 for cortex, CCR7-CCL19/CXCL13 for
  medulla/GC), not an exhaustive profiling of every gene in a 25-gene signature -- the other 22
  genes (all 12 `mg_derived`, plus CCL2/3/4/5/8/18/21, CXCL9/11, TNFSF13B) are simply not
  individually named in the paper's main text. Absence of mention isn't evidence against them, just
  outside this particular paper's stated scope (most are costimulatory molecules -- CD40/CD40LG/
  CD28/CD86/LTB -- not chemokines, and this paper's spatial analysis centers on chemokine-receptor
  pairs specifically).

  **(3) Normal-thymus baseline -- done, and it meaningfully refines the headline claim, not just
  confirms it.** Read the paper's STAR Methods data-availability table to identify what
  `Helmi_et_al`/`Suo_et_al` (the h5ad's `obs['project']` spelling; the paper's own reference list
  spells the author "Heimli") actually are: **Heimli et al. 2022** (Front. Immunol. 13:1092028,
  public GEO **GSE207205**) and **Suo et al. 2022** ("Mapping the developing human immune system
  across organs," *Science* 376, doi:10.1126/science.abo0510, public Human Cell Atlas
  fetal-immune portal) -- both peer-reviewed-published, publicly-accessible datasets, meeting the
  same ISEF Human Participants/Tissue exemption already checked for this project's other four
  datasets (section 3a). Documented in `mg_spatial.py`'s docstring and the config; safe to use.
  Loaded all three projects (59,796 spots total), applied the same cleaning as before (single
  pooled adaptive mito threshold this time: 5.85%, 98.9% retention), AUCell-scored, and compared
  medulla vs. cortex **within each project separately**:

  | Project | n medulla spots | Medulla median | Cortex median | p-value |
  |---|---|---|---|---|
  | Yasumizu_et_al (MG) | 3,734 | 0.116 | 0.051 | ~0 |
  | Heimli_et_al (normal) | 3,674 | 0.136 | 0.078 | ~0 |
  | Suo_et_al (normal) | 797 | 0.124 | 0.063 | 2.7e-279 |

  **Important, honest refinement: broad medulla-vs-cortex enrichment is NOT MG-specific** -- both
  normal-reference projects show the *same* significant medulla enrichment as the MG tissue, if
  anything with a larger absolute gap in Heimli's normal data. This makes biological sense: the
  medulla's core chemokine axis (CCL19-CCR7 organizing T-cell/DC trafficking) is normal thymic
  physiology, not an MG-specific pathological feature -- consistent with the paper's own framing
  ("CCL19-CCR7 interactions were specific to ... the medulla" in "both tumor and normal tissues").
  **What IS MG-specific, and the reason this doesn't undercut the validation:** `medulla_GC` and
  `medulla_FN1` are granular sub-niche labels that **only exist in the Yasumizu_et_al (MG) data** --
  both normal-reference projects have **zero** spots labeled `medulla_GC` or `medulla_FN1` (confirmed
  directly from the obs counts), because the depositors only subdivided medulla into a GC-containing
  cluster where ectopic germinal centers are actually present to cluster out -- consistent with the
  well-established biology that ectopic GC formation in thymic medulla is itself a pathological
  feature of MG, not a normal-thymus phenomenon. **This means `medulla_GC`'s 9.1e-70 enrichment
  result (the strongest result in this validation) is the one that's actually disease/GC-specific,
  and should be read as the headline finding rather than the broader pooled-medulla comparison**,
  which reflects generic medullary chemokine biology present in both healthy and diseased thymus.
  Revises how the original headline framing (2026-09-01, first pass) should be read -- worth noting
  explicitly rather than letting the more impressive-sounding "pooled medulla" number stand
  unqualified.

  **(2) medulla_FN1 -- done, genuine biology confirmed two independent ways, both pointing the same
  direction.** First, the paper itself directly answers this: "The stroma and medulla_FN1 regions
  were characterized by high numbers of endothelial cells, fibroblasts, and vascular smooth muscle
  cells" -- i.e. the depositors' own cellular-composition analysis groups `medulla_FN1` with
  `stroma`, not with the GC/medulla lymphoid compartment. Since this project's own result already
  found `stroma` is NOT enriched, `medulla_FN1`'s null result is exactly what a stromal-composition
  region should show -- a real, paper-confirmed biological distinction, not a data artifact. Second,
  ran `bootstrap_median_diff` (2,000 resamples) on the actual median-score gap vs. cortex, for scale
  against the two clearly-significant niches:

  | Niche | n | Median diff vs. cortex | 95% bootstrap CI | vs. medulla_GC's effect |
  |---|---|---|---|---|
  | medulla_GC | 118 | 0.099 | [0.087, 0.109] | -- |
  | medulla | 3,734 | 0.065 | [0.062, 0.067] | 66% as large |
  | medulla_FN1 | 350 | 0.004 | [0.0005, 0.0067] | **4% as large** |

  Note this CI barely excludes zero, in slight tension with the earlier Mann-Whitney test's
  BH-corrected p=0.248 (not significant) -- worth stating honestly rather than picking whichever
  number reads better: the two tests answer different questions (stochastic dominance across the
  whole distribution vs. a direct median-difference estimate) and can disagree at a genuinely
  marginal effect size like this one. Read together with the paper's compositional evidence above,
  the weight of evidence favors "real, tiny, non-GC biological signal consistent with a
  stromal-like composition" over "purely an underpowering artifact" -- but the effect, if real, is
  only ~4-6% the size of the two genuine GC/medulla effects, i.e. functionally negligible either
  way for this validation's conclusion.

**Tests to add later:** unit test that signature-derivation is deterministic given fixed input +
seed; a test that the output signature format matches what Step 4 expects (schema contract); a
test that every gene in the output signature carries a provenance tag (literature-prior or
MG-derived).

### Step 4 — Cross-disease projection (AUCell) — Status: ✅ Complete (2026-09-02)

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

**Implementation — done 2026-09-02.** Built `src/mg_thymus_map/scoring/cross_disease_projection.py`
(`label_tls_region_cells`, `derive_shared_architecture_threshold`, `proportion_exceeding`,
`threshold_sensitivity_sweep` — 8 new unit tests, 100% coverage, 115 total tests project-wide).
Deliberately reuses `signature/spatial_validation.py`'s `score_signature_aucell` and
`compare_niche_enrichment`/`bootstrap_median_diff` rather than duplicating them — those are generic
per-cell gene-set-scoring and group-comparison utilities, not spatial-specific, and the plan already
called for AUCell specifically in both places.

**"TLS-region cells" definition, and a scoping decision worth remembering.** Sjögren's GSE272409 has
no reprocessable spatial deposit (see Step 5's note on this same limitation), so unlike Step 3's
spatial niches, "TLS-region" here means fine-grained cell-state calls: `is_gc_b_cell | is_tfh`
(`signature/cell_states.py`, the same two states Step 3 operationalized for MG). FDC/HEV markers
exist too (`markers.py`) and are in principle scoreable here since Sjögren's/tonsil are whole-tissue
(unlike MG's CD45+-sorted data) — checked against the real data and found too sparse in the healthy
tonsil reference to support a reference distribution (7 FDC / 1 HEV out of 38,072 tonsil cells, vs.
1,186 GC B cells / 592 Tfh), so Step 4 is scoped to GC_B_cell/Tfh only, consistent with what Step 3
already validated.

**Data used — the real, already-processed data, not a fresh run.** `data/interim/sjogrens_with_cell_states.h5ad`
(65,938 cells) and `data/interim/tonsil_with_cell_states.h5ad` (38,072 cells) already existed on disk
from a prior session's Step 2/3 pass (gitignored, same convention as `mg_with_cell_states.h5ad`) —
full QC, CellTypist annotation, and all ten fine-grained cell-state calls already present. All 25
signature genes are present in both datasets' `var_names` (zero missing), so no `tmin` degradation to
worry about. TLS-region counts: Sjögren's 657/65,938 (570 in the 7 PSS patients, 87 in the 6 SICCA
comparator patients); tonsil 1,778/38,072.

**Real result (from `data/processed/step4_cross_disease_results.csv`,
`step4_threshold_sensitivity.csv`, `step4_headline_metric.csv`) — a genuinely positive, well-powered
finding:**

1. **Headline metric:** at a threshold set to the tonsil TLS-region cells' own median AUCell score
   (0.1069 — the "shared architecture is normal here" bar, not an arbitrary constant), **69.1%
   (394/570) of Sjögren's (PSS) TLS-region cells exceed it**, vs. tonsil's own ~50% self-rate at that
   same threshold by construction. Bootstrap 95% CI on the median difference (PSS TLS − tonsil TLS):
   **+0.0231 [0.0168, 0.0289]** — excludes zero, a real effect, not underpowered noise.
2. **Threshold-sensitivity sweep** (closing the risk-table item on arbitrary thresholds): PSS
   TLS-region cells clear the tonsil-derived bar at 95.1% (10th percentile) down to 26.7% (90th
   percentile) — always well above the reference's own self-rate at each percentile (89.9% → 10.0%).
   The finding holds across the whole threshold range, not just at one cherry-picked cutoff.
3. **Sjögren's PSS TLS-region vs. tonsil TLS-region, direct test:** one-sided Mann-Whitney,
   median 0.130 vs. 0.107, **p = 7.5×10⁻²⁶** (BH-adjusted 1.5×10⁻²⁵). SICCA TLS-region trends the
   same direction (median 0.118) but doesn't clear significance at n=87 (p=0.086) — consistent with
   SICCA being a symptomatic-but-non-autoimmune comparator, not a null population.

   **Patient-level re-verification, added 2026-09-02 — this headline result is NOT a pseudoreplication
   artifact.** After finding that item 5's PSS-vs-SICCA claim didn't survive correcting for
   within-patient cell non-independence, re-checked this comparison (the actual Step 4 headline) the
   same way, rather than assuming it was fine because it wasn't the one that broke. Patient/donor-level
   Mann-Whitney (7 PSS patient medians vs. 4 tonsil donor medians — tonsil capped at
   `max_donors=4`, see Step 1): **p = 0.021**, PSS medians (0.111–0.154) sitting above most tonsil
   donor medians (0.096–0.118). Then the same exact cluster-permutation check used for item 5
   (pooled-cell median diff, calibrated against all C(11,7)=330 possible ways to split these 11
   patients/donors into groups of 7 and 4): observed diff **+0.0231**, and **not one of the other 329
   possible relabelings produced a diff that large — exact one-sided p = 0.00303, the theoretical
   minimum achievable p-value at this sample size** (1/330). This is about as strong a confirmation
   as this dataset can mathematically produce: the true PSS/tonsil split is the single most extreme
   split possible. Saved to `data/processed/step4_pss_tonsil_patient_level_test.csv`.
4. **Specificity check** (directly answers the risk-table's "circular / trivially matches generic
   immune genes" concern): within Sjögren's, TLS-region cells score far higher than non-TLS-region
   cells of the same tissue — PSS median 0.130 vs. 0.028 (p≈7×10⁻²¹²), SICCA median 0.118 vs. 0.028
   (p≈5×10⁻¹⁹). The signature isn't just picking up generic immune-cell background.
5. **PSS vs. SICCA, within TLS-region cells only — status downgraded 2026-09-02, see below.**
   Cell-level test: PSS scores significantly higher than SICCA (median 0.130 vs. 0.118, p=0.006).
   **This claim did not survive a more careful re-check and should be treated as unconfirmed, not
   as a finding** — see the patient-level re-test immediately below, which is the actual current
   read on this specific comparison.

   **Investigation trail, added 2026-09-02 (kept in full — this is exactly the kind of thing that's
   easy to lose track of if only the final answer is kept).** The user asked whether class imbalance
   (570 PSS vs. 87 SICCA TLS-region *cells*) should be fixed with SMOTE-style oversampling before
   trusting item 5's p=0.006. It shouldn't — Mann-Whitney already handles unequal n correctly, and
   synthesizing fake SICCA cells would inflate apparent power without adding real information
   (pseudo-replication). First check: `bootstrap_median_diff` (same function as item 2 and Step 3's
   `medulla_FN1` case) on PSS-vs-SICCA cell scores gave **median diff +0.0122, 95% CI
   [-0.0070, 0.0286]** — straddling zero, unlike item 2's CI, suggesting the *effect size* wasn't
   pinned down even though the *direction* test was still significant.

   That CI result prompted the real question: is 570-vs-87 even the right n for this test? No — those
   are cells, not patients, and cells from the same patient aren't independent observations (the
   classic scRNA-seq **pseudoreplication** problem: a cell-level test implicitly assumes 570+87
   independent samples when there are really only 7+6 independent biological units, patients).
   Re-ran the comparison at the correct unit of replication: one median TLS-region AUCell score per
   patient (7 PSS patients: 0.111–0.154, tightly clustered; 6 SICCA patients: 0.036–0.149, much more
   spread out and overlapping the PSS range), one-sided Mann-Whitney U (exact method, appropriate at
   this n) — **U=24.0, p=0.365. Not significant.** The apparent cell-level significance (p=0.006) was
   most likely inflated by treating within-patient replicate cells as independent evidence — exactly
   the failure mode pseudoreplication predicts, now confirmed on this project's own real data rather
   than left as a theoretical concern.

   **One more legitimate check, same date: an exact patient-label (cluster) permutation test — more
   sensitive than the crude per-patient-median test, still not significant.** The per-patient-median
   test above is statistically valid but throws away information (how many cells each patient
   contributed, within-patient spread) by collapsing each patient to a single number. A better
   alternative keeps the *original* pooled-cell test statistic (median(PSS cells) − median(SICCA
   cells), using all 657 TLS-region cells) but calibrates its null distribution by permuting *patient*
   labels, not cell labels — the correct unit of independence — rather than assuming a parametric
   form. With only 13 patients (7 PSS / 6 SICCA), every possible split can be enumerated exactly:
   C(13,7) = 1,716 relabelings, no random sampling or seed needed. Observed pooled median diff:
   **+0.0122**. Across all 1,716 exact relabelings, 387 produced a diff at least that large —
   **exact one-sided p = 0.2255**. This is more sensitive than the per-patient-median test (p=0.365
   → p=0.226, confirming the extra cell-level granularity does recover some real signal) but still
   well short of significance. Saved to `data/processed/step4_pss_sicca_permutation_test.csv`
   (summary) and `step4_pss_sicca_permutation_null.csv` (the full null distribution).

   **Current honest status of item 5: PSS vs. SICCA is NOT a confirmed finding, and this is close to
   the statistical ceiling this dataset can support.** Three separate valid ways of testing it (naive
   cell-level, patient-median, exact cluster permutation) all show the same trend (PSS higher) but
   only the naive/invalid one reached significance — the two methods that correctly account for
   patient non-independence (p=0.365 and p=0.226) don't. At n=7/6 patients there just isn't enough
   independent biological replication to distinguish a real modest effect from chance; the fix is
   more SICCA patients (a different, larger dataset), not a cleverer test on this same 13 patients.
   This doesn't undermine items 1–4 — their effect sizes (p as low as 10⁻²¹²) are so large they would
   very likely survive the same patient-level correction, but that hasn't been explicitly re-verified
   either, and is worth doing before Step 7's write-up treats any Step 4 cell-level p-value as final.
   Data: `data/processed/step4_pss_vs_sicca_patient_level.csv` (per-patient scores),
   `step4_pss_vs_sicca_patient_level_test.csv` (patient-median test), `step4_pss_sicca_permutation_test.csv`
   (cluster permutation test); the superseded cell-level bootstrap CI remains in
   `step4_headline_metric.csv` for the record, now annotated as superseded.

**Full pseudoreplication audit, added 2026-09-02 — every remaining Step 4 claim checked at the
patient level, not just the one that broke.** After item 5's retraction, the user asked to confirm
no other item had the same hidden flaw, rather than assuming items 1–4 were fine because their
p-values looked extreme. Checked each:

- **Item 1/2 (headline 69.1% proportion):** not a p-value, but checked whether it was driven by a
  few cell-heavy patients rather than being a real per-patient pattern. It isn't — **7/7 PSS
  patients individually clear >50%** (range 52.4%–90.0%), and the pooled cell-weighted proportion
  (69.1%) matches the unweighted per-patient average (68.8%) almost exactly. Genuinely patient-wide,
  not skewed by whichever patient contributed the most cells. `data/processed/step4_specificity_patient_level.csv`
  companion diagnostic not needed here — see the per-patient table in this session's transcript;
  reproducible via `label_tls_region_cells` + `proportion_exceeding` grouped by `sample_id`.
- **Item 3 core (PSS vs. tonsil):** re-verified above — p=0.00303, the mathematical minimum
  possible at n=7/4.
- **Item 3's SICCA-vs-tonsil sub-part:** re-checked at the patient level too (6 SICCA patients vs.
  4 tonsil donors) — stays non-significant (patient-level MWU p=0.238, exact cluster permutation
  p=0.129), consistent with the already-honest "trends the same direction, doesn't clear
  significance" framing. No correction needed — this one was never overclaimed.
- **Item 4 (specificity, TLS-region vs. non-TLS-region):** re-tested as a **paired** per-patient
  comparison (each of the 13 patients' own TLS-region median vs. their own non-TLS-region median) —
  **13/13 patients individually show TLS-region scoring higher**, and the paired one-sided Wilcoxon
  signed-rank test gives **p=0.000122, the exact theoretical minimum possible at n=13** (2⁻¹³, since
  every single patient landed on the same side). Saved to
  `data/processed/step4_specificity_patient_level.csv`.

**Conclusion of the audit: items 1–4 all hold up at the correct unit of replication, several at the
mathematical maximum significance achievable given the patient counts. Only item 5 was a genuine
artifact.** Step 4 is now on solid footing across the board, not just "probably fine because the
p-values looked big."

**Bottom line:** the MG-derived TLS signature transfers to real Sjögren's disease tissue — Sjögren's
TLS-region cells look more like genuine lymphoid/TLS architecture (calibrated against a real healthy
tonsil reference) than like the rest of Sjögren's own tissue, and it's robust across a wide range of
threshold choices rather than resting on one hand-picked cutoff. **Revised 2026-09-02:** the
PSS-vs-SICCA specificity claim (item 5) does **not** hold at the patient level (p=0.365, n=7/6) and
should not be reported as confirmed — the direction trends the expected way but isn't statistically
distinguishable from chance at this sample size. Items 1–4 (the tonsil comparison, the headline
proportion, the sensitivity sweep, and the within-Sjögren's specificity check) are unaffected by this
correction and remain the actual finding; only the PSS/SICCA autoimmune-specificity refinement on top
of them is now retracted pending more data or a proper patient-level/mixed-effects re-analysis.
**Item 3's core comparison (PSS TLS-region vs. tonsil TLS-region) has since been explicitly
re-verified at the patient level, not just assumed robust because it wasn't the claim that broke —
exact cluster permutation gives p=0.00303, the mathematical minimum possible at n=7/4, i.e. the
strongest confirmation this dataset can produce.** The overall Step 4 finding (MG signature
transfers to real Sjögren's TLS tissue) is therefore unaffected by the item 5 retraction and is
independently confirmed at the correct unit of replication, not resting on a cell-level p-value that
might hide the same pseudoreplication problem that sank item 5. This
is Step 4's actual Phase 1 deliverable — the cross-disease finding Steps 1–3 were built toward.

### Step 5 — Stromal extraction — Status: ✅ Complete (2026-09-02)

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
  **Cross-disease corroboration worth remembering here (found 2026-08-28, during Step 3's
  literature research, easy to lose track of since it wasn't found while working on Step 5
  itself):** VCAM1 shows up independently in *two* separate papers, not just this one — Sengupta et
  al. 2018 (*PLoS One* 13(10):e0205464, see `signature/seed.py`'s docstring) found VCAM1
  differentially expressed in real **MG thymus** GC-rich vs. GC-poor tissue, the same gene Nayar et
  al. list above for **Sjögren's** salivary gland. Same gene, independently flagged in both halves of
  this project's actual disease pairing, not two unrelated cancer papers — worth prioritizing VCAM1
  specifically when this step's marker list is built, rather than treating it as just one entry among
  many in the Nayar list above.

**Tests to add later:** unit test of the subtraction/regression function against synthetic data
where the "shared" and "unique" signals are known/planted, confirming the function recovers the
right split.

**Implementation — done 2026-09-02.** Built `src/mg_thymus_map/stromal/extraction.py`
(`restrict_to_stromal_compartment`, `regress_out_shared_signature`, `rank_stromal_markers`,
`check_ground_truth_recovery` — 4 new unit tests including the exact planted-confound-vs-unique-signal
test the plan asked for, 100% coverage, 119 tests project-wide).

**Deviation from the plan's literal wording, decided before running anything (not after a problem
was found).** The plan says compare against "healthy tonsil/control." Checked the real data first:
tonsil has almost no stromal cells (139 fibroblasts / 23 endothelial / 663 epithelial out of
38,072 — a dissociation-protocol artifact of that atlas) and is a structurally different organ
(tonsil vs. salivary gland) regardless, so a Sjögren's-vs-tonsil stromal comparison would be
swamped by organ-identity differences unrelated to disease. Used GSE272409's own SICCA arm instead
(non-Sjögren's sicca: symptomatic, not autoimmune-confirmed) — same organ, same protocol, same
batch, differing only in disease status; matches Nayar et al.'s own PSS-vs-SICCA design, and Step
4's already-established framing of SICCA as "a symptomatic comparator, not a healthy control."
Stromal compartment = CellTypist's `Fibroblasts`/`Epithelial cells`/`Endothelial cells` broad labels
(no finer stromal typing available — Immune_All_High is an immune-cell classifier; see
`annotate/celltypist_annotate.py`'s docstring). 31,522 stromal cells total (14,010 PSS / 17,512
SICCA), well distributed across all 13 patients (1,077–4,717 cells each — no coverage gap).

**Method: regress out the shared-signature AUCell score (`sc.pp.regress_out`), then rank
PSS-vs-SICCA differential genes (`sc.tl.rank_genes_groups`, Wilcoxon).** Genes filtered to ≥1%
detection (13,700 of 25,364; confirmed all 22 Nayar et al. ground-truth genes below still present
before filtering) to keep `regress_out` tractable and reduce multiple-testing burden. Note: several
signature genes (CCL19, CCL21, CXCL13) are themselves canonical stromal/FDC/FRC products, so
stromal cells legitimately carry nonzero shared-signature scores (PSS stromal mean 0.031 vs. SICCA
0.027) — this is exactly the variation the regression is meant to strip out before testing for a
disease-specific effect, not a data error.

**Applied the Step 4 lesson immediately this time, before writing anything up as a finding, not
after.** The naive cell-level ranking (31,522 pooled cells, only 13 real patients) is the same
pseudoreplication setup that produced Step 4's retracted item 5. So every candidate gene below was
spot-checked at the patient level (one mean expression value per patient, exact/near-exact
Mann-Whitney) before being reported as a real result, rather than reporting cell-level p-values and
finding out later they don't hold up.

- **First filtered out standard technical-artifact gene classes** before even looking at candidates:
  mitochondrial (`MT-*`), ribosomal (`RPL*`/`RPS*`), immunoglobulin/`JCHAIN` genes (almost certainly
  ambient RNA from the tissue's huge plasma-cell population — 17,118 cells, the single largest type
  — leaking into "stromal"-labeled droplets; consistent with this project's already-documented,
  unresolved "ambient-RNA correction inconsistency" flagged for Step 7's Limitations), and sex-linked
  genes (`XIST` + 5 Y-linked genes — checked donor sex composition via XIST/Y-gene expression since
  this project has already caught one real sex-imbalance bug before, in the tonsil donor selection:
  PSS is 6F/1M, SICCA is 4F/2M — not as stark as the earlier tonsil case, but still asymmetric enough
  that sex-linked genes shouldn't be read as disease biology). 128/13,700 genes excluded this way.

- **The Nayar et al. ground-truth genes (immunofibroblast + pericyte/mural markers) did NOT hold up
  at the patient level — a second real finding, not a null result to hide.** All 22 were present and
  most looked cell-level "significant" (e.g. RGS5 p≈3×10⁻³⁰, CCL8 p≈8×10⁻³⁰, ACTA2 p≈3×10⁻²³) — but
  patient-level checks on the six spot-checked (RGS5, CCL8, ACTA2, MCAM, CCL21, and the specifically-
  prioritized **VCAM1**) all came back non-significant (p=0.27–0.63), and several **flipped
  direction** between the cell-level and patient-level analyses (CCL8, CCL21, VCAM1). This is very
  likely the same pseudoreplication artifact as item 5, not evidence the genes are biologically
  irrelevant — but it could also partly reflect a real scoping mismatch: Nayar et al.'s marker sets
  characterize specific, presumably rare fibroblast/pericyte *substates*, while `majority_voting`'s
  broad `Fibroblasts`/`Endothelial cells` labels (the only granularity CellTypist's immune-focused
  model supports) pool those substates together with the bulk of ordinary fibroblasts/endothelium —
  diluting a real substate-specific signal into a null when tested at the whole-broad-category level.
  Both explanations are plausible and not mutually exclusive; distinguishing them would need finer
  stromal subtyping than this project currently has (see Step 2/annotation's caveat), which is real,
  scoped-out follow-up work, not something to paper over.

- **A different, real, literature-corroborated finding emerged instead: an interferon-stimulated-
  gene signature, elevated in PSS relative to SICCA, that survives both the shared-architecture
  regression and patient-level correction.** After excluding technical artifacts, the top PSS-up
  genes are **IFI6, IFI44L, XAF1** (all bona fide interferon-stimulated genes) plus secretory/
  glandular genes **PIP, LYZ, AZGP1, MUC7**. Patient-level exact Mann-Whitney (7 PSS vs. 6 SICCA
  patients, one mean value per patient): **PIP p=0.0070, AZGP1 p=0.0175, LYZ p=0.0175, IFI6
  p=0.0111, IFI44L p=0.0111, XAF1 p=0.0111** (MUC7 weaker, p=0.0688) — real separation at this
  sample size, not the theoretical floor (which would be ≈0.00058 at n=7/6), so not a maximal/
  artifactual result either. **This is independently corroborated by established Sjögren's
  literature**: a type-I interferon signature in salivary gland glandular epithelium is one of the
  most consistently replicated findings in real Sjögren's syndrome research — this project's own
  independently-derived candidate list reproducing that signature, without having been told to look
  for it, is a genuine positive validation, structurally the same kind of check as Step 3's
  literature cross-check on CXCL13/CCL19/CXCL10. Full ranked list: 13,572 genes in
  `data/processed/step5_stromal_markers_clean.csv`; spot-check results in
  `step5_candidate_patient_level_check.csv`; ground-truth recovery table in
  `step5_ground_truth_recovery.csv`.

**Bottom line:** Step 5 did not confirm the specific Nayar et al. fibroblast/pericyte marker
panel — that comparison is honestly reported as unconfirmed at the correct unit of replication, for
reasons that are at least partly attributable to this project's own stromal-annotation resolution
limit, not necessarily a biological miss. What it did find, and what survived the same scrutiny that
caught item 5's problem in Step 4, is a real, literature-independent interferon-stimulated-gene
signature specific to true Sjögren's (PSS) stromal tissue relative to the sicca (SICCA) comparator —
a genuine disease-specific, architecture-independent finding, matching one of the field's most
replicated results for this exact disease. Only the top handful of candidates were spot-checked at
the patient level; the remainder of the 13,572-gene ranked list should be treated as a cell-level
candidate list requiring the same patient-level check before being cited as confirmed, not assumed
solid by extension.

**Full numbers appendix, added 2026-09-03 (every number this session actually produced, not just the
headline) — regenerable via the module's functions against `data/interim/sjogrens_stromal_scored.h5ad`,
kept here so nothing lives only in a terminal transcript.**

*Cell-level ranking, top 20 genes up in PSS (post-regression, technical artifacts already excluded;
`scores`/`pvals_adj` from `sc.tl.rank_genes_groups`, Wilcoxon; `logfoldchanges` came back `NaN` for
this run — a known scanpy quirk when a gene's mean in one group rounds to ~0 pre-log, harmless since
ranking here uses `scores`/`pvals_adj`, not fold-change):

| gene | score | pvals_adj | patient-level spot-checked? |
|---|---:|---:|---|
| PIP | 60.02 | 0 | ✅ p=0.0070 |
| LYZ | 52.13 | 0 | ✅ p=0.0175 |
| RP11-1143G9.4 | 49.71 | 0 | no |
| IFI6 | 49.54 | 0 | ✅ p=0.0111 |
| C6orf58 | 41.88 | 0 | no |
| PIGR | 37.32 | 3.61×10⁻³⁰² | no |
| SMR3B | 36.17 | 6.93×10⁻²⁸⁴ | no |
| MUC7 | 32.72 | 1.82×10⁻²³² | ✅ p=0.0688 (weaker, doesn't clear 0.05) |
| B2M | 30.37 | 3.11×10⁻²⁰⁰ | no |
| IFI44L | 29.26 | 6.85×10⁻¹⁸⁶ | ✅ p=0.0111 |
| ANKRD36C | 28.74 | 2.65×10⁻¹⁷⁹ | no |
| EHF | 28.60 | 1.43×10⁻¹⁷⁷ | no |
| CD24 | 27.18 | 1.88×10⁻¹⁶⁰ | no |
| FAM3D | 26.78 | 1.04×10⁻¹⁵⁵ | no |
| AZGP1 | 25.80 | 1.58×10⁻¹⁴⁴ | ✅ p=0.0175 |
| PLIN5 | 25.59 | 2.80×10⁻¹⁴² | no |
| FXYD3 | 25.36 | 1.03×10⁻¹³⁹ | no |
| OGFRL1 | 25.19 | 8.15×10⁻¹³⁸ | no |
| XAF1 | 25.18 | 9.94×10⁻¹³⁸ | ✅ p=0.0111 |
| ELF5 | 24.84 | 4.51×10⁻¹³⁴ | no |

Top 20 genes up in SICCA (post-regression, technical artifacts excluded) — **none of these were
patient-level spot-checked**, so treat this list as cell-level candidates only, not confirmed
findings; included for completeness since it's a real output of this run. They read as broadly
generic housekeeping/cytoskeletal/stress genes (S100 proteins, GAPDH, ferritin, actin/cytoskeleton),
not an obviously coherent disease-specific program the way the PSS-up list is — worth being
skeptical of this direction rather than building a story around it:

| gene | score | pvals_adj |
|---|---:|---:|
| S100A6 | -47.91 | 0 |
| S100A10 | -43.23 | 0 |
| GAPDH | -43.05 | 0 |
| FTH1 | -42.94 | 0 |
| FAU | -40.54 | 0 |
| CD99 | -40.37 | 0 |
| LGALS3 | -40.13 | 0 |
| CST3 | -37.45 | 2.67×10⁻³⁰⁴ |
| GSTP1 | -36.53 | 1.24×10⁻²⁸⁹ |
| VIM | -35.79 | 4.34×10⁻²⁷⁸ |
| CD81 | -33.80 | 6.35×10⁻²⁴⁸ |
| NACA | -33.15 | 1.72×10⁻²³⁸ |
| MGP | -32.65 | 1.81×10⁻²³¹ |
| ANXA1 | -29.74 | 5.42×10⁻¹⁹² |
| SRP14 | -29.54 | 2.21×10⁻¹⁸⁹ |
| IFITM2 | -28.63 | 5.80×10⁻¹⁷⁸ |
| EEF1D | -28.23 | 4.64×10⁻¹⁷³ |
| ACTB | -26.91 | 3.06×10⁻¹⁵⁷ |
| PFN1 | -26.72 | 5.18×10⁻¹⁵⁵ |
| CAV1 | -26.67 | 2.07×10⁻¹⁵⁴ |

*Complete ground-truth recovery table* — all 22 Nayar et al. genes, cell-level (`rank` is position in
the full 13,572-gene list sorted by `pvals_adj`; negative `score` = higher in SICCA, positive = higher
in PSS, matching the `group=sjogrens_syndrome, reference=non_sjogrens_sicca` orientation used
throughout):

| state | gene | rank | score | pvals_adj | patient-level (of the 6 spot-checked) |
|---|---|---:|---:|---:|---|
| immunofibroblast | IFNGR1 | 633 | -14.67 | 1.87×10⁻⁴⁷ | not spot-checked |
| pericyte_mural | RGS5 | 2227 | -11.58 | 2.98×10⁻³⁰ | ❌ p=0.6346, direction unchanged (up in SICCA) |
| pericyte_mural | CCL8 | 2352 | -11.49 | 8.27×10⁻³⁰ | ❌ p=0.6346, **direction flipped** (up in PSS at patient level) |
| pericyte_mural | ACTA2 | 5730 | -10.01 | 3.09×10⁻²³ | ❌ p=0.5274, direction unchanged (up in SICCA) |
| pericyte_mural | CCL2 | 5743 | -10.01 | 3.21×10⁻²³ | not spot-checked |
| pericyte_mural | MCAM | 7367 | -9.40 | 1.01×10⁻²⁰ | ❌ p=0.5822, direction unchanged (up in SICCA) |
| pericyte_mural | CCL21 | 7641 | -9.28 | 3.05×10⁻²⁰ | ❌ p=0.3141, **direction flipped** (up in PSS at patient level) |
| immunofibroblast | CD34 | 6941 | -9.59 | 1.71×10⁻²¹ | not spot-checked |
| immunofibroblast | TNFRSF1A | 8886 | -8.65 | 8.19×10⁻¹⁸ | not spot-checked |
| immunofibroblast | ICAM1 | 9415 | -8.29 | 1.58×10⁻¹⁶ | not spot-checked |
| immunofibroblast | SOCS1 | 10329 | -7.55 | 5.79×10⁻¹⁴ | not spot-checked |
| immunofibroblast | CXCL9 | 11157 | -6.67 | 3.06×10⁻¹¹ | not spot-checked |
| pericyte_mural | TNC | 11374 | -6.35 | 2.53×10⁻¹⁰ | not spot-checked |
| immunofibroblast | IFNGR2 | 11812 | -5.71 | 1.29×10⁻⁸ | not spot-checked |
| immunofibroblast / pericyte_mural | CCL19 | 11909 | -5.53 | 3.74×10⁻⁸ | not spot-checked (this is the shared-signature seed gene itself) |
| immunofibroblast | IRF1 | 11926 | +5.48 | 4.80×10⁻⁸ | not spot-checked |
| immunofibroblast | RELB | 12131 | -5.06 | 4.57×10⁻⁷ | not spot-checked |
| immunofibroblast | TNFSF13B | 12748 | -3.46 | 5.69×10⁻⁴ | not spot-checked (also a shared-signature seed gene) |
| immunofibroblast | NFKB2 | 12989 | -2.62 | 9.21×10⁻³ | not spot-checked |
| immunofibroblast / pericyte_mural | VCAM1 | 13415 | -0.77 | 0.445 | ❌ p=0.2669, **direction flipped** (up in PSS at patient level); the gene [[mg-thymus-map-step5-stromal-extraction]] flagged as worth prioritizing — did not pan out |
| immunofibroblast | CD82 | 13506 | +0.32 | 0.753 | not spot-checked |

*All 6 ground-truth genes that were spot-checked at the patient level* (7 PSS patients / 6 SICCA
patients, one mean expression value per patient, exact/near-exact one-sided Mann-Whitney in the
cell-level-favored direction):

| gene | PSS patient-mean median | SICCA patient-mean median | direction at patient level | patient MWU p |
|---|---:|---:|---|---:|
| RGS5 | 0.1666 | 0.1771 | up in SICCA (unchanged) | 0.6346 |
| CCL8 | 0.0451 | 0.0433 | up in PSS (flipped) | 0.6346 |
| ACTA2 | 0.4540 | 0.4755 | up in SICCA (unchanged) | 0.5274 |
| MCAM | 0.2552 | 0.2613 | up in SICCA (unchanged) | 0.5822 |
| CCL21 | 0.0784 | 0.0605 | up in PSS (flipped) | 0.3141 |
| VCAM1 | 0.2776 | 0.2523 | up in PSS (flipped) | 0.2669 |

*All 7 candidate genes spot-checked from the PSS-up list* (same design as above):

| gene | PSS patient-mean median | SICCA patient-mean median | patient MWU p (one-sided, up-in-PSS) |
|---|---:|---:|---:|
| PIP | 3.5510 | 2.1211 | 0.0070 |
| LYZ | 3.5933 | 2.1210 | 0.0175 |
| IFI6 | 0.4763 | 0.2396 | 0.0111 |
| IFI44L | 0.2489 | 0.1630 | 0.0111 |
| XAF1 | 0.2164 | 0.1598 | 0.0111 |
| AZGP1 | 1.4167 | 0.8887 | 0.0175 |
| MUC7 | 3.3553 | 2.8731 | 0.0688 |

*Donor sex composition check* (inferred from XIST vs. mean of 5 Y-linked genes — RPS4Y1, DDX3Y, UTY,
KDM5D, EIF1AY — per patient, since no donor demographics were loaded for GSE272409 the way the
tonsil loader captures them): **PSS 6F/1M** (GSM8401512_PSS_B is the one male), **SICCA 4F/2M**
(GSM8401521_SICCA_F and GSM8401522_SICCA_G). Both groups are female-predominant, matching real
Sjögren's/sicca epidemiology, so this isn't the same severity of confound as the earlier all-male
tonsil bug — but the imbalance is why `XIST` and the 5 Y-linked genes were excluded from the
candidate list rather than left in.

*Technical-artifact exclusion tally:* 128 of 13,700 genes excluded before ranking — mitochondrial
(`MT-*`), ribosomal (`RPL*`/`RPS*`), immunoglobulin (`IGK*`/`IGH*`/`IGL*`/`JCHAIN`), and the 6
sex-linked genes above.

*Files:* `data/processed/step5_stromal_markers_naive.csv` (no regression), `_corrected.csv`
(post-regression, all 13,700 genes, artifacts still included), `_clean.csv` (artifacts excluded,
13,572 genes — source for both tables above), `step5_ground_truth_recovery.csv` (the 22-gene table
above), `step5_candidate_patient_level_check.csv` (the two patient-level spot-check tables above);
scored interim data at `data/interim/sjogrens_stromal_scored.h5ad`. All gitignored, regenerable via
`src/mg_thymus_map/stromal/extraction.py`'s functions.

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
  deadline; **the tonsil control's sex balance is donor-level only, not cell-level** — 2 male/2 female
  donors were selected after fixing a real all-male selection bug, but the two male donors
  individually contribute more cells, so the final data is 59.9% male / 40.1% female by cell count,
  not 50/50 — a real, substantial improvement over the original bug, but not perfect parity, worth
  stating plainly rather than letting "2/2 donors" imply otherwise. Judges at a fair (per question 4's
  resolution) reward intellectual honesty about a project's boundaries — this section is a rigor
  signal, not a hedge to bury.

## 4a. Cutoffs used across the pipeline — referenced vs. adaptive, and why (added 2026-08-31)

Every numeric cutoff used in Phase 1 so far, in one place, with whether it's a fixed value taken
from the field ("referenced") or computed from this project's own data ("adaptive"/ad hoc), and why.
Added after a direct question about whether referenced cutoffs could replace the adaptive ones
throughout — the answer differs by cutoff, for a structural reason explained below the table.

| # | Cutoff | Value(s) | Referenced or adaptive | Why |
|---|---|---|---|---|
| 1 | `min_genes_per_cell` | 200 | **Referenced** | [Seurat PBMC3k Guided Clustering Tutorial](https://satijalab.org/seurat/articles/pbmc3k_tutorial.html): `CreateSeuratObject(min.features = 200)` — confirmed by fetching the page directly |
| 2 | `min_cells_per_gene` | 3 | **Referenced** | Same tutorial: `CreateSeuratObject(min.cells = 3)` — confirmed |
| 3 | Mitochondrial-% cutoff | MG 9.5%, Sjögren's 38.1%, tonsil 25.3% (median + 5×MAD, capped at 50%) | **Adaptive** | Generic referenced ranges (5% immune-only, 10–15% Sjögren's, 8–10% tonsil) were tested directly and rejected — a fixed 5% cutoff strips 39% of MG plasma cells and 50% of cycling cells, exactly the populations this project studies (Step 2) |
| 4 | Doublet-score threshold | MG 0.2990, Sjögren's 0.2769, tonsil 0.3300 (KDE valley on pooled simulated scores) | **Adaptive** | Scrublet's own per-sample auto-threshold — the closest thing to a "reference" — failed outright on 6 of 12 MG samples (threshold fit above every real cell's score); the score scale isn't portable across datasets anyway |
| 5 | Doublet KDE peak-prominence guard | `min_second_peak_prominence=0.05` | **Ad hoc, implementation-specific** | Not a biological cutoff at all — a numerical safeguard against KDE noise, set from observing a real noise peak >400× smaller than the true peak; no field reference exists for this because it's an artifact of this specific detection method |
| 6 | Normalization `target_sum` | 1e4 | **Referenced** | [scanpy `normalize_total` docs](https://scanpy.readthedocs.io/en/stable/generated/scanpy.pp.normalize_total.html); load-bearing citation is [CellTypist's own tutorial](https://celltypist.readthedocs.io/en/latest/notebook/celltypist_tutorial.html), quoted directly: *"pre-processed (and required) as log1p normalised expression to 10,000 counts per cell"* — confirmed by fetching the page |
| 7 | CellTypist confidence split | `conf_score < 0.5` (diagnostic only, not a filter) | **Not actually referenced — corrected 2026-08-31** | Fetched CellTypist's own tutorial directly to check this: it displays `conf_score` in its output (values ~0.13–0.99) but documents **no threshold** for "low confidence." The 0.5 cutoff is a generic borrowed convention (probability midpoint), not a CellTypist-specified value. Doesn't affect correctness here since it's diagnostic only — no cells are ever dropped on it — but the "Referenced" label in an earlier version of this table was wrong and is fixed here. |
| 8 | Cell-state (GC B cell / Tfh) score threshold | mean + 2×std within broad CellTypist category (`n_std=2.0`) | **Adaptive** | `scanpy.score_genes` output scale depends on the specific marker panel and this dataset's background expression — no portable literature value exists to reference |
| 9 | Communication `specificity_rank` | ≤ 0.05 | **Adaptive / ad hoc** | `liana`'s own tutorial reference (0.01) was tested directly and rejected — it excludes CD40LG-CD40 due to a tie floor in this data's rank distribution (see merge.py) |
| 10 | Communication `magnitude_rank` | ≤ 0.6 | **Adaptive / ad hoc**, picked by eyeballing a gap in 15 sorted values | No referenced value exists for this statistic at all; explicitly flagged as not validated-optimal (see merge.py's threshold-honesty note) |
| 11 | `liana` `expr_prop` (minimum per-group expressed fraction before a pair is even scored) | 0.1 | **Referenced** | [`liana.method.rank_aggregate` docs](https://liana-py.readthedocs.io/en/latest/generated/liana.method.rank_aggregate.__call__.html): default `0.1` — confirmed by fetching the page. Not set by this project's code; inherited as-is. Explains why CXCL13/ICOS/IL21 never entered scoring at all (Step 3) |
| 12 | `liana` `n_perms` | 1000 | **Referenced** | Same docs page: default `1000` — confirmed. Pinned explicitly rather than left implicit, to avoid silent reproducibility drift if `liana` changes its own default in a future release |
| 13 | AUCell threshold (Step 4) | **Not yet chosen** | N/A — open | Flagged in Risks & Mitigations as "arbitrary and drives the headline result"; the plan's intent is to benchmark against the tonsil control's own score distribution rather than pick a fixed number, for the same structural reason as #8 and #9/10 below |

*(RNG seeds — `random_seed: 0` in the pipeline config, `seed=1337` pinned in `communication.py` to match `liana`'s own default — are reproducibility settings, not filtering cutoffs, so they're not in this table.)*

**Honest caveat on rows 1–2, checked while sourcing the citation above, not assumed:** the Seurat
PBMC3k tutorial's `200`/`3` values are that specific tutorial's chosen defaults for one specific 2,700-cell
PBMC dataset that became a widely-copied convention across the field — a real, traceable reference (cited
above), but a culturally-adopted one, not a value independently derived or validated for *this* project's
three tissues. Same category of caveat as the mito-% conventions in row 3, which were tested against real
per-cell-type retention and rejected; rows 1–2 have not been given that same empirical check against MG/
Sjögren's/tonsil data specifically, and are flagged here as worth that same scrutiny if they ever become
consequential to a downstream claim.

**Why the split isn't arbitrary.** Rows 1, 2, 6, 11, 12 are all technical/methodological conventions
that don't depend on this project's specific biology — the same constant is correct regardless of
which tissue or disease is being analyzed, so a referenced value is exactly right and is what's used.
Rows 3, 4, 8, 9, 10, and the still-open row 13 all share the opposite property: each is a statistic
computed from something intrinsic to *this* dataset (this dataset's own per-cell-type mito
distribution, this dataset's own simulated doublets, this dataset's own marker-score background,
this dataset's own ligand/receptor detection rates) that doesn't transfer from a number published
against a different dataset's tissue, depth, protocol, or cell-type mix. Row 3 is the one case where
a referenced alternative was directly tested against real per-cell-type retention and shown to fail;
row 9 is the one case where the closest thing to a field reference (`liana`'s own tutorial value) was
tested and also failed, for a documented, data-specific reason (the tie floor). Rows 4, 8, and 10 have
no candidate referenced value to test in the first place, because the underlying statistic's scale
isn't standardized across studies.

## 5. Deliverables checklist

- [~] `data/DATASETS.md` (or config) — dataset manifest with provenance. **Done via config, not a
      separate file**: `configs/phase1_mg_sjogrens.yaml`'s `datasets:` block.
- [x] `src/mg_thymus_map/data/` — download + load + validate. **Done** (loaders only; the
      download itself was done manually by the user, not scripted — see Step 1's plan-vs-actual).
- [x] `src/mg_thymus_map/qc/` — QC + normalization + annotation. **Done**, with the doublet-detection
      gap noted in Step 2. Also includes `annotate/` (CellTypist wrapper, a separate subpackage) and
      batch-effect/integration code (`qc/integration.py`) that wasn't originally itemized here.
- [x] `src/mg_thymus_map/signature/` — MG TLS signature derivation. **Done (Step 3): seed signature,
      marker panels, cell-state calling, cell–cell communication analysis, the signature-merge/output
      artifact (real 25-gene signature), and spatial validation (`spatial_validation.py`, real
      AUCell-vs-niche enrichment result) all complete — 2026-09-01.**
- [x] `src/mg_thymus_map/scoring/` — AUCell projection + statistics. **Done (Step 4): real,
      well-powered cross-disease result, fully audited at the patient level — Sjögren's TLS-region
      cells score significantly higher on the MG signature than tonsil TLS-region cells (exact
      cluster permutation p=0.003, the mathematical minimum at this sample size), with real
      specificity (p=0.000122, also the exact minimum at n=13). The PSS>SICCA sub-claim did NOT
      survive patient-level correction and is retracted — 2026-09-02.**
- [x] `src/mg_thymus_map/stromal/` — subtraction / stromal extraction. **Done (Step 5): the specific
      Nayar et al. ground-truth marker panel did not hold up at the patient level (likely a stromal-
      subtyping resolution limit, not necessarily a miss), but a real, literature-corroborated
      interferon-stimulated-gene signature (IFI6/IFI44L/XAF1 + glandular genes PIP/LYZ/AZGP1)
      specific to true Sjögren's stromal tissue was found and confirmed patient-level — 2026-09-02.**
- [ ] `src/mg_thymus_map/pharma/` — DGIdb/ChEMBL clients + ranking. **Not started (Step 6).**
- [x] `tests/unit/`, `tests/integration/`, `tests/fixtures/` covering all of the above. **Done for
      Steps 1–5** (119 tests total, `make test` green, 100% coverage on covered modules including
      `mg_spatial.py`, `spatial_validation.py`, `cross_disease_projection.py`, and
      `stromal/extraction.py`); Steps 6–7 have none yet.
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

   **Follow-up, 2026-08-28: the literature prior itself was under-sourced, and has been fixed.**
   The initial implementation grounded the 12-chemokine core in one paper (Messina et al. 2012) and
   "verified" it only by finding other papers that *cited/reused* that same list — not independent
   confirmation, since a single group's original panel can miss genes other researchers separately
   found relevant. Flagged directly by the user: "please use much more research rather than just
   one paper to confirm this literature prior... one research alone may not have all the previously
   discovered genes." Did a real multi-paper search rather than relying on recalled knowledge; full
   trail and per-gene support level now lives in `signature/seed.py`'s module docstring. Findings:
   - **Uneven support within the original 12**, made explicit per-gene rather than smoothed over —
     and corrected twice already (see below), which is itself worth being upfront about rather than
     presenting the current state as if it were right the first time. Actual per-gene picture as of
     2026-08-28's second research pass (prompted by the user asking to search again in case anything
     was missed):
     - **CXCL13**: three independent sources — Hou et al. 2023's statistical re-derivation (pooled
       three published gene sets, Cox regression on TCGA); Nayar et al. 2025's Sjögren's data; and
       Yoshimitsu et al. 2025 (*Cancer Science* 116(8):2075–2085), which experimentally
       co-administered CXCL13+CCL21 in mice and showed it induces TLS formation and improves
       anti-PD-L1 efficacy — functional evidence, not just correlation.
     - **CCL19, CXCL9, CXCL10**: two independent sources each (Hou + Nayar).
     - **CXCL11**: one independent source (Hou only).
     - **CCL21**: two independent sources, corrected once already — an earlier pass wrongly lumped
       it in with Hou et al.'s re-derived panel, which it isn't part of (checked directly). Its real
       sources: Yoshimitsu et al.'s functional experiment (above), and — the strongest evidence
       found in this whole search — Sengupta et al. 2018 (*PLoS One* 13(10):e0205464), which found
       CCL21 differentially expressed between GC-rich and GC-poor **MG thymus tissue itself**, not
       a borrowed disease. The same paper independently found RGS13 and FDCSP differentially
       expressed too — both already in this project's marker panels (`GC_B_CELL_MARKERS`,
       `FDC_MARKERS`), now with an added citation there.
     - **CCL5**: two sources of different confidence — Nakamura et al. 2022 (*Front Oncol*
       12:811586, Merkel cell carcinoma, full text verified: CCL5 significantly elevated in
       TLS-positive samples, screened across 27 chemokine/receptor genes) and Xu et al. 2022
       (*Cancer Immunol Immunother*, ccRCC) reportedly finding CCL4/CCL5/CCL8/CCL19/CXCL13
       prognostically significant — this second source could only be confirmed via search-engine
       summaries (paywalled, full text never accessed), so it's flagged as a lead, not verified on
       the same footing as the other citations here.
     - **CCL4, CCL8**: one source each, the same paywalled Xu et al. lead above — same caveat.
     - **CCL2, CCL3, CCL18**: zero independent sources found across the full search (Hou,
       Dieu-Nosjean 2014, Nayar, Yoshimitsu, Sengupta, Nakamura, Xu). They stay in the seed list
       only because the 12-chemokine panel as a whole remains the field's most widely *reused* TLS
       score — that's reuse of one panel, not independent re-discovery. This "zero found" is bounded
       by the searches actually run, not a claim that no such evidence exists anywhere.
   - **Added TNFSF13B (BAFF)** to the seed signature. Not part of the original 12-chemokine panel,
     but independently and repeatedly implicated in *autoimmune* (not cancer) ectopic lymphoid
     structure formation specifically — exactly this project's context — by Bombardieri et al. 2017
     and a dedicated Nature Reviews Rheumatology piece on BAFF's role in tertiary lymphoid
     neogenesis in Sjögren's syndrome and rheumatoid arthritis. Confirmed present in MG's real gene
     panel (25,150 genes) before adding.
   - **Found a major asset for later steps**: Nayar et al. 2025 ("Molecular and spatial analysis of
     tertiary lymphoid structures in Sjogren's syndrome", *Nature Communications*,
     doi:10.1038/s41467-024-54686-0) is real single-cell/spatial ground truth for TLS in the actual
     comparison disease — not another cancer analogy. It independently corroborates CD40 as
     TLS-relevant (elevated in Sjögren's TLS-GC vs. tonsil-GC), consistent with this project's own
     liana-py finding of CD40LG-CD40 as a top hit in real MG data (see Step 3 below) — a genuine
     literature/data cross-check, found after the fact rather than fit to it. It also reports a
     TLS-associated fibroblast signature (CD34, CCL19, TNFSF13B, ICAM1, VCAM1, CD82, CXCL9) and an
     inflammatory-state signature (ICOS, IFNG, TNF, CASP8) distinguishing Sjögren's TLS from tonsil
     GC — these describe a stromal cell type and an activation state respectively, not additional
     TLS *identity* chemokines, so they were **not** folded into the seed signature. Recorded here
     as the primary reference to build from when Step 5 (Sjögren's-side stromal extraction) needs
     its own dedicated marker panel, with its own review — not bolted on hastily now.
   - `tests/unit/test_signature_seed.py` updated (13 genes now, was 12); full suite still green
     (76 tests).
   - **Papers found but deliberately not used, for the record:** Cabrita et al. 2020 (*Nature*,
     melanoma TLS) — searched repeatedly but never got past a paywall/cookie-wall to its actual gene
     list; genuinely unresolved, not ruled out. Dieu-Nosjean et al. 2014 (*Trends in Immunology*) —
     the gene list attributed to it in search results was identical to Messina/Coppola's, so
     independence vs. reuse couldn't be established. Lin et al. 2020 — never looked at directly (one
     of the three papers Hou et al. pooled). Vanhersecke et al. 2021 (*Nature Cancer*, mature-TLS/
     DC-LAMP) — no new gene-level detail found beyond what Hou's panel already covers. **Helmink et
     al. 2020** (*Nature*, melanoma) — a real independent finding (a TLS-B-cell gene set: CD19, CR2,
     CD79A, MS4A1, CD40, CD22, CD23, CD27, CD72) that was set aside without being formally recorded
     until now, because these are general B-cell/BCR-activation genes rather than GC-specific ones
     and didn't cleanly fit `GC_B_CELL_MARKERS` without blurring that panel's specificity — CD40 and
     CD22 are worth another look given they also independently surfaced in Nayar et al.'s Sjögren's
     data, but that's a deliberate future re-check, not done here.

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
| 3 | Sep 9 – Sep 15 | **Actually done early, 2026-08-28 to 2026-09-01.** Step 3 in full: TLS signature seeded with the 12-chemokine prior and refined on MG data, with provenance tagging; cell–cell communication analysis (`liana-py`/`squidpy`); **full spatial validation** — reprocess the MG Cell Reports Visium data with our own AUCell scoring against the derived signature, not the gene-overlap fallback. Unit tests for signature determinism and schema. | ~25–35 | ~4–5.5 hrs |
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
