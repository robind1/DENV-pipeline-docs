# Inputs

## Samplesheet (recommended)

Give a CSV file with `--input`. The columns are `sample,platform,fastq_1,fastq_2,fasta`, validated
against `assets/schema_input.json` before any job starts:

```text
sample,platform,fastq_1,fastq_2,fasta
DENV001,illumina,/data/DENV001_1.fastq.gz,/data/DENV001_2.fastq.gz,
DENV002,ont,/data/DENV002.fastq.gz,,
DENV003,fasta,,,/data/DENV003.fasta
```

| `platform` | Required columns | Notes |
| --- | --- | --- |
| `illumina` | `fastq_1`, `fastq_2` | Gzipped paired-end FASTQ (`.fastq.gz` or `.fq.gz`) |
| `ont` | `fastq_1` | Gzipped single-end ONT FASTQ; also requires `--medaka_model` |
| `fasta` | `fasta` | Pre-built consensus FASTA (`.fasta`, `.fa`, `.fna`, optionally `.gz`) |

```bash
nextflow run . -profile docker --input samplesheet.csv --outdir results
```

You can mix Illumina, Nanopore, and consensus FASTA samples in the same samplesheet.

### Sample ID rules

- 1 to 40 characters: letters, digits, `.`, `_`, or `-`, starting with a letter or digit
  (`^[A-Za-z0-9][A-Za-z0-9._-]{0,39}$`).
- Sample IDs must be unique, and must remain unique after FHIR ID sanitisation (for example,
  `DENV_1` and `DENV.1` both become `DENV-1` and are rejected).
- **Use laboratory accessions, not personal identifiers.** Sample IDs appear in the PHI-free
  genomic FHIR bundles. A 16-digit ID triggers a warning because it matches Indonesian NIK format.

## Legacy folder input

For compatibility with 1.0 runs, `--reads_dir` discovers files by filename suffix:

```bash
nextflow run . -profile docker --reads_dir /data/NGS --outdir results
```

Expected filenames in `--reads_dir`:
- Illumina paired-end: `<id>_1_illumina.fastq.gz` and `<id>_2_illumina.fastq.gz`
- Oxford Nanopore: `<id>_ont.fastq.gz`
- Pre-built consensus FASTA: `<id>.fasta`, `<id>.fa`, or `<id>.fna`

## Amplicon primer BEDs (`--primer_bed`)

For tiled amplicon sequencing protocols (such as [DengueSeq](https://doi.org/10.1186/s12864-024-10350-x)),
pass `--primer_bed` to trim primer sequences from aligned BAMs with `ivar trim` before variant
calling and consensus generation:

```bash
nextflow run . -profile docker \
  --input samplesheet.csv \
  --primer_bed data/primers/dengueseq \
  --skip_host_removal true \
  --outdir results
```

- `--primer_bed` can be either a **single BED file** or a **directory of BED files** (one per
  reference genome).
- When given a directory, `IVAR_TRIM` inspects the contig accession in column 1 of each `.bed` file
  and automatically selects the BED matching the sample's assigned reference accession (for example
  `NC_001477.1` → `DENV1.bed`, `EF105380.1` → `DENV2_sylvatic.bed`).
- All six official DengueSeq v1.0 primer BED files are bundled in `data/primers/dengueseq/`.

## Nanopore model (`--medaka_model`)

Whenever a run includes any `ont` sample, `--medaka_model` is required and must match the flow cell
and basecaller used to generate the reads:

- Example for R10.4.1 Kit 14 SUP basecalling: `--medaka_model r1041_e82_400bps_sup_variant_v5.0.0`
- Example for R9.4.1 HAC basecalling: `--medaka_model r941_min_hac_variant_g507`

List available models in the pinned Medaka image with:

```bash
docker run --rm quay.io/biocontainers/medaka:2.1.0--py38ha0c3a46_0 medaka tools list_models
```

## Consensus FASTA input (`platform = fasta`)

Submitted consensus genomes skip read QC, host removal, alignment, and variant calling, and proceed
directly to:
1. Header normalisation and sequence QC (`PREPARE_CONSENSUS`: length, non-N completeness, GC%,
   non-ACGTN count).
2. Pan-serotype panel classification (`minimap2 -c -x asm20`).
3. Nextclade genotyping and amino acid mutation extraction.
4. Summary report and FHIR bundle generation (with `coverage.json` marked
   `status: "submitted_consensus"`, `assessed: false`).
