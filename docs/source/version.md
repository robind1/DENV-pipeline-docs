# Version

## Pipeline version

**2.0.0**

## Software

| Tool | Version |
| --- | --- |
| FastQC | 0.12.1 |
| Trimmomatic | 0.39 |
| NanoPlot | 1.48.0 |
| chopper | 0.12.0 |
| Hostile | 2.0.2 |
| minimap2 | 2.26 |
| bwa-mem2 | 2.2.1 |
| iVar | 1.4.4 |
| samtools | 1.19.2 |
| bcftools | 1.19 |
| medaka | 2.1.0 |
| mosdepth | 0.3.6 |
| Nextclade | 3.29.1 |
| MultiQC | 1.30 |
| HL7 FHIR validator | 6.5.26 |
| Python / Java | 3.12 / 21 |

The exact versions used in a run are recorded in `pipeline_info/software_versions.yml`.

## Reference data

### Reference genomes (`data/references/`)

- DENV-1: [NC_001477.1](https://www.ncbi.nlm.nih.gov/nuccore/NC_001477.1) (`NC_0014771.fasta`, 10,735 bp)
- DENV-2: [NC_001474.2](https://www.ncbi.nlm.nih.gov/nuccore/NC_001474.2) (`NC_0014742.fasta`, 10,723 bp)
- DENV-3: [NC_001475.2](https://www.ncbi.nlm.nih.gov/nuccore/NC_001475.2) (`NC_0014752.fasta`, 10,707 bp)
- DENV-4: [NC_002640.1](https://www.ncbi.nlm.nih.gov/nuccore/NC_002640.1) (`NC_0026401.fasta`, 10,649 bp)
- DENV-2 sylvatic (Dak Ar 578): [EF105380.1](https://www.ncbi.nlm.nih.gov/nuccore/EF105380.1) (`EF105380.fasta`, 10,722 bp)
- DENV-4 sylvatic (P73-1120): [JF262780.1](https://www.ncbi.nlm.nih.gov/nuccore/JF262780.1) (`JF262780.fasta`, 10,667 bp)

### Amplicon primer BEDs (`data/primers/dengueseq/`)

- Official DengueSeq v1.0 primer BED files for DENV-1..4 and sylvatic DENV-2/4 from
  [grubaughlab/DENV_pipeline](https://github.com/grubaughlab/DENV_pipeline/tree/v1.0)
  (Vogels et al., 2024).

### Nextclade datasets

- `community/v-gen-lab/dengue/denv1`, `denv2`, `denv3`, `denv4` (Hill et al., 2024 Dengue lineage
  nomenclature).

## Changelog

See [CHANGELOG.md](https://github.com/oucru-id/DENV-to-fhir-full/blob/2.0.0-dev/CHANGELOG.md).

## References

1. Vogels, C. B. F., Hill, V., et al. (2024). *DengueSeq: a pan-serotype whole genome amplicon
   sequencing protocol for dengue virus*.
   [BMC Genomics 25:433](https://doi.org/10.1186/s12864-024-10350-x).
2. Hill, V., et al. (2024). *A new lineage nomenclature to aid genomic surveillance of dengue
   virus*. [PLOS Biology](https://doi.org/10.1371/journal.pbio.3002834).
3. Aksamentov, I., et al. (2021). *Nextclade: clade assignment, mutation calling and quality
   control for viral genomes*. [JOSS](https://doi.org/10.21105/joss.03773).
4. Grubaugh, N. D., et al. (2019). *An amplicon-based sequencing framework for accurately measuring
   intrahost virus diversity using PrimalSeq and iVar*.
   [Genome Biology 20:8](https://doi.org/10.1186/s13059-018-1618-7).
5. Stubbs, S. C. B., et al. (2020). *Assessment of a multiplex PCR and Nanopore-based method for
   dengue virus sequencing in Indonesia*.
   [Virology Journal 17:24](https://doi.org/10.1186/s12985-020-1294-6).
6. Di Tommaso, P., et al. (2017). *Nextflow enables reproducible computational workflows*.
   [Nature Biotechnology](https://www.nature.com/articles/nbt.3820).
7. HL7 International. *Genomics Reporting Implementation Guide 3.0.0*.
   [hl7.org](https://hl7.org/fhir/uv/genomics-reporting/STU3/).
