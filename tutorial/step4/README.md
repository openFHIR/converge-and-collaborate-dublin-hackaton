# Step 4: Integrating services

By this point, we have a local FHIR server capable of integrating to openEHR and to openFHIR. And we have a local
instance of openFHIR.

What's left is for us to connect all the pieces together.

## Integrating Firely Server with openFHIR

In step2, we have enabled a plugin in Firely Server that can seamlessly integrate with openFHIR. What we still need to
do is tell that plugin
where openFHIR actually lives.

To do that, add OpenFhir.BaseUrl in Firely Server's `appsettings.json`. Alternatively, if you've decided to integrate
with a sanbox instance of openFHIR, configure OAuth2 properties.

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
"ClientSecret": "<your-client-secret>",
"Scope": ""
}
}
}
```

## Configuring openEHR Integration

Now we need to tell our openFHIR Firely Plugin where openEHR CDR actually lives and based on what it should decide how
to route calls to it's local FHIR store or to an openEHR CDR.

To do that, we need the following configuration sections:

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

Copy [cdrs.yml](cdrs.yml) to the repo root and fill in the CDR URL provided on the day. Then add the volume mount
to the `firely` service in your root `docker-compose.yml`:

```yaml
volumes:
  - ./cdrs.yml:/app/cdrs.yml:ro
```

### Filters

in FhirQueryFilter section of the configuration, you define for which FHIR queries you want the plugin to invoke
openFHIR and openEHR.

Similarly, you do that in FhirCreateFilter for the create flow, where you define for which FHIR IGs you want to invoke
an openEHR flow of data.

### Applying plugin configuration for openEHR CDRs

The following goes to appsettings.json. Be aware of the root key being OpenFhirPlugin

```yaml
"OpenFhirPlugin": {
  "Interceptor": {
    "CdrsConfigFile": "/app/cdrs.yml",

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
following to appsettings.json (under base appsettings.json)

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
our Firely Server instance. For this, mount logsettings.json as you can find it in step4 subfolder inside docker
container like so:

```yaml
volumes:
  - ./logsettings.json:/app/logsettings.json:ro
```

## Assertion of the Step 4

After updating `appsettings.json` and `docker-compose.yml`, recreate Firely Server container

```bash
docker compose up -d firely
```

Verify your files match those in the step4 subfolder of this tutorial.

Additionally, test `$summary` on a random (can be non-existent) patient, just to verify operation is permitted and
enabled. If you get a "Not Implemented" response, it means that something has gone wrong.

If you get a 200, check the logs and you should see some openFHIR Plugin business logic logs.
