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
