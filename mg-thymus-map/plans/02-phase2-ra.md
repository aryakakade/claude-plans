# Phase 2 Plan — Rheumatoid Arthritis (Synovium) Extension

**Session: "RA Batch-Effect Marathon"** (2026-09-09 to 2026-09-11,
`claude.ai/code/session_01UaUH4B9As1iwjy8Yt9YpWF`, Claude Sonnet 5) — everything from the Step 1
AMP2 pivot-and-revert through the Step 4/5 batch-effect saga, the fibroblast-subtyping finding, the
intra-RA lining-vs-sublining extra analysis, its own Step 6 pharma narrowing (2026-09-11, see end of
§4 Step 6 subsection), and the Phase 3 dataset search was done in this one session. Named here so a
future session (any model) picking this back up can identify which session's work this is,
especially across a model switch.

**2026-09-11, later same session — third-disease search closed out.** After Phase 3's lupus
candidates (below) and, in a separate check this same night, Graves' disease, Hashimoto's
thyroiditis, IBD/Crohn's (SCP2959 — user personally checked, found it "requires organization email
etc."), and primary biliary cholangitis (CRA017680 — genuinely open but only 4 raw-BAM sequencing
runs vs. ~15 samples described in the source paper, and raw reads only, no processed matrix) were
each checked and ruled out, the user set a hard constraint: **genuinely open, zero-registration
data only — no DUC, no institutional email, no sign-in — because this is an ISEF submission and
extra forms aren't worth the risk.** Given six disease areas checked and all six either
scientifically insufficient or access-gated in some way, explicit recommendation given to the user:
stop the third-disease search and spend remaining time tightening Phase 1 (Sjögren's, confirmed)
and Phase 2 (RA, honest null + the intra-RA fibroblast/pharma finding above) rather than rushing a
third disease with a data-quality asterisk. **Not yet a final decision** — user has since raised
one more candidate (Smillie et al. 2019 ulcerative colitis atlas), being checked in a forked
sub-session; see `00-overview.md`'s session log for the outcome once that fork reports back.

Status: **In progress — dataset pivoted to AMP2, then reverted back, both 2026-09-09.**
`E-MTAB-11791`/`GSE283080` (Kuo et al./Miyahara et al.) **is the dataset this phase uses** — the
same conclusion as before the pivot, arrived at the long way. Briefly switched to the AMP RA/SLE
Network Phase 2 dataset (Zhang et al. 2023, *Nature*, Synapse `syn52297840`) to escape the
diagnosis-mapping blocker below, then reverted after direct checking found the specific files
carrying per-sample diagnosis (`AMP-RA.SLE_clinical.csv`, `metadata_clin_donor_singlecell.xlsx`)
sit behind a `ManagedACTAccessRequirement` — a Data Use Certificate + Intended Data Use statement,
reviewed by Synapse's Access and Compliance Team — the same *class* of gate as dbGaP's DAC, just
one layer deeper than the Dataset-collection-level check that first looked clean. Given no mishap
tolerance, not worth trading a bounded problem (an unresolved diagnosis field) for an unbounded one
(a discretionary human-review queue). **Final resolution: proceed on all 25 `E-MTAB-11791` samples
labeled generically as "inflammatory arthritis vs. OA"** — the weaker of the two options from the
original §7 item 2, chosen explicitly rather than defaulted into. Both the AMP2 detour and the
original Kuo/Miyahara research are kept below in full (nothing deleted) — real work, real lessons,
including two real access-tier findings (the E-MTAB-11791 diagnosis gap, and the AMP2 DUC/IDU gate)
that are each worth remembering if this decision is ever revisited. This is still a living document
meant to be argued with and edited, the same way Phase 1's plan doc was, not a finished spec.
Sections marked **OPEN** need a real decision before the next step can start cleanly; everything
else is a reasonable default carried over from Phase 1, subject to change.
Parent plan: [`00-overview.md`](00-overview.md). Companion: [`01-phase1-sjogrens-pilot.md`](01-phase1-sjogrens-pilot.md),
whose code and conventions this phase reuses wholesale.

## 1. Why RA, and what "replay" actually means here

RA (synovium) was always the planned Phase 2 target (see `00-overview.md`'s roadmap). It's a good
second pairing for the same structural reasons Sjögren's was a good first one: a single,
well-defined target organ (synovium), well-studied TLS biology (RA synovial TLS/ectopic germinal
centers are a textbook example, arguably *the* textbook example alongside Sjögren's), and — same
caveat as Sjögren's — real published ground truth to check the stromal target list against.

**What genuinely carries over from Phase 1 with zero code changes:** every QC/normalization/
integration/annotation function, the AUCell scoring framework, the regression + patient-level
statistical testing machinery, the DGIdb/ChEMBL clients. The MG-derived 25-gene TLS signature
itself is **not re-derived** — reusing it as-is is the entire point of testing whether it's shared.

**What does NOT carry over, and needs fresh work exactly like it did for Sjögren's:** a verified
dataset (see §7, Q1 — **OPEN**), a disease-appropriate comparator (RA vs. OA, the structural analog
of the SICCA decision), an RA-specific ground-truth paper to check the stromal list against (the
analog of Nayar et al.), and a full patient-level statistical audit of every claim before it's
reported — not assumed to be fine because Phase 1's code already handles it correctly once.

## 2. Objectives

1. Replay the full pipeline (assembly → cross-disease projection → stromal extraction →
   pharmacogenomic mapping → validation) on RA, using Phase 1's code unmodified, RA-specific config
   and data only.
2. Extend Aim 1's evidence: does the MG-derived shared architecture also transfer to RA, at the
   same statistical rigor as the Sjögren's result (patient-level, not cell-level)?
3. Extend Aim 2 concretely: compare RA's disease-specific residual (post-subtraction) against
   Sjögren's ISG signature (PIP/LYZ/IFI6/IFI44L/XAF1/AZGP1) — do they converge, or diverge? Both
   outcomes are reportable (see the discussion in this session's chat log, 2026-09-08).
3. Produce a second independent pharmacogenomic candidate list, and check whether it converges with
   Step 6's interferon-pathway/anifrolumab finding or points somewhere new.

## 3. Definition of done for Phase 2

- [x] An RA synovium scRNA-seq (or spatial) dataset is downloaded, documented, and loads cleanly.
      **Settled 2026-09-09, after a same-day pivot-and-revert (see Step 1 for the full story):**
      `E-MTAB-11791`/`GSE283080`, already fully downloaded and loading correctly
      (`load_ra_synovium`/`load_oa_synovium`, 102,758 real RA cells + 32,856 real OA cells, 10
      passing tests). **Explicit decision on the diagnosis question:** proceed on all 25 samples
      labeled generically as "inflammatory arthritis vs. OA," not the true 15-RA subset — a
      considered trade-off (see former §7 item 2), not an oversight. **ISEF-compliance check done
      and passed, 2026-09-09** — checked directly against the current Human Participants and Tissue
      & Body Fluid rule text; both `E-MTAB-11791` and `GSE283080` are public/open-access with no
      account, quiz, or DUC of any kind, the cleanest access tier this project has dealt with (see
      Step 1 for the full write-up). Same standing caveat as Phase 1's §3a: this project's own
      reading, not an official SRC ruling.
- [ ] QC + normalization + annotation produces a labeled RA dataset on the same shared cell-type
      vocabulary as Phase 1 (CellTypist `Immune_All_High`, `majority_voting=True`).
- [ ] The MG signature is AUCell-scored against RA, with the same tonsil-calibrated threshold logic
      and the same immediate patient-level audit discipline Phase 1 eventually adopted (not
      retrofitted after a retraction this time).
- [ ] RA stromal populations are isolated (RA vs. OA, or the best available comparator — see §7 Q2)
      and compared against an RA-specific ground-truth paper.
- [ ] RA's stromal marker list is queried against DGIdb/ChEMBL.
- [ ] The Sjögren's-vs-RA residual comparison (Objective 2 above) is run and reported, whichever way
      it comes out.
- [ ] Tests: same standard as Phase 1 — unit tests for anything RA-specific (a new loader, any new
      parsing logic), full `make test` green.
- [ ] A short Phase 2 findings write-up + Limitations section, same three-doc pattern as Phase 1
      (`docs/phase2_findings.md`, and only if genuinely useful, `phase2_methods.md`/
      `phase2_full_story.md` — Phase 2 should lean on Phase 1's methods doc by reference rather than
      duplicate it, since the methodology itself hasn't changed).

## 4. Step-by-step plan

### Step 1 — Data acquisition — Status: ✅ Settled 2026-09-09 (pivot to AMP2, then reverted, same day)

**Goal:** get one RA synovium dataset into the same loadable, documented state Phase 1's three
datasets are in.

**⏩ PIVOT then ⏪ REVERT, both 2026-09-09.** Switched away from `E-MTAB-11791` because its
per-sample RA-vs-other-diagnosis blocker (originally §7 item 1) turned out to be a genuine dead end
after real checking (Table S1/S2, main-text Table 1/Table 2, all read in full — none carry a
per-sample diagnosis column). Moved to the AMP RA/SLE Network Phase 2 dataset instead — see the
"AMP2 detour" block immediately below for the full record of that research, kept in full because it
found something real: the specific files needed (the per-sample clinical/diagnosis data) sit behind
a discretionary Data-Use-Certificate review, the same *class* of gate this project has consistently
avoided elsewhere. Given the user's explicit zero-mishap-tolerance instruction, reverted back to
`E-MTAB-11791`/`GSE283080` the same day rather than pursue that application. **Final decision: proceed
on all 25 `E-MTAB-11791` samples labeled generically as "inflammatory arthritis vs. OA"** — resolving
former §7 item 2 explicitly in favor of the weaker-but-immediately-usable option, a deliberate
trade-off, not an oversight.

---

#### Historical record (explored and abandoned, 2026-09-09) — the AMP2 detour

**New primary dataset + comparator briefly considered, in one deposit: AMP RA/SLE Network Phase 2**
(Zhang et al. 2023, *Nature* 623:616–624, "Deconstruction of rheumatoid arthritis synovium defines
inflammatory subtypes"; Synapse `syn52297840`). Checked directly against the paper and Synapse's
own metadata API, not assumed: **82 samples / 79 donors, 314,011 cells**, whole-tissue synovial
dissociation; every RA sample unambiguously RA by construction (stratified by treatment-response
category, not mixed arthritis subtypes); **9 OA samples from the same study, same protocol**;
CITE-seq (mRNA + 58 surface-protein ADT markers, only the mRNA side would have been used).

**Access, first pass — checked against the Dataset-collection entity itself, genuinely clean:**
`GET /entity/syn52297840/accessRequirement` and the same call on its parent project both returned
`{"totalNumberOfResults":0,"results":[]}`; `GET .../permissions` (anonymous) showed
`"canPublicRead": true`, `"isCertificationRequired": true` — reading only this far, it looked like
the same light tier already accepted for ImmPort SDY998 (free account + self-service quiz, no DAC).
**This first pass was real but incomplete** — it checked the Dataset collection record, not the
specific files inside it.

**Access, second pass — the real finding, 2026-09-09.** The user located the actual files needed
(`AMP-RA.SLE_clinical.csv`, `metadata_clin_donor_singlecell.xlsx` — both under a path literally
named "Controlled Access") and asked to verify that directly rather than take the folder name at
face value. Checking `accessRequirement` on those two specific file IDs (`syn47136972`,
`syn57405619`) directly returned a `ManagedACTAccessRequirement` ("Controlled AR - ARK Portal"):
`"isDUCRequired": true`, `"isIDURequired": true`, one-year renewal — a **Data Use Certificate +
Intended Data Use statement, reviewed by Synapse's Access and Compliance Team.** This is the same
*class* of gate as dbGaP's Data Access Committee (discretionary human review, real turnaround
time), just discovered one layer deeper than the first check reached — a genuine gap in that first
pass, not a false alarm. **Real lesson for next time: check access requirements on the specific
files actually needed, not just the collection/dataset entity that bundles them** — a container can
look open while what's inside it isn't.

**ISEF compliance, checked while this was still the leading option (kept for completeness, not
acted on further):** published in *Nature* — satisfies both the Tissue & Body Fluid and Human
Participants exemptions on the peer-reviewed-journal clause alone, same reasoning as Phase 1's §3a.
This finding is orthogonal to the access-tier problem above and would still hold if this dataset is
ever revisited.

**Real technical requirement this dataset would have introduced, noted for the record:** its files
are R serialized objects (`.rds`) per the analysis repo's README
(`github.com/immunogenomics/RA_Atlas_CITEseq`) — `raw_mRNA_count_matrix.rds`,
`fine_cluster_all_314011cells_82samples.rds`, `CTAP_donor_mapping.xlsx` — not the `.mtx`/`.h5ad`
every other loader in this project reads directly. This project's environment has been Python-only
so far; converting `.rds` would have needed R (`Matrix`/`Seurat`) as new infrastructure. Moot now,
but worth remembering if any future dataset also turns out to be R-only.

**Division-of-labor note, kept for the record:** the user explicitly asked that all Synapse
browsing/downloading be done by them directly, not by this session, out of caution about automated
access to a platform requiring account registration — checked Synapse's actual Terms of Use PDF
directly and found no prohibition on scripted access (Sage Bionetworks ships official Python/R/CLI
clients for exactly that), so this was the user's own preference, respected as such, not a
compliance requirement. Moot now that AMP2 isn't being used, but the same principle (anonymous
public-metadata checks are fine; anything needing login goes through the user) should carry forward
to any future access-gated dataset this project considers.

*(End of AMP2 detour.)*

---

#### Reinstated as the active dataset, 2026-09-09 (after the AMP2 detour above) — Kuo et al./Miyahara et al. research

**What's confirmed so far (2026-09-08 session), checked directly, not assumed:**
- The most famous RA-vs-OA synovium scRNA-seq study (Zhang et al. 2019, *Nature Immunology*, the
  AMP RA/SLE Network — exactly the comparator design this project wants) is **not on GEO**. Its data
  lives on ImmPort (SDY998) and dbGaP (phs001457.v1.p1). dbGaP is controlled-access — the same
  situation Phase 1 explicitly avoided for the MG spatial data (JGA vs. figshare, §3a). Whether
  ImmPort's SDY998 has an openly-downloadable tier (some ImmPort studies do) has **not** been
  checked yet.
- **All three GEO candidates from the initial search are now confirmed, by direct inspection of
  their real GEO records (not the search summary), to be unusable — 2026-09-08:**
  - **GSE296117** — synovial *fluid*, not tissue (immune-cell-dominated, wrong compartment for
    Step 5 the same way MG's CD45+-sorting was); true raw data is controlled-access on China's
    GSA-Human (`HRA011646`), only a single bundled `.rds` (R object) file is actually public on
    GEO; no OA/comparator arm at all (pre/post-drug-treatment design within RA patients only).
  - **GSE109449** — only 384 cells total (96 cells × 2 RA + 2 OA patients), plate-based not
    droplet-based, FACS-**sorted fibroblasts only** (zero immune cells — no Step 4 cross-disease
    projection possible), and its own record states raw data will never be available even via
    dbGaP: *"The patients did not consent to genomic data sharing."*
  - **GSE56409** — not single-cell at all: `Expression profiling by array` (bulk microarray),
    from **cultured** fibroblasts (expanded in a dish, not fresh dissociated tissue). Wrong
    technology for every stage of this pipeline.
  - **ImmPort SDY998** (the AMP RA/SLE Network's real dataset, Zhang et al. 2019) — access itself
    turned out to be genuinely open (free account only, no DAC/IRB/data-use-agreement, confirmed
    via the original authors' own download guide). But the actual scRNA-seq data is a poor
    technical fit regardless: only **5,265 cells total** across 21 samples (~250 cells/sample),
    explicitly **FACS-sorted populations** (B cells, fibroblasts, monocytes, T cells sorted
    separately), not whole-tissue dissociation — a fundamentally different, much lower-throughput
    design than MG/Sjögren's/tonsil, likely to hit the same small-sample failure modes already
    documented for Harmony/Scrublet in Phase 1. **Verdict: not used as the primary dataset**, but
    kept as a candidate ground-truth comparison paper for Step 5 (the CTAP framework), the same
    role Nayar et al. played for Sjögren's.
- **Pattern worth naming explicitly:** every candidate found by a quick keyword search failed on
  inspection, for several different reasons (wrong compartment/controlled-access, too
  small/consent-blocked, wrong technology, fabricated search-summary detail — see below). This is
  exactly the kind of thing Phase 1 learned to expect and check for up front rather than assume
  away — real time was budgeted for this, not treated as a formality, and it paid off.
- **Two more user-suggested candidates checked directly, 2026-09-09, after the dataset decision
  below was already made — both ruled out, for the record:**
  - **`GSE109248`** — not Stephenson et al. at all, despite the accession being suggested under that
    name. Checked its real GEO metadata directly: *"Genome-wide analysis of gene expression of
    cutaneous lupus and cutaneous psoriasis lesions"* — 56 skin biopsy samples, bulk microarray
    (`Expression profiling by array`), lupus/psoriasis, not RA/OA. Wrong tissue, wrong technology,
    wrong disease pairing on every axis. **The real Stephenson et al. 2018** (*Nat Commun* 9:791,
    "Single-cell RNA-seq of rheumatoid arthritis synovial tissue using low-cost microfluidic
    instrumentation") isn't on GEO at all — checked its actual Data Availability statement
    directly (PMC5824814): *"RNA sequencing data... have been deposited in dbGaP with the
    accession code phs001529.v1.p1"* — controlled-access, the same class of gate this project has
    consistently avoided (and the same accession AMP2's own Data Availability statement cited for
    Stephenson, during the now-reverted AMP2 detour above).
  - **`GSE299518`** — real, genuinely open, no gate: *"Single cell RNA sequencing on synovium,
    meniscus and cartilage of rheumatoid arthritis"* (*Nature Communications*, 2025,
    PMID 40721606). But only **3 samples total, and only 1 is synovium** (the other two are
    meniscus and cartilage from the same knee) — **no OA/comparator samples deposited in this
    accession at all**. n=1 for the tissue this project needs means zero patient-level statistics
    are possible — a hard disqualifier given this project's own standard (patient-level, not
    cell-level, testing). **Not usable as a primary dataset.** Its headline finding — CD142+
    synovial fibroblasts as a novel RA lining-layer subset — is specific and independent enough to
    be worth keeping as a candidate for Step 5's ground-truth comparison or Step 7's
    positive-control list (same role as Croft et al. 2019), just not as a data source. (One file in
    its deposit, `AllCelltype_Reference_Seurat.RDS`, is R-format — moot here since the dataset
    isn't being used, but the same `.rds` friction as the AMP2 detour if it's ever revisited.)

**✅ RESOLVED, 2026-09-08 — dataset decision made.**

**Primary dataset: `E-MTAB-11791`** (ArrayExpress) — Kuo et al.,
"Molecular maps of synovial cells in inflammatory arthritis using an optimized synovial tissue
dissociation protocol." 25 fresh synovial tissue biopsies, 23 patients, **~102,758 cells**, real
10x Genomics Chromium (v3.0/v3.1), Cell Ranger v6.0.0, genuinely open access (no DAC). **Real
per-diagnosis breakdown, checked directly (not the abstract-level summary): 15 RA, 3 psoriatic
arthritis, 4 spondyloarthritis, 3 undifferentiated arthritis** — a genuinely solid RA sample size,
larger than Sjögren's own 7 PSS patients. No OA/healthy arm in this study.

**Correction, 2026-09-09: `GSE181082` is NOT a "partial GEO mirror" of this dataset, as this doc
previously (and wrongly) stated.** Checked directly by downloading GSE181082's own GEO SOFT
metadata (`GSE181082_family.soft`, via `ftp.ncbi.nlm.nih.gov`): it is an entirely different paper
("MAFB surrogates the glucocorticoid receptor ability to induce tolerogenesis in dendritic cells,"
same Zurich/Ospelt lab, overlapping co-authors) that reuses only **2 of Kuo et al.'s 23 patients**
(patients 28 and 50 — `SB-028`/`SB-050` in Table S1's list), labeled by glucocorticoid-treatment
status ("GCs" vs. "none"), not by arthritis diagnosis. Confirmed independently against the paper's
own Data and Code Availability statement, which states plainly: "Subset of data (patients 28 and
50) also in NCBI Gene Expression Omnibus: GSE181082." Not usable for the diagnosis-mapping
question below, and should not have been cited as a mirror of the full 25-sample cohort.

**Comparator: `GSE283080`** — Miyahara et al. 2025, *JCI Insight*, "CD34hi subset of synovial
fibroblasts contributes to fibrotic phenotype of human knee osteoarthritis." **4 knee OA patients**
(2 classified inflammatory-type, 2 fibrotic-type OA synovium by the original authors — a real
subtype split within the OA group, not 4 uniform replicates), real 10x Chromium v3.1, standard
`barcodes/features/matrix` mtx triplets (same format `sjogrens.py`'s loader already parses),
GPL24676 platform, openly on GEO, no DAC.

**Honest limitations of this pairing, stated up front rather than discovered later:**
- **Cross-study comparator, not same-study** (unlike SICCA/PSS) — different institutions
  (Kuo et al.'s cohort vs. University of Tokyo), likely different exact dissociation/library
  protocols despite both using 10x Chromium. This is the same class of risk Phase 1's own risk
  table already flags for any cross-study comparison; expect to check for a batch effect the way
  Phase 1 always has, not assume it away.
- **Sample-size imbalance**: 15 RA vs. 4 OA. Workable (Sjögren's own 7-vs-6 was already thin), but
  worth stating plainly, not glossing over.
- **OA reference has internal heterogeneity** (2 inflammatory-type + 2 fibrotic-type patients) —
  worth checking whether that matters for the comparison design once the data is in hand, rather
  than assumed to be a uniform "healthy-ish" baseline the way SICCA was treated.

**Requirements this pairing was checked against, learned directly from Phase 1's own mistakes/near-misses:**
1. Whole-tissue, not sorted — ✅ both.
2. Droplet-based (10x), real per-sample cell counts in the thousands — ✅ both.
3. Ideally RA vs. OA within the same batch/protocol — ⚠️ not met; accepted as a cross-study pairing
   given how many same-study options were checked and ruled out.
4. Public/GEO or another clearly open-access repository — confirm no DAC/dbGaP application needed,
   checked directly against the actual access page, not assumed from a paper's reputation.

**Implementation — done 2026-09-08.** Built `scripts/download_ra_synovium.py` and
`scripts/download_oa_synovium.py` (both idempotent — skip a file if it already exists and is
non-empty; **honest limitation, stated plainly**: neither BioStudies nor GEO publishes an MD5 for
these files, so idempotency here is existence-based, not checksum-verified — a real gap from
Phase 1's original stated intent, same class of gap as `pipeline.py` never being wired to
orchestrate stages). `src/mg_thymus_map/data/ra_synovium.py` (`load_ra_synovium`) and
`oa_synovium.py` (`load_oa_synovium`), plus 10 new tests (5 unit + 1 integration for RA, 4 unit
for OA), all passing — 160 tests project-wide, `make test` green. Both scripts were actually run
against the real download servers this session, not just written and left untested — confirmed
live by watching real files land in `data/raw/{ra_synovium,oa_synovium}/` (gitignored, same
convention as Phase 1's raw data).

**Real bug caught immediately, before writing any loader code: `requests` cannot fetch `ftp://`
URLs.** Both GEO's and ArrayExpress's own machine-readable records give `ftp://` scheme URLs. This
project's only HTTP client (`requests`, already a Phase 1 dependency for the DGIdb/ChEMBL clients)
has no FTP support and raises `InvalidSchema` on an `ftp://` URL. **Fixed by using HTTPS on the
identical path** — confirmed directly with a HEAD request that `ftp.ncbi.nlm.nih.gov` serves the
same file tree over HTTPS before writing the real script, not assumed.

**OA loader (`oa_synovium.py`) — straightforward, closely mirrors `sjogrens.py`'s established
per-sample-mtx-triplet pattern.** One real design choice: captured each sample's inflammatory-type
vs. fibrotic-type OA classification as its own `oa_subtype` obs column (via `augment_obs`'s
`**extra` kwarg, already supported by Phase 1's code, unused until now) rather than treating the 4
OA patients as uniform replicates — matches the honest framing already in this doc's limitations
section above.

**Real download completed and the real loader run against it, 2026-09-09 — not just the synthetic
test.** `download_oa_synovium.py` pulled all 12 real files (4 samples × 3 files) cleanly.
`load_oa_synovium` against the real data: **32,856 cells, 36,601 genes** — sensible per-sample
counts (GSM8655601/Infla1: 7,613; GSM8655602/Infla2: 10,901; GSM8655603/Fibro1: 9,042;
GSM8655604/Fibro2: 5,300 — no zero-cell or corrupt samples), 18,514 inflammatory-type cells /
14,342 fibrotic-type cells. No crashes, no format surprises — this one loaded clean on the first
real run. RA's `count_matrix_filtered.mtx` (3.5 GB) is still downloading as of this note; will be
confirmed against the real loader the same way once it lands.

**RA loader (`ra_synovium.py`) — structurally different from every Phase 1 loader, confirmed by
direct inspection before writing any code, not assumed to match the familiar per-sample-triplet
shape.** E-MTAB-11791 is deposited as ONE pooled matrix (`count_matrix_filtered.mtx`, confirmed
17,057 genes × 102,758 cells — the exact real scale, not the paper's rounded "~100,000") plus flat
`genes_filtered.txt`/`barcodes_filtered.txt` lists, with per-cell sample identity encoded as a
prefix on each barcode string (e.g. `Syn_Bio_023.AAACCCAAGAGGCTGT`) rather than separate files per
sample. Matrix orientation (genes × cells, needing a transpose for AnnData) was confirmed against
the real `.mtx` header's dimension line before writing the loader, not assumed from convention.

**A second real quirk found and handled, not glossed over: the SDRF sample-metadata file's `Source
Name` values don't map 1:1 to barcode prefixes, in two different ways.** Checked by pulling the
real 25 unique barcode prefixes and the real SDRF file directly: some samples have multiple SDRF
rows differing only by a trailing `_1`/`_2`/`_3` (separate sequencing lanes/libraries of the *same*
sample, e.g. `Syn_Bio_023_1` and `Syn_Bio_023_2` both collapse to barcode prefix `Syn_Bio_023`) —
but other samples carry a letter suffix that IS part of the distinct sample identity, not a lane
suffix (`Syn_Bio_077a` and `Syn_Bio_077b` are two separate biopsies from individual 77, and must
stay two separate samples). `_index_sdrf_by_barcode_prefix` handles both correctly via
longest-match `startswith` against the real set of 25 barcode prefixes rather than a hardcoded
suffix-stripping rule that would have silently mishandled one case or the other. Unit-tested
against synthetic fixtures that specifically replicate both real patterns.

**The one real, unresolved blocker, stated plainly rather than worked around with a guess: no
per-sample RA-vs-other-diagnosis label is recoverable yet.** Checked the SDRF's own
`Characteristics[disease]` field directly — it is the generic string `"arthritis"` for **all 25
samples**, not differentiated by diagnosis. The paper's Table 2 reports the aggregate breakdown
(15 RA / 3 PsA / 4 SpA / 3 UA), but the per-sample mapping from individual ID to specific diagnosis
was not found in the SDRF, the main text, or the parts of the PMC article fetched — it most likely
lives in a supplementary table (Table S1/S2) that automated fetching could not reach this session
(ScienceDirect returned HTTP 403 to a direct fetch; guessed PMC `bin/mmc1.*` paths returned 404;
the legacy PMC Open-Access API endpoint used to bulk-fetch supplementary files also returned 404,
suggesting it's been retired/moved). **Consequence, made explicit rather than silently narrowed:**
`load_ra_synovium` currently labels every cell `disease: "inflammatory_arthritis"` — an honest
generic placeholder, not a guess at which 15 of 25 samples are RA. **This is now the single most
important open item blocking Step 2+** — see the consolidated open questions at the end of this
doc.

**Real download completed and the real loader run against the full 3.75 GB file, 2026-09-09 — not
just the synthetic test.** `download_ra_synovium.py` pulled all 4 real files cleanly (the 3.5 GB
matrix, confirmed as 3,749,874,692 bytes on disk). `load_ra_synovium` against the real data:
**102,758 cells × 17,057 genes — exactly matching the dimensions read from the raw `.mtx` header
during the earlier verification pass**, no shape mismatch. **25 samples, 23 unique individuals** —
precisely confirms the structural analysis above: individuals 77 and 98 each contributed two
separate biopsies (`Syn_Bio_077a`/`077b`, `Syn_Bio_098a`/`098b`), correctly kept as 4 distinct
samples belonging to 2 individuals by the `startswith`-based SDRF matching, not collapsed or
conflated. Per-sample cell counts range 524 (`Syn_Bio_049`, the smallest) to 7,843
(`Syn_Bio_026`) — real, expected biological/technical variation, not a bug. Sex split: 82,502
female cells / 20,256 male cells, consistent with the cohort's real sex composition (19 of 23
individuals female, per the SDRF). One benign warning (`Variable names are not unique` — some
gene symbols repeat in `genes_filtered.txt`, which has no Ensembl ID column to disambiguate by;
resolved by the loader's own `var_names_make_unique()` call immediately after, same as every other
loader in this project) — not a bug, not chased further.

**Both real datasets are now fully downloaded, loaded, and confirmed working end-to-end against
real data, not just synthetic fixtures.** Step 1 is complete. The diagnosis-mapping blocker above
is real and still unresolved — the deliberate, explicit resolution (see the top of this section and
former §7 item 2, now closed) is to proceed on all 25 samples labeled generically, not to treat this
as blocking. `load_ra_synovium`'s current `disease: "inflammatory_arthritis"` label is therefore
the correct label to build on, not a placeholder waiting to be replaced.

*(This dataset is the active plan — see the top of this section for the pivot-and-revert story.)*

### Step 2 — QC, normalization, annotation — Status: ✅ Complete 2026-09-09 (batch-effect finding investigated and resolved)

Same functions, same conventions as Phase 1 (adaptive mito%, pooled doublet threshold, CellTypist
`Immune_All_High` + `majority_voting=True`, Harmony via direct `run_harmony()` call) — no
RA-specific code, per `scripts/run_phase2_ra_step2.py`.

**Real numbers, 2026-09-09:**
- **RA:** 102,758 → 102,118 post-QC (adaptive mito threshold 30.40%) → 102,022 post-doublet-removal
  (96 flagged, 0.09%).
- **OA:** 32,856 → 29,046 post-QC (adaptive mito threshold 17.70%) → 28,041 post-doublet-removal
  (1,005 flagged, 3.46% — notably higher doublet rate than RA; not yet investigated further).
- **Cell types (CellTypist, majority_voting):** RA — 31% Macrophages, 30% Fibroblasts, 22% T cells,
  9% Endothelial, plus Monocytes/DC/B/ILC/Mast/pDC/Plasma at lower shares. OA — 50% Macrophages,
  37% Fibroblasts, 5% T cells, then DC/Plasma/Endothelial/B/ILC/Mast/Monocytes/pDC. **OA's much
  higher macrophage share and much lower T-cell share than RA is flagged, not yet explained** — it
  could be real OA biology (less lymphocytic infiltrate than RA, a textbook expectation) or an
  artifact of the batch effect below; not resolvable from cell-type proportions alone.
- Combined: 130,063 cells, 27,136 genes.

**§7 item 5's predicted cross-study batch effect is confirmed real, investigated, and mostly
explained — see §7 item 5 for the full cluster-by-cluster finding.** Pre-Harmony, most Leiden
clusters were >95% one dataset. Post-Harmony, most improved substantially; the clusters that
stayed skewed turned out to be mostly legitimate disease-biology/population-size differences (the
same class of explanation Phase 1 accepted for its own batch effect), plus one negligible 27-cell
technical artifact. **Deliberately no Harmony parameter tuning** — changing the shared integration
code specifically for RA would have broken Objective 1's cross-disease code-consistency premise.
Step 3/4 proceed on `data/interim/ra_oa_harmonized.h5ad` as-is.

**Not yet done:** the CellTypist-fibroblast-granularity audit (§7 item 6) — still open, unaffected
by the batch-effect finding above; both are real open items, not the same one.

### Step 3 — TLS signature — Status: not re-derived, reused as-is

The 25-gene MG signature is not touched. Re-deriving it from RA would defeat the point.

### Step 4 — Cross-disease projection — Status: ✅ Run 2026-09-10 — real NEGATIVE result for RA, flagged for re-investigation after Step 7

AUCell-scored RA and OA against the 25-gene MG signature (`data/processed/tls_signature.csv`),
same tonsil-calibrated threshold logic as Phase 1 (`scripts/run_phase2_ra_step4.py`, reusing
`signature/spatial_validation.py` and `scoring/cross_disease_projection.py` unmodified). Patient-
level testing applied from the start, not retrofitted, per this section's original intent.

**All 25 genes present in RA, OA, and tonsil — no gene-dropout issue.** TLS-region cell counts
(GC B cell/Tfh states): RA 1,140/102,022 (1.1%), OA 97/28,041 (0.35%), tonsil 1,778/38,072 (4.7%).
Tonsil-calibrated threshold (50th percentile of tonsil TLS-region scores): 0.106942.

**Headline result — RA does NOT replicate Sjögren's Phase 1 finding:**

| Comparison | n | Median score | Tonsil reference median | p-value | BH | Status |
|---|---|---|---|---|---|---|
| RA TLS-region vs. tonsil | 1,140 | 0.0998 | 0.1069 | 0.9999 | 0.9999 | **not significant** |
| OA TLS-region vs. tonsil | 97 | 0.1317 | 0.1069 | 0.00008 | 0.00016 | ~~confirmed~~ **superseded — see below** |

**Correction, 2026-09-10 (§7 item 9): OA's "confirmed" row above does not survive patient-level
testing** (p = 0.129 at the patient level, 4 OA patients vs. 4 tonsil donors, 70 exact
permutations — `phase2_step4_oa_tonsil_patient_level_test.csv`) — the same cell-level cross-study
artifact Step 5 independently found. **Neither RA nor OA shows genuine TLS-signature enrichment
over tonsil.** The table above is kept as originally computed (not silently edited) with this
correction stated explicitly alongside it.

Only 42.0% of RA's TLS-region cells exceed the tonsil threshold (below the ~50% a group
statistically identical to tonsil would show). Bootstrap CI for RA-vs-tonsil median difference:
**−0.0072 [−0.0105, −0.0036]** — excludes zero, so this is a real shortfall, not underpowering.
**Patient-level exact permutation test (the most rigorous test here, avoids pseudoreplication)
confirms it**: 23 RA individuals vs. 4 tonsil donors, 17,550 exact permutations, observed diff
−0.00077, **p = 0.625** — nowhere close to significant. RA vs. OA patient-level: p = 0.059 (not
significant, but trending toward OA > RA).

**One reassuring piece: specificity still holds.** Every one of the 25 RA samples and all 4 OA
samples show TLS-region cells scoring higher than that same sample's own non-TLS-region cells (all
"diff" values positive) — the signature is still meaningfully specific to TLS-organizing states
within each tissue, RA just doesn't reach tonsil's absolute baseline the way Sjögren's PSS did in
Phase 1 (69.1% exceeding threshold there, vs. RA's 42.0% here).

**Leading hypothesis, not yet confirmed either way:** this is very likely connected to the standing
decision (§7, former item 2) to proceed on all 25 mixed-diagnosis samples rather than the true
15-RA subset — PsA/SpA/UA diluting a real RA-specific signal is at least as plausible as RA
genuinely lacking shared TLS architecture with MG. **Deliberately not investigated further right
now** — the user's explicit call, 2026-09-10: finish the full pipeline through Step 7 first on the
current (all-25-samples) data, THEN circle back and check whether resolving the true RA-only subset
changes this result, rather than context-switching mid-pipeline. See §7 item 11.

**Anomaly flagged here — RESOLVED, see §7 item 9:** OA's apparent significant TLS-signature
enrichment while RA did not was checked directly and found to be a cell-level cross-study artifact,
not real — it does not survive patient-level testing (p=0.129). OA's cell-composition/doublet-rate
anomaly (§7 item 10) remains open as separate, unresolved context.

Full results: `data/processed/phase2_step4_cross_disease_results.csv`,
`phase2_step4_headline_metric.csv`, `phase2_step4_specificity_patient_level.csv`,
`phase2_step4_ra_tonsil_patient_level_test.csv`, `phase2_step4_ra_vs_oa_patient_level_test.csv`,
`phase2_step4_ra_patient_level.csv`, `phase2_step4_oa_patient_level.csv`,
`phase2_step4_oa_tonsil_patient_level_test.csv` (added 2026-09-10).

### Step 5 — Stromal extraction — Status: 🟡 Real fix applied 2026-09-10, honest null result — no RA-specific marker survives, most likely underpowered not broken

RA (all 25 samples, generically labeled) vs. OA (`GSE283080`, Step 1's reinstated cross-study
comparator), regress out shared signature, rank markers, patient-level confirm. **Needs an
RA-specific ground-truth paper** — the Nayar-et-al analog.

**Method — reused Phase 1's code exactly** (`scripts/run_phase2_ra_step5.py`,
`stromal/extraction.py` + `stromal/patient_level.py` unmodified). Restricted to stromal compartment
(CellTypist `Fibroblasts`/`Endothelial cells` — RA: 40,271 cells [30,959 Fibroblasts + 9,312
Endothelial]; OA: 10,794 cells [10,413 Fibroblasts + 381 Endothelial]; no `Epithelial cells` in
either, expected for synovium). Regressed out each cell's Step-4 TLS-signature AUCell score from
every gene (16,183 genes shared between RA/OA), then ranked RA-vs-OA markers on the residual
(Wilcoxon). Patient-level confirmation used `individual` (23 unique) for RA and `sample_id` (4) for
OA — the correct units, not the 25 RA samples/4 OA samples raw count, avoiding the exact
pseudoreplication trap this project's own code was built to catch.

**Headline: 15/15 top non-artifact candidate genes "confirmed" at patient level, BH-corrected
(IGFBP7, TPM4, KLF6, EMP1, SPARC, VIM, MCL1, ACTN1, MARCKS, FOSB, DDX3X, LRRFIP1, SLC2A3, IRF1,
NAMPT — all "up in RA").** Full ranked list: `data/processed/phase2_step5_stromal_markers.csv`.
Patient-level results: `data/processed/phase2_step5_candidate_patient_level_bh.csv`.

**🔴 Real red flag, caught before reporting this as a finding, not after — DO NOT TRUST THIS GENE
LIST YET.** Checked the Mann-Whitney U statistics directly rather than just reading BH-significant
as "confirmed": **10 of the 15 genes hit U = 92.0 exactly — the mathematical maximum possible U for
n_ra=23 vs. n_oa=4 (23×4=92), meaning complete separation: every single RA individual's value
exceeds every single OA patient's value, for 10 different genes simultaneously.** That is a far
stronger and more uniform signal than real single-gene disease biology plausibly produces across
independent genes, and OA's n=4 makes "complete separation" trivially easy to hit by any systematic
non-biological group difference, not just a real large effect. Two converging reasons to suspect
this is **not** RA-specific biology:
1. **This regression only removed the Step 4 TLS-signature score — it did NOT re-apply Step 2's
   Harmony batch correction**, which was confirmed real in Step 2 (§7 item 5's resolution). AUCell/
   regression need full-gene un-corrected expression, so whatever cross-study technical difference
   Harmony was correcting for in Step 2 is still fully present here, unaddressed, and could easily
   present as "RA vs. OA" separation that's actually "Kuo-et-al-protocol vs. Miyahara-et-al-protocol."
2. **The gene identities themselves look like a known technical confound, not disease biology.**
   None of the 15 are canonical RA synovial fibroblast markers from the literature (THY1, CD34,
   PRG4, FAP, PDPN — the field-standard panel). Several — especially **FOSB**, an immediate-early
   gene — are specifically documented in the single-cell literature as **dissociation-protocol
   stress-response artifacts** (van den Brink et al. 2017, *Nat Methods*, flagged FOS/FOSB/JUN/EGR1
   as exactly this). This project's own `TECHNICAL_ARTIFACT_PREFIXES` filter (written for this
   script, not Phase 1's original — that flagging pass was never committed, see the script's own
   docstring) only caught Ig/Hb/MT/RP gene families and **missed this entire class** of
   dissociation-stress artifact, which is precisely the kind of confound a cross-study,
   cross-protocol RA/OA pairing (§7 item 5) would produce.

**All three next steps above are now done, 2026-09-10 — full resolution below.**

**Attempt 2 (superseded): cell-level ComBat.** `scripts/run_phase2_ra_step5_combat.py` — extended
artifact filter to the immediate-early/stress family, applied ComBat at the cell level before
ranking. **0/15 original genes survived — strong confirmation the original list was pure batch
artifact.** But the replacement list was itself untrustworthy: ComBat logged "34 genes with zero
variance" and promoted biologically implausible near-zero-expression genes (an olfactory receptor,
pancreatic elastase). Root cause understood, not chased further: ComBat assumes roughly-Gaussian
per-gene batch effects, which breaks on sparse single-cell data.

**Attempt 3 (the real fix): pseudobulk selection + pseudobulk ComBat.**
`scripts/run_phase2_ra_step5_pseudobulk.py`. Validated the approach first against data already
trusted — see §7 item 12's Sjögren's-reproduction check, which confirmed the underlying
patient-level Mann-Whitney math exactly reproduces Phase 1's own PIP/LYZ numbers, and clarified
that *selection* (not confirmation) needed to move to pseudobulk, not the whole design. Then
applied to RA/OA: aggregated to one residual-expression profile per patient (27 total: 23 RA
individuals + 4 OA patients) over 8,816 non-artifact genes expressed in ≥10% of cells, ComBat-
corrected that dense pseudobulk matrix (well-behaved here, unlike cell-level — no zero-variance
warnings), selected a 15-gene shortlist by |pseudobulk median difference| (PRG4, CLU, CFD, IGFBP5,
MT2A, CRIP1, COL1A1, THBS4, PLA2G2A, TIMP3, MT1X, CRTAC1, DCN, TMEM196, HLA-B — note **PRG4 is a
real Zhang-et-al-2019/Croft-et-al-2019 ground-truth lining-fibroblast marker**, a genuinely
promising sign this selection stage is finding real biology, not artifact), then ran patient-level
Mann-Whitney confirmation restricted to just that shortlist (BH-corrected across 15, not 8,816).
**Real bug hit and fixed along the way:** the first run crashed with `ValueError: 'patient_id' is
both an index level and a column label, which is ambiguous` — the pseudobulk DataFrame's index had
inherited the name `"patient_id"` from the earlier `groupby("patient_id")` call in
`build_pseudobulk`, colliding with a column of the same name when constructing the shortlist
AnnData's `obs`. Fixed by using a fresh, unnamed `pd.Index` for that `obs` DataFrame instead of
reusing the named index directly; re-run clean.

**Result: 0/15 confirmed — but this time it looks like an honest null, not a broken method.**
U-statistics spread reasonably (27–51 out of a possible 0–92 range) — no more suspicious complete
separation. Raw p-values before correction: 0.22–0.92, nowhere near nominal significance even
uncorrected. **Most likely explanation: a genuine power problem, not absence of any RA-specific
signal** — n_oa=4 patients gives very coarse Mann-Whitney resolution regardless of true effect
size, compounded by RA's own still-unresolved mixed-diagnosis dilution (§7 item 11). **This
strengthens the case for revisiting both the RA-only subset question and, potentially, finding an
OA comparator with more than 4 patients**, rather than treating this null as RA lacking a
detectable disease-specific stromal signature. Full results:
`data/processed/phase2_step5_pseudobulk_selection_ranking.csv`,
`phase2_step5_pseudobulk_candidate_patient_level_bh.csv`.

**Historical record, not deleted:** the original cell-level result and the failed cell-level-ComBat
attempt are preserved above and in `phase2_step5_stromal_markers.csv` /
`phase2_step5_stromal_markers_combat.csv` — real work, real lessons (the exact-maximum-U-statistic
diagnostic and the ComBat-sparsity lesson are both reusable knowledge for any future cross-study
comparison this project attempts), just not the trustworthy result.

**Back to the original constraint, now that the AMP2 detour is reverted:** Kuo et al.'s own
`E-MTAB-11791` findings can't be the ground-truth comparison either, for the same circularity
reason AMP2's own paper couldn't — it's literally the data being analyzed. Real independent
candidates:
- **Zhang et al. 2019** (AMP RA/SLE Phase 1, *Nature Immunology* — its raw data was already ruled
  out above as too small/FACS-sorted for use as a *data source*, but its *reported findings* are
  still usable as ground truth, the same way Nayar et al.'s reported findings were used when
  GSE272409 had no spatial deposit) — a genuinely separate cohort from `E-MTAB-11791`.
- **Croft et al. 2019, *Nature*** ("Distinct fibroblast subsets drive inflammation and damage in
  arthritis") — a synovial fibroblast-subtyping paper found during this session's dataset search,
  not yet read in full or checked for cohort independence from `E-MTAB-11791`/`GSE283080`.
- Kuo et al.'s own paper can still serve as a *first sanity cross-check* (it's literally the same
  data, so agreement is expected and weak evidence, but stark disagreement would be a real red
  flag worth investigating) — just not as the primary independent ground truth.

**Objective 2's comparison** (RA residual vs. Sjögren's ISG signature) happens here.

### Step 6 — Pharmacogenomic mapping — Status: reused code

Same DGIdb/ChEMBL clients, RA's gene list in. Check whether the result converges with Step 6's
interferon-pathway/anifrolumab finding or points to a different mechanism.

### Step 7 — Validation — Status: reused method, fresh gene list

Positive-control analog, same reframing discipline already established (seed genes ≠ pipeline
discoveries — apply that lesson from the start this time, not after catching it live). Candidate
RA/TLS positive-control genes: CXCL13 again (independently implicated in RA lymphoid neogenesis
too, worth confirming with a real citation before using it), plus RA-specific candidates to
research (ACPA-pathway genes, or whatever Step 1's eventual ground-truth paper names).

## 5. Timeline and the lupus-nephritis question — Status: OPEN, real constraint

**Hard constraint, confirmed 2026-09-08:** submission is 2026-10-03; the user's exams start
2026-10-01, five exams, four of five already prepped, physics + revision still outstanding. Real
usable project time is **not** the full window to Oct 3 — the last several days before Oct 1 need
to be protected for exam prep, not project work. Working estimate discussed: treat **~Sep 27–28**
as the real hard cutoff for all project work, not Sep 30 or Oct 3.

**Agreed approach (2026-09-08 chat, not yet formally written up before now):** don't commit both RA
and lupus nephritis to a fixed date up front. Use RA's actual pace over its first couple of real
days (~Sep 10–12) as a checkpoint, decide by **~Sep 12–13** whether lupus nephritis is realistic as
a second full-rigor phase before the cutoff, or should be scoped down / treated as a stretch goal /
reported honestly as "in progress, preliminary" if it doesn't fit. This mirrors Phase 1's own
"one phase fully green before the next starts" principle and its explicit refusal to let deadline
pressure cut statistical corners.

**This section should be updated with the real decision once the Sep 12–13 checkpoint happens** —
don't let it go stale the way Phase 1's status banners did (caught and fixed 2026-09-08).

**2026-09-12 checkpoint, updated as promised above.** Today is the checkpoint date. Real state:
**15–16 days of usable project time remain** before the ~Sep 27–28 cutoff (today is 2026-09-12;
exams start 2026-10-01). RA's actual pace over Sep 9–12 is now a known data point for this
checkpoint: Steps 1–6 plus the intra-RA extra analysis all ran in roughly 3.5 real days, including
two full detours (the AMP2 pivot-and-revert, and Step 5's three-attempt methods fight) — RA itself
is functionally done (see `docs/phase2_findings.md`, written today). **Decision: proceed with a
third disease now** — lupus nephritis specifically is superseded (see `03-phase3-ssc.md` §4), not
merely deferred, after its two real candidates were ruled out on independent grounds. Ulcerative
colitis (`SCP259`) is the active candidate, currently blocked only on the user's mentor completing
a Terra/Google sign-in (the user is 15; Terra requires 18+, so this genuinely needs their ISEF Adult
Sponsor, not a workaround). **Systemic sclerosis (`GSE138669`, Tabib et al. 2021) was found the same
day as a fully-open backup/parallel candidate** requiring no sign-in at all — see `03-phase3-ssc.md`
§5 for the full check. With ~15 days left and Phase 3 not yet started, real risk of the
"two-new-diseases-at-once" problem this section itself warned about is low only if Phase 3 moves
fast once a dataset is in hand — worth revisiting this section again once GSE138669's TLS-biology
literature check (§5) comes back.

## 6. Risks & mitigations (RA-specific, in addition to everything already in Phase 1's §6)

| Risk | Mitigation |
|---|---|
| No clean GEO-hosted RA-vs-OA whole-tissue droplet dataset actually exists at the quality Phase 1 had | **Materialized as `E-MTAB-11791`'s diagnosis-mapping gap.** Step 1 did try to fail fast and pivot (2026-09-09, to AMP2) but the pivot target had its own real access gate (a Data Use Certificate review) — reverted same day. **Final mitigation: accept the weaker "inflammatory arthritis vs. OA" claim rather than keep chasing a perfectly clean dataset** — stated plainly in the findings write-up's Limitations, not glossed over |
| Two-new-diseases-at-once time pressure degrades statistical rigor on either RA or lupus | The Sep 12–13 checkpoint (§5) exists specifically to catch this before it happens, not after |
| CellTypist's generic immune model doesn't meaningfully resolve RA's fibroblast-like synoviocyte subtypes | Flagged explicitly in Step 2 as something to check, not assume; Step 5's Leiden-subclustering rescue approach (already built, reusable) is the fallback if the broad label proves too coarse |
| Cross-study RA/OA batch effect isn't fully corrected by Harmony (§7 item 5) | **Materialized, then resolved, 2026-09-09** — investigated the residual (same method Phase 1 used for its own batch effect); mostly explained by legitimate population-size/disease-biology differences, one negligible-scale (27-cell) technical artifact. No Harmony tuning applied — deliberately kept the exact same code as Phase 1 to preserve cross-disease comparability |
| A dataset's *collection*-level access check can look clean while specific files inside it are gated (the real lesson from the AMP2 detour, §7 item 8) | Check `accessRequirement` on the specific files actually needed, not just the container entity, before treating any future dataset's access tier as settled |

## 7. Open questions for Phase 2 (consolidated, ordered by urgency)

**1. RA-specific per-sample diagnosis — Status: ✅ RESOLVED 2026-09-09, by decision (item 2), not by
recovering the mapping.**
`E-MTAB-11791`'s per-sample diagnosis genuinely isn't recoverable from any public source checked
(SDRF, Table S1/S2, main-text Table 1/2 — all read directly). A same-day pivot to the AMP RA/SLE
Phase 2 dataset (Zhang et al. 2023) — where every RA sample is unambiguously RA by construction —
was reverted after finding its own diagnosis-bearing files gated behind a Data-Use-Certificate
review (see Step 1's "AMP2 detour"). **Resolved by explicit decision, not by ever finding the
mapping**: proceed on all 25 samples labeled generically (item 2). The two leads below are kept as
a record, in case anyone wants the true RA-only subset later — not currently being pursued:
1. Lead contact Mojca Frank Bertoncelj, `frankbertoncelj@bio.mx` (paper's own designated channel for
   this exact request) — email drafted 2026-09-09, not sent, low priority now that the phase
   doesn't depend on a reply.
2. Two GitLab analysis-code repos (`gitlab.uzh.ch/retogerber/{synovialscrnaseq,protocol_synovial}`)
   were unreachable this session (connection timeout, not 403/404) — not pursued further.

**Follow-up, 2026-09-12 — the supplement itself was actually fetched this time, and confirmed
empty of the mapping.** Every prior attempt to reach the supplement failed at the transport layer
(ScienceDirect 403, guessed PMC `bin/mmc1.*` paths 404, the retired PMC OA API 404) — so the
content was never actually seen. This session found the correct PMCID (**PMC11144743** — a prior
guess, PMC11061893, was a wrong paper entirely) via Europe PMC's search API, then pulled the real
supplementary archive from Europe PMC's `supplementaryFiles` endpoint (WebFetch choked on the
24.8MB response, so downloaded directly with `curl`, and read the resulting `mmc1.pdf`, all 28
pages, directly — not summarized secondhand). **Confirmed: this paper's entire supplement is
Figures S1-S18 plus exactly two tables — Table S1 (per-sample neutrophil histology/scRNA-seq
detection, 18 samples) and Table S2 (demographics for 2 unrelated flow-cytometry proof-of-concept
patients, one with septic oligoarthritis, one with early RA).** Neither table, nor anything else in
the 28 pages, carries a per-sample RA/PsA/SpA/UA diagnosis column. **This upgrades the blocker from
"couldn't reach the source that might resolve it" to "reached the source and confirmed it doesn't
exist here"** — the decision in item 2 below was already being treated as final, and this removes
the last bit of uncertainty about whether an unread supplement might have changed that.

**2. Proceed now on all 25 samples, or wait for the RA-only subset? — Status: ✅ RESOLVED 2026-09-09.**
**Decision: proceed now on all 25 samples**, labeled generically as "inflammatory arthritis vs. OA."
Made explicitly after the AMP2 detour (a real attempt to get the cleaner "wait" option without the
time cost) ran into its own access gate. Accepted trade-off, not a default: PsA/SpA/UA dilute any
RA-specific signal, so this phase's claims are weaker than "RA vs. OA" — this should be stated
plainly in the eventual findings write-up's Limitations section, not glossed over.

**3. RA ground-truth paper — Status: ✅ DONE 2026-09-10 — both papers actually read (real fetches,
not recalled), real marker gene panels extracted, and successfully applied (see §7 item 6).**
Kuo et al.'s own findings remain circular (same data being analyzed), so not used. Two genuinely
independent papers read in full and verified:
- **Zhang et al. 2019**, *Nat Immunol* (PMC6602051, DOI 10.1038/s41590-019-0378-1) — real
  cohort/data independent of `E-MTAB-11791`/`GSE283080`. Four fibroblast subtypes with real,
  extracted marker genes: **SC-F1** (CD34+ sublining): CD34. **SC-F2** (HLA-DRAhi sublining):
  HLA-DRA, IFI30, IL6, CXCL12. **SC-F3** (DKK3+ sublining): DKK3, CADM1, COL8A2. **SC-F4** (CD55+
  lining): CD55, PRG4 — plus HBEGF/CLIC5/HTRA4/DNASE1L3 reported by the paper itself as higher in
  OA than leukocyte-rich RA within this same lining population, a directly relevant cross-check.
- **Croft et al. 2019**, *Nature* 570:246–251 (PMC6690841) — mouse single-cell data (F1–F5
  fibroblast clusters) explicitly validated against human sorted populations in the same paper.
  THY1 discriminates sublining (F1–F4, THY1+: PDPN, FAP, THY1) from lining (F5, THY1−: PDPN, FAP,
  PRG4, CLIC5, TSPAN15).

**Applied successfully, 2026-09-10:** used both papers' real marker panels to score RA's own
stromal subclusters (§7 item 6) — 24/28 subclusters clearly matched a specific published substate.
This is the actual ground-truth comparison this item always needed, done with real data now, not
deferred further.

**4. ISEF compliance check for `E-MTAB-11791`/`GSE283080` — Status: ✅ DONE 2026-09-09.**
Checked directly against the current Human Participants and Tissue & Body Fluid rule text (same
framework as Phase 1's §3a, and the same rule text checked for AMP2 during the detour). Both
datasets are public/open-access — no account, no quiz, no DUC/DAC of any kind, the cleanest access
tier of any dataset this project has used. Clears both exemptions on the publicly-available-database
clause alone (and, incidentally, `E-MTAB-11791` also clears on the peer-reviewed-journal clause,
same as AMP2 did). Standing caveat, same as Phase 1's §3a: this project's own reading of the rule
text, not an official SRC ruling — worth a direct mentor/SRC confirmation before treating as final.

**5. Cross-study batch effect, RA vs. OA — Status: ✅ RESOLVED 2026-09-09 — investigated, mostly
explained, no code change.** Checked directly via `compute_cluster_composition` before and after
Harmony, then investigated what the still-skewed post-Harmony clusters actually contain (cell
type, sample-level breakdown, OA subtype) — the same diagnostic Phase 1 used for its own batch
effect (`docs/phase1_full_story.md`, Step 2), not a new method invented for RA.

**Deliberately did NOT tune Harmony's parameters (`theta`, etc.) to force tighter mixing** — doing
that specifically for RA would break Objective 1's core premise ("replay the full pipeline...
using Phase 1's code unmodified"), turning a cross-disease comparison into a per-disease-tuned one.
Flagged explicitly by the user before any investigation started.

**Findings, cluster by cluster (of the ones that stayed skewed post-Harmony):**
- **Cluster 24** (673 cells, 63% OA, ~88% Plasma cells): OA has ~8× RA's plasma-cell proportion
  (1.5% vs 0.17% of each dataset). **Same phenomenon as Phase 1's own accepted precedent**
  ("Sjögren's has ~100× more plasma cells than MG") — a real population-size difference between
  diseases, not a batch artifact.
- **Clusters 3, 7, 23** (10,611 / 7,444 / 724 cells, Fibroblast- or Macrophage-dominated,
  54–61% OA): each spread across **many samples from both RA and OA**, not dominated by one or two
  samples — the signature of real disease-associated biology (an OA-enriched cell state), not an
  uncorrected technical effect.
- **Cluster 31** (170 cells, 169 OA/1 RA, Macrophages): spread across **all 4** OA samples — a
  small but plausible real OA-specific macrophage state.
- **Cluster 33** (27 cells, all OA): **81% from one single sample** (`GSM8655602_Infla2`) — this
  one genuinely does look like a technical/sample-specific artifact. Small enough (0.02% of all
  cells) to be irrelevant to any downstream analysis; noted, not chased further.

**Conclusion:** the residual batch effect is mostly legitimate cross-disease biology plus one
negligible-scale artifact, matching Phase 1's own experience closely enough that no special
handling is needed. Proceed to Step 3/4 on `data/interim/ra_oa_harmonized.h5ad` as-is.

**6. CellTypist fit for RA's fibroblast-like synoviocytes — Status: ✅ RESOLVED 2026-09-10 — the
label WAS too coarse, and subclustering recovers real, literature-matched substructure.**
Checked directly (`scripts/run_phase2_ra_fibroblast_subtyping.py`): Leiden-subclustered RA's
stromal compartment (40,271 cells → 28 subclusters, `stromal/subtyping.py`'s reusable machinery,
unmodified — the same tool built for Phase 1's Nayar-et-al check) and scored each subcluster
against real published RA fibroblast marker sets (Zhang et al. 2019's four subtypes SC-F1
CD34+/SC-F2 HLA-DRAhi/SC-F3 DKK3+/SC-F4 CD55+, and Croft et al. 2019's THY1+/− lining-sublining
axis — genes verified directly against both papers' real text, not recalled from memory, see §7
item 3's ground-truth citations). **24 of 28 subclusters (86%) clearly matched a specific
published substate** (outlier gap ≥ 0.1): 7 clusters → SC-F1 (CD34+ sublining), 4 → SC-F2
(HLA-DRAhi sublining), 3 → SC-F3 (DKK3+ sublining), 8 → SC-F4 (CD55+ lining), 1 → Croft's
THY1+ sublining, 1 → Croft's THY1− lining; only 4 unassigned. **Real, clean finding, no
cross-study batch-effect confound at all** (this is entirely within RA's own data, no OA/tonsil
comparison involved) — `Immune_All_High`'s broad `Fibroblasts` label was genuinely hiding
literature-recognizable RA fibroblast heterogeneity, exactly as Phase 1's risk table anticipated.
Full results: `data/processed/phase2_fibroblast_subcluster_marker_scores.csv`,
`phase2_fibroblast_subcluster_dominant_substate.csv`.

**7. Lupus nephritis timing — Status: 🟡 OPEN, gated on RA's real pace.**
Real deadline context: submission 2026-10-03, but the user's exams start 2026-10-01 (five exams,
four of five prepped as of 2026-09-08) — true project cutoff is closer to ~Sep 27–28, not the
literal submission date (see §5). Decision point ~Sep 12–13, once RA's actual pace past Step 1 is
known — the 2026-09-09 pivot-then-revert (a second real, unplanned research detour, this one
resolved same-day) is itself a data point for that checkpoint, worth remembering when it comes.

**Sub-lead checked 2026-09-12: user proposed dropping lupus nephritis and switching the third
disease to the Smillie et al. 2019 ulcerative-colitis atlas (Broad Single Cell Portal `SCP259`),
sourced via CELLxGENE's `.h5ad` instead of SCP directly, specifically to route around SCP's own
access gate. Checked directly against both platforms' real metadata — the CELLxGENE route does
NOT work for this:**
- CELLxGENE's copy of this exact study (collection `33d19f34-87f5-455b-8ca5-9023a2e5453d`, DOI
  `10.1016/j.cell.2019.06.029` — confirmed same paper as `SCP259`) is genuinely open, no
  sign-in (`https://datasets.cellxgene.cziscience.com/774d77e7-210d-4237-8ca8-8d2bb841a2e4.h5ad`,
  a direct public URL) — but its own curated metadata shows **only 34,772 cells, disease field =
  `normal` only, tissue = colon/caecum epithelium only.** That is a small epithelial-only slice of
  the real atlas (366,650 cells, 18 UC patients + 12 healthy, all compartments per the paper's own
  abstract) — **no UC-patient cells, no stromal/immune compartment, at all in this file.** CELLxGENE
  curation frequently republishes only a subset of a study's original deposit; this is exactly that
  case, not a partial-download artifact.
- Broad's own `SCP259` page confirms the real full dataset (365,492 cells, 30 individuals, both UC
  and healthy) lives there, but its download page states plainly **"Please sign in to
  download data"** — a sign-in requirement, which is exactly the class of gate the user's own
  2026-09-11 hard constraint (zero-registration, no sign-in of any kind, stated earlier in this
  section) rules out. Same conclusion as the AMP2/dbGaP/Synapse gates already ruled out elsewhere in
  this doc, just a lighter-weight version of the same gate.
- **Net finding: neither route gets a usable UC dataset under this project's own access rule** — the
  open copy has no disease cells, and the copy with disease cells isn't open. Not recommending
  further pursuit of `SCP259`/Smillie et al. specifically; a genuinely open GEO-hosted UC dataset
  would need to be found fresh if UC is still wanted as the third disease.
- **Incidental find, worth recording even though off-target:** CELLxGENE also hosts Perez et al.
  2022 (*Science*, DOI `10.1126/science.abf1970`) — 1.26M PBMCs, 162 SLE cases + 99 controls,
  genuinely open, no sign-in, real case/control lupus data. **Not a fit for this project's design**
  despite solving the "can't find open lupus data" problem in isolation — it's peripheral blood, not
  a target organ, so it has no organ-resident TLS/stromal biology for Steps 3-6 to operate on the
  way thymus/salivary-gland/synovium tissue does. Kept here as a data point, not a recommendation to
  pivot lupus nephritis to lupus PBMC.

**8. AMP2 acquisition mechanics — Status: ✅ CLOSED 2026-09-09 (abandoned, not completed).**
Was open while AMP2 was the leading option; moot now that Step 1 reverted to `E-MTAB-11791`/
`GSE283080`. Kept here, not deleted, as the record of why: the specific files needed
(`AMP-RA.SLE_clinical.csv`, `metadata_clin_donor_singlecell.xlsx`) turned out to require a
`ManagedACTAccessRequirement` (Data Use Certificate + Intended Data Use statement, reviewed by
Synapse's Access and Compliance Team) — see Step 1's "AMP2 detour" for the full finding. Worth
revisiting only if this phase's approach changes again.

**9. OA's "significant" TLS-signature enrichment vs. tonsil — Status: ✅ RESOLVED 2026-09-10, was a
cell-level artifact, not real.** Original flag: OA TLS-region cells (n=97) scored significantly
higher than tonsil's reference at the cell level (p=0.00008, BH p=0.00016) while RA's 1,140
TLS-region cells did not (p=0.9999) — backwards from what Objective 2 predicts, and never accepted
at face value.

**Checked directly, 2026-09-10** (`scripts/run_phase2_ra_step4_oa_tonsil_patient_level.py`):
applied the same rigorous patient-level exact permutation test RA already got (4 OA patients vs.
4 tonsil donors, 70 exact permutations — RA's own analogous test used 23 individuals vs. 4 donors,
17,550 permutations). **Real bug hit and fixed along the way, stated plainly:** the first run
crashed with `KeyError: 'is_gc_b_cell'` — the script never ran the GC-B-cell/Tfh cell-state scoring
(`score_cell_state`/`call_cell_state` against `GC_B_CELL_MARKERS`/`TFH_MARKERS`) on OA before
calling `label_tls_region_cells`, which needs those columns already present. Fixed by adding the
same two scoring calls used for RA, re-run clean. **Result: observed diff +0.0188, p = 0.129 — not
significant.** The
cell-level "confirmed" result does not survive patient-level scrutiny, exactly the same failure
mode Step 5 (§7 item 12) independently found in a completely different analysis. **Real
conclusion: neither RA nor OA shows genuine TLS-signature enrichment over tonsil once tested
properly — the "backwards" asymmetry was never a real biological surprise, it was two different
analyses hitting the same cell-level cross-study artifact.** This also means Step 4's headline
cross-disease table (`phase2_step4_cross_disease_results.csv`) should be read as: OA's "confirmed"
row is superseded by this patient-level test and should not be cited as a positive finding.

**10. OA's cell composition and doublet rate are both anomalous relative to RA — Status: 🟡 OPEN,
partially diagnosed 2026-09-10, root cause understood for the doublet half, cell-composition half
still unexplained.**

**Doublet-rate half — checked directly, real finding.** Inspected RA and OA's Scrublet score
distributions and pooled thresholds directly (not just the flagged rates): RA's pooled threshold
(0.3908) sits far out past its own real-cell 99th percentile (0.1609) — the KDE valley-finding
landed near the extreme tail rather than a genuine bimodal split. OA's threshold (0.1956) sits
between its 95th/99th percentiles (0.1468/0.3283) — a normal-looking fit. **5 of RA's 25 samples
got zero flagged doublets at all** — the same class of failure `doublets.py`'s own docstring
documents Phase 1 hitting once already (6/12 MG samples, fixed by switching to the pooled-threshold
method) — recurring here through the pooled method itself, not the per-sample method it replaced.
**Deliberately not silently patched with an RA-specific parameter tweak** — same principle as the
Harmony-tuning question earlier in this project: changing a QC knob specifically because RA's
result looks inconvenient would be exactly the kind of per-disease cherry-picking this project
avoids. Documented as a known, real QC limitation instead. Practical impact is likely small
(probably a few hundred to low-thousands of missed doublets out of 102,022 cells) relative to the
much larger cross-study batch effect already dominating Steps 4–5's problems, but it's a real,
stated limitation for the eventual write-up, not swept away.

**Cell-composition half — still open.** OA's cell-type mix differs sharply from RA's: 49.7%
Macrophages / 5.4% T cells (OA) vs. 30.9% Macrophages / 21.6% T cells (RA). Whether this reflects
real OA biology (plausible — OA is less classically an adaptive-immune/TLS disease than RA) or
residual cross-study technical difference remains unresolved. No longer directly relevant to
item 9 (now resolved — see above), but still relevant to interpreting Step 2's cell-type-proportion
numbers honestly.

**11. Does resolving the RA-only subset change Step 4's (and later steps') results? — Status: 🔴
OPEN, deliberately deferred, explicit user decision 2026-09-10.** The current negative RA-vs-tonsil
result (item above) is most plausibly explained by the all-25-mixed-diagnosis-samples decision
(§7's former item 2) diluting a real RA-specific signal — but this is a hypothesis, not confirmed.
**Explicit decision: finish Steps 5–7 first on the current data, then circle back and check whether
narrowing to the true RA-only subset (if the diagnosis mapping ever resolves — see the dormant
email-to-Kuo-et-al.-corresponding-author lead, formerly §7 item 1) changes any of the downstream
findings**, rather than context-switching mid-pipeline now. This item is the anchor to come back to
— don't let it go stale the way other status banners in this project have (already caught and fixed
twice, 2026-08-31 and 2026-09-08).

**12. Step 5's 15-gene RA-vs-OA marker list is likely contaminated by the uncorrected cross-study
batch effect — Status: ✅ RESOLVED 2026-09-10 — real fix applied, honest null result, see Step 5
for the full pseudobulk write-up.** Same root cause as items 9/10, now showing up a third time, in a
different analysis. Original evidence: 10 of 15 "confirmed" genes hit the mathematically maximum
possible Mann-Whitney U statistic (complete separation, 23 RA individuals vs. 4 OA patients); the
list contained no canonical RA fibroblast markers (THY1/CD34/PRG4/FAP/PDPN) but did contain FOSB, a
documented dissociation-protocol stress-response artifact gene (van den Brink et al. 2017,
*Nat Methods*).

**Tested directly, 2026-09-10 (`scripts/run_phase2_ra_step5_combat.py`): applied ComBat batch
correction (`batch_key='dataset'`) to expression before the TLS-score regression, plus an extended
artifact filter covering the FOS/JUN/EGR1/HSP immediate-early gene family.** Result:
**0 of the original 15 genes survive — complete turnover, the strongest possible confirmation that
the original list was pure batch artifact, not RA biology.**

**But the replacement list is itself untrustworthy, a new and separate problem.** ComBat logged
"Found 34 genes with zero variance," and its new top-20 genes (CELA1, OR2C3, TCN1, CALCR, AQP6,
SHISA9, ...) have no plausible connection to synovial biology, near-zero expression values
(~1e-6–1e-4), and none reach significance at patient level either (p 0.06–1.0). **Root cause,
understood, not a bug to chase further: ComBat assumes roughly-Gaussian per-gene batch effects, an
assumption that breaks on sparse single-cell data** — it manufactures apparent signal out of
correction noise on near-zero-count genes rather than correcting real ones. This is a known,
documented limitation of ComBat specifically for scRNA-seq, not something wrong with this
pairing's data.

**Resolution, 2026-09-10: pseudobulk selection + pseudobulk ComBat built and run.** First validated
against trusted data (the Sjögren's-reproduction check below in this item's history — confirmed the
patient-level math exactly reproduces Phase 1's own PIP/LYZ numbers), then applied to RA/OA: 0/15
shortlisted genes confirmed, but this time with U-statistics spread reasonably (27–51 of a possible
0–92) and raw p-values 0.22–0.92 — an honest null, not the artifact signature (exact-maximum U,
implausible gene identities) both earlier attempts showed. **Most likely a genuine power problem**
(n_oa=4 patients) compounded by RA's mixed-diagnosis dilution, not evidence RA lacks a real
disease-specific stromal signature. See Step 5 above for the full write-up and file list.
**Do not carry any of the three gene lists (original, cell-level-ComBat, or the pseudobulk
shortlist) into Step 6 as a confirmed finding** — none reached a trustworthy positive result;
Step 6 needs to either wait for a cleaner RA-only subset (§7 item 11) or proceed with this
honestly-null Step 5 result disclosed as a limitation.

---

## Extra analysis (2026-09-11, explicitly NOT a Step 5 substitute): intra-RA lining vs. sublining fibroblasts

**Why this exists, and why it's kept separate from Step 5 above.** After Step 5's honest null, a
mentor-proposed pivot suggested comparing "pathogenic" vs. "bystander" RA fibroblast subclusters
within RA only, sidestepping OA's n=4 entirely. Reviewed directly before building anything: the
"pathogenic vs. bystander" framing wasn't supported by the literature as stated (conflated two
papers' independent axes, asserted pathogenicity from marker identity alone), and the proposed
`groupby='leiden_subcluster'` + `rank_genes_groups` implementation would have been cell-level, not
patient-level — reintroducing pseudoreplication this project has already fixed twice. **Explicit
user decision, 2026-09-11: keep Step 5's RA-vs-OA result as the primary, plan-consistent finding;
build this as a clearly-labeled additional analysis answering a different question (a druggable
target within RA), not a replacement.** Objective 3 (RA residual vs. Sjögren's ISG signature) still
needs an actual RA-vs-comparator residual, which this analysis does not produce.

**Design, built to avoid the specific problems found above:**
1. **Lining vs. sublining, not pathogenic vs. bystander** — anchored to Croft et al. 2019's real
   functional evidence (adoptive-transfer experiments: THY1− lining fibroblasts selectively
   mediate bone/cartilage damage; THY1+ sublining fibroblasts drive inflammation), not asserted.
   Lining group = clusters matching `zhang_SC_F4_CD55_lining` or `croft_lining_THY1neg` (§7 item 6's
   subtyping). Sublining group = `zhang_SC_F1/F2/F3` or `croft_sublining_THY1pos`. Unassigned
   clusters excluded.
2. **Paired, not unpaired, test.** Every RA patient can contribute both lining and sublining
   cells — a real paired design (Wilcoxon signed-rank on per-patient group means), more powerful
   than an unpaired test at the same n, and structurally immune to the OA-comparator power problem
   since OA is never involved.
3. **No cell-level pre-screen** — tests every sufficiently-expressed gene directly at the paired
   patient level, BH-corrected across all of them, rather than shortlisting via `rank_genes_groups`
   first (the exact step that was circular in the mentor's original proposal).

**Result: 23/23 RA individuals qualified** (≥10 cells in both groups — the full cohort, no
exclusions). **5,610 of 8,889 tested genes significant after BH correction.** Marker-panel sanity
check (genes used to define lining/sublining, excluded from the candidate list) matched the
literature cleanly: PDPN/FAP/PRG4/COL8A2/CD55/THY1/TSPAN15/CLIC5 up in lining; IL6/CXCL12/CADM1/
HLA-DRA/CD34 up in sublining (DKK3 was a minor, small-magnitude exception — Zhang describes it as
sublining, here it's marginally higher in lining, 0.255 vs. 0.204).

**Real second circularity check, run before trusting the count — this is genuinely different from
the mentor's proposal's cell-level circularity, and only partially mitigated by it:** the
lining/sublining *grouping itself* came from Leiden clustering on 2,000 HVGs computed from this
same expression data — so even without cell-level `rank_genes_groups`, testing genes correlated
with those same HVGs against clusters built from them is still not fully independent.
- Of the 5,610 significant genes, 613 were literally in the 2,000-gene HVG set used for
  clustering — removed.
- Of the remaining 4,997, many top hits were ribosomal genes (RPL30, RPS28, RPS20, RPL7, RPL36) —
  **the same technical-artifact family already filtered in every Step 5 script, but omitted from
  this new script's first draft.** Caught and fixed: extended the same `TECHNICAL_ARTIFACT_PREFIXES`
  filter here. 90 more genes removed.
- **Result after both filters: 4,907 genes still significant.** This is not itself evidence of a
  remaining artifact — lining and sublining are genuinely, robustly distinct fibroblast programs
  (independently established by Croft/Zhang), so a large, real transcriptional difference between
  them, with strong within-patient consistency (n=23/23), is biologically plausible, not
  suspicious the way the RA-vs-OA maximum-U pattern was.

**Honest conclusion: BH-significance alone is no longer a selective filter here — 4,907 genes is
too many to treat as a discrete "druggable target list."** This validates that the pipeline
correctly recovers a real, strong, literature-consistent biological axis (useful on its own), but
**picking actual drug-target candidates needs a stronger secondary filter — effect size, exclusion
from any HVG set used anywhere in this pipeline, and/or actual DGIdb/ChEMBL druggability (Step 6)
as the real differentiator — not p-value ranking, which is saturated.** Not yet done.

Full results: `data/processed/phase2_intra_ra_lining_vs_sublining.csv` (all 8,889 genes tested),
`phase2_intra_ra_lining_vs_sublining_non_hvg.csv` (HVG-filtered), `..._clean.csv` (HVG- and
artifact-filtered, the 4,907). Script: `scripts/run_phase2_ra_intra_lining_sublining.py`.

### Step 6 for this extra analysis — DGIdb/ChEMBL narrowing — Status: ✅ Run 2026-09-11

**Narrowing method.** All 4,907 clean candidates are already BH-significant (n=23 gives real
power) — p-value has no discriminating power left among them, same problem flagged above. Used
effect size instead: `log2((lining_median + 0.01) / (sublining_median + 0.01))`, distribution
inspected (range 0.006–3.5, long right tail), threshold set at `|log2fc| > 2` (≥4-fold
per-patient-mean difference) — 68 / 4,907 genes clear it. Queried DGIdb (one batched GraphQL call)
and ChEMBL (per-gene target→mechanism→molecule chain) for all 68, same `pharma/` module and
`combine_and_rank` Phase 1's Step 6 used.

**Result: 14 / 68 genes had any DGIdb/ChEMBL hit at all (54 had none — expected, most human genes
have no listed drug). Only 2 gene-drug pairs are FDA-approved (max_phase = 4).** Most of the
79 raw pairs found are noise, not real candidates — endogenous ligands DGIdb catalogs for receptor
genes (C5AR2's own ligand C5a; FZD4's ligand Norrin), uncharacterized research compounds (NB23-BI,
BAY-069, FZM1), or literal carbohydrates DGIdb lists as galectin-binding partners (LGALS9 →
"lactose, anhydrous") — not therapeutics. Filtering to what's actually a real drug:

- **MAPK11 (p38β MAPK), up in sublining (log2fc −3.19, lining median 0.007 vs. sublining 0.145)**
  — hits regorafenib (FDA-approved, max_phase 4, multi-kinase inhibitor incl. p38β), losmapimod
  (max_phase 3, selective p38 inhibitor), RWJ-67657 (max_phase 1). Mechanistically coherent —
  p38 MAPK is textbook RA synovial-fibroblast inflammatory signaling — **but flagged honestly**:
  losmapimod and other selective p38 inhibitors have a real history of *failing* RA trials
  (tested for RA in the 2000s–2010s, discontinued for lack of efficacy/tolerability; losmapimod's
  later approvals pursued COPD/cardiovascular, not RA) — worth stating in any write-up as a known
  caveat, not just listing the hit.
- **LRRC32 (GARP), up in sublining (log2fc −3.02)** — hits livmoniplimab, a clinical-stage
  anti-GARP:TGFβ1 antibody. Plausible: GARP-presented latent TGFβ activation fits sublining
  fibroblasts' known inflammatory/TGFβ-driven program.
- **CD79B, up in sublining (log2fc −2.66) — flagged as a likely contamination artifact, not a
  fibroblast finding.** Its one approved-drug hit (polatuzumab vedotin) is an anti-CD79b
  antibody-drug conjugate for B-cell lymphoma — but CD79B is a B-cell-receptor signaling
  component with no established fibroblast biology. RA sublining tissue is exactly where ectopic
  lymphoid aggregates/TLS form (the whole basis of this project), so CD79B turning up "in
  sublining fibroblasts" most plausibly reflects ambient RNA / spatial contamination from
  genuinely adjacent B cells, not fibroblast-intrinsic expression — same category of risk as the
  IGKC/IGHG immunoglobulin artifacts already filtered out of Step 5, just not caught by the
  RPS/RPL/IG prefix filter since CD79B doesn't match those prefixes. **Do not present this as a
  drug-repurposing finding.**
- **Lining side produced no clean approved-drug hits** — its two hits (ADGRG2 → NB23-BI, SIX3 →
  BAY-069) are uncharacterized probe compounds, and PDGFC → sunitinib is a broad multi-target
  kinase inhibitor (VEGFR/PDGFR), not a targeted mechanism.

**Bottom line: one real, defensible repurposing candidate for the sublining/inflammatory program
(MAPK11/p38β, with the trial-history caveat stated up front) plus one plausible secondary
(LRRC32/GARP); nothing clean for the lining/destructive program.** This is a much smaller, more
honest result than the raw 79-pair table would suggest — reporting the full table without this
filtering would overstate the finding.

Full results: `data/processed/phase2_intra_ra_step6_dgidb_raw.csv`,
`phase2_intra_ra_step6_chembl_raw.csv`, `phase2_intra_ra_step6_ranked.csv`. Script:
`scripts/run_phase2_ra_intra_step6_pharma.py`.
