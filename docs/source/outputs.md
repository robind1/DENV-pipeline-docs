# Outputs

All results are written under `--outdir` (default `results/`):

```text
results/
├── pipeline_info/
│   ├── execution_report_<timestamp>.html
│   ├── execution_timeline_<timestamp>.html
│   ├── execution_trace_<timestamp>.txt
│   ├── pipeline_dag_<timestamp>.html
│   └── software_versions.yml
├── qc/
│   ├── fastqc/               <id>*_fastqc.{zip,html}                  (Illumina)
│   ├── nanoplot/             <id>_NanoStats.txt, <id>_NanoPlot-report.html (ONT)
│   ├── host_removal/         <id>.hostile.log                         (when enabled)
│   ├── coverage/             <id>.coverage.json, <id>.mask.bed,
│   │                         <id>.per-base.bed.gz, <id>.mosdepth.summary.txt
│   └── multiqc/              multiqc_report.html, multiqc_data/
├── serotyping/
│   └── <id>.serotype.json
├── variants/
│   └── <id>.filtered.vcf.gz(.tbi)
├── consensus/
│   └── <id>.consensus.fasta
├── genotyping/
│   ├── <id>.nextclade.csv
│   ├── <id>.nextclade.json
│   └── <id>.genotype.json
├── reports/
│   └── <id>.summary_report.txt
└── fhir/
    ├── genomic/              <id>.genomic.fhir.json                   (PHI-free)
    ├── clinical/             shared.clinical.json, <fid>.clinical.json,
    │                         external_refs.json, clinical_summary.tsv (personal data)
    ├── bundle/               <fid>.bundle.fhir.json, link_summary.tsv (personal data)
    ├── validation/           validation_summary.tsv, validation_<n>.json (when enabled)
    └── upload/               <fid>.bundle.upload.json                 (when uploaded)
```

## Key output files

| File | Description |
| --- | --- |
| `serotyping/<id>.serotype.json` | Serotype call, confidence, selected reference accession/FASTA, sylvatic flag, per-serotype read counts/fractions/identity, and `coinfection_suspected` flag. |
| `qc/coverage/<id>.coverage.json` | Coverage assessment status (`assessed`, `submitted_consensus`, or `unmapped`), reference accession, mean/median/min/max depth, breadth at `>=1x`, `>=min_depth`, `>=30x`, and masked base count. |
| `qc/coverage/<id>.mask.bed` | 0-based BED intervals where depth `< consensus_min_depth` that were masked with `N` in the consensus. |
| `consensus/<id>.consensus.fasta` | Depth-masked haploid consensus genome (`><id>`), with 0 IUPAC ambiguity bases for Illumina/ONT runs. |
| `variants/<id>.filtered.vcf.gz` | Normalised, depth/quality/AF-filtered haploid VCF applied to build the consensus. |
| `genotyping/<id>.genotype.json` | Structured Nextclade genotype, major/minor lineage, clade code, QC status (`PASS`/`FAIL`), `qc_reasons`, `raw_clade`, masked consensus statistics, and structured lists of AA substitutions, deletions, and insertions. |
| `reports/<id>.summary_report.txt` | Human-readable summary report with sequence/coverage QC, serotype & co-infection check, Nextclade genotype/QC status, and AA changes grouped by viral protein (`C`, `prM`, `E`, `NS1`–`NS5`). |
| `fhir/genomic/<id>.genomic.fhir.json` | PHI-free FHIR R4 `collection` Bundle containing `DiagnosticReport`, `Observation`s (classification, region-studied QC, embedded consensus sequence, and AA variants), `Device`, and `Provenance`. |
| `fhir/bundle/<fid>.bundle.fhir.json` | Self-contained FHIR R4 `transaction` Bundle linking genomic and clinical resources (only generated when `--patient_metadata` is supplied). |
| `pipeline_info/software_versions.yml` | Exact tool versions, workflow version, FHIR profile, and reference panel used for the run. |
