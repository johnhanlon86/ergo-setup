Docker based Ergo setup. This is a fork of [Ergo Setup](https://github.com/abchrisxyz/ergo-setup) but this repo runs less components and runs on a single docker compose network. This repo does not include a node as this is connected to remotely.

Install [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/), then follow the instructions below for the components you'd like to run.

> Containers are configured to expose certain ports for convenient access within a home lab context. Modify the `ports` values in the `docker-compose.yml` files if needed.

```
# Make sure that you are in the graphql directory
cd graphql

# Volume and network expected by graphql compose file.
docker volume create --name=ergo_redis
docker network create ergo-graphql

# Set the EXPLORER_VERSION variable in `./build.sh` and `docker-compose.yml`.
# Update this to the latest version at https://github.com/ergoplatform/explorer-backend/tags

# Set the VERSION variable for the graphql component in `docker-compose.yml`.
# Set the ARG VERSION variable for the graphql component in `/graphql/Dockerfile`.
# Update this to the latest version at https://github.com/capt-nemo429/ergo-graphql/tags

# To connect to your node:
# Edit the master-nodes field in explorer\explorer-backend.conf to point it to your node's IP
master-nodes = ["http://your-ip:9053"]
# In the docker-compose.yml, for the graphql component,
# update the ERGO_NODE_ADDRESS variable with your node's IP
ERGO_NODE_ADDRESS: http://your-ip:9053

# In the docker-compose.yml, for the graphql component,
# under environment, set the `NETWORK` field to `MAINNET` or `TESTNET` as required

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
docker compose stop graphql
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

The database volume is mapped, not named. Mapped volumes make it easier to handle  Postresql upgrades.

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
