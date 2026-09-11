# Phase 3 Plan — Lupus Nephritis Extension

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
