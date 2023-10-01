---
title: "Migrate from GitHub"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 2
---

Depending on your goals, Gitea can be used simply as a mirror, or you can go all-in and migrate completely over. As already mentioned, we are in the process of the doing the latter, though we started this process with mirroring some GitHub repos in Gitea. A full transition involves understanding the nuances in the process, most of which we've experienced, and is documented below.

### Setup your org

Since the defaults are setup for `cerc-io`, you'll want to install Gitea like previously described but modify [this line](https://github.com/cerc-io/hosting/blob/4bdf0f7d25756f753303ddb436da7dded25502e9/gitea/initialize-gitea.sh#L7) prior to `build-containers`. It should match exactly the GitHub organization you are mirroring or migrating from.

### Mirror repositories

At this point, the Gitea console gives you the ability to select any repo and mirror it. Select all your core repos and mirror them. Not only do you now have an up-to-date backup for when GitHub goes down, it is also possible to run workflows separately (or alongside) existing GitHub workflows. This not only provides redundancy, but independence as well (anyone can run your stack with relative ease).

### Migrate repos

Mirroring is a great first step, but migrating will actually reducing reliance on the centralized service that is GitHub. We have a [migrate repo script](https://github.com/cerc-io/hosting/blob/main/gitea/migrate-repo.sh) that you [can call in a loop](https://github.com/cerc-io/hosting/pull/42). Alternatively, the Gitea console provides functionality to migrate repositories manually. If selected, migration brings Issues and Pull Requests directly from GitHub.

You can migrate, and develop your code on Gitea, while still using GitHub as a mirror, and for workflows.

### Workflows

The `act` integration in Gitea makes it fairly seamless to duplicate, enhance, migrate, or any combination thereof, your existing GitHub workflows. Note that some configuration of you existing workflows is required, e.g., adding [this line](TODO).

- a Gitea repo will first look for files in `.gitea/workflows` and run those Actions, as you are already used to using GitHub Actions
- if `.gitea/workflows` does not exist, Gitea will run the Actions in `.github/workflows`
- if both `.gitea/workflows` and `.github/workflows` exist, Gitea will run only the former
- with a mirror setup, having both `.gitea/workflows` and `.github/workflows` means you operate with redundancy and run different components of CI/CD in different locations.

For example, stack orchestrator still uses GitHub for the active codebase and publishing releases, but some computationally intensive stacks are tested in Gitea, everytime the mirror is updated (not currently true though?)
