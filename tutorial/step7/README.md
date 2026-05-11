# Step 7: Create Medical Devices Mappings

In this step you will write FHIRConnect mappings for the IPS Medical Devices section, which was not covered by the
existing mappings provisioned in Step 5.

The full field-level mapping is specified in [`../step6/Medica_ Devices.csv`](../step6/Medica_%20Devices.csv).

## Background

The target FHIR resources are `DeviceUseStatement` and `Device` (IPS profile: `DeviceUseStatement-uv-ips`).

The openEHR archetypes involved are:
- `EVALUATION.device_summary.v0` — the top-level device record, maps to `DeviceUseStatement`
- `CLUSTER.device.v1` (nested inside `device_summary`) — the device details, maps to `Device`

## Mapping Table

### Section level

| openEHR archetype | openEHR name | FHIR path | Notes |
|-------------------|--------------|-----------|-------|
| `SECTION.adhoc.v1` | IPS Medical Devices | `Composition.section:sectionMedicalDevices` | Section header |
| `EVALUATION.absence.v2` | Absence of information | — | Used when no device data is available |
| `/items[…absence.v2]/data[at0001]/items[at0002]/value` | Absence statement | — | Free text, no FHIR mapping |
| `/items[…absence.v2]/data[at0001]/items[at0005]/value` | Reason for absence | `Composition.section:sectionMedicalDevices.emptyReason` | |
| `EVALUATION.clinical_synopsis.v1` | Clinical synopsis | — | Narrative only |
| `/items[…clinical_synopsis.v1]/data[at0001]/items[at0002]/value` | Synopsis | `Composition.section:sectionMedicalDevices.text` | Human-readable narrative |

### Entry level: `EVALUATION.device_summary.v0` → `DeviceUseStatement`

| openEHR name | openEHR path | FHIR path | Cardinality | Notes |
|--------------|--------------|-----------|-------------|-------|
| Status | `/items[…device_summary.v0]/data[at0001]/items[at0002]/value` | `DeviceUseStatement.status` | 1..1 | Requires concept map — openEHR values (`Never`, `Current`, `Previous`) do not align with FHIR `DeviceUseStatementStatus` |
| Description | `/items[…device_summary.v0]/data[at0001]/items[at0015]/value` | `DeviceUseStatement.text` | 0..1 | |
| Start date | `/items[…device_summary.v0]/data[at0001]/items[at0022]/items[at0008]/value` | `DeviceUseStatement.timing[x]:timingPeriod.start` | 0..1 | FHIR timing is a choice of `Timing`, `Period`, `dateTime` |
| End date | `/items[…device_summary.v0]/data[at0001]/items[at0022]/items[at0009]/value` | `DeviceUseStatement.timing[x]:timingPeriod.end` | 0..1 | |
| Body site | `/items[…device_summary.v0]/data[at0001]/items[at0022]/items[at0012]/value` | `DeviceUseStatement.bodySite` | 0..1 | Preferred binding: SNOMEDCTBodyStructures |

### Nested cluster: `CLUSTER.device.v1` → `Device`

| openEHR name | openEHR path | FHIR path | Cardinality | Notes |
|--------------|--------------|-----------|-------------|-------|
| Device name | `/items[…device.v1]/items[at0001]/value` | `Device.deviceName.name` | 1..1 | |
| Type | `/items[…device.v1]/items[at0003]/value` | `Device.type` | 0..1 | Preferred binding: Medical Devices IPS (`http://hl7.org/fhir/uv/ips/ValueSet/medical-devices-uv-ips`) |
| Description | `/items[…device.v1]/items[at0002]/value` | `Device.text` | 0..1 | |
| UDI | `/items[…device.v1]/items[at0021]/value` | `Device.udiCarrier.deviceIdentifier` | 0..1 | |
| Manufacturer | `/items[…device.v1]/items[at0004]/value` | `Device.manufacturer` | 0..1 | |
| Date of manufacture | `/items[…device.v1]/items[at0005]/value` | `Device.manufactureDate` | 0..1 | |
| Date of expiry | `/items[…device.v1]/items[at0007]/value` | `Device.expirationDate` | 0..1 | |
| Serial number | `/items[…device.v1]/items[at0020]/value` | `Device.serialNumber` | 0..1 | |
| Model number | `/items[…device.v1]/items[at0023]/value` | `Device.modelNumber` | 0..1 | |
| Batch/Lot number | `/items[…device.v1]/items[at0006]/value` | `Device.lotNumber` | 0..1 | |
| Software version | `/items[…device.v1]/items[at0025]/value` | `Device.version.value` | 0..1 | |
| Other identifier | `/items[…device.v1]/items[at0024'Other identifier']/value` | `Device.identifier` | 0..* | |
| Distinct identifier | `/items[…device.v1]/items[at0024'Distinct identifier']/value` | `Device.distinctIdentifier` | 0..1 | |
| Comment | `/items[…device.v1]/items[at0008]/value` | `Device.note` | 0..1 | openEHR 0..1 vs FHIR 0..* — cardinality mismatch |

## Create the Mapping Files

Create the following files in `fhirconnect/ips/`:

```
fhirconnect/ips/ips.medical_devices.v1.yml              # entry-level mapping: device_summary → DeviceUseStatement + Device
fhirconnect/ips/ips.section_medical_devices.v1.yml      # section mapping
fhirconnect/ips/core/evaluation/org/openehr/
  evaluation/device_summary.v0.yml                      # device_summary archetype mapping
  device_summary_conceptmap.json                        # status concept map (Never/Current/Previous → FHIR codes)
```

Use `ips.problem_diagnosis.v1.yml` as a structural reference for the entry-level mapping and
`ips.section_problem_list.v1.yml` for the section wiring pattern.

## Reload and Verify

Restart the openFHIR container from the **repo root** to bootstrap the new mappings:

```bash
docker compose restart openfhir
```

Verify the mapping was loaded:

```bash
curl http://localhost:8083/fc/context
```

The response should now include the medical devices mapping.

## Test

Re-run the `$summary` call from Step 6:

```http
GET http://localhost:4080/Patient/{patient-id}/$summary
```

The returned IPS Bundle should now include a populated `DeviceUseStatement` section alongside the problems and
allergies from Step 6.
