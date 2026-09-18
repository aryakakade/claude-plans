# Literature search — can the other four diseases reach the Sjögren's drug set or Aim 1? (2026-09-18)

**User's question:** can Aim 1 be confirmed for RA, SSc, PF, UC, and is there any literature route
to saying they also have JAK-inhibitor / anifrolumab-type drugs — did we miss something crucial?
**Answer in one line:** no legitimate route confirms Aim 1 for them (pre-registered tests are
final), but the literature supplies (a) a verified mechanistic bridge between Aim 1 and Aim 2 that
explains *why* Sjögren's is the match, and (b) a JAK-inhibitor evidence line for every disease,
reached by different upstream cytokines. Every paper below was verified by PMID on Europe PMC.

## The bridge we had not connected

**Cufi et al. 2014, *J Autoimmun*, PMID 24393484 — "Central role of interferon-beta in thymic
events leading to myasthenia gravis":** IFN-β "increased the expression of the chemokines CXCL13
and CCL21" by thymic epithelial cells and, in vivo, "favored the recruitment of B cells"; it also
induced α-AChR (the MG autoantigen). Also Cufi et al. 2013 *Ann Neurol* PMID 23280437 (dsRNA/poly(I:C)
signalling); Sikorski et al. 2025 *Mediastinum* PMID 40666538 (hyperplastic MG thymus "exhibits
heightened interferon (IFN) signaling"). **So in MG, type-I IFN is upstream of the thymic TLS.**
That links the two aims: Aim 2's type-I IFN programme is MG's own TLS driver.

| disease | type-I IFN stromal programme (Phase 4) | MG-map TLS (Aim 1) | reading |
|---|---|---|---|
| MG (literature) | IFN-β drives thymic CXCL13/CCL21 → B-cell recruitment | — (reference) | driver + TLS-competent tissue |
| **Sjögren's** | present (p = 0.007) | **confirmed** | driver + TLS-competent tissue → MG-like |
| SSc skin | present (p = 0.002) | not confirmed | driver present; skin does not build GC-TLS (SSc TLS reported in lung) |
| UC | absent (all compartments) | not confirmed | TLS present, different drivers (IFN-γ, OSM/IL-11) |
| PF | borderline (p = 0.086) | not confirmed | weak driver; IPF aggregates lack germinal centres |
| RA | untestable (OA n = 4) | not confirmed | IFN signature only in a patient subset |

Hypothesis this generates (not a claim): **MG-like TLS appears where the type-I IFN driver meets a
tissue that builds germinal-centre TLS.** Sjögren's is the only disease tested with both.

## Per-disease drug and TLS literature

**RA.** JAK inhibitors approved (tofacitinib, van Vollenhoven et al. 2012 NEJM PMID 22873531;
baricitinib, upadacitinib, filgotinib). No anifrolumab. Type-I IFN signature only in a subpopulation
(van der Pouw Kraan et al. 2007 PMID 17223656). Ectopic lymphoid structures only in the
lympho-myeloid synovial pathotype (Humby et al. 2009 PLoS Med PMID 19143467; 2019 ARD PMID 30878974);
B-cell-poor patients respond better to tocilizumab than rituximab, 63% vs 36% (R4RA, Humby et al.
2021 Lancet PMID 33485455; Rivellese et al. 2022 Nat Med PMID 35589854). **Missed angle:** our RA
test averaged all patients (10/25 not even RA), diluting any lympho-myeloid subset.

**SSc.** Anifrolumab phase 3 DAISY, 306 patients, primary endpoint revised CRISS-25 at 52 weeks
(Khanna et al. 2024 Clin Exp Rheumatol PMID 39152751); rationale review Radić 2026 PMID 41682782.
JAK: tofacitinib vs cyclophosphamide RCT in early diffuse SSc, greater mRSS reduction (Amin Khan
et al. 2026 Cureus PMID 41809291 — small, single-centre, low-impact journal); upadacitinib case
report (PMID 40520078). TLS: lung (SSc-ILD, Zhu 2026), not skin (Geroldinger-Simić 2026).
**Missed angle:** tissue — our data is skin; the TLS literature is lung.

**PF.** No approved JAK use in IPF; JAK2 inhibition antifibrotic in preclinical IPF models (Milara
et al. 2018 Thorax PMID 29440315; Respir Res PMID 29409529); tofacitinib used clinically in
dermatomyositis-ILD (Chen et al. 2019 NEJM PMID 31314977, case). Lymphocyte aggregates accumulate
with disease (Todd et al. 2013 PMID 23576879) but IPF lymphoid structures are "devoid of … germinal
centers", with variable B-cell follicles (Marchal-Sommé et al. 2006 J Immunol PMID 16670278).
Type-I IFN tracks *less* fibrosis (Nathwani 2026 PMID 42627972). **Missed angle:** heterogeneity
(3 of 17 patients supplied 62% of our PF TLS-region cells) — but IPF structures lack GCs, so an
MG match is not expected biologically.

**UC.** JAK inhibitors approved (tofacitinib PMID 28467869; upadacitinib). No anifrolumab. Basal
lymphoid aggregates correlate with inflammation severity, r = 0.9 (Sipos et al. 2010 PMID
20132083), with "abnormal follicular architecture" (Yeung et al. 2000 Gut PMID 10896913); review
McNamee 2016 PMID 27579025. Our data agrees (3.3% vs 1.4% TLS-organizing cells). Drivers: IFN-γ,
OSM/IL-11 (our Check B; West 2017 PMID 28368383).

**MG itself.** Tofacitinib in 19 refractory MG patients reduced glucocorticoid need and improved
scores via Th17.1/JAK-STAT3 (Zhao et al. 2026 Neurotherapeutics PMID 41547654).

## Honest limits for the write-up

- JAK inhibitors are used across many immune diseases, so sharing the *class* is weak evidence of
  shared biology; the specific statement is the *upstream route* (type-I IFN → anifrolumab applies
  only to Sjögren's, SSc, plausibly MG).
- None of this changes a pre-registered outcome. Aim 1 remains 1 of 5.

## Proposed last test (not run; needs the user's go)

**IFN–TLS coupling, pre-registered, deletable:** within each dataset, do patients with a stronger
fibroblast type-I IFN composite (Phase 4 file) have higher MG-map TLS scores (Step 4 patient
medians)? Directly tests the Cufi-derived bridge on data already on disk. Small n per disease and
shared "global inflammation" confounding are stated limits; it cannot confirm Aim 1.
