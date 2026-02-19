# Installation

## Prerequisites
To run this pipeline, you need the following prerequisites:

*   [Nextflow](https://www.nextflow.io/)
*   [Python](https://www.python.org/)
*   [FastQC](https://github.com/s-andrews/FastQC)
*   [MultiQC](https://github.com/MultiQC/MultiQC)
*   [Trimmmomatic](https://github.com/usadellab/Trimmomatic)
*   [Chopper](https://github.com/wdecoster/chopper) 
*   [BWA-MEM2](https://github.com/bwa-mem2/bwa-mem2) 
*   [minimap2](https://github.com/lh3/minimap2)
*   [bcftools](https://github.com/samtools/bcftools)
*   [samtools](https://github.com/samtools/samtools)
*   [blastn](https://github.com/ncbi/sra-tools/tree/master)
*   [hostile](https://github.com/bede/hostile)
*   [Nextclade](https://github.com/nextstrain/nextclade)
*   [FHIR validator](https://github.com/hapifhir/org.hl7.fhir.validator-wrapper)

## Setup
1.  Clone the repository for local installation:
    ```bash
    git clone https://github.com/oucru-id/DENV-to-fhir-full.git
    cd DENVgenomicpipeline
    ```
2.  Install Nextflow:
    ```bash
    curl -s https://get.nextflow.io | bash
    ```
3.  Testing the Nextflow install:
    ```bash
    nextflow -v
    ```
