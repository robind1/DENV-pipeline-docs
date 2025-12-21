# Data Processing

## Main Workflow Controller
**File:** `main.nf`
The main workflow handles channel processing and parallel execution. It automatically detects input data types (Illumina or Nanopore), performs trimming and host removal, determines the serotype, and then routes data to the specific alignment and consensus generation workflows.

## Pre-processing & Host Removal
**Files:** `trimming.nf`, `host_removal.nf`
1.  **Trimming**:
    *   **Illumina**: `Trimmomatic` (Quality trimming, adapter removal).
    *   **Nanopore**: `chopper` (Quality and length filtering).
2.  **Host Removal**:
    *   **Tool**: `hostile`.
    *   **Process**: Aligns reads against a host reference (human) to remove non-viral reads using `bowtie2` (Illumina) or `minimap2` (Nanopore).

## Serotyping
**File:** `serotyping.nf`
Determines the Dengue serotype to select the appropriate reference for alignment.
1.  **Tool**: `BLASTn`.
2.  **Process**: Maps a subset of reads against a database of Dengue reference sequences (DENV-1, 2, 3, 4, and Sylvatic strains).

## Nanopore (Long-Read) Workflow
**File:** `nanopore.nf`
For Oxford Nanopore Technologies (ONT) sequencing data.
1.  **QC**: `FastQC`.
2.  **Reference Selection**: Selects the specific reference genome based on the determined serotype.
3.  **Alignment**:
    *   **Tool**: `minimap2`.
4.  **Consensus Generation**:
    *   **Tools**: `bcftools`.

## Illumina (Short-Read) Workflow
**File:** `illumina.nf`
For Illumina paired-end sequencing data.
1.  **QC**: `FastQC`.
2.  **Reference Selection**: Selects the specific reference genome based on the determined serotype.
3.  **Alignment**:
    *   **Tool**: `bwa-mem2`.
4.  **Consensus Generation**:
    *   **Tools**: `bcftools`.

## Genotyping & Variant Analysis
**File:** `genotyping.nf`
Performs detailed characterization of the consensus sequence.
1.  **Tool**: `Nextclade`.
2.  **Process**:
    *   Assigns **Genotype** and **Lineage** (Major/Minor) based on the Dengue dataset.
    *   Identifies amino acid **Mutations**.

## Reporting
**File:** `report.nf`
1.  **MultiQC**: Aggregates FastQC and trimming logs.
2.  **Dengue Report**: A text report summarizing Serotype, Genotype, Lineage, Coverage, and Mutations.

## FHIR Converter
**File:** `fhir.nf`
Converts genomic analysis results into HL7 FHIR R4 standard resources.
1.  **Input Parsing**: Reads consensus sequence stats, serotyping, and Nextclade results.
2.  **Resource Creation**:
    *   **Observation**: For Dengue Classification (Serotype, Genotype, Lineage).
    *   **Observation**: For Viral Consensus Genome Sequence (Sequence string, length, coverage).
    *   **Observation**: For each detected Genetic Variant.
    *   **DiagnosticReport**: Report for overall Dengue analysis.
