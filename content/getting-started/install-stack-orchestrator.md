---
title: "Install Stack Orchestrator"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

[Stack Orchestrator](https://github.com/cerc-io/stack-orchestrator) is a command-line tool for building and deploying all the components of the Laconic stack. It also includes tooling to improve the decentralization of any web3 application.

Installation detail will depend on the existing settings of your machine. If using a fresh Linux machine, check out [this script](https://github.com/cerc-io/stack-orchestrator/blob/main/scripts/quick-install-linux.sh) for a quick setup.

These pre-requisites are required:

- `jq` [Install](https://jqlang.github.io/jq/download/)
- `python3` [Install](https://www.python.org/downloads/)
- `docker` [Install](https://docs.docker.com/get-docker/)
- `docker-compose` [Install](https://docs.docker.com/compose/install/)


**Note:** if installing docker-compose via package manager (as opposed to Docker Desktop), you must install the plugin, e.g., on Linux:

```bash
mkdir -p ~/.docker/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

Next, install the latest release of Stack Orchestrator

```bash
curl -L -o laconic-so https://github.com/cerc-io/stack-orchestrator/releases/latest/download/laconic-so
```

Give it permission:
```bash
chmod +x laconic-so
```

Verify operation:
```bash
./laconic-so version
```

```bash
Version: 1.1.0-f55a14b-202308221833
```

For a more permanent setup, move the binary to `~/bin` and add it your `PATH`.
