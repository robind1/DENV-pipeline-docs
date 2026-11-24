# Pipeline components

## 1. Reference panel preparation (`PREPARE_REFERENCE`)

`BUILD_SEROTYPE_PANEL` reads `data/references/panel.tsv` and tags all six Dengue reference genomes
with their serotype and sylvatic status (e.g. `>DENV-1|NC_001477.1`,
`>DENV-2_SYLVATIC|EF105380.1`) before indexing them once into `denv_ref.mmi` with `minimap2 -d`.

| Accession | FASTA file | Serotype | Sylvatic | Nextclade dataset |
| --- | --- | --- | --- | --- |
| `NC_001477.1` | `NC_0014771.fasta` | `DENV-1` | no | `community/v-gen-lab/dengue/denv1` |
| `NC_001474.2` | `NC_0014742.fasta` | `DENV-2` | no | `community/v-gen-lab/dengue/denv2` |
| `NC_001475.2` | `NC_0014752.fasta` | `DENV-3` | no | `community/v-gen-lab/dengue/denv3` |
| `NC_002640.1` | `NC_0026401.fasta` | `DENV-4` | no | `community/v-gen-lab/dengue/denv4` |
| `EF105380.1` | `EF105380.fasta` | `DENV-2` | yes | `community/v-gen-lab/dengue/denv2` |
| `JF262780.1` | `JF262780.fasta` | `DENV-4` | yes | `community/v-gen-lab/dengue/denv4` |

## 2. Read QC and optional host depletion (`READ_QC`)

- **Illumina**: `FASTQC` inspects input reads while `TRIMMOMATIC` trims paired-end reads
  (`LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:36`).
- **Oxford Nanopore**: `NANOPLOT` computes read quality and length distributions while `CHOPPER`
  filters reads by `--nanopore_min_q` (default `7`) and `--nanopore_min_l` (default `200`).
- **Host depletion** (`HOSTILE_CLEAN`): unless `--skip_host_removal true` is passed, `hostile`
  depletes human reads using `bowtie2` (Illumina) or `minimap2` (ONT). Pass `--hostile_index` to
  use a pre-staged local Hostile cache directory offline.

## 3. Pan-serotype classification (`SEROTYPING`)

1. `SEROTYPE_MAP` maps up to 180,000 reads (or the full submitted FASTA) against `denv_ref.mmi`
   using `minimap2 -c --secondary=no` with platform-specific presets (`-x sr` for Illumina,
   `-x map-ont` for ONT, `-x asm20` for FASTA). The `-c` flag computes exact base-level CIGAR match
   counts in PAF column 9 so identity filtering works accurately on noisy ONT reads as well as
   Illumina reads.
2. `SEROTYPE_CLASSIFY` (`bin/serotype_classifier.py`) retains the best hit per query
   (`alen >= 50` and `identity >= 0.70` for reads; `alen >= 200` and `identity >= 0.75` for FASTA)
   and aggregates counts by serotype and reference accession:
   - Requires at least `--min_serotype_reads` (default `10`, or `1` for FASTA) mapped DENV reads.
     Otherwise the sample is labelled `unknown` and routed to `CONSENSUS_UNMAPPED` (never silently
     aligned to DENV-1).
   - Selects the best matching reference FASTA within the winning serotype, including sylvatic
     references (`EF105380.fasta`, `JF262780.fasta`) when those have the highest support.
   - Flags `coinfection_suspected: true` when a second serotype reaches both
     `>= min_serotype_reads` and `>= coinfection_min_fraction` (default `0.10`) of mapped DENV
     reads.

## 4. Amplicon alignment, primer trimming, and masked consensus

### Illumina (`CONSENSUS_ILLUMINA`)

1. `BWAMEM2_ALIGN` aligns trimmed paired-end reads to the selected serotype reference and sorts
   and indexes the BAM with `samtools`.
2. `IVAR_TRIM` (when `--primer_bed` is given) soft-clips amplicon primer sequences using `ivar trim`
   (`-q 20 -m 30 -e`), automatically matching the BED file in a primer directory whose contig in
   column 1 equals the selected reference accession, then re-sorts and re-indexes the BAM.
3. `MOSDEPTH` computes exact per-base coverage (`<id>.per-base.bed.gz`) and summary statistics.
4. `SUMMARIZE_COVERAGE` (`bin/summarize_coverage.py`) writes `<id>.coverage.json` (mean/median/min/max
   depth, breadth at `>=1x`, `>=consensus_min_depth`, and `>=30x`, and masked base count) and
   `<id>.mask.bed` covering every reference interval with depth `< consensus_min_depth` (including
   any uncovered reference ends).
5. `CALL_VARIANTS_ILLUMINA` calls haploid variants using:
   ```bash
   bcftools mpileup -a FORMAT/AD,FORMAT/DP -d ${params.mpileup_max_depth} -Q 20 -q 20 -f ref bam \
     | bcftools call --ploidy 1 -mv -Ou \
     | bcftools norm -f ref -m -any -Ou \
     | bcftools filter -i "FORMAT/DP>=${min_depth} && QUAL>=${min_qual} && (FORMAT/AD[0:1]/FORMAT/DP)>=${min_af}"
   ```
   Using `--ploidy 1` and allele-fraction filtering eliminates spurious heterozygous IUPAC
   ambiguity codes (`R`, `Y`, `M`, `S`, etc.) in the consensus.
6. `BUILD_CONSENSUS` runs `bcftools consensus -f ref -m <id>.mask.bed <id>.filtered.vcf.gz` so
   every position below `--consensus_min_depth` is masked as `N` rather than filled with reference
   sequence.

### Oxford Nanopore (`CONSENSUS_NANOPORE`)

1. `MINIMAP2_ALIGN` aligns trimmed ONT reads with `minimap2 -x map-ont --secondary=no -L --MD`
   and sorts/indexes the BAM.
2. Optional `IVAR_TRIM` when `--primer_bed` is given.
3. `MOSDEPTH` and `SUMMARIZE_COVERAGE` compute per-base coverage and `<id>.mask.bed` identically to
   Illumina.
4. `MEDAKA_VARIANT` runs `medaka inference` with `--medaka_model`, `medaka vcf`,
   `medaka tools annotate`, `bcftools norm -m -any`, and filters to ALT calls with
   `INFO/DP >= consensus_min_depth` and `QUAL >= variant_min_quality || FORMAT/GQ >= variant_min_quality`.
5. `BUILD_CONSENSUS` produces the depth-masked `<id>.consensus.fasta`.

## 5. Nextclade genotyping and QC gating (`GENOTYPING`)

1. `NEXTCLADE_DATASET_GET` fetches each unique serotype dataset required by the batch once per run
   (or stages pre-fetched directories from `--nextclade_dir`).
2. `NEXTCLADE_RUN` aligns each consensus genome offline against its serotype dataset, emitting
   `<id>.nextclade.csv` and `<id>.nextclade.json`.
3. `PARSE_NEXTCLADE` (`bin/parse_nextclade.py`):
   - Parses structured Hill et al. (2024) Dengue lineage codes (e.g. `1V_D.1.1` → `Genotype V`,
     `Lineage D`, `Sub-lineage 1.1`; `2III_C.2` → `Genotype III`, `Lineage C`, `Sub-lineage 2`).
   - Extracts all amino acid **substitutions**, **deletions**, and **insertions**, plus frameshifts
     and nucleotide mutation counts.
   - Applies the **QC gate**: if `serotype == "unknown"`, consensus is empty, masked completeness
     `< --min_completeness` (default `0.80`), or Nextclade `qc.overallStatus == "bad"`, then
     `qc_status` is `FAIL` and `genotype` / `major_lineage` / `minor_lineage` / `clade` are
     reported as **`Indeterminate`** (with explicit `qc_reasons` and the unfiltered `raw_clade`
     preserved for auditability).

## 6. Reporting and FHIR (`MULTIQC`, `SAMPLE_REPORT`, `FHIR`)

- `MULTIQC` aggregates FastQC, NanoPlot, and mosdepth reports into a single HTML report.
- `SAMPLE_REPORT` writes a plain-text report per sample (`reports/<id>.summary_report.txt`).
- `FHIR` runs the three-layer FHIR export described in [FHIR output](FHIR.md).
