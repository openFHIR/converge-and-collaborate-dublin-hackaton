# Step 1: Setup Firely Server

## License

Download an evaluation license from Firely website: Request a trial license at https://fire.ly/firely-server-trial/.

You'll receive it in a mail in a minute or two and it is a prerequisite for our next step.

Once you have it, place it in a .secret subfolder of this workspace (root, not step1 folder).

## Docker compose

This is a first step of the tutorial where we also start building our complete docker-compose file.
The [docker-compose.yml](docker-compose.yml) you see on the step1 folder is what your docker compose should contain at
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
```

### Firely Server

Now we can set up our Firely Server. For this to work, we require 2 additional files:  
(1) appsettings.json  
(2) a trial Firely Server License

#### appsettings

configure persistence parameters, see [appsettings.json](appsettings.json)

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
    volumes:
      - ./../../.secret/firelyserver-license.json:/app/firely-license.json
      - ./appsettings.json:/app/appsettings.json:ro
    depends_on:
      mongodb:
        condition: service_healthy
 ```

## Assertion of the Step 1

You can see if you've sucessfully set up Firely Server by going to http://localhost:4080/metadata in a browser or in
your prefered REST testing application (i.e. Postman, Bruno, ..).

Additionally, you're welcome to compare your docker-compose.yml file with [docker-compose.yml](docker-compose.yml)