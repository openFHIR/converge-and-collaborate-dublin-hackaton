# Step 4: Integrating services

By this point, we have a local FHIR server capable of integrating to openEHR and to openFHIR. And we have a local
instance of openFHIR.

What's left is for us to connect all the pieces together.

## Integrating Firely Server with openFHIR

In step2, we have enabled a plugin in Firely Server that can seamlessly integrate with openFHIR. What we still need to
do is tell that plugin
where openFHIR actually lives.

To do that, add OpenFhir.BaseUrl in Firely Server's `appsettings.instance.json`. Alternatively, if you've decided to
integrate
with a sandbox instance of openFHIR, configure OAuth2 properties.

For local openFHIR (Step 3):

```json
"OpenFhirPlugin": {
"OpenFhir": {
"BaseUrl": "http://openfhir:8080"
}
}

```

Alternatively, if using the sandbox instead of a local instance:

```json
"OpenFhirPlugin": {
"OpenFhir": {
"BaseUrl": "https://sandbox.open-fhir.com",
"OAuth2": {
"TokenUrl": "https://sandbox.open-fhir.com/auth/realms/open-fhir/protocol/openid-connect/token",
"ClientId": "<your-client-id>",
"ClientSecret": "<your-client-secret>"
}
}
}
```

## Configuring openEHR Integration

Now we need to tell our openFHIR Firely Plugin where openEHR CDR actually lives and based on what it should decide how
to route calls to it's local FHIR store or to an openEHR CDR.

To do that, we need the following configuration sections to Firely's appsettings.instance.json:

`Interceptor` is a configuration property telling our Interceptor where exactly openEHR CDRs live and how it can connect
to them.

`FhirCreateFiler` tells our plugin for which incoming FHIR payloads it should route to openEHR CDR instead of to its own
local FHIR store.

`FhirQueryFilter` tell ours plugin which FHIR Queries should be translated to AQLs and invoked on integrated CDR rather
than on its own local FHIR store.

### CDRs

Configuring integrated openEHR CDRs happens with a yaml file referenced in CdrsConfigFile.

Example of cdrs.yml as follows:

```yml
- id: local
  name: EHRbase
  baseUrl: http://ips.open-fhir.com/ehrbase/rest
  authMethod: basic
  basicAuth:
    username: ehrbase-user
    password: SuperSecretPassword
```

Copy [cdrs.yml](cdrs.yml) to your `config/` folder and fill in the CDR URL provided on the day. It will be
available inside the container at `/app/config/cdrs.yml`, which matches the `CdrsConfigFile` value already set in
`appsettings.instance.json`.

### Filters

in FhirQueryFilter section of the configuration, you define for which FHIR queries you want the plugin to invoke
openFHIR and openEHR.

Similarly, you do that in FhirCreateFilter for the create flow, where you define for which FHIR IGs you want to invoke
an openEHR flow of data.

### Applying plugin configuration for openEHR CDRs

The following goes to appsettings.instance.json. Be aware of the root key being OpenFhirPlugin (so the same root you've probably
added before already when adding OpenFhir endpoints - don't add it twice)

```yaml
"OpenFhirPlugin": {
  "Interceptor": {
    "CdrsConfigFile": "/app/config/cdrs.yml",

                   // Profiles that trigger the OpenEHR store flow (FhirCreateMiddleware).
                   // Any POST whose resource meta.profile matches one of these URLs is forwarded to the CDR.
    "FhirCreateFilter": {
      "InterceptedProfiles": [
        "http://hl7.org/fhir/uv/ips/StructureDefinition/Composition-uv-ips"
      ]
    },

                   // Rules that trigger the OpenEHR query flow (FhirQueryMiddleware).
                   // Rules are evaluated in order; the first match wins.
                   // Use _resourceType to match the last path segment of the URI.
                   // Use * as a value to match any non-absent value.
    "FhirQueryFilter": {
      "Rules": [
        {
          "TemplateId": "International Patient Summary",
          "FhirQuery": { "_resourceType": "AllergyIntolerance" }
        },
        {
          "TemplateId": "International Patient Summary",
          "FhirQuery": { "_resourceType": "Condition", "verification-status": "confirmed" }
        },
        {
          "TemplateId": "International Patient Summary",
          "FhirQuery": { "_resourceType": "DeviceUseStatement" }
        }
      ]
    }
  }
},

```

## $summary operation

Firely Server requires one additional configuration to enable the `$summary` operation, for which you need to add the
following to appsettings.instance.json (under base appsettings.instance.json)

```yaml
"Operations": {
  "$summary": {
    "Name": "$summary",
    "Level": [ "Instance" ],
    "Enabled": true,
    "RequireAuthorization": "Never",
    "RequireTenant": "WhenTenancyEnabled"
  }
},
```

actual business logic/implementation of this operation is provided by openFHIR Firely Plugin.

## Additional logging

To be able to debug if something is going wrong (or right), it may be a good idea to add additional logging output to
our Firely Server instance. Copy [logsettings.json](logsettings.json) to your `config/` folder — Firely will pick it
up automatically from the same directory as `appsettings.instance.json`.

## Assertion of the Step 4

After updating `appsettings.instance.json` and `docker-compose.yml`, restart Firely Server container

```bash
docker compose restart firely
```

Verify your files match those in the step4 subfolder of this tutorial.

Additionally, test `$summary` on a random (can be non-existent) patient, just to verify operation is permitted and
enabled. If you get a "Not Implemented" response, it means that something has gone wrong. If you get a
`No EHR ID found for patient .. on CDR 'local'` error, it means you are where you want to be. We'll fix this error in step 6.