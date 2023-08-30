---
title: "Install Codegen"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

### Install yarn

For development:
- Install Node using `nvm`: [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)
- Install `yarn` with: `npm install -g yarn`

For testing:
- To only install `yarn`, on Linux, see [here](https://stackoverflow.com/a/53471064)

### Get the code

```bash
git clone https://github.com/cerc-io/watcher-ts
cd watcher-ts
```

### Install the required packages

```bash
yarn
```

### Build the tool

```bash
yarn build
```

### Verify operation

```bash
yarn codegen
```
