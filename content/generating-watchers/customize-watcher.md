---
title: "Customize Watcher"
date: 2022-12-30T09:19:28-05:00
draft: true
weight: 5
---

### Customize

* Indexing on an event:

  * Edit the custom hook function `handleEvent` (triggered on an event) in `src/hooks.ts` to perform corresponding indexing using the `Indexer` object.

  * While using the indexer storage methods for indexing, pass `diff` as true if default state is desired to be generated using the state variables being indexed.

* Generating state:

  * Edit the custom hook function `createInitialState` (triggered if the watcher passes the start block, checkpoint: `true`) in `src/hooks.ts` to save an initial `State` using the `Indexer` object.

  * Edit the custom hook function `createStateDiff` (triggered on a block) in `src/hooks.ts` to save the state in a `diff` `State` using the `Indexer` object. The default state (if exists) is updated.

  * Edit the custom hook function `createStateCheckpoint` (triggered just before default and CLI checkpoint) in `src/hooks.ts` to save the state in a `checkpoint` `State` using the `Indexer` object.

### Development

* `lint`

  Command to check lint issues in files

  ```bash
  yarn lint
  ```

  To fix lint issue

  ```bash
  yarn lint --fix
  ```

* `version:set`

  Command to set cerc-io package versions in package.json template

  ```bash
  yarn version:set <VERSION>
  ```

  Example

  ```bash
  yarn version:set 0.2.17
  ```



### Known Issues

* Currently, `node-fetch v2.6.2` is being used to fetch from URLs as `v3.0.0` is an [ESM-only module](https://www.npmjs.com/package/node-fetch#loading-and-configuring-the-module) and `ts-node` transpiles to import  it using `require`.
