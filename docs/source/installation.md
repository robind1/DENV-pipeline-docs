# Installation

## Requirements

You need [Nextflow](https://www.nextflow.io/) >= 25.04 and **one** of the execution profiles below.
If using Nextflow 26.04, use **26.04.2 or later**.

| Profile | Provides the tools via |
| --- | --- |
| `-profile docker` (or `podman`) | pinned [BioContainers](https://biocontainers.pro/) images |
| `-profile singularity` (or `apptainer`) | the same images (e.g. on HPC) |
| `-profile conda` (or `mamba`) | pinned bioconda / conda-forge packages |
| none (`standard`) | tools already on `PATH` |

Without a container or conda profile, the following tools must be available on `PATH`:
- **All runs**: `minimap2` (2.26), `nextclade` (3.29.1), Python 3.10+ (standard library only;
  `openpyxl` only if reading `.xlsx` clinical metadata)
- **Illumina input**: `fastqc` (0.12.1), `trimmomatic` (0.39), `bwa-mem2` (2.2.1), `samtools`
  (1.19.2), `bcftools` (1.19), `mosdepth` (0.3.6), `multiqc` (1.30); `ivar` (1.4.4) if using
  `--primer_bed`; `hostile` (2.0.2) unless `--skip_host_removal true`
- **ONT input**: `NanoPlot` (1.48.0), `chopper` (0.12.0), `minimap2` (2.26), `samtools` (1.17+),
  `medaka` (2.1.0), `bcftools` (1.19), `mosdepth` (0.3.6), `multiqc` (1.30); `hostile` unless
  `--skip_host_removal true`
- **FHIR validation** (optional, when `--skip_fhir_validation false`): Java >= 17

If your system only provides Debian/Ubuntu's `trimmomatic.jar`, pass
`--trimmomatic_jar /usr/share/java/trimmomatic.jar`.

### Notes for container runs

- The Python container (`quay.io/biocontainers/python:3.12.12`) does not include `openpyxl`. Supply
  clinical metadata as **CSV** in container runs, or use `-profile conda`.
- `MEDAKA_VARIANT` sets a writable `HOME` inside the task work directory so Medaka models can be
  fetched automatically inside rootless Docker containers (`-u $(id -u):$(id -g)`).
- By default (`--skip_host_removal false`), `hostile` downloads a human reference index unless
  `--hostile_index` points to a pre-staged Hostile cache directory. For pure viral amplicon runs
  where host depletion is not required, pass `--skip_host_removal true`.

## Get the pipeline

```bash
git clone -b 2.0.0 https://github.com/oucru-id/DENV-to-fhir-full.git
cd DENV-to-fhir-full
```

Or run directly from GitHub with Nextflow:

```bash
nextflow run oucru-id/DENV-to-fhir-full -r 2.0.0 -profile test_stub -stub-run
```

## Verify the installation

```bash
# 1. Wiring and schema check (no containers or tools needed)
nextflow run . -profile test_stub -stub-run

# 2. End-to-end on the 4 canonical DENV RefSeq genomes (Docker)
nextflow run . -profile test_fasta,docker

# 3. Unit tests (optional, requires pytest)
pytest -q
```

## Offline dataset staging (optional)

By default, DENVtoFHIR downloads each needed Nextclade dataset
(`community/v-gen-lab/dengue/denv1..4`) once per run. For air-gapped or reproducible offline runs,
pre-fetch all four datasets once and pass `--nextclade_dir`:

```bash
tools/nextclade/fetch_datasets.sh db/nextclade
nextflow run . -profile docker --input samplesheet.csv --nextclade_dir db/nextclade
```

## Uploading to a FHIR server

Upload runs **only** when `--fhir_server_url` is set, and only uploads linked transaction bundles
(requiring `--patient_metadata`). Credentials are stored as Nextflow secrets, never as CLI
parameters or tracked files:

```bash
nextflow secrets set DENVFHIR_CLIENT_ID     '<client id>'
nextflow secrets set DENVFHIR_CLIENT_SECRET '<client secret>'
# Optional, if the gateway also requires an X-Api-Key header:
nextflow secrets set DENVFHIR_API_KEY       '<api key>'
```

Then run with `--fhir_server_url` and `--fhir_auth_url`:

```bash
nextflow run . -profile docker \
  --input samplesheet.csv \
  --patient_metadata site/patients.csv \
  --organization_metadata site/organization.csv \
  --practitioner_metadata site/practitioner.csv \
  --fhir_profile satusehat \
  --fhir_server_url https://<server>/fhir \
  --fhir_auth_url https://<sso>/realms/<realm>
```

For manual token testing outside Nextflow, `tools/satusehat/get_access_token.py` saves tokens to
`$DENVFHIR_HOME/access_token.json` (default `~/.denvfhir/access_token.json`, mode `0600`) outside
the repository.
