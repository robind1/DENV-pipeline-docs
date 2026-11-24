# Overview

[DENVtoFHIR](https://github.com/oucru-id/DENV-to-fhir-full) is a Nextflow pipeline for Dengue virus
(DENV-1 to DENV-4) genomic analysis from Illumina paired-end amplicon/WGS reads, Oxford Nanopore
reads, or pre-built consensus FASTA sequences. It produces serotype calls with mixed-serotype
co-infection checks, depth-masked haploid consensus genomes, Nextclade genotype and lineage
classifications, and HL7 FHIR R4 resources conforming to the
[Genomics Reporting IG 3.0.0](https://hl7.org/fhir/uv/genomics-reporting/STU3/).

## Key features

- **Multi-platform input**: Illumina paired-end reads, Oxford Nanopore reads, or pre-assembled
  consensus FASTAs, given together in one samplesheet.
- **Pan-serotype panel & co-infection detection**: maps reads against reference
  panel (`data/references/panel.tsv`: DENV-1..4 RefSeq plus sylvatic DENV-2 and DENV-4), flags
  suspected mixed-serotype co-infections.
- **Amplicon-aware haploid consensus with low-depth masking**:
  - Optional amplicon primer trimming via `ivar trim` (`--primer_bed`), with the official
    [DengueSeq](https://doi.org/10.1186/s12864-024-10350-x) primer BED files bundled in
    `data/primers/dengueseq/`.
  - Haploid Illumina variant calling (`bcftools mpileup` + `bcftools call --ploidy 1 -mv` with DP,
    QUAL, and allele-fraction filtering) and ONT variant calling (`medaka`), eliminating diploid
    IUPAC ambiguity inflation.
  - Per-base `mosdepth` coverage masks every position below `--consensus_min_depth` (default `10`)
    with `N` (`bcftools consensus -m`), so reported sequence completeness matches real sequencing
    breadth.
- **QC-gated Nextclade genotyping**: fetches each needed serotype dataset
  (`community/v-gen-lab/dengue/denv1..4`) once per run—or uses offline pre-fetched datasets via
  `--nextclade_dir`—extracts amino acid substitutions, deletions, and insertions, and gates
  genotype/lineage calls to **Indeterminate** when completeness falls below `--min_completeness`
  (default `0.80`) or Nextclade QC is `bad`.
- **QC reporting**: FastQC (Illumina) or NanoPlot (ONT), optional human host depletion (Hostile),
  per-base `mosdepth` coverage summaries, MultiQC, and per-sample text reports.
- **Genomic FHIR without patient data, by default.** The genomic report refers to the patient by
  id only and embeds the masked consensus genome sequence (`--fhir_include_sequence true`).
  Clinical resources are built in a separate, optional step and joined into a self-contained
  `transaction` bundle in a third step.
- **Two FHIR profiles**:
  - `generic` (default): plain HL7 Genomics Reporting IG 3.0.0.
  - `satusehat`: adds Indonesian SATUSEHAT platform.

## What a run produces

For each sample:
- serotype call (`serotyping/<id>.serotype.json`) with read breakdown, sylvatic flag, and
  co-infection flag
- depth-masked consensus FASTA (`consensus/<id>.consensus.fasta`) and filtered VCF
  (`variants/<id>.filtered.vcf.gz`)
- per-base coverage summary (`qc/coverage/<id>.coverage.json`) and low-depth mask BED
- Nextclade genotype, lineage, and amino acid mutation calls (`genotyping/<id>.genotype.json`)
- human-readable summary report (`reports/<id>.summary_report.txt`)
- a PHI-free FHIR genomic bundle (`fhir/genomic/<id>.genomic.fhir.json`) including the consensus
  sequence
- with `--patient_metadata`, a FHIR `transaction` bundle (`fhir/bundle/<id>.bundle.fhir.json`)
  linking the genomic report to Patient, Specimen, Encounter, ServiceRequest, Organization, and
  Practitioner

## Repository layout

```text
DENV-to-fhir-full
├── main.nf                  entry point and parameter validation
├── workflows/denvfhir.nf    main workflow
├── subworkflows/local/      prepare_reference, read_qc, serotyping, consensus_illumina,
│                            consensus_nanopore, genotyping, fhir, utils_params
├── modules/local/           one process per tool or step (with container, conda, and stub)
├── bin/                     Python scripts (serotyping, coverage, Nextclade parser, genomic/
│                            clinical/link FHIR builders, validation summary, upload, reports)
├── conf/                    resources (base.config), output layout (modules.config), test profiles
├── assets/                  samplesheet schema, FHIR CodeSystems, metadata templates
├── data/
│   ├── references/          6 DENV reference genomes and panel.tsv
│   └── primers/dengueseq/   official DengueSeq primer BED files (DENV1..4 + sylvatics)
├── tools/
│   ├── nextclade/           fetch_datasets.sh for offline Nextclade staging
│   ├── satusehat/           get_access_token.py helper (tokens stored in ~/.denvfhir)
│   └── dev/                 check_no_phi.py PHI and credential guard
├── tests/                   pytest suite and test profile data
├── nextflow.config          parameters, manifest, and execution profiles
└── nextflow_schema.json     parameter schema (nf-schema)
```
