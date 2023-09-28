---
title: "Creating a Stack"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 3
---

The Azimuth stack demonstrates a full example using watcher - the only thing missing is the RPC and GQL endpoints for `ipld-eth-server`.

### stack.yaml

Starting with the `stack.yml`, we see:

```yaml
version: "1.0"
name: azimuth
repos:
  - github.com/cerc-io/azimuth-watcher-ts@v0.1.1
containers:
  - cerc/watcher-azimuth
pods:
  - watcher-azimuth
```

### Dockerfile

Note: the line `yarn && yarn build` would, in most webapp situations, be replaced by `yarn global add @some-registry/my-watcher`. This process is documented in the "Self Hosting" and "WebApp" sections of the documentation.

The single container is built from this Dockerfile:

```Dockerfile
FROM node:18.16.0-alpine3.16

RUN apk --update --no-cache add git python3 alpine-sdk

WORKDIR /app

COPY . .

RUN echo "Building azimuth-watcher-ts" && \
    yarn && yarn build

RUN echo "Install toml-js to update watcher config files" && \
    yarn add --dev --ignore-workspace-root-check toml-js
```
via this `build.sh` script:

```bash
#!/usr/bin/env bash
# Build cerc/watcher-azimuth

source ${CERC_CONTAINER_BASE_DIR}/build-base.sh

# See: https://stackoverflow.com/a/246128/1701505
SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )

docker build -t cerc/watcher-azimuth:local -f ${SCRIPT_DIR}/Dockerfile ${build_command_args} ${CERC_REPO_BASE_DIR}/azimuth-watcher-ts
```

### docker-compose.yml

Then, the `docker-compose-watcher-azimuth.yml` (some parts omitted for brevity):

```yaml
version: '3.2'

services:
  # Starts the PostgreSQL database for watchers
  watcher-db:
    restart: unless-stopped
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=vdbm
      - POSTGRES_MULTIPLE_DATABASES=azimuth-watcher,azimuth-watcher-job-queue,censures-watcher,censures-watcher-job-queue,claims-watcher,claims-watcher-job-queue,conditional-star-release-watcher,conditional-star-release-watcher-job-queue,delegated-sending-watcher,delegated-sending-watcher-job-queue,ecliptic-watcher,ecliptic-watcher-job-queue,linear-star-release-watcher,linear-star-release-watcher-job-queue,polls-watcher,polls-watcher-job-queue
      - POSTGRES_EXTENSION=azimuth-watcher-job-queue:pgcrypto,censures-watcher-job-queue:pgcrypto,claims-watcher-job-queue:pgcrypto,conditional-star-release-watcher-job-queue:pgcrypto,delegated-sending-watcher-job-queue:pgcrypto,ecliptic-watcher-job-queue:pgcrypto,linear-star-release-watcher-job-queue:pgcrypto,polls-watcher-job-queue:pgcrypto,
      - POSTGRES_PASSWORD=password
    volumes:
      - ../config/postgresql/multiple-postgressql-databases.sh:/docker-entrypoint-initdb.d/multiple-postgressql-databases.sh
      - watcher_db_data:/var/lib/postgresql/data
    ports:
      - "0.0.0.0:15432:5432"
    healthcheck:
      test: ["CMD", "nc", "-v", "localhost", "5432"]
      interval: 20s
      timeout: 5s
      retries: 15
      start_period: 10s

  # Starts the azimuth-watcher server
  azimuth-watcher-server:
    image: cerc/watcher-azimuth:local
    restart: unless-stopped
    depends_on:
      watcher-db:
        condition: service_healthy
    env_file:
      - ../config/watcher-azimuth/watcher-params.env
    environment:
      CERC_SCRIPT_DEBUG: ${CERC_SCRIPT_DEBUG}
      CERC_IPLD_ETH_RPC: ${CERC_IPLD_ETH_RPC}
      CERC_IPLD_ETH_GQL: ${CERC_IPLD_ETH_GQL}
    working_dir: /app/packages/azimuth-watcher
    command: "./start-server.sh"
    volumes:
      - ../config/watcher-azimuth/watcher-config-template.toml:/app/packages/azimuth-watcher/environments/watcher-config-template.toml
      - ../config/watcher-azimuth/merge-toml.js:/app/packages/azimuth-watcher/merge-toml.js
      - ../config/watcher-azimuth/start-server.sh:/app/packages/azimuth-watcher/start-server.sh
    ports:
      - "3001"
    healthcheck:
      test: ["CMD", "nc", "-vz", "localhost", "3001"]
      interval: 20s
      timeout: 5s
      retries: 15
      start_period: 5s
    extra_hosts:
      - "host.docker.internal:host-gateway"

  # Starts the censures-watcher server
  censures-watcher-server:
    image: cerc/watcher-azimuth:local
    restart: unless-stopped
    depends_on:
      watcher-db:
        condition: service_healthy
    env_file:
      - ../config/watcher-azimuth/watcher-params.env
    environment:
      CERC_SCRIPT_DEBUG: ${CERC_SCRIPT_DEBUG}
      CERC_IPLD_ETH_RPC: ${CERC_IPLD_ETH_RPC}
      CERC_IPLD_ETH_GQL: ${CERC_IPLD_ETH_GQL}
    working_dir: /app/packages/censures-watcher
    command: "./start-server.sh"
    volumes:
      - ../config/watcher-azimuth/watcher-config-template.toml:/app/packages/censures-watcher/environments/watcher-config-template.toml
      - ../config/watcher-azimuth/merge-toml.js:/app/packages/censures-watcher/merge-toml.js
      - ../config/watcher-azimuth/start-server.sh:/app/packages/censures-watcher/start-server.sh
    ports:
      - "3002"
    healthcheck:
      test: ["CMD", "nc", "-vz", "localhost", "3002"]
      interval: 20s
      timeout: 5s
      retries: 15
      start_period: 5s
    extra_hosts:
      - "host.docker.internal:host-gateway"

  # Starts the claims-watcher server
  claims-watcher-server:
    image: cerc/watcher-azimuth:local
  
  # Starts the conditional-star-release-watcher server
  conditional-star-release-watcher-server:
    image: cerc/watcher-azimuth:local
  
  # Starts the delegated-sending-watcher server
  delegated-sending-watcher-server:
    image: cerc/watcher-azimuth:local
  
  # Starts the ecliptic-watcher server
  ecliptic-watcher-server:
    image: cerc/watcher-azimuth:local

  # Starts the linear-star-release-watcher server
  linear-star-release-watcher-server:
    image: cerc/watcher-azimuth:local

  # Starts the polls-watcher server
  polls-watcher-server:
    image: cerc/watcher-azimuth:local

  # Starts the gateway-server for proxying queries
  gateway-server:
    image: cerc/watcher-azimuth:local
    restart: unless-stopped
    depends_on:
      azimuth-watcher-server:
        condition: service_healthy
      censures-watcher-server:
        condition: service_healthy
      claims-watcher-server:
        condition: service_healthy
      conditional-star-release-watcher-server:
        condition: service_healthy
      delegated-sending-watcher-server:
        condition: service_healthy
      ecliptic-watcher-server:
        condition: service_healthy
      linear-star-release-watcher-server:
        condition: service_healthy
      polls-watcher-server:
        condition: service_healthy
    environment:
      CERC_SCRIPT_DEBUG: ${CERC_SCRIPT_DEBUG}
    working_dir: /app/packages/gateway-server
    command: "yarn server"
    volumes:
      - ../config/watcher-azimuth/gateway-watchers.json:/app/packages/gateway-server/dist/watchers.json
    ports:
      - "0.0.0.0:4000:4000"
    healthcheck:
      test: ["CMD", "nc", "-vz", "localhost", "4000"]
      interval: 20s
      timeout: 5s
      retries: 15
      start_period: 5s
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  watcher_db_data:
```

### Configuration

A handful of config files are found [here](https://github.com/cerc-io/stack-orchestrator/tree/main/app/data/config/watcher-azimuth):

```bash
ls app/data/config/watcher-azimuth
```

```bash
gateway-watchers.json  merge-toml.js  start-server.sh  watcher-config-template.toml  watcher-params.env
```
