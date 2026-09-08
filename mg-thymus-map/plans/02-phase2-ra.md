# Phase 2 Plan — Rheumatoid Arthritis (Synovium) Extension

Status: **Draft — planning stage, not started.** This is a first pass meant to be argued with and
edited, the same way Phase 1's plan doc was — not a finished spec. Sections marked **OPEN** need a
real decision before Step 1 can start; everything else is a reasonable default carried over from
Phase 1, subject to change.
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

- [ ] An RA synovium scRNA-seq (or spatial) dataset is downloaded, documented, ISEF-compliance-
      checked (same two-exemption check as §3a of the Phase 1 doc), and loads cleanly — **blocked
      on §7 Q1**.
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

### Step 1 — Data acquisition — Status: 🟡 Dataset chosen (2026-09-08), download/loader not yet built

**Goal:** get one RA synovium dataset into the same loadable, documented state Phase 1's three
datasets are in.

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

**✅ RESOLVED, 2026-09-08 — dataset decision made.**

**Primary dataset: `E-MTAB-11791`** (ArrayExpress; partial GEO mirror `GSE181082`) — Kuo et al.,
"Molecular maps of synovial cells in inflammatory arthritis using an optimized synovial tissue
dissociation protocol." 25 fresh synovial tissue biopsies, 23 patients, **~102,758 cells**, real
10x Genomics Chromium (v3.0/v3.1), Cell Ranger v6.0.0, genuinely open access (no DAC). **Real
per-diagnosis breakdown, checked directly (not the abstract-level summary): 15 RA, 3 psoriatic
arthritis, 4 spondyloarthritis, 3 undifferentiated arthritis** — a genuinely solid RA sample size,
larger than Sjögren's own 7 PSS patients. No OA/healthy arm in this study.

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

### Step 2 — QC, normalization, annotation — Status: reused code, fresh data

Same functions, same conventions (adaptive mito%, pooled doublet threshold, CellTypist +
`majority_voting=True`, Harmony via direct `run_harmony()` call). New judgment call: audit whether
CellTypist's broad `Fibroblasts` label is doing anything meaningful for RA's fibroblast-like
synoviocytes (FLS) — Phase 1 found this workable for Sjögren's but never tested against a synovium
context specifically.

### Step 3 — TLS signature — Status: not re-derived, reused as-is

The 25-gene MG signature is not touched. Re-deriving it from RA would defeat the point.

### Step 4 — Cross-disease projection — Status: reused code, fresh run

AUCell-score RA against the MG signature, same tonsil-calibrated threshold. **Apply patient-level
testing to every claim before it is ever reported**, not as a retrofit — Phase 1 only adopted this
discipline after item 5's retraction; Phase 2 should start with it.

### Step 5 — Stromal extraction — Status: reused code, real fresh research needed

RA vs. OA (or whatever comparator Step 1 lands on), regress out shared signature, rank markers,
patient-level confirm. **Needs an RA-specific ground-truth paper** — the Nayar-et-al analog.
Candidates to check: Zhang et al. 2019 itself (even if its raw data isn't usable, its *reported
findings* may be, the same way Nayar et al.'s reported findings were used when GSE272409 had no
spatial deposit), or newer work (a 2023 *Nature* paper on RA synovium inflammatory subtypes turned
up in this session's search, not yet read in full). **Objective 2's comparison** (RA residual vs.
Sjögren's ISG signature) happens here.

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

## 6. Risks & mitigations (RA-specific, in addition to everything already in Phase 1's §6)

| Risk | Mitigation |
|---|---|
| No clean GEO-hosted RA-vs-OA whole-tissue droplet dataset actually exists at the quality Phase 1 had | Step 1 must be allowed to fail fast and pivot (e.g., to a different comparator, or RA-only with a different reference) rather than force-fit a bad dataset to protect the schedule |
| Two-new-diseases-at-once time pressure degrades statistical rigor on either RA or lupus | The Sep 12–13 checkpoint (§5) exists specifically to catch this before it happens, not after |
| CellTypist's generic immune model doesn't meaningfully resolve RA's fibroblast-like synoviocyte subtypes | Flagged explicitly in Step 2 as something to check, not assume; Step 5's Leiden-subclustering rescue approach (already built, reusable) is the fallback if the broad label proves too coarse |

## 7. Open questions for Phase 2

1. **Which RA dataset?** — Status: **✅ Resolved 2026-09-08**
   `E-MTAB-11791` (RA, 15 patients) + `GSE283080` (OA comparator, 4 patients). See §4 Step 1 above
   for the full verification trail — 9 real GEO/ArrayExpress/ImmPort records were checked directly
   before landing here, several search-summary claims turned out to be wrong on inspection.

2. **RA's comparator** — Status: **✅ Resolved 2026-09-08, as a cross-study pairing (not same-study)**
   Is OA (osteoarthritis) synovium the right comparator (same-study, same protocol, differs mainly
   in disease status — the SICCA pattern), or does the eventual dataset choice force a different
   option (e.g., a cross-study normal-joint reference, weaker methodologically)?

3. **RA ground-truth paper** — Status: **OPEN, leading candidates identified**
   Zhang et al. 2019's reported CTAP findings (data inaccessible for direct reprocessing, but the
   paper's own conclusions are usable the way Nayar et al.'s were for Sjögren's), and/or Kuo et
   al.'s own E-MTAB-11791 paper's reported findings (since we're using their actual data, their
   paper's own cell-state calls are a natural first cross-check). Needs the same kind of direct
   read Nayar et al. got before being cited.

4. **Lupus nephritis timing** — Status: **OPEN**, see §5
   Gated on RA's real pace, decision point ~Sep 12–13.
