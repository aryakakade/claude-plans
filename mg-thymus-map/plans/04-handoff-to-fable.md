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

**Real, honest read: the literature-prior half is carrying the actual signal; the MG-derived half
(found via `liana-py` cell-communication analysis run only on MG's own data — see
`docs/phase1_methods.md` §3) looks MG-specific rather than universally portable, and mixing it
into the full signature appears to dilute a real signal in at least RA's case.** RA's p=0.056
is *suggestive*, not confirmed — say that plainly if you report it, don't round it up to a
finding. This was NOT checked for SSc or PF yet (both would need a fresh reprocess since their
full-gene TLS-scored objects were never cached to disk — see §4 below).

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
