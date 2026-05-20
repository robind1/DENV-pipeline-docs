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
4. Get Access Token (FHIR Upload)
```bash
python scripts/get_access_token.py
```
5. Basic Run
```bash
nextflow run main.nf
```
6. Run with FHIR Upload
> Get the access token first before running with upload.
```bash
nextflow run main.nf \
  --fhir_server_url "https://<BASE_URL>/fhir"
```
