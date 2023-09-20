---
title: "Self Hosting"
description: "Using Stack Orchestrator to self host your entire codebase"
weight: 5
---

As a whole, the Laconic stack provides tooling for improving the decentralization of any given application.

The `package-registry` stack simplifies the process of setting up your own self-hosted [Gitea](https://docs.gitea.com) instance. This makes it quick and painless to 1) mirror all your Github repos and 2) have a backup ready to go for when GitHub goes down.

We host mirrors of all our repos on our own Gitea instance here: [https://git.vdb.to](https://git.vdb.to). Another benefit that we use extensively is the [act integration](https://gitea.com/gitea/act) included in the `package-registry` stack. This gives you the ability to easily run CI/CD workflows on servers of your choosing, with very little modification to **existing GitHub workflows**.

For example, [this test file](https://github.com/cerc-io/stack-orchestrator/blob/main/.gitea/workflows/test.yml) will trigger an [action](https://git.vdb.to/cerc-io/stack-orchestrator/actions/runs/574) in our self-hosted stack orchestrator mirror. This is very useful if your testing workloads run into limits on GitHub.

This stack was introduced earlier in a demo that included laconicd. In this section, we dive deeper into all the features of our self-hosting solution and relevant configurations.
