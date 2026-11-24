# Limitations

- **Mixed-serotype co-infections**: `SEROTYPE_CLASSIFY` detects and flags suspected co-infections
  (when the second serotype represents >= `--coinfection_min_fraction` of mapped DENV reads), and
  surfaces that warning in the summary report and FHIR `DiagnosticReport`/`Observation`. However,
  downstream alignment and consensus reconstruction currently build a consensus genome for the
  **primary serotype only** (competitive pan-panel assignment rather than iterative per-serotype
  reconstruction).
- **Sylvatic lineages with non-sylvatic primers**: `data/references/panel.tsv` includes sylvatic
  DENV-2 (`EF105380.1`) and DENV-4 (`JF262780.1`) references and `data/primers/dengueseq/` includes
  their corresponding BED coordinates, but the DengueSeq primer scheme itself was designed against
  human-outbreak genotypes and can show reduced coverage (60–94%) on divergent sylvatic isolates.
- **Amplicon dropouts and Nextclade QC gating**: masking positions below `--consensus_min_depth`
  (default `10`) as `N` prevents reference sequence bleed-through, which is scientifically
  accurate, but samples with >1,100 masked `N` bases (roughly <89% completeness) can receive
  Nextclade `qc.overallStatus = "bad"` even when completeness exceeds `--min_completeness` (`0.80`).
  In those cases, DENVtoFHIR reports genotype/lineage as `Indeterminate` while preserving
  `raw_clade` in `genotyping/<id>.genotype.json` and `reports/<id>.summary_report.txt`.
- **Primer trimming requires matching scheme coordinates**: pass `--primer_bed data/primers/dengueseq`
  only for libraries amplified with DengueSeq primers. For other amplicon schemes (e.g. Stubbs et
  al. 2020 or custom panels), supply the matching primer BED file(s) or omit `--primer_bed`.
- **Submitted consensus FASTA (`platform = fasta`)**: read-level depth and breadth cannot be
  measured from a FASTA alone, so `coverage.json` is recorded as `assessed: false`
  (`status: "submitted_consensus"`).
- **ONT Medaka model selection**: `--medaka_model` must match the flow cell and basecaller used
  (e.g. `r941_min_hac_variant_g507` for R9.4.1 HAC vs `r1041_e82_400bps_sup_variant_v5.0.0` for
  R10.4.1 Kit 14 SUP). Using a mismatched model degrades indel accuracy.
- **FHIR validation default**: `--skip_fhir_validation` currently defaults to `true` in `2.0.0-dev`
  while Genomics Reporting IG 3.0.0 profile validation is finalised. Enable it with
  `--skip_fhir_validation false` when validating bundles before production upload.
- **Network access for runtime databases**: unless `--nextclade_dir` (and `--hostile_index` when
  host removal is enabled) are pre-staged locally, the first run requires internet access to fetch
  Nextclade datasets and/or the Hostile index.
