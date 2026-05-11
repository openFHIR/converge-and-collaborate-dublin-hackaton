# Step 3: Install openFHIR

By this point, we have a local Firely Server running with the openFHIR plugin loaded. The plugin is capable of
integrating with an openEHR CDR and with openFHIR, but those connections are not wired up yet — that happens in Step 4.

What we need first is a local instance of openFHIR to connect to.

## Installing openFHIR

(If you would not want to set up this locally, you can jump to Step 3.alt and use a sandbox instance in cloud)

openFHIR can be run locally as a docker container and leverage existing MongoDB we've already setup for Firely Server to store mappings.

Add the section below to your `docker-compose.yml` in the **repo root**, then run `docker compose up -d` from there.
This will start openFHIR alongside the already-running services without recreating them.

```yaml
  openfhir:
    image: openfhir/openfhir:latest
    container_name: openfhir
    ports:
      - "8083:8080"
    environment:
      - SPRING_DATA_MONGODB_URI=mongodb://admin:admin@mongodb:27017/openfhir?authSource=admin
      - SPRING_DATA_MONGODB_DATABASE=openfhir
      - OPENFHIR_DB_TYPE=mongo
    depends_on:
      mongodb:
        condition: service_healthy
```

## Assertion of the Step 3

In your preferred RESTful API testing tool (i.e. Postman, Bruno) or simply in a browser go to http://localhost:8083/health, which should return a 200 OK with "UP" message. This means openFHIR is running locally.

Additionally, feel free to compare your docker compose with that inside this step3 subfolder.

# Step 3.alt: Using Sandbox
If you'd rather use a sanbdox instance of openFHIR Enterprise, please visit https://sandbox.open-fhir.com and create an account.