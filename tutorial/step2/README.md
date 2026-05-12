# Step 2: Install openFHIR Firely Plugin

For Firely Server to work as a facade on top of an openEHR CDR, it needs to have a plugin available for intercepting
FHIR API invocations and routing them to openFHIR and openEHR.

A plugin for this is already available at: https://github.com/openFHIR/openfhir-firely-plugin

Building that repository would give you a DLL file containing the business logic described above. For the purpose of
this session, you can find the .dll files in this project as well.

## Installing the plugin

Move the two .dll files you find in this step to a folder inside `tutorial/workspace/.plugins`.

We need to make sure Firely Server picks them up by mounting the `.plugins` folder into the container.
Add the following under the `firely` service's `volumes` in your `docker-compose.yml`:

```yaml
volumes:
  - ./.plugins:/app/plugins:ro
```

And last thing is to enable this plugin in `appsettings.instance.json` of FirelyServer. For this, add the following section in
appsettings.instance.json:

(notice `OpenFhirFirelyPlugin` at the bottom of the `Include`, this enables our plugin)

```yaml
 "PipelineOptions": {
  "PluginDirectory": "./plugins",
  "Branches": [
    {
      "Path": "/",
      "Include": [
        "Vonk.Core",
        "Vonk.Plugin.Operations",
        "Vonk.Fhir.R3",
        "Vonk.Fhir.R4",
        "Vonk.Repository.MongoDb.MongoDbVonkConfiguration",
        "Vonk.Subscriptions",
        "Vonk.Plugin.Smart",
        "Vonk.UI.Demo",
        "Vonk.Plugin.ConvertOperation.ConvertOperationConfiguration",
        "Vonk.Plugin.BinaryWrapper",
        "Vonk.Plugin.Audit",
        "Vonk.Plugin.UpdateNoOp.UpdateNoOpConfiguration",
        "Vonk.Plugin.UpdateNoOp.PatchNoOpConfiguration",
        "Vonk.Plugin.UpdateNoOp.ConditionalUpdateNoOpConfiguration",
        "Vonk.Plugin.SearchAnonymization",
        "Vonk.Plugins.Terminology",
        "OpenFhirFirelyPlugin"
      ],
      "Exclude": [
        "Vonk.Subscriptions.Administration",
        "Vonk.Plugin.Audit.Integrity"
      ]
    },
    {
      "Path": "/administration",
      "Include": [
        "Vonk.Core",
        "Vonk.Fhir.R3",
        "Vonk.Fhir.R4",
        "Vonk.Repository.MongoDb.MongoDbTaskConfiguration",
        "Vonk.Repository.MongoDb.MongoDbAdminConfiguration",
        "Vonk.Repository.Memory.MemoryAdministrationConfiguration",
        "Vonk.Subscriptions.Administration",
        "Vonk.Plugins.Terminology",
        "Vonk.Administration",
        "Vonk.Plugin.BinaryWrapper"
      ]
    }
  ]
},
```

## Assertion of the Step 2

After applying the changes, recreate Firely Server from the **`tutorial/workspace/` directory**:

```bash
docker compose up firely --force-recreate
```

Check the container logs:

**Linux / macOS:**
```bash
docker compose logs firely | grep -i "OpenFhir"
```

**Windows (PowerShell):**
```powershell
docker compose logs firely | Select-String "OpenFhir"
```

**Windows (Command Prompt):**
```cmd
docker compose logs firely | findstr /i "OpenFhir"
```

You should see a line indicating that the `OpenFhirFirelyPlugin` was loaded. If you see an error about a missing
assembly, verify that both `.dll` files are present in `tutorial/workspace/.plugins/` and that the volume mount path is correct.

Feel free to compare your `appsettings.instance.json` and `docker-compose.yml` with those in this step2 subdirectory. They
should more or less align after completion of this step.