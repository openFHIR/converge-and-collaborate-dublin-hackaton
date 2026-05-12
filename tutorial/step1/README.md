# Step 1: Setup Firely Server

## License

Download an evaluation license from Firely website: Request a trial license at https://fire.ly/firely-server-trial/.

You'll receive it in a mail in a minute or two and it is a prerequisite for our next step.

Once you have it, place it in the `config/` folder inside `tutorial/workspace/` and name it
`firely-license.json`. This folder will also hold `appsettings.instance.json` and other config files as the tutorial progresses.

## Docker compose

Throughout this tutorial you will maintain a single `docker-compose.yml` file in the **`tutorial/workspace/`** directory of
this repository. Each step adds services or config to that file. The `docker-compose.yml` inside each step folder is
a reference snapshot showing what yours should look like at the end of that step.

All `docker compose` commands should be run from the **`tutorial/workspace/`** directory. See [workspace/README.md](../workspace/README.md)
for an overview of the expected folder structure.

The [docker-compose.yml](docker-compose.yml) you see in the step1 folder is what your docker-compose should contain at
the end of this step.

### MongoDB

Firely server requires a persistence layer. Could be MSSQL or MongoDB. For the prupose of this session, we'll set up a
local MongoDB instance.

Add the following container in your docker compose file

```yaml
  mongodb:
    image: mongo:7
    command: [ "mongod", "--logpath", "/dev/null" ]
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin
    volumes:
      - mongodb-data:/data/db
    healthcheck:
      test: [ "CMD", "mongosh", "--eval", "db.adminCommand('ping')" ]
      interval: 5s
      timeout: 5s
      retries: 10
```

### Firely Server

Now we can set up our Firely Server. For this to work, we require 2 files in the `config/` folder inside `tutorial/workspace/`:

(1) `appsettings.instance.json`  
(2) `firely-license.json` — your trial license from above

#### appsettings

Create it with sections below and place it in the `config/` folder. You can compare it with the reference [appsettings.instance.json](appsettings.instance.json) in this step folder.

```yaml
"Repository": "MongoDb",
"MongoDbOptions": {
  "ConnectionString": "mongodb://admin:admin@mongodb:27017/fhir?authSource=admin",
  "EntryCollection": "vonkentries"
},
"Administration": {
  "Repository": "MongoDb",
  "MongoDbOptions": {
    "ConnectionString": "mongodb://admin:admin@mongodb:27017/fhir_admin?authSource=admin",
    "EntryCollection": "vonkadmin"
  }
},
 ```

#### Firely Server container in docker compose

```yaml
  firely:
    image: firely/server:6.7.0
    ports:
      - "4080:4080"
    environment:
      - VONK_PATH_TO_SETTINGS=/app/config
    volumes:
      - ./config:/app/config
      - vonk-imported:/app/vonk-imported
    depends_on:
      mongodb:
        condition: service_healthy
 ```

The `./vonk-imported` bind mount persists Firely Server's conformance resource import history across container
recreations. Without it, Firely re-imports all conformance resources on every restart, adding ~5 minutes to startup.

Create the `vonk-imported/` directory inside `tutorial/workspace/` before starting the stack so that it has the correct permissions.

**Linux / macOS:**
```bash
mkdir -p vonk-imported && chmod 777 vonk-imported
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path vonk-imported
```

Run these from the `tutorial/workspace/` directory.

Finally, set everything up by running from the `tutorial/workspace/` directory:

```bash
docker compose up -d
```

## Assertion of the Step 1

You can see if you've sucessfully set up Firely Server by going to http://localhost:4080/metadata in a browser or in
your prefered REST testing application (i.e. Postman, Bruno, ..).

Do note that initial boot of Firely Server takes a while to load all conformance resources (~5 min). If you're getting
an OpeartionOutcome with 423 Locked, feel free to move on to Step 2 and revisit this `assertion` after some time.

Additionally, you're welcome to compare your docker-compose.yml file with [docker-compose.yml](docker-compose.yml).

