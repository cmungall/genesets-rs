# cGSA-derived curated interpretations

Curated GO interpretations of gene sets drawn from the **cGSA benchmark** (Wang
et al., *Knowledge-guided contextual gene set analysis with large language
models*, Bioinformatics 2026, btag214; https://github.com/ncbi-nlp/cGSA). These
are kept in a **separate corpus** from `curation/genesets/` on purpose — see
"Why isolated" below.

The source data is ingested under `evals/cgsa/` (see that directory's README).
Each set here is one differentially-expressed-gene (DEG) module from the
benchmark, curated with the same discipline as the native corpus
(`curation/README.md` is the authority): every association grounded in the
**actual** DEG membership, every ontology id OLS-verified and swept for
obsolescence, biology-first `category`, orthogonal `recovery_status`, and
`insight`.

## Layout
- `genesets/CGSA_<SET>.yaml` — one interpretation per DEG module.
- `genesets/cgsa_members.gmt` — the module memberships (from the benchmark).
- `genesets/manifest.tsv` — index (same columns as the native manifest).

## Current sets

Eight DEG modules across six studies, spanning melanoma, glioblastoma, diabetic
cardiomyopathy and Tfh-cell regulation, in both human and mouse.

**PMC10202813 melanoma modules** — one study whose DEGs the benchmark splits into
modules, linked as `SERIES:CGSA_PMC10202813_MODULES` to show that distinct modules
of one study resolve to **contrasting** GO interpretations:

| set | series_role | defining biology | grounding note |
|-----|-------------|------------------|----------------|
| `CGSA_PMC10202813_3_INTERFERON` | interferon_response | type I/II IFN response, antiviral defense | clean ISG panel (OAS/MX/IFIT/ISG15) — `annotation_supported`, confirmatory |
| `CGSA_PMC10202813_6_PIGMENT` | pigmentation_and_proliferation | melanin biosynthesis + melanosome | pigment core is **thin** (OCA2), co-mixed with a proliferation signal (CDC45/AURKB/PBK) |
| `CGSA_PMC10202813_2_SENESCENCE` | senescence_and_immune | cellular senescence + T-cell activation | two coherent arms (CDKN2A/TP53; PTPRC/TIGIT/SELL) plus melanoma drivers |

**PMC9232499 glioblastoma modules** — paired arms of one study, linked as
`SERIES:CGSA_PMC9232499_MODULES`:

| set | series_role | defining biology | grounding note |
|-----|-------------|------------------|----------------|
| `CGSA_PMC9232499_1_GBM_MIGRATION` | migration_ecm | cell migration + ECM remodeling | clean (Mmp9/Mmp10/Col1a1/Adamts2/Cyr61) |
| `CGSA_PMC9232499_2_GBM_STRESS` | stress_defense | innate/IFN defense response | GT's broad "response to stress" recorded `nonspecific`; the informative arm is Gbp1/Ciita/Fgl2 |

**Standalone modules:**

| set | defining biology | grounding note |
|-----|------------------|----------------|
| `CGSA_PMID30017245_MELANOMA_MITOSIS` | mitotic cell cycle | 12/12 canonical mitotic markers (CDK1/CCNB1/AURKB/PLK1/…) |
| `CGSA_PMC8831505_1_TFH_HYPOXIA` | hypoxia + glycolysis | HIF/glycolytic block (Slc2a1/Ldha/Hk2/Pdk1/Bnip3) in Tfh cells |
| `CGSA_PMC10761883_2_DIABETIC_CM_HIF` | hypoxia/HIF + PPAR/lipid | HIF targets (Bnip3/Adm/Angptl4/Vegfa) + glycolysis; KEGG "HIF-1 signaling" → GO response to hypoxia |

Several modules were chosen precisely because they are **mixed** DEG clusters:
they exercise the membership-grounding rule (the paper's top ground-truth term
can be carried by very few set members), which is the within-cluster heterogeneity
cGSA's community detection is built to handle.

### Curation caveat: ground-truth vs membership mismatch

Not every benchmark set is curatable. Several sets' paper-derived ground-truth
pathways do **not** match their released DEG membership — e.g. `PMC11285963-1`
(pancreatic cancer) carries an epithelial/tumor membership (KRT/CEACAM/MUC1) but
its ground truth is entirely immune (T-cell activation, neutrophil
degranulation), and the two rheumatoid-arthritis lymphocyte sets list B-/T-cell
ground truth while only 1–2 canonical lymphocyte markers are actually present.
Curating those against the ground truth would violate the ground-in-membership
rule, so they are **skipped**. Every set here was screened for marker support
before curation.

## Why isolated from `curation/genesets/`

The native gold is scored by `scripts/score_method_vs_benchmark.py`, which globs
`curation/genesets/*.yaml` and measures recall against **human GOA** enrichment.
The cGSA benchmark is 52%% mouse and consists of large (~800-gene) DEG contrasts;
dropping those into the human-GOA precision/recall gold would silently distort
`recall_core`. Keeping them here preserves the native benchmark's integrity
while still subjecting every cGSA-derived YAML to the same 4-gate validator.

## Validate

```bash
# same 4 gates as the native corpus (structural, term id+label, reference, obsolescence)
uv run --project python/genesets-workflows --extra curation \
  genesets-workflows curate validate curation/cgsa/genesets/CGSA_PMC10202813_3_INTERFERON.yaml
```

> Environment note: the validator's OAK/eutils import chain needs
> `pkg_resources`, which setuptools ≥ 81 removed. If validation dies with
> `ModuleNotFoundError: No module named 'pkg_resources'`, pin `setuptools<81` in
> the workflows venv (already declared in the `curation` extra).
