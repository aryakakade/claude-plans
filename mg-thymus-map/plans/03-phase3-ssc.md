# Phase 3 Plan — Systemic Sclerosis (SSc) Extension (history: lupus nephritis → ulcerative colitis → SSc, see §4-§5)

**2026-09-14 update — file renamed `03-phase3-lupus.md` → `03-phase3-ssc.md`, retitled from SSc's
own dataset (GSE138669), not lupus nephritis or UC.** Real status as of this rename: SSc has real
completed results through Step 6 (Steps 1, 2, 4, 5, 6 all done — see §5/§6/§7 below), the strongest
disease-specific finding in Phase 2 or 3 so far (a 14-gene confirmed fibroblast signature + one
real pharma candidate, brensocatib). Lupus nephritis was superseded (§4), and UC (`SCP259`) is
still blocked on the Terra 18+ age gate as of this note — SSc became the active Phase 3 disease by
simply being the one that could proceed without waiting on that. **2026-09-12 update — disease
pivoted from lupus nephritis to ulcerative colitis.** See §4 below. Kept the original title and all
lupus-nephritis research in place (nothing deleted), same historical-record convention
`02-phase2-ra.md` used for its own AMP2 detour — this file is now Phase 3's plan doc for whichever
disease it ends up covering, not exclusively a lupus doc.

**Session: "RA Batch-Effect Marathon"** (`claude.ai/code/session_01UaUH4B9As1iwjy8Yt9YpWF`, Claude
Sonnet 5, 2026-09-11) — same session as Phase 2's dataset-search work; see `02-phase2-ra.md`'s
header for the full session scope.

Status: **Not started — dataset search only, 2026-09-11.** No code has been written for Phase 3.
This document exists because real, substantive dataset research was done tonight (prompted by the
user asking whether to freeze Phase 2 and pivot to Phase 3), and per this project's own convention
(every real finding gets written down, including negative ones), it's recorded here rather than
lost. **Explicit decision, 2026-09-11: do not start Phase 3 yet** — Phase 2 isn't green (§7 item 11
still open, the intra-RA extra analysis still needs a druggability-based follow-up filter), and
starting a third disease before the second is settled repeats exactly the risk Phase 2's own plan
doc §5 already named ("two-new-diseases-at-once time pressure degrades statistical rigor").
Parent plan: [`00-overview.md`](00-overview.md).

## 1. Why lupus nephritis

Per `00-overview.md`'s roadmap, lupus nephritis was always the planned third disease for testing
whether the MG-derived TLS signature generalizes beyond Sjögren's (Phase 1, confirmed) and RA
(Phase 2, inconclusive at current data resolution). Kidney is a structurally different target organ
from thymus/salivary-gland/synovium, and lupus nephritis's tubulointerstitial fibrosis has real,
independently-published fibroblast/myofibroblast biology to check the stromal target list against
— the same kind of ground-truth comparator Phase 1 (Nayar et al.) and Phase 2 (Zhang/Croft et al.)
each used.

## 2. Dataset search, 2026-09-11 — two candidates checked directly, neither yet usable

**Candidate 1: ImmPort SDY997** (Arazi et al. 2019, *Nat Immunol*, "The immune cell landscape in
kidneys of patients with lupus nephritis," PMC6726437) — the user initially asked about this under
the mistyped accession "SDY99"; verified the real one directly against the paper, not assumed.

**Checked directly, ruled out:**
- **24 lupus nephritis (LN) patients + 10 healthy-control kidney biopsies** (living donor) — a
  real same-study disease-vs-control design, no cross-study batch-effect risk. Genuinely good on
  this one axis.
- **But only 2,736 leukocytes + 145 epithelial cells = 2,881 total kidney cells** — smaller than
  even this project's smallest existing arm (OA's 28,041).
- **Wrong compartment**: 90% of sorted cells were CD45+ (immune), 10% CD45−CD10+. No meaningful
  stromal/fibroblast population reported at all — the same structural problem Phase 1 already
  documented for MG's own CD45+-sorted data (`GSE233180`). A Step-5-style stromal extraction has
  essentially nothing to work with here.
- **Technology mismatch**: most samples used CEL-Seq2 (plate-based, low-throughput); only 2
  control samples used 10x Chromium — inconsistent within its own dataset and a different
  platform than every other dataset this project has used.
- **Access not confirmed even for what little is usable**: paper states raw data is on dbGaP
  (`phs001457.v1.p1`, controlled) and clinical/serological data on ImmPort (SDY997) — does not
  clearly state the processed count matrix itself is openly downloadable, only that an
  "interactive browser" exists for viewing.
- **Verdict: not usable.** Real numbers, not assumption — see chat log 2026-09-11 for the full
  PMC-sourced quotes.

**Candidate 2: Synapse `syn64064827`** (Gurajala et al., *Sci Transl Med*, "A population-scale
transcriptional atlas of blood and tissue in lupus nephritis," DOI 10.1126/scitranslmed.aec0512)
— found via a downstream analysis paper (Raparia, Hoover et al. 2026, *Ann Rheum Dis*, "Spatially
distinct macrophage subsets drive myofibroblast heterogeneity and maladaptive fibrosis in lupus
nephritis," DOI 10.1016/j.ard.2026.07.014) that cited it as its primary data source (reference 15
in that paper) — checked back to the actual originating atlas paper, not used secondhand.

**Genuinely promising, real numbers:**
- **538,194 single-cell + 142,881 single-nucleus kidney profiles, 155 LN patients + 30 healthy
  preimplantation-transplant controls**, plus 327,326 single-cell blood profiles — a huge,
  well-powered, same-study cohort.
- **Confirmed whole-tissue with a real stromal compartment**, via the downstream myofibroblast
  paper built on this same data: 16,187 stromal cells — mesangial, fibroblasts, myofibroblasts,
  endothelial cells, and 2 VSMC subclusters, with 4 fibroblast + 2 myofibroblast + 10 VSMC + 5
  endothelial subclusters at high resolution. This is the AMP-SLE analog of what Zhang/Croft
  provided for RA — real published subtype structure to validate a subtyping check against, if
  Phase 3 ever starts.
- **Access — checked directly against Synapse's API, same method as AMP2 and confirmed genuinely
  stricter, 2026-09-11:**
  - `GET /entity/syn64064827/accessRequirement`: `{"totalNumberOfResults":0,"results":[]}` — no
    listed public access requirements, same as AMP2's RA data looked at first glance.
  - **But `GET /entity/syn64064827/permissions` (anonymous): `"canView": false`,
    `"canPublicRead": false`** — this entity isn't even anonymously *viewable*, let alone
    downloadable. AMP2's RA Dataset entity (`syn52297840`) was `canPublicRead: true` at this same
    check — this lupus dataset is a strictly tighter tier from the very first anonymous probe.
  - **Not yet resolved**: whether this becomes a self-service DUC (like RA's clinical files
    turned out to be) or something requiring institutional sponsorship needs a logged-in check —
    same division of labor already established for Synapse (user checks, this session doesn't
    touch login/session). Real risk, given the empty `accessRequirement` list combined with
    `canView: false` doesn't cleanly match either pattern seen so far this project (AMP2's RA data
    was publicly viewable with the gate only on specific child files; this is gated at the
    container level from the start).

## 2b. 2026-09-11, later same night — closed out against the new open-access constraint

After the user personally checked SCP2959 (Kong et al. Crohn's atlas, Broad Single Cell Portal)
and found it "requires organization email etc.", they set a firm, project-wide constraint: **only
genuinely open, zero-registration data — no DUC, no institutional email, no sign-in — because this
is an ISEF submission and extra forms aren't worth the risk.**

Re-checked `syn64064827` against that bar specifically: **fails outright.** The anonymous
`permissions` check above already found `canView: false, canPublicRead: false` — this is gated at
the container level from the very first look, not a self-service form like AMP2's RA clinical
files turned out to be. For a multi-consortium kidney-biopsy atlas at this scale, that pattern
almost always means institutional sponsorship or a full DUC review, not a quick click-through.
SDY997 was already ruled out on the merits (§2, Candidate 1) independent of this constraint.

**Verdict: no viable open-access lupus dataset found as of 2026-09-11, despite genuine search.**
Combined with five other disease areas checked the same night (RA's own diagnosis-mapping issues
aside — Graves', Hashimoto's, Crohn's/IBD, primary biliary cholangitis — each either
scientifically insufficient or gated in some way; full list in `02-phase2-ra.md`'s session header),
explicit recommendation given to the user: **stop the third-disease search and spend remaining
time tightening what's already real** (Phase 1 Sjögren's, confirmed; Phase 2 RA, honest null +
the intra-RA fibroblast/pharma finding) rather than rushing a third disease with a data-quality
asterisk under real deadline pressure. **Phase 3 lupus nephritis is, as of this note, not going
forward** unless a genuinely open dataset surfaces that clears this bar cleanly on the first check.

**Not fully closed**: the user raised one more disease candidate the same night (Smillie et al.
2019 ulcerative colitis atlas) immediately after this recommendation was given — being checked in
a forked sub-session started 2026-09-11. If it's genuinely open and sufficiently powered, it would
supersede this section's "stop the search" recommendation for lupus specifically (UC would become
the third disease, not lupus) — outcome not yet known at the time of this note.

## 2c. 2026-09-11, same night — UC candidate checked, ruled out on two independent grounds

Fork investigated the accession the user gave for this (`GSE114374`). Two separate problems found,
neither assumed — both confirmed directly:

1. **`GSE114374` is not the Smillie et al. paper at all.** It's Kinchen/Chen/Simmons et al. 2018
   (*Cell*), a different colonic-mesenchyme study — only 2 UC + 2 healthy human patients (plus
   mouse DSS-colitis samples), 10 samples total. **Same class of mistake as the earlier GSE181082
   mix-up this session already caught** (an accession that sounds/looks right but is a different
   paper) — worth remembering as a recurring failure mode when a dataset is named from memory
   rather than looked up directly.
2. **The real Smillie et al. 2019 paper (PMID 31348891, PMCID PMC6662628, *Cell*) has no GEO
   deposit at all** — confirmed via PubMed→GEO cross-link (empty) and a direct GEO search (zero
   hits). Its only home is the **Broad Single Cell Portal, `SCP259`**, confirmed live:
   "Please sign in to download data." **This is the exact same access tier (Broad SCP sign-in)
   the user already personally hit and rejected via SCP2959** ("requires organization email
   etc.") — fails the open-access constraint outright, for the identical reason.

**On the merits, moot given the gate, but worth recording**: the real dataset would have been
strong — 366,650 cells, 18 UC + 12 healthy donors, inflamed/non-inflamed/healthy tissue, 8
published fibroblast subtypes including WNT5B+ inflammation-associated fibroblasts (a real
stromal-subtype ground truth, the same kind Zhang/Croft provided for RA). **Ruled out on access
alone, not on scientific merit** — if the user's constraint ever relaxes, or a future collaborator
already has SCP access, this is worth revisiting ahead of the other candidates checked tonight.

**Verdict stands: no third disease found tonight that clears the open-access bar.** The §2b
recommendation (stop the search, tighten Phase 1 + Phase 2) now applies without a pending
exception.

## 3. Status and next step

**Phase 3 is contingent, in order, on:**
1. Phase 2 reaching a settled state (§7 item 11 resolved one way or another; the intra-RA extra
   analysis getting its druggability-based follow-up filter) — per this project's own "one phase
   green before the next starts" discipline.
2. The user personally checking `syn64064827`'s real access tier while logged into Synapse (the
   one thing this session can't do itself) — if it turns out to need the same
   `ManagedACTAccessRequirement`-style DUC review AMP2's RA data needed, that's a real decision
   point (apply and wait, or treat Phase 3 as out of scope for this deadline), not a quick fix.
3. Remaining real time before the ~Sep 27–28 cutoff (Phase 2's plan doc §5) — not yet
   re-evaluated against this new information as of this note.

**No code has been written. No data has been downloaded.** This document is a dataset-search
record only, to be picked up from here if/when Phase 3 actually starts.

## 4. 2026-09-12 — constraint clarified, pivoting to ulcerative colitis (Smillie et al. 2019, `SCP259`)

**The zero-registration constraint (§2b) is narrower than it read, by the user's own clarification
today: a free Google-account sign-in (no DUC, no institutional email, no discretionary human
review) is acceptable — the constraint was against exactly those heavier gates, not against
sign-in as such.** Explicit user statement: *"I'll sign into google, that's no issue, I just do
not want to run from my original goal."* This does not reopen AMP2 (Synapse's
`ManagedACTAccessRequirement` DUC+IDU review) or `syn64064827` (Synapse, `canView: false` at the
container level, almost certainly the same class of gate) — both remain ruled out as genuinely
heavier tiers. It specifically reopens Broad Single Cell Portal-hosted data, which §2c already
confirmed uses plain Google OAuth2 (`/single_cell/users/auth/google_oauth2`), the same tier as
the many free-account-only repositories this project has already accepted elsewhere (e.g. ImmPort
SDY998 in Phase 2's own dataset search).

**Re-confirmed directly, 2026-09-12, against `SCP259`'s real live page (not from memory or §2c's
prior note):** 365,492 cells, 21,784 genes, 18 UC patients + 12 healthy controls — matches §2c's
figures. Download tab confirmed present, gated behind Google sign-in only.

**Decision: `SCP259` (Smillie et al. 2019, *Cell*, PMID 31348891, PMCID PMC6662628) is now the
active Phase 3 dataset — lupus nephritis is superseded, not merely deferred**, given §2's two real
lupus candidates were ruled out on independent grounds (SDY997: too small/wrong compartment/CEL-Seq2
technology mismatch; `syn64064827`: gated at the Synapse container level, a DUC-class tier the
user's clarification does not reach). Nothing about lupus nephritis's candidates changed today —
only UC's access tier did.

**Real strength of this dataset for Phase 3's purpose, from §2c, worth restating now that it's
active:** 8 published fibroblast subtypes including WNT5B+ inflammation-associated fibroblasts — a
genuine independent ground-truth comparator for Step 5's stromal extraction, the same role
Zhang/Croft played for RA and Nayar et al. for Sjögren's. Single-cell (not single-nucleus), same
general droplet-based technology class as every other dataset this project uses.

### Step 4 — cross-disease projection — Status: ✅ Run 2026-09-12 23:33-23:40 IST — real negative result, same pattern as RA

`scripts/run_phase3_ssc_step4.py`. Reprocesses SSc fresh from raw (QC->doublets->normalize->
annotate, same functions/thresholds as Step 2), **plus Harmony+Leiden clustering and
`correct_immune_mislabels` run again here** — a real, necessary departure from Phase 2's RA/OA
Step 4 pattern (which calls `call_tls_region_states` directly against raw `majority_voting`),
because GC-B-cell/Tfh calling is restricted to cells already broadly labeled "B cells"/"T cells"
(`call_cell_state`'s `within_cell_types`) — running that against SSc's raw, contaminated
`majority_voting` would let mislabeled keratinocytes dilute the threshold. Documented in the
script's own module docstring, not a silent deviation.

**Timeline**: script launched 23:33:04 IST, CellTypist annotation + Harmony converged by 23:37:40
(7 iterations), full script (AUCell scoring + all statistical tests + CSV writes) finished
23:40:19 — total runtime ~7m15s.

**Real numbers, this run**: 58,763 cells post-QC/doublet-removal (same as Step 2, though the exact
doublet-filtered set can differ slightly run-to-run per the unseeded-Harmony caveat above).
**Immune-mislabel correction reassigned 12,245 / 58,763 cells (20.8%)** this run — in the same
ballpark as Step 2's ~21-27% (12,245+6,804(unaffected Endothelial baseline)... the exact
per-category breakdown wasn't re-printed this run, only the total reassignment count; the pattern
matches Step 2's finding, not a new or different result). One marker gene, IL21, was missing from
the Tfh panel after gene filtering — noted, not chased (the panel has other genes; a single
missing gene out of a multi-gene panel is a minor gap, not a repeat of the E-MTAB-11791 gene-
dropout problem).

**Both AUCell scoring runs found every one of the 25 signature genes present** (SSc: 0 missing,
tonsil: 0 missing, confirming Step 3's signature transfers gene-for-gene onto skin's larger
24,924-gene panel with no dropout issue, unlike RA's fully-present-but-worth-checking-each-time
convention).

**TLS-region cell counts, the first real power concern found here**: only 319 / 58,763 cells
(0.54%) called `is_gc_b_cell | is_tfh` — far lower than RA's 1.1% or Sjögren's much higher rate.
Split: 221 in SSc (disease) samples (0.65% of 33,919), 98 in healthy-control samples (0.39% of
24,844). Tonsil, for reference: 1,778 / 38,072 (4.7%). **This is very likely genuine biology, not a
QC artifact** — skin is not classically a TLS-organizing tissue the way tonsil/synovium/salivary
gland are (consistent with §5's own TLS-literature caveat, checked before this dataset was even
downloaded), so a sparse TLS-region population is the expected outcome, not a red flag.

**Tonsil-calibrated threshold: 0.106942** — identical to Phase 1/2's own derivation (same tonsil
object, same method), confirming the shared reference is being applied consistently across every
phase of this project.

**Cross-disease results, cell-level (not yet the primary evidence — patient-level below is):**

| group | n | median score | tonsil reference median | p-value | BH p | status |
|---|---|---|---|---|---|---|
| SSc (disease) TLS-region | 221 | 0.105705 | 0.106942 | 0.671 | 0.9999 | not significant |
| SSc (healthy control) TLS-region | 98 | 0.075478 | 0.106942 | 0.9999 | 0.9999 | not significant |

**Headline metric**: 46.6% of SSc(disease) TLS-region cells exceed the tonsil threshold (103/221)
— below the ~50% a group statistically identical to tonsil would show, but only barely.
Bootstrap CI for the SSc-vs-tonsil median difference: **[-0.008091, 0.002928] — includes zero.**
**This is a real, meaningful difference from RA's own negative result**: RA's bootstrap CI
excluded zero (a real, if modest, shortfall below tonsil); SSc's CI straddles zero, meaning SSc is
statistically indistinguishable from tonsil in either direction, not even a "real shortfall."

**Patient-level tests (the primary evidence, per this project's own standing convention):**

| comparison | n (group A) | n (group B) | exact permutations | observed diff | p-value |
|---|---|---|---|---|---|
| SSc (disease) vs tonsil | 12 patients | 4 tonsil donors | 1,820 | -0.006064 | 0.676 |
| healthy control vs tonsil | 10 patients | 4 tonsil donors | 1,001 | -0.020853 | 0.845 |
| SSc vs healthy control (same-study) | 12 patients | 10 patients | — (Mann-Whitney) | median 0.097 vs 0.083 | 0.391 |

All 22 samples contributed a patient-level median (every sample had at least one TLS-region cell,
confirmed via the specificity table — no sample dropped from the analysis for lack of coverage,
unlike some of RA's finer checks).

**Conclusion: SSc skin shows no significant TLS-signature enrichment over tonsil, in either
direction, and no significant SSc-vs-healthy-control difference either — a clean, honest negative,
not a borderline or ambiguous one.** Reads consistently with §5's own pre-registered caveat: SSc
skin was flagged before download as lacking RA/Sjögren's-level a priori TLS literature support, and
that is exactly what the data now shows. This is the expected, honest outcome of the exploratory
framing already committed to in this doc — not a surprise, and not evidence against the MG-derived
signature's validity elsewhere (Sjögren's, Phase 1, remains the confirmed positive).

**Specificity check**: every one of the 22 samples individually shows higher TLS-region scores than
that same sample's own non-TLS-region cells (all 22 "diff" values in
`phase3_step4_specificity_patient_level.csv` are positive, range 0.002-0.135) — the signature
remains meaningfully specific to TLS-organizing states within skin tissue, it just doesn't reach
tonsil's absolute baseline. Same pattern Phase 1/2 found: specificity holds even when the
cross-tissue enrichment comparison doesn't.

Full results: `data/processed/phase3_step4_cross_disease_results.csv`,
`phase3_step4_headline_metric.csv`, `phase3_step4_specificity_patient_level.csv`,
`phase3_step4_ssc_tonsil_patient_level_test.csv`, `phase3_step4_control_tonsil_patient_level_test.csv`,
`phase3_step4_ssc_vs_control_patient_level_test.csv`, `phase3_step4_ssc_patient_level.csv`,
`phase3_step4_control_patient_level.csv`.


## 5. 2026-09-12 — fourth-disease candidate checked: GSE135893 (pulmonary fibrosis, Habermann et al.)

User provided this accession directly with specific claims (114,396 cells, 20 PF/10 control lungs,
31 cell types, 4 discrete fibroblast populations + smooth muscle + mesothelial, a HAS1hi subtype
found only in IPF lungs, raw+processed data on GEO with no gate, an HCA mirror as a backup).
**Checked every claim directly against the real GEO SOFT record and the actual paper — all
confirmed accurate except one, not assumed true because the user stated it confidently.**

- **Real paper**: Habermann et al. 2020, *Science Advances* 6(28) (PMC7439444, PMID 32832598),
  "Single-cell RNA-sequencing reveals profibrotic roles of distinct epithelial and mesenchymal
  lineages in pulmonary fibrosis." >114,000 cells, 31 cell types — matches.
- **Sample count resolved exactly like RA's own dataset**: GEO lists 24 "pulmonary fibrosis (PF)"
  samples, not 20 — checked the real per-sample titles directly, not just the aggregate count: 4
  patients (VUILD59, 60, 61, 62) each contributed 2 samples, so 24 samples = 20 unique PF
  **patients** + 10 control samples = 10 unique control patients, exactly matching the paper's "20
  PF and 10 control lungs." Same structural pattern as `E-MTAB-11791`'s 25-samples/23-individuals
  wrinkle — not a new problem, a recognized one.
- **Mesenchymal-compartment claims confirmed directly against the paper's real text**: four
  discrete fibroblast populations plus separately-identified smooth muscle and mesothelial cells;
  the HAS1hi subtype was composed *entirely* of cells from IPF lungs, not detected in controls,
  localizing near-exclusively to subpleural regions colocalizing with COL1A1 — all as the user
  described.
- **Access confirmed genuinely open**: real GEO series supplementary files (barcodes/genes/matrix
  triplet plus a full annotated `.rds` object and IPF-specific metadata CSV), no DAC/controlled-
  access note anywhere, same tier as this project's cleanest datasets (E-MTAB-11791/GSE283080).
- **One claim not confirmed, flagged rather than repeated as fact**: the paper's own Data
  Availability statement mentions only GEO (`GSE135893`) and a GitHub code repo
  (`github.com/tgen/banovichlab/`) — no mention of a Human Cell Atlas mirror. A live search of
  CZ CELLxGENE's public collections found no Habermann/pulmonary-fibrosis entry either. **Doesn't
  matter in practice** since GEO's own direct download already has no gate to route around, but
  the HCA-backup claim specifically shouldn't be repeated as confirmed.

**Verdict: this is a genuinely excellent, verified candidate** — same-study case/control design,
real independent per-sample diagnosis labels (unlike RA), a real published stromal ground-truth
(the four fibroblast subtypes) analogous to what Zhang/Croft provided for RA, and the cleanest
access tier available. Not yet started (no download, no code) — kept here as a strong fourth option
if time allows after SSc/UC, or as a stronger substitute for either if one of them runs into
trouble. Same "one phase green before starting the next" discipline as everywhere else in this
project applies before actually starting it.

## 6. 2026-09-12/13 — GSE135893 (pulmonary fibrosis) download complete

Download launched 2026-09-12 23:33:18 IST, completed 2026-09-13 09:46 IST
(`scripts/download_pf_lung.py`) — all 4 files landed at their exact expected byte sizes (confirmed
against the HEAD-request sizes checked before downloading, no truncation/corruption):
`barcodes.tsv.gz` 936,873 bytes, `genes.tsv.gz` 129,668 bytes, `matrix.mtx.gz` 1,076,137,992 bytes
(~1.08GB), `IPF_metadata.csv.gz` 3,354,014 bytes. **Deliberately not downloading
`GSE135893_ILD_annotated_fullsize.rds.gz` (22.4GB, R-format)** — same reasoning as the AMP2
detour's `.rds` friction (this project is Python-only and builds its own independent QC/
annotation pipeline rather than reusing the original authors' derived Seurat object, same practice
as every other dataset here). **Loaded and confirmed against real data, 2026-09-13 — Step 1 done.**
`src/mg_thymus_map/data/pf_lung.py` (`load_pf_lung`). Real structural finding, checked before
trusting anything: GEO's pooled `matrix.mtx.gz` is a RAW (unfiltered) matrix — 220,213 barcodes,
not the real cell count, same "raw not filtered" surprise as SSc's per-sample `.h5` files, just
pooled instead of per-sample. **The real, QC-passed 114,396-cell list — matching the paper's own
headline count exactly — lives in `GSE135893_IPF_metadata.csv.gz`**, whose row index is the cell
barcode; the loader restricts the pooled matrix to that index rather than re-deriving cell-calling.
Two real bugs caught before trusting the loader: (1) `mmread` handles `.gz` transparently (checked
directly, confirmed real dimensions 33,694 genes × 220,213 cells before assuming), but a
straight-`open()` line-reader does NOT — genes.tsv.gz/barcodes.tsv.gz needed `gzip.open(path,
"rt")` instead, a bug that would have silently produced garbage gene/barcode strings if untested;
(2) none, `Sample_Name` was confirmed directly (not assumed) to correctly collapse the 4
two-library patients (`ILD59-1`/`ILD59-2` → `VUILD59`, etc.) into one identity each, matching
GEO's own sample-title structure.

**Real strength over `E-MTAB-11791`'s RA data, confirmed directly: per-cell diagnosis IS
recoverable here.** `Diagnosis` column gives the fine-grained label (IPF 57,682 / NSIP 8,638 / cHP
7,535 / Sarcoidosis 4,891 / ILD 4,006 / Control 31,644 cells); `Status` collapses this to a clean
binary (Disease 82,752 / Control 31,644) already provided by the depositors, not inferred. Loader
preserves both: `disease` uses the binary `Status` (`pulmonary_fibrosis`/`healthy_control`, matching
this project's other loaders' convention), `diagnosis_detail` keeps the granular value for a future
IPF-only subset analysis — the paper's own headline HAS1hi finding is IPF-specific, not "PF"
broadly, worth revisiting if a coarse PF-vs-control result is ambiguous.

**Final real counts: 114,396 cells × 33,694 genes, 30 unique patients (20 disease + 10 control)** —
exactly matching the paper. 5 new unit tests (a full synthetic round-trip, matching E-MTAB-11791's
own test rigor for a pooled-matrix loader), 174 passing project-wide.

**Step 2 (QC/CellTypist/Harmony/immune-mislabel correction) run 2026-09-13 19:40:15 IST**
(`scripts/run_phase3_pf_step2.py`) — applies the immune-contamination correction from the outset
this time, not as an afterthought, since SSc already established it as a real risk class for any
tissue with a large non-immune fraction. Real numbers: 114,396 → 109,270 cells post-QC/doublets.
The depositors' own cell-type calls (metadata's `population` column, not used by this project's
own annotation, but informative context) show a substantial non-immune share: Epithelial 37,325 +
Mesenchymal 5,232 + Endothelial 10,446 = 52,997/114,396 (46.3%) — immune still the plurality
(61,393, 53.7%), a less extreme ratio than SSc's epithelial-majority skin.

**A second, different real bug found here (2026-09-14), not the same one SSc found.**
Before/after correction counts came out byte-identical again — but this time the flagging logic
itself had genuinely failed, not just been non-deterministic. Checked directly: cluster 24 (1,488
cells, 80.3% plurality "ILC", real PTPRC-positive fraction 3.4% — clearly non-immune, clearly
mislabeled) was never flagged. Root cause: **with 109K cells split across 54 Leiden clusters, two
clusters ended up with *exactly* zero PTPRC-positive cells** — never happened on SSc's smaller run.
The `flag_non_immune_clusters` gap-search adds a small epsilon before taking `log()`; the gap from
those exact-zero clusters to the next (still tiny but nonzero) value looked artificially enormous
in log space purely because of how close the epsilon sits to zero, so `argmax` locked onto that
spurious split at the very bottom of the distribution instead of the real 4.7%→54.5% boundary
further up — silently dropping cluster 24 (and everything above it) from the flagged set entirely,
the same "silent no-op instead of an error" failure shape as SSc's bug, different root cause.

**Fixed in `annotate/immune_contamination.py`**: a candidate split is now only eligible if its
*high* side clears `fallback_fraction_threshold` (3%) — a gap entirely within the near-zero noise
floor isn't a real immune/non-immune boundary, whatever it looks like in log space. New regression
test added reproducing this exact scenario (two exact-zero clusters adjacent to a real 4.7%→54.5%
boundary) — 7 tests in this module now, 175 passing project-wide. **Confirmed SSc's already-reported
results are unaffected**: checked its final run's real PTPRC fractions directly — minimum was
0.000195, never exactly zero, so this specific epsilon artifact never had anything to trigger on;
the real gap it used (0.5%→5.2% in the actual run) still clears the new eligibility check
unchanged. No SSc numbers need revision.

**Step 2 re-run, 2026-09-14 14:59:34 → 16:44 IST, with the fix — Step 2 done.** Real final numbers:
114,396 → 109,407 post-QC (mito threshold 15.93%) → 109,270 post-doublet-removal (137 flagged,
0.13% — very clean). Disease split: 78,557 pulmonary fibrosis / 30,713 healthy control.

**CellTypist (`majority_voting`): Macrophages 37,952 (the largest population — real lung biology,
alveolar macrophages are genuinely abundant, not a red flag), Epithelial cells 31,215, Endothelial
cells 10,499, Monocytes 6,695, T cells 6,663, ILC 5,305, DC 3,690, Fibroblasts 3,574, B cells
2,309, Mast cells 615, Plasma cells 579, pDC 174.**

**Correction result, this time genuinely small and proportionate — a good sign the fix isn't
over-correcting.** Only ~1,166 / 109,270 cells (1.1%) reassigned, vs. SSc's ~21% — consistent with
lung's CellTypist annotation being largely accurate to begin with (unlike SSc's epithelial-majority
skin, lung's real biology already matches an immune-classifier-friendly composition reasonably
well). The one real correction: ILC 5,305 → 4,390 (−915, matching the exact 1,488-cell mislabeled
cluster found during debugging, most of which resolved to Fibroblasts — 3,574 → 4,662, +1,088) and
Endothelial cells 10,499 → 10,337 (−162). Everything else moved by single digits or not at all.
Final corrected counts saved as `cell_type_corrected` in `data/interim/pf_harmonized.h5ad`.

### Step 4 — cross-disease projection — Status: ✅ Run 2026-09-14 17:13-17:22 IST — cell-level "signal" that does not survive patient-level testing, same artifact pattern already seen twice

`scripts/run_phase3_pf_step4.py`, mirroring SSc's own Step 4 script exactly. 109,270 cells
reprocessed fresh (same QC numbers as Step 2), immune-mislabel correction reassigned 1,622 cells
(1.5%, consistent with Step 2's small/proportionate correction). All 25 signature genes present
in both PF and tonsil, no dropout.

**TLS-region cells: 418 / 109,270 total (331 disease / 87 control)** — sparse, though notably
less sparse than SSc's 0.54% (PF's is 0.38%, similar order, both far below tonsil's 4.7%).

**Cell-level result: both PF (disease) and healthy-control TLS-region cells score significantly
*higher* than tonsil** (PF disease: median 0.132 vs. tonsil's 0.107, p=1.1×10⁻¹⁴, BH p=2.3×10⁻¹⁴;
control: median 0.143, p=6.5×10⁻⁶) — headline bootstrap CI for the PF-vs-tonsil median difference
excludes zero ([0.017, 0.033]). **On its own this would read as a striking positive** — the
opposite pattern from both RA and SSc, and structurally plausible: pulmonary fibrosis (especially
IPF) has real, independently documented lymphoid-aggregate/TLS pathology in fibrotic lung tissue,
unlike SSc skin.

**Patient-level tests (the primary evidence, per this project's own standing convention) — do
NOT confirm it, the same cell-level pseudoreplication artifact already caught twice in this
project (RA's Step 4, OA-vs-tonsil):**

| comparison | n | exact permutations | observed diff | p-value |
|---|---|---|---|---|
| PF (disease) vs tonsil | 17 patients vs. 4 tonsil donors | 5,985 | +0.0123 | **0.241** |
| healthy control vs tonsil | 8 patients vs. 4 tonsil donors | 495 | −0.0307 | **0.806** |
| PF vs healthy control (same-study) | 17 vs. 8 patients | Mann-Whitney | median 0.116 vs. 0.073 | **0.449** |

(3 of 20 PF patients and 2 of 10 control patients had zero TLS-region cells at all, so contribute
no patient-level median — real, not a bug, consistent with how sparse this cell state is here.)

**Conclusion: like RA and SSc, PF lung's TLS-signature transfer is not confirmed once tested at
the correct statistical level.** The cell-level "signal," while numerically striking, does not
survive the patient-level test this project has trusted as primary evidence since Phase 1's own
pseudoreplication lesson (a cell-level p=0.006 that became p=0.365 patient-level) — same failure
mode, not a new one.

**A real, honest secondary observation, worth flagging even though it doesn't change the primary
conclusion**: per-sample TLS-vs-non-TLS diffs (the specificity table) show real heterogeneity
across PF patients — TILD015 (133 TLS cells, diff +0.041), VUILD60 (diff +0.100), VUILD61 (diff
+0.109) show a much stronger pattern than VUILD59 (diff −0.045) or TILD010 (diff −0.057, only 1
TLS cell). This is consistent with real, independently-documented clinical heterogeneity in how
prominently individual IPF/PF patients develop lymphoid aggregates — worth a closer look (e.g.
against `diagnosis_detail`, since this dataset's fine-grained IPF/NSIP/cHP/Sarcoidosis label might
explain some of this spread) if this dataset is pursued further, but not chased today.

**Specificity still holds broadly**: most samples individually show higher TLS-region than
non-TLS-region scores (20 of 25 samples with any TLS-region cells show positive diff); the
negative-diff samples (TILD010, VUILD59, VUHD66, VUHD68, VUHD70) all have very few TLS-region
cells (1-8), consistent with small-n noise rather than a real reversal.

Full results: `data/processed/phase3_pf_step4_cross_disease_results.csv`,
`phase3_pf_step4_headline_metric.csv`, `phase3_pf_step4_specificity_patient_level.csv`,
`phase3_pf_step4_pf_tonsil_patient_level_test.csv`,
`phase3_pf_step4_control_tonsil_patient_level_test.csv`,
`phase3_pf_step4_pf_vs_control_patient_level_test.csv`, `phase3_pf_step4_pf_patient_level.csv`,
`phase3_pf_step4_control_patient_level.csv`.

### Step 5 — stromal extraction — Status: 🟡 real methodological problem found and fixed 2026-09-14, second run in progress

`scripts/run_phase3_ssc_step5.py`. Uses `cell_type_corrected` (not raw `majority_voting`) for the
compartment restriction, per Step 2/4's contamination finding. Same-study/same-protocol design
(unlike RA's cross-study pairing), so this uses Phase 1's original simple method directly —
cell-level regress-out + rank + patient-level confirm — not RA's pseudobulk-ComBat workaround,
which exists specifically for a cross-study batch effect this dataset doesn't have.

**First run, 2026-09-14 16:40-16:43 IST, restricted to Phase 1/2's default `STROMAL_CELL_TYPES`
(Fibroblasts+Epithelial cells+Endothelial cells) — real problem found, not trusted.** SSc stromal:
49,815 cells (Epithelial 22,112 + Fibroblasts 21,076 + Endothelial 6,627), disease split 28,454 SSc
/ 21,361 control. **Top 20 markers were almost entirely keratinocyte differentiation genes**: SFN,
KRT10, KRT1, KRT14, KRT5, DMKN, LY6D, LGALS7B, FXYD3, S100A11, PERP, AQP3, KLF5 — the exact same
gene family this project used to diagnose Step 2's CellTypist contamination bug, not real
fibroblast/myofibroblast biology, and nothing resembling Tabib et al.'s own reported SFRP2/
myofibroblast markers. Root cause: `STROMAL_CELL_TYPES`'s default bundling of "Epithelial cells"
with "Fibroblasts" made sense for Sjögren's (glandular epithelium is coherently "stroma" there) and
RA (epithelial cells barely present), but in skin "Epithelial cells" = keratinocytes, a massive
(44% of the mixed compartment), biologically distinct population whose own real SSc-specific
differentiation changes swamped the fibroblast signal this dataset was actually chosen for.
**Kept, not deleted**: `phase3_step5_stromal_markers_mixed_compartment_v1.csv`,
`phase3_step5_candidate_patient_level_bh_mixed_compartment_v1.csv`.

**Second run, 16:48:23 → 16:52 IST, restricted to Fibroblasts only — a real, strong, biologically
coherent positive finding.** SSc stromal (fibroblasts only): 21,021 cells (10,036 SSc / 10,985
control). **14 of 15 candidate genes clear BH correction at patient level** — the strongest
positive stromal result this project has found in Phase 2 or 3 (RA's own stromal extraction was
an honest null, 0/15). Real, literature-recognizable ECM/fibrosis genes: **POSTN** (periostin,
p_bh=0.017), **BGN** (biglycan, p_bh=0.005), **THY1** (CD90, a canonical fibroblast-activation
marker — also the same gene Croft et al. used to define RA's sublining fibroblast axis,
p_bh=0.005), **SPARC** (p_bh=0.017), **PCOLCE** (procollagen C-endopeptidase enhancer, directly
involved in collagen fibril assembly, p_bh=0.003), **ASPN** (asporin, p_bh=0.031), **IGFBP4**
(p_bh=0.020), **NNMT** (nicotinamide N-methyltransferase, p_bh=0.014), **CTSC** (cathepsin C,
p_bh=0.003) — all up in SSc. **Interesting secondary thread, not yet chased further**: IFITM1/2/3
(interferon-induced transmembrane proteins) are also strongly significant (p_bh=0.003-0.005,
IFITM3 the single strongest hit in the whole list) — a possible echo of Phase 1's own
interferon-stimulated-gene finding in Sjögren's (PIP/LYZ/IFI6/IFI44L/XAF1/AZGP1), worth checking
directly against real IFITM-SSc literature before treating as a real cross-disease convergence
rather than coincidence. **Only XIST fails** (p_bh=0.767, correctly a sex-linked gene that
shouldn't have been in the candidate list in the first place — see the artifact-filter gap below).

**A real process gap caught while reviewing this result, fixed, not chased further with a rerun
since it doesn't change the reported result**: this script's `TECHNICAL_ARTIFACT_PREFIXES`/
`STRESS_RESPONSE_ARTIFACT_GENES` filter, copied from RA's Step 5, was missing Phase 1's own
established sex-linked gene filter (`docs/phase1_methods.md` §5: "XIST + 5 Y-linked genes").
XIST reaching the top-20 candidate list and only failing by chance at the confirmation stage (not
because the filter caught it) is exactly the kind of near-miss this project's own convention exists
to prevent. Fixed by adding `SEX_LINKED_ARTIFACT_GENES` to the script — doesn't change the 14
confirmed genes above, since XIST was never among them, but the filter is now complete for any
future rerun.

Full results: `data/processed/phase3_step5_stromal_markers.csv` (fibroblast-only, real),
`phase3_step5_candidate_patient_level_bh.csv` (fibroblast-only, real, the 14/15-confirmed table
above), `phase3_step5_stromal_markers_mixed_compartment_v1.csv` and
`phase3_step5_candidate_patient_level_bh_mixed_compartment_v1.csv` (the flawed first attempt,
kept as historical record, not deleted).

### Step 6 — DGIdb/ChEMBL pharmacogenomic mapping — Status: ✅ Run 2026-09-14 17:10-17:11 IST — one real, defensible candidate

`scripts/run_phase3_ssc_step6_pharma.py`. Queries the literal 14-gene confirmed list directly
(Phase 1's original Step 6 pattern), not an effect-size-narrowed shortlist — unlike the intra-RA
extra analysis's 4,907-gene list, 14 already-small confirmed genes need no further narrowing.

**7 of 14 genes had any DGIdb/ChEMBL hit at all; 18 raw gene-drug pairs; zero FDA-approved
(max_phase=4) hits** — consistent with the pattern already seen in Phase 1 (ISGs) and RA
(structural/ECM genes are historically hard direct drug targets, no surprise here either). Most
raw hits are the same class of noise already documented elsewhere in this project — antiserum,
environmental estrogen, Freund's adjuvant, calcium gluconate, progesterone, lysozyme (endogenous
reagents/research compounds DGIdb catalogs, not real therapeutic candidates).

**One real, defensible finding: CTSC (cathepsin C) → brensocatib, max_phase 3.0 (2 independent
sources, DGIdb + ChEMBL), action type INHIBITOR.** Verified directly, not assumed real from the
name alone: brensocatib is a genuine, currently-in-development oral DPP1 (dipeptidyl peptidase
1 — CTSC's own protein product) inhibitor, confirmed via Europe PMC search against real trial
literature — Phase II/III trials for non-cystic-fibrosis bronchiectasis (WILLOW, ASPEN) and a
Phase IIa trial for cystic fibrosis, both neutrophilic airway-inflammation indications.
**Mechanistically coherent** (CTSC/DPP1 activates neutrophil serine proteases; CTSC is
patient-level up in SSc fibroblasts here, p_bh=0.003, the second-strongest hit in the whole list).
**Honest caveat, stated plainly rather than overclaimed**: brensocatib has never been tested in
SSc as far as this check found — this is a genuine, novel repurposing hypothesis grounded in a
real mechanistic link, not an established SSc therapy or even a currently-hypothesized one in the
literature. The same category of finding as RA's MAPK11/regorafenib (real drug, real mechanism,
untested in this specific disease) — worth stating with the same honesty in any eventual write-up.

**One secondary, weaker candidate**: PENTOSAN POLYSULFATE (an approved interstitial-cystitis drug)
also hit CTSC, but with only 1 source and a less-characterized mechanistic relationship — not as
clean as brensocatib, kept for the record but not elevated to a primary finding.

Full results: `data/processed/phase3_step6_dgidb_raw.csv`, `phase3_step6_chembl_raw.csv`,
`phase3_step6_ranked.csv`.

**Step 6, pathway-level pivot — run 2026-09-15 (Fable) — SSc returns Phase 1's exact drug set.**
`scripts/run_phase3_ssc_step6_pathway_pivot.py`. Phase 1's own documented move
(`docs/phase1_methods.md` §6): the literal ISG list had almost no drug hits (ISGs are downstream
effectors), so it queried one level upstream — IFNAR1/IFNAR2/JAK1/JAK2/TYK2/STAT1/STAT2,
inhibitory direction — and that is where anifrolumab and the JAK inhibitors came from. SSc's
Step 5 residual carries the same class of signal, so the same pivot was applied, unchanged.
Result: **anifrolumab (IFNAR1, FDA-approved, 2 independent sources) + 16 distinct approved
JAK1/JAK2/TYK2 inhibitors** (baricitinib, tofacitinib, upadacitinib, filgotinib, deucravacitinib,
ruxolitinib, abrocitinib, ...) — Phase 1's exact set.

**Correction to an earlier statement in this doc and in `docs/phase3_ssc_findings.md`**: only
**IFITM3 (p_bh=0.003) and IFITM2 (p_bh=0.005)** are among the 14 patient-level-confirmed genes.
IFITM1 was in the top-20 marker list but fell outside the 15-gene shortlist that went to
patient-level testing. Two confirmed ISGs, not three.

**Honest weight of this result, stated before anyone over-reads it**: the pivot's gene set is
fixed a priori, so the drug list is mechanically the same for any disease — it proves nothing on
its own. The scientific claim ("SSc's residual converges with Sjögren's on a type-I-IFN
component, hence the same drug class") rests on two things: (1) SSc's own confirmed ISG hits,
which are real but a minority of a residual dominated by ECM/fibrosis genes; and (2) independent
literature, **both verified directly against the real abstracts, not a summary**:
- Bryon et al. 2025, *Arthritis & Rheumatology*, PMID 39415484 (primary research): "SSc skin
  biopsies showed the highest levels of type I IFN response"; SSc dermal-fibroblast exosomes
  induce an IFN signature in keratinocytes, and "inhibition of TBK or JAK activity suppressed" it.
- Radić et al. 2026, *J Clin Med*, PMID 41682782 (review): proposes anifrolumab (anti-IFNAR1)
  for SSc, citing "upregulation of interferon-stimulated genes (ISGs)" and ongoing trials.
(3) is what turns two discovery genes into a claim: `scripts/run_phase3_ssc_step7_isg_convergence.py`
(launched 2026-09-15) tests an a-priori canonical ISG panel — IFI6/IFI44L/XAF1 (Phase 1's own),
ISG15, MX1, IFI27, IFI44, OAS1, IFIT1, IFIT3, RSAD2, STAT1 — with IFITM2/IFITM3 deliberately
excluded (they are the discovery; re-testing them would be circular, Phase 1's Step 7 lesson),
patient-level, one-sided UP (a genuine a-priori direction), BH within the family.

**Step 7 result, 2026-09-15 — cleared, cleanly.** 17,920 fibroblasts (12 SSc / 10 control
patients; count differs from Step 5's 21,021 per the documented Harmony-stochasticity effect on
the correction). **10 of 12 a-priori ISGs clear BH correction; 11 of 12 point up in SSc; all
three of Phase 1's own Sjögren's ISGs confirm** — IFI6 p_bh=0.004, ISG15 0.004, IFI27 0.004,
IFI44 0.006, IFI44L 0.006, STAT1 0.006, XAF1 0.020, MX1 0.020, OAS1 0.036, IFIT3 0.036. IFIT1
points the wrong way (p=0.80); RSAD2 near-zero in both arms (p=0.36). U-statistics 89-110 of a
maximum 120 — not the complete-separation artifact RA's first Step 5 showed. Results:
`data/processed/phase3_step7_isg_convergence_patient_level_bh.csv`.

**Decision, made before the result and held**: SSc is now reportable as the second disease
reaching Sjögren's drug set — *the same type-I-IFN target (anifrolumab / JAK inhibitors), reached
through a disease-specific stromal residual, even though the TLS architecture did not transfer.*
Aim 1 remains a clean negative and is reported as such. UC is now optional, not required for this
goal; PF stays paused.

**Aim 1 is unchanged by any of this: SSc's TLS-signature transfer remains a clean negative.** If
Step 7 confirms the ISG component, the SSc story is "the same drug target reached through a
disease-specific stromal residual, despite non-shared TLS architecture" — reportable and, if
anything, more interesting than a simple replication. If Step 7 is null, the brensocatib/CTSC
finding stands on its own and the IFN convergence gets reported as suggestive-but-unconfirmed.

**Tonsil 4→8 rebuild, completed 2026-09-15**: `data/interim/tonsil_with_cell_states_8donors.h5ad`,
61,069 cells, 8 donors, TLS-region 4.72% (vs. 4.67% in the 4-donor object — nearly identical,
reassuring). **Per the user's explicit instruction, Sjögren's vs. MG is not being touched**: the
4-donor reference stays the primary for every reported number; the side-by-side re-run
(`rerun_step4_against_8donor_tonsil.py`) is off the critical path — at most a sensitivity
appendix later, not a replacement.

## 7. 2026-09-12 — two more user-suggested candidates checked, one is a real find

User asked about "systemic sclerosis, Tabib et al. (`GSE138286`)" and "Crohn's (`GSE125527`)" while
waiting on the SCP259 sign-in. Both accessions checked directly against GEO's real records (via
`eutils` esummary + the SOFT family file, since GEO's own web page was blocking automated fetches
with a reCAPTCHA this session) — same recurring failure mode this project has hit before
(GSE181082, GSE114374): a remembered accession is close but not the actual paper.

**`GSE138286` is not Tabib et al. at all** — it's "The transcriptomes of PC3 cells with or without
KLF16 interference," an unrelated 2-sample bulk RNA-seq prostate-cancer-cell-line study. **The real
Tabib et al. paper is `GSE138669`** (Tabib et al. 2021, *Nature Communications*, PMID 34042322,
"Myofibroblast transcriptome indicates SFRP2+ fibroblast progenitors in systemic sclerosis skin") —
found via Europe PMC title search, confirmed against its own Data Availability statement.

**`GSE138669` — genuinely excellent, real numbers, checked directly:**
- **22 whole-skin biopsies, same-study/same-protocol case-control: 12 SSc + 10 healthy controls**
  (confirmed per-sample via the SOFT file's own `condition: CONTROL`/`condition: SSC` field — a
  clean per-sample disease label, the exact thing `E-MTAB-11791` was missing for RA).
- **Whole-skin digest, not sorted** — "Miltenyi Whole Skin Dissociation Kit," all cell types
  including the fibroblast/stromal compartment this pipeline's Step 5 needs. Structurally *better*
  than the RA/OA pairing: same-study, same protocol (unlike RA's cross-study Kuo/Miyahara pairing),
  the same design quality as Sjögren's PSS/SICCA in Phase 1.
- **10x Genomics (v1/v2 chemistry), per-sample Cell Ranger `raw_feature_bc_matrix.h5` files** — the
  exact format Phase 1's MG loader (`scanpy.read_10x_h5`) already reads, no new loader logic needed
  structurally, unlike RA's pooled-matrix surprise.
- **Genuinely open GEO deposit, no DAC/controlled-access note anywhere** — checked the full SOFT
  record, no gate at all, not even a sign-in (a friendlier tier than SCP259's Google OAuth).
- **Not yet checked**: whether SSc skin shows real TLS/ectopic-lymphoid biology (tertiary lymphoid
  structures/lymphocytic aggregates are documented in SSc skin and lung in some literature, but this
  needs a direct citation check before assuming the pipeline's core premise transfers, the same
  diligence given to RA and lupus nephritis's ground-truth papers) — first thing to check before
  writing any loader code.
- **~2,800–3,270 cells/sample × 22 samples**, roughly 62,000–68,000 cells total (paper's own
  reported means) — comparable scale to Phase 1's Sjögren's arm.

**`GSE125527` is real but a poor fit for this pipeline's design, not a Crohn's dataset at all.**
Checked directly (SOFT file): it's Boland et al. 2020 (*Science Immunology*, PMID 32826341),
"Heterogeneity and clonal relationships of adaptive immune cells in ulcerative colitis" — **UC, not
Crohn's.** 103 samples (62 control/41 UC), rectum/ileum/PBMC, but **every single sample is CD45+
FACS-sorted immune cells only** — no stromal/epithelial compartment at all, the same structural
disqualifier already documented for MG's own `GSE233180` and lupus nephritis's `SDY997`. Genuinely
open on GEO (real UMI tables, no gate), but Step 5 has nothing to extract here. Kept as a
data point, not pursued as a primary dataset — `SCP259` already covers UC with a full compartment
range.

**TLS-literature check, done 2026-09-12 before writing any code, per this project's own standard
of checking the ground-truth premise before building on it.** Real finding, honest and mixed: SSc's
documented tertiary lymphoid structure / ectopic germinal-center biology is concentrated in the
**lung** (Zhu et al. 2026, *Arthritis Res Ther*, PMC12910954 — Tfh cells organizing into TLS in
SSc-associated interstitial lung disease specifically), not skin. For skin specifically, the
established finding is different and weaker for this project's purposes: a **diffuse perivascular
lymphocytic infiltrate, predominantly CD4+ T cells, without organized follicle/germinal-center
architecture** (Geroldinger-Simić et al. 2026, *J Dtsch Dermatol Ges*, PMC13059067 — spectral flow
cytometry on SSc skin, no B-cell aggregates or lymphoid organization reported). Checked the actual
`GSE138669` paper (Tabib et al. 2021) directly too — it doesn't address immune/lymphoid populations
at all, scoped entirely to fibroblast heterogeneity.

**What this means, stated plainly rather than glossed over:** unlike RA and Sjögren's — both
textbook ectopic-lymphoid-neogenesis diseases with strong a priori literature support going in —
SSc *skin* doesn't have the same pre-established rationale for the TLS signature to land on
anything. The pipeline's Step 4 doesn't strictly require pre-diagnosed histological TLS (it
operationally scores whatever GC-B-like/Tfh-like cells it empirically finds by marker expression in
the real data), so the analysis can still run and produce a real result either way — but a negative
result here would be markedly less surprising/informative than RA's was, since the field never
strongly expected skin specifically to show this. **Decision, 2026-09-12: proceed with `GSE138669`
anyway, framed honestly as exploratory** (testing whether shared architecture extends even to a
tissue without strong prior TLS expectation), not claiming RA/Sjögren's-level a priori rationale.
This framing distinction belongs in the eventual findings write-up's Limitations section from the
start, not retrofitted later.

**Recommendation: `GSE138669` (systemic sclerosis) is worth pursuing now, in parallel with UC**,
specifically because it needs no sign-in at all — it doesn't have to wait on the mentor the way
`SCP259` does. If the TLS/ground-truth check above holds up, this may end up a stronger third-disease
candidate than UC on data-quality grounds alone (same-study design, real per-sample labels,
familiar file format), independent of the access-tier question.

**Division of labor for the actual download, per this project's established Synapse-era
convention** (anonymous public-metadata checks are fine for this session; anything needing login
goes through the user) — **explicit user choice today: user signs in and generates the bulk-download
command, pastes it here, this session runs it.** Broad SCP's bulk-download mechanism: after
signing in on the study page, the Download tab's "Bulk Download" action generates a short-lived
(~30 min) `curl` command carrying an auth token — two lines, roughly:
```
curl "https://singlecell.broadinstitute.org/single_cell/api/v1/bulk_download/generate_curl_config?accessions=SCP259&auth_code=<code>&directory=all&context=study" -o cfg.txt
curl -K cfg.txt
```
**Not yet done: user hasn't generated/pasted this command yet.** Once pasted, this session will
run it (token expires fast, so it should be generated right before pasting, not sat on), inspect
the real file layout (expression matrix format, cluster/metadata files — not yet known, SCP studies
vary), and write `download_uc_colon.py` + a loader following the same pattern as Phase 2's RA/OA
scripts, checking real dimensions against what's actually on disk before trusting any code.

**Blocked, 2026-09-12: the user is 15, and Terra (which SCP259's download routes through) requires
account holders to be 18+.** Real, structural blocker, not a workaround-able inconvenience — see
[[user-technical-background]] memory. Resolution in progress: the user's ISEF Adult Sponsor
(parent/teacher/mentor) will create their own Terra account under their own identity and either
download the files directly or generate the bulk-download command. **Given this real delay, §5's
`GSE138669` (systemic sclerosis) is worth starting in parallel** — it needs no sign-in at all, so
it isn't gated on the mentor's availability the way UC is.

**Downloaded and loaded against real data, 2026-09-12 — Step 1 done for SSc.**
`scripts/download_ssc_skin.py` pulled all 22 real per-sample Cell Ranger `raw_feature_bc_matrix.h5`
files (579MB total) from GEO's open file server, same HTTPS-not-ftp fix as RA/OA. One real network
timeout hit mid-download (GEO's server, not a bug) — the script's existing skip-if-present
idempotency handled the resume cleanly, no code change needed.

**Real structural finding, checked before writing the loader, not assumed:** these are Cell
Ranger's *raw* (unfiltered) matrices — 737,280 barcodes per sample, the full 10x v2 chemistry
whitelist size, not cell-called output. The paper's own Methods state cells were kept only if they
expressed ≥200 genes, no other cell-calling method — `src/mg_thymus_map/data/ssc_skin.py`
(`load_ssc_skin`) applies that exact filter via `sc.pp.filter_cells(min_genes=200)`. **Validated
directly against the paper's own reported numbers before trusting it**: per-sample counts
reproduce the paper's stated means closely (control mean 2,934 here vs. paper's 2,822; SSc mean
3,242 here vs. paper's 3,082) — real per-sample variation, not a systematic mismatch. **Final real
counts: 68,249 cells × 33,538 genes, 22 samples (12 SSc / 10 control), matching CONDITION_MAP's
per-sample labels confirmed directly against the real GEO SOFT record.** 5 new unit tests, 165
passing project-wide, `make test` green.

**Step 2 done, 2026-09-12 — QC/CellTypist/Harmony, same functions as Phase 1/2, no SSc-specific
code.** `scripts/run_phase3_ssc_step2.py`. Real numbers: 68,249 → 59,051 post-QC (adaptive mito
threshold 13.16%) → 58,763 post-doublet-removal (288 flagged, 0.49% — a much cleaner doublet rate
than RA's 0.09%/OA's 3.46%, no anomaly to chase). Post-QC disease split: 33,919 SSc / 24,844
healthy control cells.

**CellTypist (`majority_voting`): Epithelial cells 18,583, Fibroblasts 12,182, T cells 9,291, ILC
7,056, Endothelial cells 6,804, Macrophages 2,395, DC 1,352, Mast cells 569, B cells 329, Erythroid
148, Plasma cells 54.** Real, expected difference from every other dataset this project has used:
**epithelial cells are the largest single population** — genuine skin biology (keratinocytes),
whereas RA/OA/Sjögren's/MG were all immune-cell-dominated. **ILC's 7,056 cells (12% of all cells)
is unusually high for skin** and not yet explained — flagged as an open item, possibly
`Immune_All_High`'s broad model miscalling a skin-resident population (the same class of
CellTypist-granularity question already resolved for RA's fibroblasts via subclustering, §
`02-phase2-ra.md` §7 item 6) rather than genuine ILC abundance; not chased further yet.

**One real structural difference from Phase 2's design, not a code change:** SSc's disease-vs-
control split is intra-study (one GSE, both conditions already in `load_ssc_skin`'s output) — there
is no second cross-study dataset to combine the way RA+OA were. Harmony was therefore run with
`batch_key="sample_id"` (correcting inter-patient variation across the 22 biopsies, the standard
scRNA-seq use of Harmony) rather than `batch_key="dataset"`, and `compute_cluster_composition` was
read against `disease` instead of `dataset` — the diagnostic question here is whether disease status
drives real biological cluster separation (expected — this is literally Tabib et al.'s own reported
finding, SSc-specific myofibroblast/SFRP2+ populations) rather than an uncorrected batch artifact.

**Real finding: two notably skewed pre-Harmony clusters, not yet individually investigated.**
Overall SSc:control ratio is 57.7%:42.3%; most pre-Harmony clusters sit near that baseline, but
cluster 26 (90.8% SSc) and cluster 33 (88.4% control, 11.6% SSc) stand out. Post-Harmony, no
cluster is nearly this skewed (worst cases ~70-75% one condition) — Harmony visibly reduced
per-patient variance, but real disease-associated compositional shifts plausibly remain (expected,
not itself a red flag the way RA/OA's cross-study skew was). **Investigated, 2026-09-12 — resolved, and it uncovered something bigger than the original
question.** Checked clusters 0/2/12 (the most SSc-skewed post-Harmony clusters that run) directly:
all three spread across all 22 samples (top single-sample share 14.9-17.4%) — not the
single-sample-dominated pattern that flagged RA/OA's real technical artifacts, so not obviously a
batch effect. But their `majority_voting` composition was strange: cluster 2 (5,081 cells) was
labeled 57% "T cells" / 41% "Epithelial cells" within one Leiden cluster — two very different
broad cell types in what should be one coherent population. Real marker genes for the whole
cluster (`rank_genes_groups`) were unambiguous: SFN, DMKN, KRT14, KRT1, S100A14, KRT10, KRT5,
KRTDAP — textbook keratinocyte/epidermal genes, zero T-cell identity.

**Confirmed directly against real marker expression, not inferred from cluster identity alone**:
the 2,911 cells CellTypist called "T cells" within this cluster have CD3D/CD2/TRBC2 at noise level
(mean 0.009/0.004/0.003) and keratinocyte markers just as high as the cells correctly labeled
"Epithelial cells" in the same cluster (KRT14 mean 5.2 vs. 4.1) — these are misclassified
keratinocytes, not a real T-cell population.

**Checked how far this extends, not assumed to be isolated to one cluster: it's dataset-wide.**
Every one of CellTypist's nominally-immune labels showed real keratinocyte contamination by the
same direct check (mean KRT14+KRT1+KRT10 expression, %KRT14-positive): "T cells" 49.6% KRT14+
(only 18.9% CD3D+), "ILC" 28.8% KRT14+, "B cells" 24.9%, "Macrophages" 32.6%, "DC" 36.0%, "Mast
cells" 16.9%. **This fully explains the "ILC unusually high for skin" oddity flagged right after
Step 2's first run** — it isn't real ILC abundance.

**Root cause: not a bug introduced this session, but a known, previously-undiagnosed gap this
project's own code has documented since Phase 1.** `annotate/celltypist_annotate.py`'s docstring
already says: "CellTypist's models are immune-cell classifiers -- they don't cover non-immune
stromal/epithelial types... Cells that don't get a confident immune call need a separate
marker-based pass (added alongside this, not yet built)." It never mattered enough to matter until
SSc skin — the first tissue in this project where non-immune cells are the clear majority (~68%
epithelial+fibroblast+endothelial here, vs. immune-dominated for MG/Sjögren's/tonsil/RA/OA).

**The real, decisive, generalizable signal: PTPRC (CD45), the standard pan-leukocyte marker used
for exactly this immune-vs-structural distinction in flow cytometry/IHC** (the same marker Kuo et
al.'s own RA flow-cytometry validation figures used for CD45+/CD45- gating). Checked per Leiden
cluster: 26 of 28 clusters showed <1.5% PTPRC-positive cells (correctly non-immune, whatever
CellTypist called them); a handful showed 5-66% (real immune signal) -- an order-of-magnitude gap,
not a subtle borderline case.

**Fix built and tested**: `src/mg_thymus_map/annotate/immune_contamination.py`
(`correct_immune_mislabels`) -- for each Leiden cluster whose PTPRC signal says non-immune but
whose CellTypist plurality label claims otherwise, relabels every cell in that cluster to
whichever structural marker panel (epithelial/fibroblast/endothelial) fits best, a cluster-level
decision rather than a per-cell threshold (real per-cell CD3D dropout makes a single-gene per-cell
rule unreliable -- checked directly: even genuine CD3D+ T cells show a nonzero keratinocyte score
from ambient RNA, and the CD3D-negative "T cells" subset's keratinocyte score sits on a continuum,
not a clean bimodal split). 6 unit tests, all passing.

**Real result of the fix**: T cells 9,291 -> ~3,300-4,400 (real per-run variation, see below);
ILC 7,056 -> 377; Epithelial cells 18,583 -> ~22,000-23,000; Fibroblasts 12,182 -> ~21,000. Overall
non-immune fraction went from 64% to ~86% -- much more consistent with real whole-skin-digest
biology (immune cells are a minority of dissociated skin, typically 10-25% in the literature) than
the raw CellTypist output.

**A second real problem found while validating the fix, not swept under the rug: the correction
was a silent no-op on one rerun.** Ran the identical Step 2 script twice on identical code/data —
got 28 clusters the first time, 31 the second, and the second run's `flag_non_immune_clusters` (a
"largest gap in the distribution" heuristic) found no gap large enough to clear its threshold,
silently returning zero corrections instead of erroring. Root cause: `harmonypy.run_harmony`'s
internal k-means initialization isn't seeded by this project's `integrate_with_harmony` wrapper —
neither `sc.tl.leiden` nor scanpy's Scrublet wrapper is the culprit (both default to a fixed
`random_state`). **Same class of issue as an already-fixed Phase 1 bug** ("Pin liana's random seed
explicitly instead of relying on its implicit default") — not patched at the Harmony-wrapper level
this time, since that's shared code RA/OA's already-audited results depend on and a bigger change
than this specific problem warrants right now. Instead, `flag_non_immune_clusters` got a
`fallback_fraction_threshold` (3%, chosen with real margin inside the gap both real runs showed:
non-immune clusters stayed under 1.4% in both, immune ones started at 5.2%+ in both) — robust to
a noisier clustering run even without a clean gap. Verified against a synthetic no-gap case (new
unit test) and confirmed the fix still fires correctly on the real SSc data after this change.
**Practical implication for interpreting exact cell counts here**: because Harmony's clustering
isn't yet reproducible run-to-run, exact post-correction cell-type counts will vary by roughly
±15% between runs (see the T-cell range above) — the qualitative finding (contamination is real,
severe, and now corrected) is solid; the precise final numbers should be read as "this order of
magnitude," not exact until Harmony is seeded, which is a real, stated limitation now, not a
silently-accepted one.

**Final clean run, 2026-09-12, with the fallback threshold in place — this is the version saved to
`data/interim/ssc_harmonized.h5ad` (both `majority_voting` and `cell_type_corrected` columns
present, use the latter downstream):** T cells 9,291 → 3,308; ILC 7,056 → 376; B cells 329 → 318;
Macrophages 2,395 → 1,970; DC 1,352 → 1,347; Mast cells 569 → 569 (untouched, no flagged cluster
had Mast cells as plurality); Epithelial cells 18,583 → 23,063; Fibroblasts 12,182 → 21,061;
Endothelial cells 6,804 → 6,625; Erythroid 148 → 73; Plasma cells 54 → 53. Final non-immune
fraction: 50,749 / 58,763 = 86.4%.

**Real timeline, updated 2026-09-12 (today is the Sep 12–13 checkpoint `02-phase2-ra.md` §5 called
for):** ~15–16 days of usable project time remain before the ~Sep 27–28 cutoff. Phase 2 (RA) is
functionally done as of today (`docs/phase2_findings.md`/`phase2_methods.md`/`phase2_full_story.md`
written). Phase 3 has not started (no data downloaded, no code written, for either UC or SSc) —
real risk if both the mentor-dependent UC path and a fresh SSc path both drag on. **Practical
recommendation: start `GSE138669` now** (fully open, can proceed immediately) **while UC waits on
the mentor**; if SSc turns out strong on its own (pending the TLS-literature check in §5), that
may resolve the "which third disease" question without needing to wait on UC at all.

## 8. 2026-09-15 — PF's patient-heterogeneity investigated; a cross-dataset QC finding; where next

**PF's TLS-region cell concentration in a few patients, investigated — no clean diagnosis-subtype
explanation, but confirmed real, not a sample-size artifact.** Joined the specificity table against
the real per-cell `Diagnosis`/`Sample_Source` metadata: the top 3 contributing patients
(TILD015=133 cells, VUILD62=39, VUILD60=33 — 62% of all 331 disease TLS-region cells from just 3
of 17 patients) are a mix of IPF and NSIP, not one coherent subtype. IPF as a whole is 240/331
(72.5%) of disease TLS cells, roughly matching its share of the cohort, but shows huge internal
spread (1 to 133 cells per patient) — the concentration isn't explained by the coarse diagnosis
label. **Confirmed this is a real rate difference, not just "TILD015 happened to have more cells
total"**: TLS-region cells as a % of each sample's own total cells shows TILD015 at 2.11%, 2-4x
every other sample (next highest: VUILD55 at 0.99%) — VUILD59, for comparison, has the *most*
total cells of any sample (14,148) but the *lowest* TLS rate (0.02%). Real, unexplained
patient-level heterogeneity — plausibly consistent with documented IPF literature on variable
lymphoid-aggregate density, but this dataset's metadata doesn't have a field (fibrosis stage,
biopsy site) that would confirm that specifically. Flagged as a real open thread, not resolved.

**Cross-dataset QC finding: the pooled doublet-threshold tail-landing problem is not RA-specific.**
Checked directly (lightweight QC-only reprocess of SSc and PF, no CellTypist/Harmony needed) —
both show the same pattern RA's plan doc flagged as a possible one-off (§7 item 10 there): the
pooled KDE-valley threshold lands past the real-cell 99th percentile. SSc: 0.2770 vs. p99=0.1972.
PF: 0.3798 vs. p99=0.1842, with **8 of 30 samples getting zero flagged doublets at all** (a more
extreme version of RA's 5/25). Phase 1's original three datasets (38K-83K cells) never showed
this; every dataset since (RA/SSc/PF, all 59K-109K cells) does — looks like a general weakness of
the pooled-valley method at larger cell counts, not a per-dataset quirk. **Deliberately not fixed
today**: the threshold applies identically to both arms of each disease-vs-control comparison, so
it under-flags symmetrically — a real, now-generalized QC limitation worth stating plainly in the
eventual write-up, but not a plausible explanation for any of today's actual results (SSc's real
fibroblast finding, or the negative TLS-transfer tests) in either direction. A proper fix would
touch shared code Sjögren's own confirmed result also depends on — worth doing carefully, not
under today's time pressure.

**Where next, given the user now has UC's downloaded data and wants to fully close out one disease
before starting another:** SSc is functionally complete through Step 6 with a real, defensible
positive finding (the 14-gene fibrosis signature + brensocatib) — the natural point to write up its
formal `docs/phase3_ssc_findings.md`/`methods.md`/`full_story.md` (matching Phase 1/2's pattern)
before starting UC, rather than leaving it as scattered plan-doc entries. PF lung stays paused at
Step 4 (real but unconfirmed TLS signal, Steps 5-6 not run) — not abandoned, just not the next
priority given the user's own stated preference for serial completion over parallel partial
progress. UC is the next disease to start once SSc's write-up is done, chosen over further PF work
because gut-associated lymphoid tissue has much stronger independent literature support for real
TLS/lymphoid-follicle biology than either SSc skin or general PF lung did — a fair reason to
prioritize it, not an attempt to engineer a particular result.
