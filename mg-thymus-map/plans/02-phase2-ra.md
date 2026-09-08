# Phase 2 Plan — Rheumatoid Arthritis (Synovium) Extension

Status: **In progress.** Step 1 (data acquisition) has real code — dataset chosen, download
scripts written and run for real, loaders written and tested (160 tests project-wide, `make test`
green) — as of 2026-09-08. This is still a living document meant to be argued with and edited, the
same way Phase 1's plan doc was, not a finished spec. Sections marked **OPEN** need a real decision
before the next step can start cleanly; everything else is a reasonable default carried over from
Phase 1, subject to change. **One real blocker found while building Step 1, not yet resolved:** no
per-sample RA-specific diagnosis is recoverable from the data's own machine-readable metadata —
see the consolidated open questions at the end of this doc, item 1.
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

- [~] An RA synovium scRNA-seq (or spatial) dataset is downloaded, documented, and loads cleanly.
      **Downloaded and loadable as of 2026-09-08** (`load_ra_synovium`/`load_oa_synovium`, 10
      passing tests) — but "RA" is currently 25 samples of mixed inflammatory arthritis, not a
      confirmed RA-only subset (see the open questions). **ISEF-compliance check not yet done** —
      needs the same explicit two-exemption check §3a of the Phase 1 doc ran for every other
      dataset (both sources here are public/open, so it's expected to pass, but hasn't been
      formally checked and written down yet).
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

## 7. Open questions for Phase 2 (consolidated, ordered by urgency)

**1. RA-specific per-sample diagnosis — Status: 🔴 OPEN, blocks everything past Step 1, most urgent
item in this whole document.**
`E-MTAB-11791`'s machine-readable metadata (the SDRF file) does not distinguish RA from psoriatic
arthritis/spondylarthritis/undifferentiated arthritis at the per-sample level — checked directly,
confirmed uniformly `"arthritis"` for all 25 samples. The paper's Table 2 gives the aggregate split
(15 RA/3/4/3) but not the individual-to-diagnosis mapping. That mapping is almost certainly in a
supplementary table (Table S1/S2) that automated fetching couldn't reach this session (ScienceDirect
403'd a direct fetch; guessed PMC binary paths 404'd). **Concrete next action, needs the user:**
open the paper directly in a browser (`doi.org/10.1016/j.isci.2024.109707`, iScience/Cell Press,
genuinely open-access — the 403 was very likely a bot-blocking measure, not a real paywall) and
pull Table S1/S2, or email the corresponding author if the browser also can't reach it. Until this
resolves, `load_ra_synovium` labels every cell the honest generic `"inflammatory_arthritis"`, and
Step 2 onward has a real decision to make (see item 2 below).

**2. Proceed now on all 25 samples, or wait for the RA-only subset? — Status: 🔴 OPEN, decision
needed regardless of how item 1 resolves.**
Two real options, not obviously the same answer: (a) run Step 2+ now on all 25 samples labeled
generically as "inflammatory arthritis vs. OA" — gets moving immediately, but is a measurably
weaker, less precise claim than "RA vs. OA" (PsA/SpA/UA are biologically distinct diseases, not RA
subtypes, mixing them dilutes any RA-specific signal); or (b) wait until item 1 resolves, then run
on the true 15-RA subset only — the scientifically cleaner claim, matching what the plan's
Objectives actually promise, but blocks all downstream progress on item 1's timeline. **No
recommendation locked in yet — this needs the user's call**, ideally informed by how quickly item 1
looks resolvable.

**3. RA ground-truth paper — Status: 🟡 OPEN, candidates identified, not yet read in full.**
Zhang et al. 2019's reported CTAP findings (data inaccessible for direct reprocessing, but usable
as reported findings the way Nayar et al. was for Sjögren's), and/or Kuo et al.'s own E-MTAB-11791
paper's reported cell-state findings (natural first cross-check, since it's literally the same
data). Needs the same direct full read Nayar et al. got before being cited as ground truth.

**4. ISEF compliance check for the two new datasets — Status: 🟡 OPEN, expected to pass, not yet
formally done.**
Phase 1's plan doc has a dedicated §3a explicitly checking every dataset against Society for
Science's Human Participants and Tissue & Body Fluid exemptions, with direct citations to the rule
text. `E-MTAB-11791` and `GSE283080` are both public/open-access, so this is expected to clear
easily, but it hasn't been formally written up the way §3a does for Phase 1's four datasets — worth
doing before treating Step 1 as fully closed out, not just assumed fine because the data downloaded
without a login wall.

**5. Cross-study batch effect, RA vs. OA — Status: 🟡 OPEN, can't be checked until Step 2 runs.**
Flagged as a real risk in §4 Step 1 and the risk table — different institutions/cohorts (Kuo et
al.'s European/multi-site cohort vs. Miyahara et al.'s University of Tokyo cohort), despite both
using 10x Chromium. Phase 1's own batch-effect check (cluster by dataset-of-origin before and after
Harmony) is the reusable tool for this — apply it here before trusting any RA-vs-OA comparison, not
after.

**6. CellTypist fit for RA's fibroblast-like synoviocytes — Status: 🟡 OPEN, can't be checked until
Step 2 runs.**
Flagged in §4 Step 2 and the risk table. Phase 1's own Leiden-subclustering rescue approach
(`stromal/subtyping.py`, already built and reusable) is the fallback if `Immune_All_High`'s broad
`Fibroblasts` label proves too coarse for RA's specific FLS subtypes.

**7. Lupus nephritis timing — Status: 🟡 OPEN, gated on RA's real pace.**
Real deadline context: submission 2026-10-03, but the user's exams start 2026-10-01 (five exams,
four of five prepped as of 2026-09-08) — true project cutoff is closer to ~Sep 27–28, not the
literal submission date (see §5). Decision point ~Sep 12–13, once RA's actual pace past Step 1 is
known — item 1 above (a real, unplanned research blocker hit on Day 1 of Phase 2) is itself a data
point for that checkpoint, worth remembering when it comes.
