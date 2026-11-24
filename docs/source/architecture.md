# Pipeline architecture

Every sample follows the branch for its sequencing platform (`illumina`, `ont`, or `fasta`), then
joins the shared serotyping, genotyping, reporting, and FHIR steps. Samples whose serotype cannot
be determined (`unknown`) are routed to `CONSENSUS_UNMAPPED` rather than falling back to a default
reference, so they still emit an explicit QC-failed genomic FHIR bundle. Clinical resource
generation, transaction linking, FHIR validation, and server upload run only when their
corresponding parameters are enabled.

```mermaid
flowchart TD
    IN[/"Samplesheet or --reads_dir"/]
    IN --> ILL_QC["Illumina QC<br/>FastQC · Trimmomatic · (Hostile)"]
    IN --> ONT_QC["ONT QC<br/>NanoPlot · chopper (Q7) · (Hostile)"]
    IN --> FA["Submitted FASTA<br/>PREPARE_CONSENSUS"]

    PANEL["PREPARE_REFERENCE<br/>6-genome tagged minimap2 panel"] --> SERO
    ILL_QC & ONT_QC & FA --> SERO["SEROTYPING<br/>minimap2 -c · serotype_classifier.py<br/>(sylvatic & co-infection flags)"]

    SERO -->|Illumina mapped| ILL_CONS["CONSENSUS_ILLUMINA<br/>bwa-mem2 · (ivar trim) · mosdepth<br/>bcftools haploid · masked consensus"]
    SERO -->|ONT mapped| ONT_CONS["CONSENSUS_NANOPORE<br/>minimap2 · (ivar trim) · mosdepth<br/>medaka · masked consensus"]
    SERO -->|Unmapped| UNM["CONSENSUS_UNMAPPED<br/>explicit unmapped QC record"]

    ILL_CONS & ONT_CONS & UNM & FA --> GENO["GENOTYPING<br/>Nextclade (1 dataset fetch/serotype)<br/>parse_nextclade.py (QC gate)"]

    ILL_QC & ONT_QC & ILL_CONS & ONT_CONS --> MQC["MULTIQC"]
    GENO --> REP["SAMPLE_REPORT"]
    GENO --> GEN["FHIR_GENOMIC<br/>PHI-free collection + embedded sequence"]

    META[/"Clinical metadata<br/>(optional)"/] --> CLIN["FHIR_CLINICAL"]
    GEN & CLIN --> LINK["FHIR_LINK<br/>transaction bundle per sample"]
    GEN & LINK --> VAL["FHIR_VALIDATE<br/>(optional, IG 3.0.0)"]
    VAL --> UP["FHIR_UPLOAD<br/>(optional, Nextflow secrets)"]
```
