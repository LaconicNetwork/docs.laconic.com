---
title: "Install Gitea"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

In the Gitea x Laconicd stack demo, we used the local Gitea instance to publish NPM packages, which were consumed by the build for a basic `laconicd` fixturenet. That quick demo intentionally omitted many details about the Gitea self-hosting solution that we will cover below.

### Build and Deploy Gitea

Recall the commands to build and deploy Gitea

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


### Configure the hostname gitea.local

How to do this depends on your operating system  but usually involves editing a `hosts` file. For example, on Linux add this line to the file `/etc/hosts` (needs sudo):

```
127.0.0.1       gitea.local
```

Test with:

```
ping gitea.local
```
```
PING gitea.local (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.147 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.033 ms
```

Although not necessary in order to build and publish packages, you can now access the Gitea web interface at: [http://gitea.local:3000](http://gitea.local:3000) using these credentials: `gitea_admin/admin1234` (Note: please properly secure Gitea if public internet access is allowed).

// END HERE but dive into gitea more, and explain everything that is going on, the hosting repo, etc.

