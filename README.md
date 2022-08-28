Docker based Ergo setup. This is a fork of [Ergo Setup](https://github.com/abchrisxyz/ergo-setup) but this repo runs less components and runs on a single docker compose network (the node component runs on the same network as the explorer components).

Install [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/), then follow the instructions below for the components you'd like to run.

> Containers are configured to expose certain ports for convenient access within a home lab context. Modify the `ports` values in the `docker-compose.yml` files if needed.

```
# Make sure that you are in the node directory
cd node

# Volume and network expected by node compose file.
docker volume create --name=ergo_redis
docker network create ergo-node

# In the docker-compose.yml, the node version image is set to `ergoplatform/ergo:v4.0.41`.
# Update this to the latest version at https://github.com/ergoplatform/ergo/releases

# Set the EXPLORER_VERSION variable in `./build.sh` and `docker-compose.yml`.
# You can use any tag from the explorer repository: https://github.com/ergoplatform/explorer-backend

# If using another node than the one defined in this stack,
# edit the master-nodes field in explorer\explorer-backend.conf to point it to your node

# In the docker-compose.yml, for the node component,
# within the `command:` change `--mainnet` to `--testnet` as required

# In the docker-compose.yml, for the graphql component,
# under environment, set the `NETWORK` field to `MAINNET` or `TESTNET` as required

# Set the apiKeyHash in the ergo.conf file (if wallet functionality is required).
ergo {
    node {
        mining = false
    }
}

scorex {
  restApi {
    # node which exposes restApi in firewall should define publicly accessible URL of it
    #publicUrl = "https://example.com:80"
    # "ergo" in http://localhost:9053/swagger#/utils/hashBlake2b returns the below Blake2b hash
    apiKeyHash = "383a6c2f1d1a01241f4b2b465a2beca2528781fe9c3c7a8c9c43d811074b12a0"
  }
}

# Choose a password for the database
echo POSTGRES_PASSWORD=ergo2022 > db/db.secret

# Same password but different env variable for the GraphQL service
echo DB_USER_PWD=ergo2022 >> db/db.secret

# Check that the passwords have saved in the db.secret file
cat ./db/db.secret

# Run the build script
./build.sh

# If receiving an error `./build.sh: Permission denied`
chmod +x build.sh

# Start all services in one go...
docker compose up -d --build

# Stopping the service
# Using stop before/instead of down seems to cause less db corruption issues
docker compose stop node
docker compose down

# ...or only the ones you need
docker compose build db graphql
docker compose up --no-start
docker compose start db
docker compose start graphql

# Check their status
docker ps --filter name=graphql -a

# Stop all explorer services
docker compose down
# ...or
docker compose stop graphql
docker compose stop db
```

### GraphQL

The graphql server will run over http on port 3001. To use it with clients requiring https, put it behind a reverse proxy.

### Database volume

Unlike the node, the database volume is mapped, not named. Mapped volumes make it easier to handle  Postresql upgrades.

This means you may have to edit the path in `docker-compose.yml` and ensure the specified path exists on your system:

```
services:
  db:
...
    volumes:
      # Mapped volume for easier Postgres upgrades.
      # Make sure the path exists or edit it here.
      # See also readme_pg_upgrade.md
      - /var/lib/explorer_pg/14/data:/var/lib/postgresql/data
...
```

### Initial sync

~~Syncing from scratch will take a long time (weeks). Two solutions:~~

~~1. Drop indexes and constraints from the database and restore them once initial sync is done.~~
~~2. Get a snapshot from someone~~

Sync performance seems to have improved significantly and can complete in 6hrs or less. Still, if in a hurry, above points remain valid.
