---
title: "Testing your stack"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 4
---

The `act` integration in Gitea makes it fairly seamless to duplicate, enhance, migrate, or any combination thereof, your existing GitHub workflows. Note that some configuration of you existing workflows is required, e.g., adding [this line](TODO).

- a Gitea repo will first look for files in `.gitea/workflows` and run those Actions, as you are already used to using GitHub Actions
- if `.gitea/workflows` does not exist, Gitea will run the Actions in `.github/workflows`
- if both `.gitea/workflows` and `.github/workflows` exist, Gitea will run only the former
- with a mirror setup, having both `.gitea/workflows` and `.github/workflows` means you operate with redundancy and run different components of CI/CD in different locations.

For example, stack orchestrator still uses GitHub for the active codebase and publishing releases, but some computationally intensive stacks are tested in Gitea, everytime the mirror is updated (not currently true though?)
