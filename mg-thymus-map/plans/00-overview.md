# MG Thymus Map — Overall Project Plan

Status: **DRAFT — planning only, no code written yet.**
Source material: [`docs/project-introduction.txt`](../docs/project-introduction.txt) (extracted from the original PDF), by Arya Kakade — *"A Thymus-Anchored Map of Tertiary Lymphoid Structures Across Multiple Autoimmune Diseases."*

---

## 1. Project summary (from the source document)

Autoimmune diseases build ectopic **Tertiary Lymphoid Structures (TLS)** — organized immune
aggregates that resemble lymph nodes but form in the wrong place (thymus, salivary gland,
synovium, thyroid, etc.). Standard treatments are broad immunosuppressants that damage healthy
tissue along with diseased tissue.

**Scientific aims (verbatim from the intro doc):**
1. Determine whether peripheral autoimmune diseases — SLE, RA, Sjögren's Syndrome, MG, and
   Graves' Disease — build TLS using the *same* central immune architecture that Myasthenia
   Gravis (MG) uses in the thymus.
2. Use that shared architecture as a **subtractive baseline** to isolate organ-specific therapeutic
   targets, per disease.

**Stated methods (5 steps), which this plan turns into a pipeline:**
1. **Data Assembly** — scRNA-seq for MG (thymus), RA, SLE, Sjögren's, Graves', + healthy tonsil
   (control).
2. **TLS Isolation** — find the cell interactions that drive MG; extract the shared immune structure.
3. **Cross-Disease Projection** — score that MG-derived structure against the other disease
   datasets via **AUCell**.
4. **Stromal Extraction** — subtract the shared immune signal to expose non-immune, disease-local
   stromal targets (e.g. thymic epithelial cells, synovial fibroblasts).
5. **Pharmacogenomic Mapping** — query those stromal targets against **DGIdb** / **ChEMBL** for
   existing drugs.

## 2. Engineering translation

Each scientific method step becomes a Python pipeline stage that takes one AnnData object (or a
collection of them) in and produces a well-defined, tested artifact out. The whole project is a
sequence of these stages run once per disease, plus a final cross-disease comparison layer.

```
raw scRNA-seq  →  QC/preprocess  →  cell-type annotation  →  [MG only] TLS signature
       │                                                              │
       └──────────────────────────────┬───────────────────────────────┘
                                       ▼
                     AUCell projection of MG signature onto disease X
                                       ▼
                     stromal extraction (subtract shared signal)
                                       ▼
                     stromal marker genes for disease X
                                       ▼
                     DGIdb / ChEMBL lookup → candidate drug targets
```

The pipeline is built and fully validated on **one** disease pair first (MG vs. Sjögren's — see
Phase 1), then replayed per-disease for the rest. This keeps the expensive parts (getting real
public scRNA-seq data to load and QC cleanly, getting AUCell scoring right, getting the subtraction
logic right) concentrated in one phase instead of six.

## 3. Guiding principles

- **Python-only.** `scanpy`/`anndata` as the core data model (the de-facto standard for scRNA-seq
  in Python, interoperable with the wider ecosystem: `scvi-tools`, `decoupler`, `squidpy`).
- **Small, pure, testable functions.** Biological pipelines are notoriously hard to debug after the
  fact; every stage should be a function that takes typed inputs and returns typed outputs, testable
  on tiny synthetic AnnData fixtures without downloading real data.
- **Config-driven, not notebook-driven.** Notebooks are fine for exploration but the pipeline itself
  runs from versioned YAML configs + CLI/Makefile targets, so every phase is reproducible.
- **Raw data never lives in git.** Datasets are tens of MB–GBs; only download scripts, checksums,
  and small derived/test fixtures are committed.
- **Fail loudly on bad biology inputs.** Schema/sanity checks (gene symbols present, expected
  `obs` columns, no all-zero cells, etc.) at every stage boundary, not just at the end.
- **One phase fully green (tests + README + a runnable Makefile target) before the next phase
  starts.**

## 4. Phase roadmap

| Phase | Focus | Depends on | Key deliverable |
|---|---|---|---|
| 0 | Foundations & infra | — | Repo scaffold, env, data conventions, test harness (no biology yet) |
| 1 | **MG vs. Sjögren's pilot** | 0 | End-to-end pipeline proven on one disease pair — see [`01-phase1-sjogrens-pilot.md`](01-phase1-sjogrens-pilot.md) |
| 2 | Extend → Rheumatoid Arthritis (synovium) | 1 | Pipeline replayed on RA; pipeline made disease-agnostic (config, not code, changes per disease) |
| 3 | Extend → SLE | 1–2 | Same, for SLE |
| 4 | Extend → Graves' Disease (thyroid) | 1–2 | Same, for Graves' |
| 5 | Full 5-disease integration + healthy tonsil control | 1–4 | Cross-disease comparison of "how much of each disease's TLS is MG-shared vs. private"; refined shared-architecture signature using all diseases jointly, not just MG→X pairwise. **Also the guaranteed fallback for MG-side stromal extraction (thymic epithelial cells)** if Phase 1 doesn't get to it as a stretch goal — required by the source doc's Aim 2, not optional; see [`01-phase1-sjogrens-pilot.md`](01-phase1-sjogrens-pilot.md) §7, question 7 |
| 6 | Cross-disease pharmacogenomic mapping | 4–5 | Ranked, deduplicated candidate drug/target list across diseases, with provenance |
| 7 | Reproducibility & dissemination | all | Frozen environment, final report/notebook, polished README, (optionally) a poster/paper draft — **not** PyPI packaging or a general-purpose release; see §9, question 4 |

Phases 2–6 are placeholders at this level of detail — each will get its own `0N-phase-*.md` once
Phase 1 has validated the pipeline shape and surfaced anything that needs to change in the general
design. Re-planning them in detail now would mean re-planning them again after Phase 1 anyway.

## 5. Proposed repository layout

*(For reference only — nothing below is created yet.)*

```
mg-thymus-map/
├── README.md                     # setup + how to run + phase status (kept current every phase)
├── Makefile                      # make setup / make test / make phase1 / ...
├── pyproject.toml                # deps + tool config (pytest, ruff, etc.)
├── configs/
│   └── phase1_mg_sjogrens.yaml   # dataset paths, thresholds, signature params
├── data/                         # gitignored: raw/ interim/ processed/
├── src/mg_thymus_map/
│   ├── data/                     # download + load + validate datasets
│   ├── qc/                       # QC, normalization, batch correction
│   ├── annotate/                 # cell-type annotation
│   ├── signature/                # TLS signature derivation (from MG)
│   ├── scoring/                  # AUCell cross-disease projection
│   ├── stromal/                  # subtraction / stromal extraction
│   ├── pharma/                   # DGIdb / ChEMBL clients + mapping
│   └── pipeline.py               # orchestrates stages per config
├── tests/
│   ├── unit/                     # pure-function tests, synthetic fixtures
│   ├── integration/               # small end-to-end run on tiny fixture data
│   └── fixtures/
└── notebooks/                    # exploratory only, not authoritative
```

Planning docs (this file and phase plans) and the original source material live outside this repo,
in `claude-plans/mg-thymus-map/plans/` and `claude-plans/mg-thymus-map/docs/` respectively —
not inside the code repo itself.

## 6. Tech stack candidates (pending decisions — see Open Questions)

| Concern | Candidate | Notes |
|---|---|---|
| Core data model | `scanpy` + `anndata` | Standard for scRNA-seq in Python |
| Env/deps | `conda`/`mamba` via `environment.yml`, or `uv`/`poetry` via `pyproject.toml` | Bioinformatics stack (leiden, scanpy, scvi-tools) often easier via conda-forge/bioconda |
| Batch correction / integration | `scanpy.external` (Harmony) or `scvi-tools` | Needed once we combine MG + disease + control in one embedding |
| Cell-type annotation | `celltypist` (reference-based) or marker-gene scoring | Need a consistent ontology across organs |
| Gene-set / AUCell scoring | `decoupler-py` (has an AUCell-equivalent, actively maintained) vs. `pyscenic.aucell` | `decoupler` is lighter-weight and doesn't require the full SCENIC regulon pipeline |
| Cell–cell communication (for "TLS isolation") | `squidpy` / `liana-py` | Python equivalents of CellPhoneDB/CellChat |
| Drug/target lookup | `requests` against DGIdb GraphQL API and ChEMBL REST API | Both have public, unauthenticated APIs |
| Testing | `pytest` + `pytest-cov` | Standard |
| Data validation | `pandera` (for `obs`/`var` DataFrames) or hand-written schema checks | |

## 7. Testing strategy (applies from Phase 0 onward)

- **Unit tests**: every pure transformation (QC filter thresholds, signature scoring math, the
  subtraction step, drug-mapping response parsing) gets tests against small synthetic AnnData
  objects built in `tests/fixtures/` — no network, no real data, runs in seconds.
- **Integration tests**: one tiny end-to-end run (a handful of cells, a handful of genes) through
  the whole Phase 1 pipeline, checked into `tests/fixtures/`, to catch stage-boundary breakage.
- **Contract/schema tests**: each stage's input/output is validated against an explicit schema
  (expected `obs` columns, no unexpected NaNs, expected value ranges for scores) so a bad upstream
  change fails at the boundary, not three stages later.
- **External API tests**: DGIdb/ChEMBL clients are tested against recorded fixtures (e.g.
  `vcr.py`/cassettes) rather than hitting the live network in CI.
- `make test` runs the whole suite; it is expected to pass before any phase is considered done.

## 8. Documentation plan

- `README.md` at the repo root is the single entry point: what the project is, how to set up the
  environment, how to run each phase (`make phase1`, etc.), and current phase status. It gets a
  short update at the end of every phase — not left to drift.
- Each phase's detailed plan lives in `plans/0N-*.md` (this file's siblings, in
  `claude-plans/mg-thymus-map/`, outside the code repo) and is updated in place if the phase's
  actual approach diverges from the plan (with a short "deviation from plan" note, not a silent
  rewrite).

## 9. Open questions

Project-level questions only — these affect the project as a whole (infra, process, scope), not one
disease/phase's biology. Phase-specific open questions live in each phase's own plan doc (e.g.
Phase 1's are in [`01-phase1-sjogrens-pilot.md`](01-phase1-sjogrens-pilot.md), section 7).

1. **Compute environment** — Status: **Resolved**
   Local laptop, university/lab HPC, or cloud (AWS/GCP)? scRNA-seq datasets can be several GB
   raw; integration/embedding steps (Harmony/scVI) are the most memory-hungry part. This
   affects whether we design for out-of-core processing.
   **Resolution:** Local development machine, no HPC/cloud commitment for Phase 1. With Phase
   1's accessions confirmed (see [`01-phase1-sjogrens-pilot.md`](01-phase1-sjogrens-pilot.md) §7,
   question 1), the actual combined scale is known: GSE233180 (MG) ~29,700 cells, GSE272409
   (Sjögren's) ~31,000 cells, and the Human Tonsil Atlas subsampled to a comparable order of
   magnitude for the control (its scRNA-seq modality has 209,786 cells in the discovery cohort
   alone — we don't need all of it as a baseline reference). A sparse count matrix at that combined
   scale is a few GB, comfortably within a modern laptop's RAM; Harmony/scVI integration operates
   on the PCA-reduced embedding (~50 dims), not the full gene matrix, so it stays cheap too.
   Design choices to keep as cheap headroom regardless: use `anndata`'s backed-mode for the
   initial load/QC pass (filter before loading eagerly), and prefer scVI's minibatch training over
   full-batch Harmony if the joint integration step ever gets tight. Treat a single on-demand
   high-RAM/GPU cloud instance (or free university/lab HPC access, if available) as an escape hatch
   for one specific step if it chokes locally — not something to provision upfront. Revisit once
   Phase 5 (all 5 diseases + full integration) is in view, since that's a meaningfully larger combined
   dataset.

2. **Package manager** — Status: **Resolved**
   conda/mamba, or pip + `uv`/`poetry`? Affects `pyproject.toml` vs. `environment.yml` in Phase 0.
   **Resolution:** pip + `uv`, with `pyproject.toml` (not conda/mamba). The historical reason to
   reach for conda in single-cell Python — needing conda-forge/bioconda prebuilt binaries for
   compiled deps (HDF5, BLAS, LLVM linking) — is much weaker now: every package in this
   project's actual stack (`scanpy`, `anndata`, `leidenalg`/`python-igraph` for clustering,
   `scvi-tools`/PyTorch for integration, `harmonypy`, `celltypist`, `decoupler-py`, `squidpy`/
   `liana-py`) ships reliable PyPI wheels, including for macOS arm64 (this machine's platform).
   `uv` is much faster than conda/mamba for installs and lockfile resolution (matters for
   `make setup`/`make test` iteration speed), and `uv.lock` gives fully reproducible installs,
   matching the reproducibility principle in §3. `pyproject.toml` also keeps tool config (pytest,
   ruff, etc.) in one file and avoids an environment.yml → pyproject.toml migration if Phase 7
   ever wants this pip-installable. Fallback if some future dependency lacks a PyPI wheel: a narrow
   documented exception for that one package, not a switch of the whole project's package
   manager.

3. **Timeline/deadline** — Status: **Resolved**
   Is there a deadline this needs to hit (e.g. a science fair, symposium, or course deadline)? This
   affects how much time Phase 0 infra work should take before biology work starts.
   **Resolution:** Hard deadline — submission to the IRIS National Fair (India) by **2026-09-30**,
   ~34 days out from this plan (corrected from an initially-stated 2026-09-15, which was a
   self-imposed buffer rather than the real deadline). With the corrected timeline, Phase 1 proceeds
   at full rigor as detailed in §§3–7 of
   [`01-phase1-sjogrens-pilot.md`](01-phase1-sjogrens-pilot.md) — proper Phase 0 infra, full spatial
   validation (not the lightweight fallback), and full test coverage, not a stripped-down MVP. See
   that doc's §8 for the week-by-week schedule.

4. **Engineering rigor target** — Status: **Resolved**
   Is the end goal a polished, reusable open-source pipeline (CI, packaging, PyPI-able), or a
   well-tested but scoped research codebase whose main output is the scientific findings
   (paper/poster)? This changes how much to invest in things like CLI ergonomics and packaging
   in Phase 7.
   **Resolution:** A rigorously tested, reproducible **research codebase** — engineered to a high
   standard because that standard is what wins a fair, not because it's being shipped as a tool for
   others. Given the goal is IRIS National Fair (India), an ISEF-affiliated fair: judges read the
   report/poster and, for software-heavy categories, often interview about the code and
   methodology — they don't install the package or care about a release process. So invest heavily
   in: tests that catch real bugs (not padding for coverage numbers), full reproducibility (pinned
   `uv.lock`, seeded randomness, versioned configs — already the plan's direction), honest
   statistical methodology (already emphasized in Step 4/7), and a codebase clean enough to defend
   any part of if asked. Do **not** invest in PyPI packaging, CLI polish for external users, contribution
   guidelines, or semantic versioning — none of that serves the award goal. Phase 7's "packaging"
   deliverable means freezing the environment and finalizing the report, not publishing a package.
