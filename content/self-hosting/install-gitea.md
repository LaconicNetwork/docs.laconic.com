---
title: "Install Gitea"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

In the Gitea x Laconicd stack demo, we used the local Gitea instance to publish NPM packages, which were consumed by the build for a basic `laconicd` fixturenet. That quick demo intentionally omitted many details about the Gitea self-hosting solution that we will cover below.

### Build and deploy

Below are the commands to build and deploy a pre-configured Gitea instance. It includes `act` and the runner needed for Gitea Actions to work.


```bash
laconic-so --stack build-support build-containers
laconic-so --stack package-registry setup-repositories
laconic-so --stack package-registry build-containers 
laconic-so --stack package-registry deploy up
```

Let's break them down one by one:

```bash
laconic-so --stack build-support build-containers
```

The `build-support` stack is core functionality and does not have a repo; its build files are contained within Stack Orchestrator [here](https://github.com/cerc-io/stack-orchestrator/tree/main/app/data/container-build/cerc-builder-js). It contains various utilities to handle the complexities of packaging and publishing npm packages. These complexities are normally outsourced to centralized service provideers.


The output will include a token. This token was generated with these [scopes](https://github.com/cerc-io/hosting/blob/main/gitea/initialize-gitea.sh#L47), therefore upon logging in, you can create new tokens with specific scopes, as you would on GitHub.

Next, we clone the repositories:

```bash
laconic-so --stack package-registry setup-repositories
```

Recall that `laconic-so` will clone repositories to `CERC_REPO_BASE_DIR`, which defaults to `~/cerc`.

At this point, you can navigate to the cloned repos (e.g., `~/cerc/hosting`) and modify the contents to suite the details of your organization - for example, at the top of [this file](https://github.com/cerc-io/hosting/blob/main/gitea/initialize-gitea.sh)

Then, build the containers. Because of how docker does caching, you'll want to use the `--force-rebuild` flag each time you modify source code in `CERC_REPO_BASE_DIR`.

```bash
laconic-so --stack package-registry build-containers 
```

Finally, deploy the whole stack

```bash
laconic-so --stack package-registry deploy up
```

Now, gitea is running. See the containers with:

```bash
docker ps
```

### Configure the hostname

The way to do this depends on your operating system  but usually involves editing a `hosts` file. For example, on Linux add this line to the file `/etc/hosts` (needs sudo):

```bash
127.0.0.1       gitea.local
```

Test with:

```bash
ping gitea.local
```
```bash
PING gitea.local (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.147 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.033 ms
```

You can now access the Gitea web interface at: [http://gitea.local:3000](http://gitea.local:3000) using these credentials: `gitea_admin/admin1234`. Please properly secure Gitea if public internet access is allowed.

### Make it your own



### Available resources
