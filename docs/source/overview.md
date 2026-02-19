# Overview
[This pipeline](https://github.com/oucru-id/DENV-to-fhir-full) is a Nextflow-based workflow designed for the analysis of pan serotype Dengue virus (DENV1-4) genomic data. It processes raw sequencing data (Illumina or Nanopore) to identify Dengue virus serotypes, perform genotyping/lineage classification, generate consensus sequences, detect gene mutation, and generate a FHIR-compliant genomics bundle. 

## Key Features
* **Multi-platform Support**: Processes raw read data from diverse platforms.
* **Comprehensive Typing**: Serotyping (DENV-1 to DENV-4), and genotyping classification.
**Host DNA Removal**: Removes human host contamination from clinical samples.
* **Gene mutation**: Gene mutation detection by Nextstrain.
* **Clinical Integration**: Merges genomic data with clinical metadata.
* **Quality Control**: QC reporting with MultiQC.

## Key Outputs
* Genome assembly
* Gene mutation list
* Serotype and Genotype classification
* FHIR-compliant genomic reports 
* Quality control metrics
