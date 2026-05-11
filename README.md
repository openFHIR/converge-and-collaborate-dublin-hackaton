# converge-and-collaborate-dublin-hackaton

Let's build resources for Converge and Collaborate, Dublin 2026

## Lets Build: openEHR-to-FHIR IPS Integration

This session walks through building a working FHIR facade on top of an openEHR CDR, using the International Patient
Summary (IPS) as the target use case.

The facade sits between FHIR clients and an openEHR CDR, translating FHIR requests into AQL queries and openEHR
compositions into FHIR resources on the fly, using [FHIRConnect](https://github.com/openFHIR/fhirconnect-spec) mappings
as the translation layer.

### Target Architecture

![Target Architecture](target_architecture.png)

| Flow | Diagram |
|------|---------|
| Querying FHIR resources (e.g. `GET /AllergyIntolerance`) | ![FHIR Query](fhir_query.png) |
| Storing FHIR resources (e.g. `POST` IPS Bundle) | ![FHIR Store](fhir_store.png) |
| Fetching the IPS summary document (`$summary`) | ![FHIR Summary](fhir_summary.png) |

### Tutorial Steps

Each step builds on the previous one, incrementally adding services to a single `docker-compose.yml` that is fully
runnable at the end of each step. Follow the README in each subfolder.

| Step | What you build | Folder |
|------|---------------|--------|
| 1 | Firely Server + MongoDB running locally | [tutorial/step1](tutorial/step1/) |
| 2 | openFHIR Firely Plugin installed into Firely Server | [tutorial/step2](tutorial/step2/) |
| 3 | openFHIR running locally | [tutorial/step3](tutorial/step3/) |
| 4 | All services wired together — Firely → openFHIR → openEHR CDR | [tutorial/step4](tutorial/step4/) |
| 5 | Existing IPS FHIRConnect mappings bootstrapped into openFHIR | [tutorial/step5](tutorial/step5/) |
| 6 | Create a patient, ingest an IPS Bundle, test existing mappings end-to-end | [tutorial/step6](tutorial/step6/) |
| 7 | New FHIRConnect mappings for the IPS Medical Devices section | [tutorial/step7](tutorial/step7/) |

### Repository Structure

```
fhirconnect/ips/    — FHIRConnect mapping files for the IPS (problems, allergies, medications stub)
tutorial/step*/     — Step-by-step instructions, docker-compose.yml, and config files for each step
```

### Prerequisites

- Docker and Docker Compose
- A Firely Server trial license — request one at https://fire.ly/firely-server-trial/
- A REST client (Postman, Bruno, or curl)

### Resources

- [openFHIR Firely Plugin](https://github.com/openFHIR/openfhir-firely-plugin)
- [openFHIR Enterprise documentation](https://open-fhir.com/documentation/2.2.0/installation.html)
- [FHIRConnect specification](https://github.com/openFHIR/fhirconnect-spec)
- [IPS Implementation Guide](https://hl7.org/fhir/uv/ips/)
