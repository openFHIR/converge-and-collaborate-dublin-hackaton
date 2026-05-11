# Step 6: Testing Existing IPS Mappings

In this step we create a patient, ingest an IPS Bundle, and verify that the existing mappings provisioned in Step 5
(problems and allergies) work end-to-end through the facade.

## Testing

### 1. Create a Patient

First create a Patient through the FHIR facade. This also provisions an EHR in the openEHR CDR and stores the EHR ID
back as a Patient identifier.

```http
POST http://localhost:4080/Patient
Content-Type: application/fhir+json

{
  "resourceType": "Patient",
  "active": true,
  "name": [{ "use": "official", "family": "Duck", "given": ["Donald"] }],
  "gender": "male"
}
```

Note the `id` returned — you will use it as `{patient-id}` below. Also note the `identifier` containing the openEHR
EHR ID; the plugin uses this to route subsequent requests to the correct EHR.

### 2. FHIR to openEHR (store an IPS Bundle)

POST an IPS Bundle to the FHIR facade. Because the bundle's `Composition` resource carries the IPS profile
(`http://hl7.org/fhir/uv/ips/StructureDefinition/Composition-uv-ips`), the plugin's `FhirCreateFilter` will intercept
it and store the data as an openEHR composition in the CDR instead of in the local FHIR store.

```bash
curl -X POST http://localhost:4080/ \
  -H "Content-Type: application/fhir+json" \
  --data-binary @ips_example.json
```

Make sure to adjust the Composition.subject.reference to reference your patient created above.

### 3. openEHR to FHIR (retrieve the IPS summary)

```http
GET http://localhost:4080/Patient/{patient-id}/$summary
```

The plugin will query the CDR via AQL, call openFHIR to map the composition, and return a FHIR IPS Bundle. You should
see populated `Condition` and `AllergyIntolerance` sections. The medical devices section will be empty until Step 7.