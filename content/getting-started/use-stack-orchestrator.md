---
title: "Use Stack Orchestrator"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 2
---

Typically, a stack will be run like so:

```
laconic-so --stack myStack setup-repositories
laconic-so --stack myStack build-containers
laconic-so --stack myStack deploy-system up
```

whereby `myStack` is defined in a `.yaml` file and specifies the variables for each command

## setup-repositories

Clone a single repository:

```
laconic-so setup-repositories --include github.com/cerc-io/go-ethereum
```

Clone the repositories for a stack:

```
laconic-so --stack fixturenet-eth setup-repositories
```

Pull latest commits from origin:

```
laconic-so --stack fixturenet-eth setup-repositories --pull
```

Use SSH rather than https:

```
laconic-so --stack fixturenet-eth setup-repositories --git-ssh
```

## build-containers

Build a single container:

```
laconic-so build-containers --include cerc/go-ethereum
```

Build the containers for a stack:

```
laconic-so --stack fixturenet-eth build-containers
```

Force full rebuild of container images:

```
laconic-so build-containers --include fixturenet-eth --force-rebuild
```

## deploy-system

Note: aliased to `deploy`

Deploy a stack:

```
laconic-so --stack fixturenet-eth deploy up
```

View a stack:

```
laconic-so --stack fixturenet-eth deploy ps
```

Shut down a stack:

```
laconic-so --stack fixturenet-eth deploy down
```

## build-npms

Note: requires an env var set because these build & publish IIRC, and gitea up and running

Build a single package:

```
laconic-so build-npms --include laconic-sdk
```

Build the packages for a stack:

```
laconic-so --stack fixturenet-laconicd build-npms
```

Force full rebuild of packages:

```
laconic-so build-npms --include fixturenet-laconicd --force-rebuild
```
