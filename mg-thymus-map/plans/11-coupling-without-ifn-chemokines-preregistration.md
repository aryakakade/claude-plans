# Pre-registration — Phase 8: does the interferon ↔ TLS coupling survive without the map's interferon-inducible genes?

**Written 2026-09-19, before any statistic was computed. Frozen on commit.** Changes only as dated
addenda in §7. Additive and deletable (`phase8_*` files, its script, its run log, this document).
Every earlier result stands as reported whatever this finds; official Sjögren's and SSc files are
read only.

## 1. Why

Phase 6 found that patients with a stronger fibroblast type-I interferon programme have higher
MG-map TLS scores (ρ = +0.26, p = 0.0048, 110 patients). The obvious objection: the 25-gene map
contains three chemokines that interferon itself induces — **CXCL9, CXCL10, CXCL11** — so part of
the link could be interferon correlating with interferon. This test removes them.

## 2. Hypothesis

**H1:** the Phase 6 coupling holds when the TLS score uses the 22-gene map (25 minus CXCL9,
CXCL10, CXCL11). **H0:** no positive coupling with the 22-gene map.

## 3. Method — Phase 6 unchanged except the TLS score

- **x (interferon):** `phase4_ifn_composite.csv`, unchanged, same five lines as Phase 6.
- **y (TLS):** per-patient median AUCell score of TLS-region cells (GC-B | Tfh) with the **22-gene
  map**, computed now. **Whole-object scoring, then cell selection** (the Phase 7 lesson). Same
  patient units and arms as Phase 6. Cell states: Sjögren's — Phase 1 calls in the saved object;
  UC — the calls saved by UC Step 4; RA/OA — `call_tls_region_states` on `majority_voting` as in
  Phase 2 Step 4; PF — raw rebuild restricted to Step 2's cells with its corrected labels copied,
  calls on `cell_type_corrected` (as Phases 4–5); SSc — fresh reprocess (the Phase 3 chain), calls
  on `cell_type_corrected`.
- **Same-run reference:** in the same run and on the same cells, the **full 25-gene** medians are
  also computed and the Phase 6 test repeated on them. This separates the effect of removing the
  three genes from run-to-run variation (SSc's reprocess is not bit-identical, because Harmony is
  unseeded).
- **Statistic:** within-dataset percentile ranks; pooled Spearman; stratified permutation (x
  shuffled within dataset), 10,000 permutations, seed 2026, **one-sided positive**. **H1 supported
  at p < 0.05.**
- **Secondary (reported, not pass criteria):** per-dataset ρ; leave-one-dataset-out; bootstrap 95%
  CI for ρ (10,000 resamples within dataset).
- **Multiplicity:** this becomes test 11 in the Phase 7 family; BH q is reported across all 11.

## 4. Outcomes

- p < 0.05: "the coupling does not depend on the map's interferon-inducible chemokines" —
  Finding 3 strengthened.
- p ≥ 0.05: "the coupling is carried at least partly by interferon-inducible genes in the map" —
  Finding 3 is reported with that qualification.

## 5. Not allowed

No other gene removals; no other signature; no dropping datasets or patients; no two-sided
switch; no substituting official medians for the 22-gene line.

## 6. Outputs

`data/processed/phase8_coupling_22gene_patients.csv`, `phase8_coupling_22gene_results.csv`;
script `scripts/run_phase8_coupling_22gene.py`; log `docs/phase8_coupling_runlog.md`.

## 7. Addenda (dated; the text above is frozen)
