# FHIR output

DENVtoFHIR splits FHIR generation into three layers—the same architecture used by TBtoFHIR 2.0 and
KPtoFHIR 2.0—so genomic results remain free of personal data by default and can be linked to
clinical records only when needed.

| Layer | Output | Contains | Personal data |
| --- | --- | --- | --- |
| 1. Genomic (always) | `fhir/genomic/<id>.genomic.fhir.json` | `DiagnosticReport`, `Observation`s (classification, region-studied QC, embedded consensus sequence, AA variants), `Device`, `Provenance`. Subject is `Patient/<fid>-patient` only. | **No** |
| 2. Clinical (with `--patient_metadata`) | `fhir/clinical/` | `Patient`, `Specimen`, `Encounter`, `ServiceRequest` per sample; shared `Organization`, `Practitioner`, `PractitionerRole`. | **Yes** |
| 3. Linked transaction (with `--patient_metadata`) | `fhir/bundle/<fid>.bundle.fhir.json` | One FHIR `transaction` Bundle per sample with all internal references resolved to deterministic `urn:uuid:` fullUrls. | **Yes** |

## Profiles

- **`--fhir_profile generic`** (default): emits HL7 FHIR R4 resources conforming to
  [Genomics Reporting IG 3.0.0](https://hl7.org/fhir/uv/genomics-reporting/STU3/)
  (`genomic-report`, `region-studied`, `variant`), with LOINC `51969-4` and `81247-9` on
  `DiagnosticReport`. No Kemkes extensions or NIK identifiers are emitted.
- **`--fhir_profile satusehat`**: adds Indonesian Ministry of Health SATUSEHAT StructureDefinitions
  (`https://fhir.kemkes.go.id/r4/StructureDefinition/*`), the `genomics` tag
  (`http://terminology.kemkes.go.id/sp#genomics`), NIK and `sys-ids.kemkes.go.id` identifiers,
  `administrativeCode` / `citizenshipStatus` extensions, and Dengue Blood specimen defaults when
  specimen columns are blank.

## Genomic resources

### `DiagnosticReport` (`<fid>-genomic-report`)

- `category`: `http://terminology.hl7.org/CodeSystem/v2-0074#GE` (Genetics)
- `code`: LOINC `51969-4` (*Genetic analysis report*) + `81247-9` (*Master HL7 genetic variant
  reporting panel*) in `generic`; `81247-9` in `satusehat`; text `"Dengue Virus Genomic Analysis Report"`
- `result`: references to every `Observation` in the bundle
- `conclusion`: human-readable summary naming serotype, genotype, lineage/clade, consensus
  completeness, confidence (or explicit QC-FAIL reasons when `Indeterminate`), and any
  co-infection warning
- `conclusionCode`: coded serotype (`denv-serotype`) and, when QC passes, coded Nextclade clade
  (`denv-lineage`)
- `presentedForm`: base64-encoded PHI-free XHTML summary

### Classification `Observation` (`<fid>-classification`)

- `code`: `denvfhir-observation-code#dengue-classification` (replaces the invalid serology LOINC
  `31343-7` used in v1)
- `valueCodeableConcept`: coded serotype (`denv-serotype`) with descriptive text
- `component`s: `serotype`, `genotype`, `major-lineage`, `minor-lineage`, `clade` (`denv-lineage`),
  `classification-confidence`, `sequencing-qc` (`PASS`/`FAIL`), `sylvatic-flag` (`valueBoolean`),
  `coinfection-flag` (`valueBoolean`), and `serotype-read-fraction` (`%`)
- `note`: attached when `coinfection_suspected` is `true`, reporting the secondary serotype, read
  count, and read fraction

### Coverage & QC `Observation` (`<fid>-region-studied`)

- `meta.profile`: `http://hl7.org/fhir/uv/genomics-reporting/StructureDefinition/region-studied`
- `code`: LOINC `53041-0` (*DNA region of interest panel*)
- `component`s: reference accession (LOINC `48013-7`), `sequence-length` (`{bp}`),
  `sequence-completeness` (`%`), `ambiguous-bases` (`%`), `gc-content` (`%`), `sequencing-qc`, and
  (when read coverage was assessed) `coverage-breadth` (`%` at `>=consensus_min_depth`) and mean
  depth (LOINC `82121-5`, `{fold}`)

### Consensus Sequence `Observation` (`<fid>-consensus-sequence`)

Included by default (`--fhir_include_sequence true`) whenever the consensus sequence is non-empty:
- `code`: LOINC `86206-0` + `denvfhir-observation-code#viral-consensus-sequence`
- `valueString`: full depth-masked consensus nucleotide sequence (~10.7 kb)
- `component`s: `sequence-length`, `sequence-completeness`, `ambiguous-bases`, and `gc-content`

Pass `--fhir_include_sequence false` to omit the nucleotide string from FHIR bundles.

### Variant `Observation`s (`<fid>-var-<n>`)

One `Observation` per detected amino acid substitution, deletion, or insertion from Nextclade:
- `meta.profile`: `http://hl7.org/fhir/uv/genomics-reporting/StructureDefinition/variant`
- `category`: `laboratory`
- `code`: LOINC `69548-6` (*Genetic variant assessment*), `valueCodeableConcept`: LOINC `LA9633-4`
  (*Present*)
- `component`s:
  - LOINC `48018-6` (*Gene studied [ID]*): viral protein name as plain `text` (`C`, `prM`, `E`,
    `NS1`, `NS2A`, `NS2B`, `NS3`, `NS4A`, `NS4B`, `NS5` — human HGNC codes are not emitted)
  - LOINC `48005-3` (*Amino acid change (pHGVS)*): HGVS protein coding (e.g. `p.D37N`, `p.K12del`,
    `p.100insGS`) with `<ref_acc>:<gene>:<hgvs_p>` text
  - LOINC `48013-7` (*Genomic reference sequence [ID]*): reference accession (e.g. `NC_001477.1`)
  - LOINC `48019-4` (*DNA change type*): Sequence Ontology coding (`SO:0001583` `missense_variant`,
    `SO:0001822` `inframe_deletion`, or `SO:0001821` `inframe_insertion`)

### `Device` (`denvfhir-pipeline`) and `Provenance` (`<fid>-provenance`)

- `Device` records `DENVtoFHIR`, the pipeline version, source repository URL, and key thresholds
  (`consensus-min-depth`, `consensus-min-af`, `variant-min-quality`, `min-completeness`,
  `primer-trimmed`).
- `Provenance` links the `DiagnosticReport` and every `Observation` to `Device/denvfhir-pipeline`
  and the performing `Organization`.

## Project CodeSystems

Concepts without standard LOINC/SNOMED codes use CodeSystems defined in `assets/fhir/` under
`http://terminology.spheres.id/CodeSystem/`:

| CodeSystem | Purpose |
| --- | --- |
| `denv-serotype` | `DENV-1`, `DENV-2`, `DENV-3`, `DENV-4`, `unknown` |
| `denv-lineage` | Nextclade Dengue lineage codes (Hill et al. 2024, e.g. `1V_D.1.1`, `2III_C.2`, `3I_A.1`, `4II_A.3`) |
| `denvfhir-observation-code` | Classification, consensus sequence, coverage breadth, QC, sylvatic, and co-infection observation/component codes |
| `pipeline-property` | `Device.property` configuration keys |

## Validation and upload

- **Validation**: `FHIR_VALIDATE` and `FHIR_VALIDATION_SUMMARY` are wired into the workflow using
  HAPI FHIR `validator_cli.jar` 6.5.26 in batches (`--fhir_validation_batch_size 50`). Currently
  `--skip_fhir_validation` defaults to `true`; pass `--skip_fhir_validation false` to validate
  genomic and linked bundles against `hl7.fhir.uv.genomics-reporting#3.0.0` and `assets/fhir/`.
- **Upload**: runs only when `--fhir_server_url` is set, one bundle at a time (`maxForks 1`) with
  exponential backoff on HTTP 429, using Nextflow secrets (`DENVFHIR_CLIENT_ID`,
  `DENVFHIR_CLIENT_SECRET`, and optionally `DENVFHIR_API_KEY`). When validation is enabled, only
  linked bundles with 0 errors are uploaded.
