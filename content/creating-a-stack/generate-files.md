---
title: "Generate Files"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

```bash
laconic-so --stack new-stack init
```

creates a new directory called `new-stack` with the following file structure:

```
├── compose/
    ├── docker-compose-new-stack.yml
├── config/
    ├── new-stack/
        ├── environment.toml
├── container-build/
    ├── cerc-new-stack/
        ├── build.sh
        ├── Dockerfile
```

... see https://github.com/cerc-io/stack-orchestrator/issues/513
