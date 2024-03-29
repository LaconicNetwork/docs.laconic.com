---
title: "Install Codegen"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

Install yarn:

- Install Node using `nvm`: [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)
- Install `yarn` with: `npm install -g yarn`

Get the code:

```bash
git clone https://github.com/cerc-io/watcher-ts
cd watcher-ts
```

Install the required packages and build the tools:

```bash
yarn && yarn build
```

Enter the directory:

```bash
cd packages/codegen
```

Verify operation:

```bash
yarn codegen --version
```
```bash
yarn run v1.22.19
$ ts-node src/generate-code.ts --version
0.2.55
Done in 8.84s.
```

You are now ready to generate a watcher using the code generator.
