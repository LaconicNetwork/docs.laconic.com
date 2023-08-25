---
title: "Install Stack Orchestrator"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

Stack Orchestrator

### Pre-requisites

- `python3` [Install](https://www.python.org/downloads/)
- `docker` [Install](https://docs.docker.com/get-docker/)
- `docker-compose` [Install](https://docs.docker.com/compose/install/)

If using a fresh Ubuntu Digital Ocean droplet, check out [this script](https://github.com/cerc-io/stack-orchestrator/blob/main/scripts/quick-install-linux.sh) for a quick setup.

**WARNING**: if installing docker-compose via package manager (as opposed to Docker Desktop), you must install the plugin, e.g., on Linux:

```
mkdir -p ~/.docker/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

Next, install the latest release of Stack Orchestrator

```
curl -L -o laconic-so https://github.com/cerc-io/stack-orchestrator/releases/latest/download/laconic-so
```

Give it permission:
```
chmod +x laconic-so
```

Verify operation:
```
./laconic-so version
```

```
Version: 1.1.0-f55a14b-202308221833
```

For a more permanent setup, move the binary to `~/bin` and add it your `PATH`.

## Stack Orchestrator

The `laconic-so` CLI tool makes it easy to experiment with various components of the stack. It allows you to quickly and seamlessly experiment with watchers. Because it uses docker/docker-compose, several commands in this tutorial will leverage the ability to execute commands directely in the containers. This, for example, means that `yarn` doesn't need to be installed on your local machine.
