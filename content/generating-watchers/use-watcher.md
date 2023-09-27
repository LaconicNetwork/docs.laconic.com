---
title: "Use Watcher"
date: 2022-12-30T09:19:28-05:00
draft: true
weight: 4
---

TODO: organize/explain

* If the watcher is an `active` watcher:

  * Run the job-runner:

    ```bash
    yarn job-runner
    ```

  * To watch a contract:

    ```bash
    yarn watch:contract --address <contract-address> --kind <contract-kind> --checkpoint <true | false> --starting-block [block-number]
    ```

  * To fill a block range:

    ```bash
    yarn fill --start-block <from-block> --end-block <to-block>
    ```

  * To create a checkpoint for a contract:

    ```bash
    yarn checkpoint --address <contract-address> --block-hash [block-hash]
    ```

  * To reset the watcher to a previous block number:

    * Reset state:

      ```bash
      yarn reset state --block-number <previous-block-number>
      ```

    * Reset job-queue:

      ```bash
      yarn reset job-queue
      ```

  * To export the watcher state:

    ```bash
    yarn export-state --export-file [export-file-path]
    ```

  * To import the watcher state:

    ```bash
    yarn import-state --import-file <import-file-path>
    ```

  * To inspect a CID:

    ```bash
    yarn inspect-cid --cid <cid>
    ```
