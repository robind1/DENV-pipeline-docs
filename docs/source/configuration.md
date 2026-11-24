# Configuration

All parameters are validated against `nextflow_schema.json` using `nf-schema` before execution
starts. Run `nextflow run . --help` to list them from the command line.

## Parameters

### Input and output

| Parameter | Default | Description |
| --- | --- | --- |
| `input` | | Samplesheet CSV (`sample,platform,fastq_1,fastq_2,fasta`). |
| `reads_dir` | | Legacy input directory (`<id>_{1,2}_illumina.fastq.gz`, `<id>_ont.fastq.gz`, `<id>.fasta`). |
| `outdir` | `results` | Output directory. (`--results_dir`) |

### Reference data

| Parameter | Default | Description |
| --- | --- | --- |
| `reference_dir` | `data/references` | Directory containing the Dengue reference FASTA files. |
| `reference_panel` | `data/references/panel.tsv` | TSV mapping accession, FASTA filename, serotype, sylvatic flag, and Nextclade dataset. |

### Read QC and host removal

| Parameter | Default | Description |
| --- | --- | --- |
| `trimmomatic_jar` | | Path to `trimmomatic.jar` when `trimmomatic` is not on `PATH`. |
| `nanopore_min_q` | `7` | `chopper` minimum read quality for ONT reads (standard ONT viral amplicon pass threshold). |
| `nanopore_min_l` | `200` | `chopper` minimum read length (bp) for ONT reads. |
| `skip_host_removal` | `false` | Skip human host depletion with Hostile (recommended `true` for pure viral amplicon runs without a staged Hostile index). |
| `hostile_index` | | Pre-staged Hostile cache directory for offline host removal. |
| `hostile_index_name` | `human-t2t-hla` | Hostile index name. |

### Amplicon primer trimming and consensus calling

| Parameter | Default | Description |
| --- | --- | --- |
| `primer_bed` | | Primer BED file or directory of per-reference BED files (e.g. `data/primers/dengueseq`) for `ivar trim`. |
| `ivar_min_quality` | `20` | `ivar trim` minimum base quality (`-q`). |
| `ivar_min_length` | `30` | `ivar trim` minimum read length after trimming (`-m`). |
| `ivar_include_no_primer` | `true` | Pass `-e` to `ivar trim` to retain reads with no matched primer. |
| `medaka_model` | | Medaka model for ONT variant calling; **required** whenever ONT samples are present. |
| `medaka_batch_size` | `100` | Medaka inference batch size. |
| `consensus_min_depth` | `10` | Minimum per-base depth; positions below this depth are masked as `N` in the consensus FASTA. |
| `consensus_min_af` | `0.5` | Minimum alternate allele fraction (`AD[1]/DP`) for Illumina haploid consensus variant inclusion. |
| `variant_min_quality` | `20` | Minimum `QUAL` (Illumina) or `QUAL`/`GQ` (ONT) for variant inclusion. |
| `mpileup_max_depth` | `10000` | Maximum per-position depth (`-d`) passed to `bcftools mpileup` on Illumina BAMs. |

### Serotyping and Nextclade genotyping

| Parameter | Default | Description |
| --- | --- | --- |
| `min_serotype_reads` | `10` | Minimum mapped DENV reads required to call a serotype (always `1` for `fasta` input). |
| `coinfection_min_fraction` | `0.10` | Flag suspected mixed-serotype co-infection when the second serotype has >= this fraction of mapped DENV reads (and >= `min_serotype_reads`). |
| `nextclade_dir` | | Pre-fetched directory containing `denv1/`, `denv2/`, `denv3/`, `denv4/` Nextclade datasets for offline runs. |
| `nextclade_dataset_tag` | | Optional pinned Nextclade dataset tag when downloading datasets at runtime. |
| `min_completeness` | `0.80` | Minimum masked consensus completeness (non-N fraction, 0–1) required to report a definitive genotype/lineage; below this, genotype is `Indeterminate`. |

### FHIR output

| Parameter | Default | Description |
| --- | --- | --- |
| `fhir_profile` | `generic` | `generic` (HL7 Genomics Reporting IG 3.0.0) or `satusehat` (Kemkes profiles, tags, NIK identifiers, and extensions). |
| `fhir_include_sequence` | `true` | Embed the masked consensus genome sequence as an `Observation` (`valueString`) inside the genomic FHIR bundle. |
| `performer_org_id` | | FHIR id of the performing laboratory `Organization`. |
| `performer_org_name` | | Display name of the performing laboratory. |
| `identifier_system_base` | | Base URI for business identifiers in the `generic` profile (omitted if unset). |

### Clinical metadata (optional, contains personal data)

| Parameter | Default | Description |
| --- | --- | --- |
| `patient_metadata` | | Patient metadata CSV or XLSX; see [Clinical metadata](clinical_metadata.md). |
| `organization_metadata` | | Performing organization table (first row used). |
| `practitioner_metadata` | | Responsible practitioner table (first row used). |
| `external_refs` | | JSON `{fhir_id: {"Patient": "...", "Encounter": "..."}}` for pre-existing server resources. |
| `require_clinical` | `false` | Fail if any sample lacks a clinical metadata row (default: warn, genomic bundle only). |
| `bundle_request_method` | `PUT` | `PUT` (idempotent client-assigned IDs) or `POST` (server-assigned IDs with `ifNoneExist` on NIK in `satusehat`). |
| `publish_clinical` | `true` | Publish intermediate clinical resources to `<outdir>/fhir/clinical/`. |

### FHIR validation

| Parameter | Default | Description |
| --- | --- | --- |
| `skip_fhir_validation` | `true` | Skip the HL7 FHIR validator step. Pass `--skip_fhir_validation false` to enable batched validation. |
| `fhir_validator` | HAPI FHIR 6.5.26 URL | Local path or URL to `validator_cli.jar`. |
| `fhir_ig` | `hl7.fhir.uv.genomics-reporting#3.0.0` | Implementation guide package for validation. |
| `fhir_tx_server` | `n/a` | Terminology server URL, or `n/a` for offline validation. |
| `fail_on_validation_error` | `true` (`generic`) / `false` (`satusehat`) | Stop the run if any validated bundle reports errors. |
| `fhir_validation_batch_size` | `50` | Number of FHIR JSON files validated per validator JVM invocation. |

### FHIR upload

| Parameter | Default | Description |
| --- | --- | --- |
| `fhir_server_url` | | FHIR server base URL. Upload runs only when this is set (and requires `--patient_metadata`). |
| `fhir_auth_url` | | OAuth2/OIDC realm base URL for client-credentials token exchange. |
| `fhir_upload_api_key` | `false` | Also send the `DENVFHIR_API_KEY` Nextflow secret as an `X-Api-Key` header. |

### Resource limits

| Parameter | Default | Description |
| --- | --- | --- |
| `max_cpus` | `min(8, host CPUs)` | Maximum CPUs any task may request. |
| `max_memory` | `16 GB` | Maximum memory any task may request. |
| `max_time` | `24h` | Maximum wall time any task may request. |
| `trace_report_suffix` | start timestamp | Suffix for `pipeline_info/` execution reports. |

Tasks failing with out-of-memory or transient exit codes (`104`, `130..145`) retry automatically
with increased memory up to `--max_memory`.

## Execution profiles

- `standard` (default): local executor, host `PATH`
- `slurm`: SLURM HPC executor
- `docker`, `podman`, `singularity`, `apptainer`: pinned BioContainers images
- `conda`, `mamba`: pinned conda-forge / bioconda environments
- `test_stub`: wiring test across `illumina`, `ont`, and `fasta` (`-stub-run`)
- `test_fasta`: end-to-end test on the 4 canonical DENV RefSeq genomes
- `test`: default quick test profile
