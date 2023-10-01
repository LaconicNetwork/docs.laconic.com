---
title: "Run IPFS (kubo)"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

The Kubo stack currently uses the native IPFS docker image, therefore a single command will do:

```bash
laconic-so --stack kubo deploy up
```

If running locally, visit: http://localhost:5001/webui and explore the functionality of the WebUI.

If running in the cloud, visit `IP:5001/webui` and you'll likely see this error: "Could not connect to the IPFS API". To fix it:

1. Get the container name with `docker ps`:

2. Go into the container:

```bash
laconic-so --stack kubo deploy exec ipfs sh
```

3. Enable CORS as described in point 2 of the error message. Copy/paste/run each line in sequence, then run `exit` to exit the container.

4. Restart the container:

```bash
laconic-so --stack kubo deploy down
laconic-so --stack kubo deploy up
```

5. Refresh the `IP:5001/webui` URL in your browser, you should now be connected to IPFS.


### The stack definition

The `kubo` stack is the most minimal possible. It has a `stack.yml` that looks like:

```yaml
version: "1.0"
name: kubo 
description: "Run kubo (IPFS)"
repos:
containers:
pods:
  - kubo 
```
and a `docker-compose-kubo.yml` that looks like:

```yaml
version: "3.2"
services:
  ipfs:
    image: ipfs/kubo:master-2023-02-20-714a968
    restart: always
    volumes:
      - ./ipfs/import:/import
      - ./ipfs/data:/data/ipfs
    ports:
      - "0.0.0.0:8080:8080"
      - "0.0.0.0:4001:4001"
      - "0.0.0.0:5001:5001"
```

Because it uses an existing docker image, it runs with a single command:

```bash
laconic-so --stack kubo deploy up
```

One could, with a relatively light lift, add the files required to build and deploy Kubo from source, using the Stack Orchestrator framework. Having the ability to do with relative ease is one goals of Stack Orchestrator.
