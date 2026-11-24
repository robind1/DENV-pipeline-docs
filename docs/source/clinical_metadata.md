# Clinical metadata

The clinical step is optional. It runs when `--patient_metadata` is given and converts up to three
tables into FHIR resources. Its outputs are kept separate from the genomic bundles (`fhir/genomic`),
which never contain personal data:
- **Patient table** (`--patient_metadata`): one row per sample, producing `Patient`, `Specimen`,
  `Encounter`, and `ServiceRequest`.
- **Organization table** (`--organization_metadata`, optional): the performing laboratory,
  producing `Organization`.
- **Practitioner table** (`--practitioner_metadata`, optional): the responsible clinician,
  producing `Practitioner` and `PractitionerRole`.

```bash
nextflow run . -profile docker \
  --input samplesheet.csv \
  --patient_metadata site/patients.csv \
  --organization_metadata site/organization.csv \
  --practitioner_metadata site/practitioner.csv \
  --performer_org_id 100000001
```

Header-only templates live in `assets/metadata_templates/`.

A sample with no row in the patient table still receives a PHI-free genomic bundle and is recorded
as `missing_clinical` in `fhir/bundle/link_summary.tsv`. Pass `--require_clinical true` to make
missing clinical rows a pipeline error.

## File format

- CSV (UTF-8, optional BOM) or `.xlsx` (requires `openpyxl`; use CSV in container runs).
- All cells are read as text so NIKs and Indonesian administrative codes retain leading zeros.
- Comment lines starting with `#` before the header row are ignored.
- Blank cells are omitted from the FHIR resource rather than filled with placeholders (except
  Dengue specimen defaults in `--fhir_profile satusehat`, where Blood specimen SNOMED `119297000`
  and collection method `BLD` are applied when omitted).
- Structural errors fail fast rather than producing silent empty output: missing file, missing
  `sample_id` column, duplicate `sample_id`, or two sample IDs colliding on the same FHIR ID.

## Patient table

`sample_id` is required; all other columns are optional. Unknown columns are ignored with a
warning.

| Column | Used for | Notes |
| --- | --- | --- |
| `sample_id` | links the row to the genomic sample | exact match (or without `_ont`/`_illumina` suffix); never a prefix match |
| `family_name`, `given_name` | `Patient.name` | |
| `gender` | `Patient.gender` | `laki-laki`/`pria`/`male`/`m`, `perempuan`/`wanita`/`female`/`f` |
| `birth_date` | `Patient.birthDate` | `YYYY`, `YYYY-MM`, or `YYYY-MM-DD`; invalid dates are omitted and flagged |
| `age`, `diagnosis_date`, `treatment_history`, `contact_number` | reserved | kept for schema compatibility with TB/KP metadata tables |
| `nik` | `Patient.identifier` (`satusehat` only) | 16 digits; non-16-digit values are omitted and flagged |
| `address`, `city`, `state` | `Patient.address` | |
| `country` | `Patient.address.country` | optional; `satusehat` defaults to `ID` |
| `province_code`, `city_code`, `district_code`, `village_code` | `administrativeCode` extension (`satusehat` only) | |
| `latitude`, `longitude` | `geolocation` extension | decimal degrees |
| `citizenship_status` | `citizenshipStatus` extension (`satusehat` only) | e.g. `WNI`, `WNA` |
| `specimen_type_code`, `specimen_type_display` | `Specimen.type` (SNOMED CT) | e.g. `119297000` / `Blood specimen` |
| `specimen_collection_method_code`, `_display`, `_text` | `Specimen.collection.method` (`v2-0488`) | e.g. `BLD` / `Blood` |
| `specimen_quantity_value`, `specimen_quantity_unit` | `Specimen.collection.quantity` (UCUM) | e.g. `2.0`, `mL` |
| `specimen_collected_date` | `Specimen.collection.collectedDateTime`, `Encounter.period.start` | FHIR date or dateTime |
| `specimen_received_date` | `Specimen.receivedTime` | FHIR date or dateTime |
| `encounter_id` (optional) | external `Encounter` reference | if supplied, no `Encounter` is created and `external_refs.json` points to the existing server id |
| `location_id` (optional) | `Encounter.location` | external `Location/<id>` reference |

## Organization table

Only the first data row is used. `org_id` is required.

| Column | Used for |
| --- | --- |
| `org_id` | `Organization.id` and identifier |
| `name`, `alias` | `Organization.name`, `Organization.alias` |
| `type_code`, `type_display`, `type_text` | `Organization.type` |
| `phone`, `email` | `Organization.telecom` |
| `address_line`, `city`, `state`, `country` | `Organization.address` |
| `province_code`, `city_code`, `district_code`, `latitude`, `longitude` | address extensions |

## Practitioner table

Only the first data row is used. `practitioner_id` is required.

| Column | Used for |
| --- | --- |
| `practitioner_id` | `Practitioner.id` |
| `nik` | `Practitioner.identifier` (`satusehat` only) |
| `name`, `phone`, `gender`, `birth_date` | `Practitioner` demographics |
| `str_kki_number`, `qualification_period_start` | `Practitioner.qualification` (`satusehat` only) |
| `role_id`, `role_code`, `role_display`, `role_text` | `PractitionerRole` (created when `role_id` is set) |

## Pre-existing server resources (`--external_refs`)

If `Patient`, `Encounter`, or other resources already exist on the destination FHIR server, pass a
JSON map with `--external_refs`:

```json
{
  "DENV-001": {
    "Patient": "P000123456",
    "Encounter": "E000987654"
  }
}
```

`link_fhir_bundle.py` omits the matching local resources from the transaction bundle and rewrites
references to the server IDs.

## Handling personal data safely

Keep real clinical metadata CSV/XLSX files, `fhir/clinical/`, and `fhir/bundle/` out of version
control. `.gitignore` excludes them by default, and `tools/dev/check_no_phi.py` blocks commits
containing unlabelled CSV data rows, real 16-digit NIKs, `.xlsx`/`.db` files, or credential keys.
Pass `--publish_clinical false` if you only want the linked bundles written to `--outdir`.
