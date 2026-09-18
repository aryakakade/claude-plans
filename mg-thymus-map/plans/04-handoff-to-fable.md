# Handoff to Fable — where things stand, and what to do next

**Written 2026-09-15 by Claude Sonnet 5, session
`claude.ai/code/session_01BXjsokGjdHP7hTWKcme1sV`, at the user's explicit request** ("I would like
fable to do all of this... make a doc talking about the plan so that fable knows what to do
next"). This is the entry point — read this first, then follow the pointers below into the full
detail. Everything referenced here is already committed to git (both `mg-thymus-map` and
`claude-plans` repos).

## 1. What this project is, in one paragraph

An ISEF/IRIS science-fair project (`docs/project-introduction.txt`) testing whether autoimmune
diseases build tertiary lymphoid structures (TLS) using the same shared immune architecture MG
(myasthenia gravis) uses in the thymus (Aim 1), then using that shared signal as a subtractive
baseline to find disease-specific stromal drug targets (Aim 2). Full roadmap: `00-overview.md`.
The user is 15, doing this largely solo with an ISEF Adult Sponsor (parent) helping only with
account-gated downloads — see `[[user-technical-background]]`-style context in memory if you have
access to it, otherwise: don't assume CS/computing fluency, do assume real scientific judgment.

**Real deadline**: submission 2026-10-03; user's exams start 2026-10-01 (5 exams) — true usable
cutoff is ~2026-09-27/28, not the literal submission date.

## 2. Current state, phase by phase

- **Phase 1 (Sjögren's vs. MG): complete, confirmed positive.** Aim 1 confirmed (p=0.003,
  patient-level). Aim 2 found a real ISG-driven stromal signature + anifrolumab/JAK-inhibitor
  pharma hit. Docs: `docs/phase1_findings.md`/`methods.md`/`full_story.md`.
- **Phase 2 (RA): complete, honest null + real secondary finding.** Aim 1 not confirmed
  (p=0.625). A reframed intra-RA lining-vs-sublining analysis found a real positive (MAPK11/
  LRRC32 pharma candidates). Docs: `docs/phase2_findings.md`/`methods.md`/`full_story.md`, plan:
  `02-phase2-ra.md`.
- **Phase 3, SSc (`GSE138669`): complete through Step 6, real positive finding.** Aim 1 not
  confirmed, cleanly (bootstrap CI includes zero — a cleaner negative than RA's, whose CI excluded
  zero). Aim 2 found a real, strong positive: 14/15 genes confirmed BH-significant (POSTN/BGN/
  THY1/SPARC/PCOLCE/ASPN/IGFBP4/NNMT/CTSC + IFITM1/2/3), plus one verified pharma candidate (CTSC
  → brensocatib, a real Phase II/III DPP1 inhibitor, never tested in SSc). Two real methodology
  bugs found and fixed along the way (CellTypist immune-mislabeling on skin; a stromal-compartment
  definition that included keratinocytes). Docs: `docs/phase3_ssc_findings.md`/`methods.md`/
  `full_story.md`, plan: `03-phase3-ssc.md`.
- **Phase 3, PF lung (`GSE135893`): paused at Step 4, not abandoned.** Loader + Step 2 done. Step
  4 shows a striking cell-level TLS-transfer signal that does NOT survive patient-level testing
  (p=0.241) — the third time this exact pseudoreplication pattern has shown up in this project
  (after RA's own Step 4 and OA-vs-tonsil). A real, unexplained patient-heterogeneity thread
  (3 patients contribute 62% of TLS-region cells) was investigated but not resolved — no clean
  diagnosis-subtype explanation, confirmed to be a real rate effect not a sample-size artifact.
  Steps 5-6 not run. Plan detail: `03-phase3-ssc.md` §6, §8 (yes, PF's detail lives in the file
  named for SSc — see that file's own header for why).
- **UC (Smillie et al. 2019, `SCP259`): not started, but the user now has the data.** Blocked
  for a while on Terra requiring 18+ (the user is 15); the user's dad (ISEF Adult Sponsor)
  completed the sign-in and download. **You need to ask the user how to access the actual files**
  — get the bulk-download command they used, or wherever the files landed locally. Full search
  history: `03-phase3-ssc.md` §2c, §4-§5 (the UC/lupus/PF candidate-search record).

## 2b. UPDATE, later on 2026-09-15 (Fable) — SSc now matches Sjögren's pathway and drug; goal met

The user's stated goal was one more disease matching Sjögren's on "the same pathway, the same
drug," on time, without messy statistics. **Done, via SSc's Aim 2, with Sjögren's untouched.**
Applied Phase 1's own upstream type-I-IFN pathway query to SSc → anifrolumab + 16 approved JAK
inhibitors (Phase 1's exact set), then earned the claim non-circularly: an a-priori panel of 12
canonical ISGs (Phase 1's own IFI6/IFI44L/XAF1 included; the discovery genes IFITM2/IFITM3
excluded), fibroblasts only, patient-level, one-sided, BH within family — **10/12 cleared, 11/12
up, all three Sjögren's ISGs confirmed in SSc.** Two citations verified against real abstracts
(Bryon 2025 PMID 39415484; Radić 2026 PMID 41682782). Full record: `03-phase3-ssc.md` (Step 6
pivot + Step 7), `docs/phase3_ssc_findings.md` §4b.

**Headline for SSc**: the same type-I-IFN drug target, reached through a disease-specific stromal
residual, even though the TLS architecture (Aim 1) did not transfer — which stays a clean
negative, reported as such.

**Consequences for the rest of this doc**: §4a (tonsil expansion) is DONE as a rebuild
(`tonsil_with_cell_states_8donors.h5ad`, 61,069 cells, TLS 4.72% vs 4.67%) but the side-by-side
re-run is deliberately NOT run — the user instructed that Sjögren's vs. MG not be touched; the
4-donor reference stays primary everywhere. §4b (UC) is now optional — the only remaining honest
shot at Aim 1, not needed for the drug goal, and expensive (~366K cells). Recommended use of the
remaining time: turn `docs/phase3_ssc_findings.md` from its bullet outline into submission-ready
prose, then the same for Phase 2 if time allows. Start UC only if the user explicitly wants Aim 1
tested once more and provides the data access.

## 3. A real, important methodological finding from today — read before doing anything else

**Three diseases tested (RA, SSc, PF) have all failed to confirm Aim 1 (TLS-signature sharing) at
the patient level, despite real variation in how much genuine TLS biology each tissue has.** This
is not obviously "wrong disease every time." A diagnostic run today
(`scripts/analyze_signature_subset_performance.py`, results in
`data/processed/signature_subset_diagnostic.csv`) split the 25-gene MG-derived signature into its
two provenance halves (`data/processed/tls_signature.csv`'s `provenance` column: 13
`literature_prior` genes vs. 12 `mg_derived` genes) and re-ran the patient-level test with each
half alone:

| Disease | Full 25-gene | Literature-prior (13) alone | MG-derived (12) alone |
|---|---|---|---|
| Sjögren's (confirmed) | p=0.012 | p=0.039 (still significant) | p=0.842 (noise) |
| RA | p=0.625 | **p=0.056 (borderline)** | p=0.987 (worse than noise) |
| OA | p=0.129 | p=0.786 | p=0.814 |
| SSc (added later 2026-09-15, fresh reprocess) | p=0.584 | p=0.115 | p=0.923 |
| PF lung (same) | p=0.291 | p=0.502 | p=0.795 |

**Real, honest read: the literature-prior half is carrying the actual signal; the MG-derived half
(found via `liana-py` cell-communication analysis run only on MG's own data — see
`docs/phase1_methods.md` §3) looks MG-specific rather than universally portable, and mixing it
into the full signature appears to dilute a real signal in at least RA's case.** RA's p=0.056
is *suggestive*, not confirmed — say that plainly if you report it, don't round it up to a
finding. SSc and PF were checked later the same day (rows above): neither approaches
significance with the literature-prior half (SSc 0.115, PF 0.502); the MG-derived half is noise
in both. The literature-prior subset alone therefore forms a gradient — Sjögren's 0.039, RA 0.056,
SSc 0.115, PF 0.502, OA 0.786 — in which only Sjögren's clears. This bears directly on UC: the
frozen pre-registration (`05-uc-preregistration.md`) keeps the full 25-gene signature as the
primary test and allows the literature-prior subset only as a labeled exploratory secondary.

## 4. Two concrete, already-scoped next steps (do these before UC, or alongside it)

### 4a. Tonsil reference expansion (4 → 8 donors) — a real power improvement, not p-hacking

Every single patient-level test in this project (Phase 1 through the diagnostic above) has
compared against only **4 tonsil donors**, chosen originally just to keep tonsil roughly the same
scale as Phase 1's other two datasets — not because more weren't available.
`src/mg_thymus_map/data/healthy_tonsil.py`'s own docstring already says the full atlas has more.
**Checked directly today: 8 real, unambiguous single-donor libraries are usable right now, no new
download needed** (the raw data is already local, 4.2GB in `data/raw/healthy_tonsil/`) — 5 male /
3 female. The rest of the full atlas uses multiplexed "hashed" libraries needing HTO
demultiplexing to resolve per-cell donor identity, out of scope for a quick expansion.

**Why this is legitimate and not cherry-picking**: it's a general, disease-agnostic increase in
the reference group's sample size, applied uniformly to every past and future comparison — it
doesn't touch what counts as significant, only the power behind "what does normal lymphoid tissue
look like." The user explicitly proposed this themselves.

**Status 2026-09-15 (Fable, same session after the model switch): scripts written and committed,
rebuild in flight.** `scripts/rebuild_tonsil_reference_8donors.py` writes a NEW
`data/interim/tonsil_with_cell_states_8donors.h5ad` (4-donor file left untouched);
`scripts/rerun_step4_against_8donor_tonsil.py` then recomputes every Step 4 test against both
references, full-25 and literature-prior-13, into
`data/processed/step4_tonsil_4_vs_8donor_comparison.csv`. Confirmed before launching: all 8
donors' raw matrices present on disk; the cached 4-donor object used donors BCLL-10/11/12/14-T
(38,072 cells, 1,778 TLS-region), so the rebuild adds BCLL-2/8/9/13-T. If the rebuild has
finished when you read this, run the re-run script next; if it hasn't, check for the 8-donor
h5ad before assuming.

**Why it's real, bounded work, not a quick tweak**: tonsil is the shared reference for every
result reported so far. Doing this properly means:
1. Reload tonsil with `max_donors=8` (or reload without a cap and confirm exactly 8 come back),
   run the same QC/CellTypist/Harmony chain, recompute `is_gc_b_cell`/`is_tfh` cell states.
2. **Re-run every downstream Step 4 test that depends on the tonsil reference** — Sjögren's
   (Phase 1's own confirmed result!), RA, OA, SSc (and eventually PF, UC) — to get updated
   p-values against the larger reference.
3. This WILL change Sjögren's own reported numbers, even if only slightly. Be transparent about
   that in whatever doc you update — don't silently swap the number, show the before/after and
   say why.
4. Do this **carefully, not rushed** — re-opening Phase 1's confirmed result is exactly the kind
   of thing that needs the same audit discipline that result already got once.

### 4b. Test UC (and reconsider RA) with the literature-prior-13-gene subset, not just the full signature

Given §3's finding, when you get UC data loaded and reach Step 4, run the cross-disease projection
**twice**: once with the full 25-gene signature (for continuity with every prior phase), and once
with just the 13 `literature_prior` genes. If UC has real gut-lymphoid-tissue TLS biology (it has
the strongest literature support of anything tried so far — see `03-phase3-ssc.md` for why), the
literature-prior subset is the more honest test of whether that transfers, given what §3 found.

**Do not go further and retroactively "fix" the signature to only use literature_prior genes
everywhere** without the user's explicit sign-off — that changes Phase 1's own definition of the
signature after the fact, which needs to be a deliberate, disclosed decision, not something slid
in quietly because it makes later results look better.

## 5. Standing rules, already established, do not relitigate without cause

- **No p-hacking.** The user asked directly whether the pipeline should be "optimized" so more
  diseases confirm. The answer given, and the one to keep giving: no — tuning statistics,
  cell-calling, or compartment definitions until something confirms would undermine the whole
  project's credibility, including Sjögren's real result. The cell-level-vs-patient-level
  discrepancy (caught 3x now: RA, OA, PF) is the safeguard working correctly, not a bug.
- **The user wants one disease fully completed (write-up and all) before the next starts.** SSc
  got that treatment (§2). Do the same for UC once it's run — don't leave it as scattered
  plan-doc entries.
- **Every real finding gets written down, including negative ones** — this project's own
  long-standing convention, visible throughout `03-phase3-ssc.md` and the docs/ folder. Keep it
  up: real bugs, real dead ends, real caveats, not just the clean final numbers.
- **Division of labor on gated platforms**: anonymous public-metadata checks are fine to do
  directly; anything needing login goes through the user (or their Adult Sponsor). Never suggest
  or normalize a second account/identity to route around an age or access gate — that's a real
  integrity risk for a competition submission, not a technicality. (This came up for UC/Terra
  already — resolved correctly, the user's dad did it under his own identity.)
- **Citations get verified, not assumed** — every pharma hit, every TLS-literature claim in this
  project has been checked against a real source before being reported (see brensocatib's
  verification in `docs/phase3_ssc_full_story.md` for the standard to match).

## 6. Where the detail actually lives (don't re-derive what's already written down)

- `00-overview.md` — project roadmap, scientific aims, original source doc.
- `01-phase1-sjogrens-pilot.md`, `02-phase2-ra.md`, `03-phase3-ssc.md` — the full, real,
  blow-by-blow development record for each phase, including every bug and dead end.
- `docs/phase{1,2,3_ssc}_{findings,methods,full_story}.md` — the clean, formal write-ups.
- `data/processed/signature_subset_diagnostic.csv` — today's raw diagnostic output.
- Memory (if you have access to the same memory store this session used): search for
  `mg-thymus-map` entries — `mg_thymus_map_phase3_uc_pivot.md` has the most recent, dense summary
  of exactly where things stand as of this handoff.

## 7. UPDATE 2026-09-16 00:30 IST — UC done: Aim 1 NOT confirmed (pre-registered, p = 1.000)

UC data (SCP259) landed 2026-09-15 23:37 IST; Steps 1, 2 and 4 ran overnight, exactly per
`05-uc-preregistration.md` (see its Addendum 2 for the outcome). Primary: 18/18 UC patients
contributed, UC (inflamed) vs. tonsil one-sided exact permutation **p = 1.000** — UC's TLS-region
cells score *below* tonsil (every patient below every donor). Not confirmed; Steps 5–7 not run;
**Aim 1 scoreboard 1/5 (Sjögren's only).** Exploratory lit-prior-13 line: UC above tonsil
(p = 0.0012) *but so is healthy colon* (p = 0.0033) — not UC-specific, no claim. Full timestamped
history: `mg-thymus-map/docs/phase3_uc_runlog.md` (branch `worktree-ssc-findings-prose`,
unmerged). Real, reportable descriptive biology: inflamed UC colon has 3.26% TLS-organizing cells
vs. 1.24% non-inflamed / 1.35% healthy / 4.67% tonsil (9× the GC-B cells of non-inflamed) — TLS
are *present* in UC; they are just not drawn on the MG map. Next: write-up only.

## 8. UPDATE 2026-09-18 — science complete; one correction; write-up-ready drug finding

**Phases added after §7, all pre-registered, all deletable extras, Sjögren's/SSc official files
untouched (user's hard rule):**
- **Phase 4 — type-I IFN fibroblast programme** (`06-ifn-convergence-preregistration.md`):
  present in **2 of 5** — Sjögren's (p = 0.007, 9/9 non-discovery genes up) and SSc (p = 0.002,
  reproduces Step 7); PF borderline (p = 0.086); UC absent (p = 0.78); RA underpowered.
- **Phase 5 — MG-vs-tonsil contrast signature** (`07-contrast-signature-preregistration.md`):
  **batch-driven, 0 of 5** (stress/annotation genes; healthy tissues also above tonsil). Future
  work needs a same-study healthy thymus.

**Correction to §7.** §7's "every UC patient below every tonsil donor" was largely a sequencing-
depth artifact: against depth-matched tonsil, UC is indistinguishable (p = 0.74). Aim 1 is still
not confirmed in UC (see `05-uc-preregistration.md` Addendum 3). SSc has the same depth handicap;
a depth-matched SSc check was run 2026-09-18 (result appended below when complete).

**Write-up-ready finding — "same drug class, different mechanism".** Type-I IFN is absent in
every UC compartment, so anifrolumab has no rationale in UC. Inflamed UC instead up-regulates
IFN-γ (epithelium, macrophages) and the OSM/IL-11 inflammatory-fibroblast programme (fibroblasts,
epithelium, T cells, macrophages). All signal through JAK1/JAK2, and JAK inhibitors are already
approved for UC (tofacitinib PMID 28467869; upadacitinib). Sjögren's and SSc reach JAK
inhibitors through type-I interferon (which also makes anifrolumab relevant); UC reaches them
through IFN-γ and OSM/IL-11, matching clinical use. This is a stronger, more precise statement
than "UC doesn't share the drug" and belongs in the discussion.

**Final scoreboard:** MG map transfers 1/5 (Sjögren's); type-I IFN stromal programme 2/5
(Sjögren's, SSc) → anifrolumab + JAK inhibitors; UC reaches JAK inhibitors by a different
cytokine route; contrast signature batch-driven. **Next: write-up only.**

**SSc depth check result (2026-09-18, exploratory, `mg-thymus-map/docs/exploratory_depth_checks.md`).**
SSc TLS-region cells: median 1,982 UMIs / 748 genes vs tonsil 3,378 / 1,471. Against tonsil thinned
to SSc depth (5 seeds), SSc goes from diff −0.006 (p = 0.676) to diff 0.000 to +0.018, **p = 0.11–0.47
— still not confirmed**; healthy skin p = 0.38–0.76. The fresh reprocess reproduced the official
SSc medians exactly. Depth handicaps shallow datasets modestly but rescues neither SSc nor UC.
**Aim 1 scoreboard unchanged: 1 of 5.**
