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

A single melanoma study (PMC10202813), whose DEGs the benchmark splits into
modules, is the first demonstration. Three modules are curated and linked as
`SERIES:CGSA_PMC10202813_MODULES` to show that distinct modules of one study
resolve to **contrasting** GO interpretations:

| set | series_role | defining biology | grounding note |
|-----|-------------|------------------|----------------|
| `CGSA_PMC10202813_3_INTERFERON` | interferon_response | type I/II IFN response, antiviral defense | clean ISG panel (OAS/MX/IFIT/ISG15) — `annotation_supported`, confirmatory |
| `CGSA_PMC10202813_6_PIGMENT` | pigmentation_and_proliferation | melanin biosynthesis + melanosome | pigment core is **thin** (OCA2), co-mixed with a proliferation signal (CDC45/AURKB/PBK) |
| `CGSA_PMC10202813_2_SENESCENCE` | senescence_and_immune | cellular senescence + T-cell activation | two coherent arms (CDKN2A/TP53; PTPRC/TIGIT/SELL) plus melanoma drivers |

The pigment and senescence modules were chosen precisely because they are
**mixed** DEG clusters: they exercise the membership-grounding rule (the paper's
top ground-truth term can be carried by very few set members), which is exactly
the within-cluster heterogeneity cGSA's community detection is built to handle.

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
