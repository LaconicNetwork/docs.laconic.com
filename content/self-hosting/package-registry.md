---
title: "A Package Registry"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 3
---

The stack is named `package-registry` because part of the design goal was ownership of all aspects of the code delivery pipeline. A benefit of Gitea is that it includes the ability to publish docker and npm packages directly to it. This is usually a laborious, complex process to setup, configure, and maintain. We've sought to simplify this.

We saw in previous tutorials: 1) using an env var to pull a package from the Laconic NPM registry and 2) publishing an NPM package to a local Gitea server, and in both cases, using this to build a particular stack.

Here, we'll dive a bit more into the features of the registry and how we use it.



- a GitHub repo (active codebase) -> publishes packages to our Gitea

