# Output Files
The primary output is the **HL7 FHIR** bundle, which integrates genomic typing, lineage assignment, and mutation findings with clinical metadata.

## FHIR Genomic Observations
Converts annotated genomic features into standardized FHIR **Observation** resources.
### Observation Resources
The pipeline generates specific Observation resources for:
*   **Dengue Classification**: 
    *   **Serotype**: The specific Dengue virus serotype (DENV-1, DENV-2, DENV-3, DENV-4).
    *   **Genotype**: The specific genotype within the serotype (e.g., Genotype I, Genotype II).
    *   **Lineage**: Major and Minor lineage assignments based on Nextclade analysis.
    *   **Confidence**: Confidence level of the assignment.
*   **Viral Consensus Genome**:
    *   **Sequence**: The full consensus nucleotide sequence.
    *   **Metrics**: Sequence length, coverage, and completeness (N content).
*   **Genetic Variants**:
    *   **Gene ID**: Viral gene symbol (e.g., *E*, *NS1*, *NS5*).
    *   **Amino Acid Change**: The specific mutation in HGVS notation (e.g., `p.Val123Ile`).

## Clinical Data Integration
Merges the FHIR Genomics Observations with patient, facility, and practitioner information to create a complete genomic diagnostic report document.

### DiagnosticReport Resource
*   **Conclusion**: A summary including the Serotype, Genotype, and Lineage.
*   **Links**: References the Patient, Specimen, and all generated Observations.

## Analysis Logic
### Serotyping
Determined by mapping reads against a database of reference sequences (DENV-1 through 4 and Sylvatic strains) using BLASTn.
*   **Logic**: The serotype with the highest number of high-identity mapped reads is selected.

### Genotyping & Lineage
*   **Clade Assignment**: Assigns specific clades (e.g., `1I.A`, `3III_B.3.2`) which are parsed into Genotypes and Lineages.
*   **Mutation Calling**: Identifies amino acid substitutions relative to the serotype-specific reference.

## Output Directory Structure
```text
results/
├── qc/
│   ├── multiqc_report.html       
├── host_removal/
│   └── *.hostile_log.txt        
├── serotyping/
│   ├── *.serotype.json          
├── consensus/
│   ├── *_consensus.fasta  
├── genotyping/
│   ├── *.genotype_lineage.json   
├── fhir/
│   └── *.fhir.json               
├── fhir_merged/
│   └── *.merged.fhir.json       
├── reports/
│ └── *_dengue_report.txt
```
