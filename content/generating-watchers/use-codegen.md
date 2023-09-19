---
title: "Use Codegen"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 2
---

In `packages/codegen`, Create a `config.yaml` file in the following format:

```yaml
contracts:
    # Contract name:
  - name: SimpleContract
    # Contract file path or an url:
    path: ../../docs/SimpleContract.sol
    # Contract kind (should match name of dataSource in {subgraphPath}/subgraph.yaml if subgraphPath provided)
    #kind: Example1

# Output folder path (logs output using `stdout` if not provided).
outputFolder: ../test-watcher

# Code generation mode [eth_call | storage | all | none] (default: none).
mode: none

# Kind of watcher [lazy | active] (default: active).
kind: lazy

# Watcher server port (default: 3008).
port: 3008

# Solc version to use (optional)
# If not defined, uses solc version listed in dependencies
solc: v0.8.0+commit.c7dfd78e

# Flatten the input contract file(s) [true | false] (default: true).
flatten: true

# Path to the subgraph build (optional).
# Can set empty contracts array when using subgraphPath.
# subgraphPath: ../graph-node/test/subgraph/example1/build

# NOTE: When passed an *URL* as contract path, it is assumed that it points to an already flattened contract file.
```

Create the following example contract:



Generate a watcher from the example contract:

```
yarn codegen --config-file ./example.config.yaml
```

The flag `--continue-on-error` can be used to continue generation if any unhandled data types are encountered. TODO explain this in the context of current and future codegen.

This will create a folder containing the generated code at the path provided in config file. Let's explore it's contents in `watcher-ts/packages/simple-contract-watcher`.

## TODO

```
ls
```
then explain
