# Phase 1 Plan — MG (Thymus) vs. Sjögren's Syndrome Pilot

Status: **DRAFT — planning only, no code written yet.**
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

**This is a proposed rationale, not a confirmed one — see Open Questions below**, particularly if
you already have specific datasets in mind that point toward a different disease being the easier
first pairing.

## 2. Objectives

1. Prove the full 5-stage pipeline (assembly → TLS isolation → projection → stromal extraction →
   pharmacogenomic mapping) runs end-to-end on real public data for one disease pair.
2. Produce a first-pass answer to: *does Sjögren's syndrome build its TLS using transcriptional
   architecture shared with MG's thymic TLS, and if so, how much (quantified), and what's left
   over (stromal/organ-specific)?*
3. Establish the reusable shape (interfaces, config schema, test fixtures) that Phases 2–4 will
   replay per-disease with minimal code changes.

## 3. Definition of done for Phase 1

- [ ] MG thymus, Sjögren's salivary gland, and healthy tonsil scRNA-seq datasets are downloaded,
      documented (source, accession, license), and load cleanly into `anndata` objects.
- [ ] QC + normalization + cell-type annotation produces a labeled MG dataset and a labeled
      Sjögren's dataset with a shared, consistent cell-type vocabulary.
- [ ] An MG-derived TLS gene/cell-state signature exists, with a written rationale for how it was
      derived.
- [ ] That signature has been scored (AUCell) against the Sjögren's dataset, with a quantified
      "proportion shared" metric, benchmarked against the healthy tonsil control.
- [ ] Stromal populations in Sjögren's have been isolated after subtracting the shared signal, with a
      ranked marker-gene list.
- [ ] Those markers have been queried against DGIdb/ChEMBL, producing a ranked candidate
      target/drug list with provenance (which database, which gene, which compound).
- [ ] Every stage above has unit tests (synthetic fixtures) and the full pipeline has one integration
      test (tiny fixture data) — all passing via `make test`.
- [ ] `README.md` documents how to set up the environment and run the Phase 1 pipeline
      end-to-end via `make phase1` (or equivalent).
- [ ] A short written summary of Phase 1 findings exists (even if preliminary/negative) — a
      pipeline that runs but finds no shared signal is still a valid, useful Phase 1 result.

## 4. Step-by-step plan

### Step 1 — Data assembly

**Goal:** get MG (thymus), Sjögren's (salivary gland), and healthy tonsil scRNA-seq into local,
documented, loadable form.

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

**Tests to add later:** a loader unit test against a tiny synthetic `.h5ad` fixture (shape/column
checks) for each of the three loader code paths (two GEO, one Zenodo/Bioconductor-derived); a
manifest schema test (every entry in `DATASETS.md`/config has the required fields).

### Step 2 — QC, normalization, cell-type annotation

**Goal:** each dataset becomes a clean, annotated `AnnData` on a shared cell-type vocabulary.

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

**Tests to add later:** unit tests for QC filter functions on synthetic data with known
outliers/doublets planted in; a check that annotation output only ever uses labels from the
approved shared vocabulary (fails loudly on an unexpected label).

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
  "shared architecture" finding is anchored to prior publications, not purely novel. Judges at a
  fair (per question 4's resolution) reward intellectual honesty about a project's boundaries — this
  section is a rigor signal, not a hedge to bury.

## 5. Deliverables checklist

- `data/DATASETS.md` (or config) — dataset manifest with provenance
- `src/mg_thymus_map/data/` — download + load + validate
- `src/mg_thymus_map/qc/` — QC + normalization + annotation
- `src/mg_thymus_map/signature/` — MG TLS signature derivation
- `src/mg_thymus_map/scoring/` — AUCell projection + statistics
- `src/mg_thymus_map/stromal/` — subtraction / stromal extraction
- `src/mg_thymus_map/pharma/` — DGIdb/ChEMBL clients + ranking
- `tests/unit/`, `tests/integration/`, `tests/fixtures/` covering all of the above
- `README.md` — updated with Phase 1 setup/run instructions
- `Makefile` — `make setup`, `make test`, `make phase1`
- A short Phase 1 findings write-up, including the spatial/ground-truth cross-check (Steps 3, 5, 7 —
  reprocessed Visium data for MG's thymoma/hyperplasia paper; reported-findings comparison
  only for Sjögren's GSE272409, since it has no reprocessable spatial deposit) and a required
  Limitations section (Step 7)

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
     patients (10 female, 2 male; 24 samples total) — the thymic-hyperplasia-associated subtype
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
     >556,000 cells, 121 annotated cell types/states, healthy tonsil — purpose-built as a reference
     atlas rather than a side comparison in an unrelated study. **Not a single GEO accession**: raw
     data is on ArrayExpress (**E-MTAB-13687**), processed expression matrices are on Zenodo, and
     it's also distributed as the `HCATonsilData` Bioconductor package — note this for Step 1's
     download/load script, since it needs different handling than a plain GEO `GSE` pull. Bonus:
     it includes matched spatial transcriptomics, which extends the spatial-validation pattern
     already used for MG (Step 3) and Sjögren's (Step 5) — tonsil's own real germinal centers are a
     natural extra positive control for Step 4's cross-disease projection.
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

| Week | Dates | Work | Est. hours | Avg/day (~6 active days) |
|---|---|---|---|---|
| 1 | Aug 26 – Sep 1 | Phase 0: proper repo scaffold (`uv` + `pyproject.toml`, `src/mg_thymus_map/` layout, `configs/`, `tests/{unit,integration,fixtures}`), `pytest` + coverage wired up, `Makefile` (`setup`/`test`/`phase1`). Kick off downloads for all 3 datasets (GSE233180, GSE272409, Human Tonsil Atlas) — start early since the tonsil atlas's Zenodo/Bioconductor distribution needs its own loader path. | ~15–20 | ~2.5–3.5 hrs |
| 2 | Sep 2 – Sep 8 | Steps 1–2 in full: load, QC, normalize, and annotate all 3 datasets onto the shared cell-type vocabulary; batch/dataset-effect check and integration (Harmony/scVI) if needed. Unit tests for QC filters and vocabulary validation written alongside, not bolted on after. This is historically where real-data pipelines lose the most time to surprises (mismatched gene symbols, unexpected metadata gaps) — the extra week of buffer here is deliberate. | ~30–40 | ~5–6.5 hrs |
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
