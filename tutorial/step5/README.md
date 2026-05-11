# Step 5: FHIRConnect Mappings

In this step, we'll provision existing IPS FHIRConnect mappings to our openFHIR setup. This can be done in two ways, either
by bootstrapping them or by provisioning them through a RESTful API (https://open-fhir.com/documentation/2.2.0/stateconfiguration.html).

## Bootstrapping openFHIR State

[fhirconnect](../../fhirconnect) subdirectory already includes all existing IPS mappings. The only thing we need to do
is mount this subdirectory to the configured bootstrap location ot our openFHIR container and recreate the container (restart the engine).

Engine will scan this directory and provision all FHIRConnect mappings in there. Add the following under `volumes`

```yaml
  - ./../../fhirconnect/:/app/bootstrap/
```

then recreate the openFHIR container from the **repo root**, so it picks up the new volume mount and restarts the engine:

```bash
docker compose up -d openfhir
```

You should see in the startup log that these files are being provisioned.

## Assertion of Step 5

`GET http://localhost:8083/fc/context` should give you provisioned context model mapping for our IPS.