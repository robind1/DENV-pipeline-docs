# Overview
[This pipeline](https://github.com/oucru-id/DENV-to-fhir-full) is a Nextflow-based workflow designed for the analysis of pan serotype Dengue virus (DENV1-4) genomic data. It processes raw sequencing data (Illumina or Nanopore) to identify Dengue virus serotypes, perform genotyping/lineage classification, generate consensus sequences, detect gene mutation, and generate a FHIR-compliant genomics bundle. 

## Key Features
* **Multi-platform Support**: Processes raw read data from diverse platforms.
* **Comprehensive Typing**: Serotyping (DENV-1 to DENV-4), and genotyping classification.
* **Host DNA Removal**: Removes human host contamination from clinical samples.
* **Gene mutation**: Gene mutation detection by Nextstrain.
* **Clinical Integration**: Merges genomic data with clinical metadata.
* **Quality Control**: QC reporting with MultiQC.

## Key Outputs
* Genome assembly
* Gene mutation list
* Serotype and Genotype classification
* FHIR-compliant genomic reports 
* Quality control metrics

## Directory Structure

```
denv-to-fhir-full
├── main.nf                             # Main workflow
├── nextflow.config                     # Configuration and parameters
├── workflows/
│   ├── illumina.nf                     # Illumina sub-workflow
│   ├── nanopore.nf                     # Nanopore sub-workflow
│   ├── trimming.nf                     # Read trimming
│   ├── host_removal.nf                 # Host read removal
│   ├── serotyping.nf                   # Serotype classification
│   ├── genotyping.nf                   # Genotype classification
│   ├── fhir.nf                         # FHIR Bundle generation
│   ├── validate_fhir.nf                # FHIR validation
│   ├── merge_clinical_data.nf          # Clinical metadata merge
│   ├── upload_fhir.nf                  # FHIR server upload
│   ├── report.nf                       # QC and sample report generation
│   └── utils.nf                        # Utility functions
├── scripts/
│   ├── annotated_to_fhir.py            # Consensus-to-FHIR converter
│   ├── clinical_metadata_parser.py     # Patient/org/practitioner parser
│   ├── serotype_classification.py      # Serotype classifier
│   ├── generate_dengue_report.py       # Per-sample report generator
│   ├── get_access_token.py             # FHIR access token retriever
│   ├── upload_fhir.py                  # FHIR upload script
│   ├── merge_clinical_fhir.py          # FHIR genomics + clinical data merger
│   └── get_versions.py                 # Software version collector
├── data/
│   ├── NGS/                            # Input FASTQ files
│   ├── references/                     # Reference genomes (DENV-1–4 & Sylvatic)
│   ├── patient_clinical_metadata.csv   # Patient metadata
│   ├── organization_metadata.csv       # Organization metadata
│   └── practitioner_metadata.csv       # Practitioner metadata
└── tools/
    └── fhir-validator.jar              # HL7 FHIR validator
```
