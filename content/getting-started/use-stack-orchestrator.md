---
title: "Use Stack Orchestrator"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 2
---

Typically, a stack will be run like so:

```bash
laconic-so --stack myStack setup-repositories
laconic-so --stack myStack build-containers
laconic-so --stack myStack deploy-system up
```

whereby `myStack` is defined in a `.yaml` file and specifies the variables for each command

### setup-repositories

Clone a single repository:

```bash
laconic-so setup-repositories --include github.com/cerc-io/go-ethereum
```

Clone the repositories for a stack:

```bash
laconic-so --stack fixturenet-eth setup-repositories
```

Pull latest commits from origin:

```bash
laconic-so --stack fixturenet-eth setup-repositories --pull
```

### build-containers

Build a single container:

```bash
laconic-so build-containers --include cerc/go-ethereum
```

Build the containers for a stack:

```bash
laconic-so --stack fixturenet-eth build-containers
```

Force full rebuild of container images:

```bash
laconic-so build-containers --include fixturenet-eth --force-rebuild
```

### deploy-system

Note: aliased to `deploy`

Deploy a stack:

```bash
laconic-so --stack fixturenet-eth deploy-system up
```

View a stack:

```bash
laconic-so --stack fixturenet-eth deploy-system ps
```

Shut down a stack:

```bash
laconic-so --stack fixturenet-eth deploy-system down
```

### build-npms

Note: requires `CERC_NPM_REGISTRY_URL` and `CERC_NPM_AUTH_TOKEN` set. If the former is unset, it will default to a local Gitea on port 3000.

Build and publish a single package:

```bash
laconic-so build-npms --include laconic-sdk
```

Build and publish packages for a stack:

```bash
laconic-so --stack fixturenet-laconicd build-npms
```

Force full rebuild of packages:

```bash
laconic-so build-npms --include fixturenet-laconicd --force-rebuild
```
