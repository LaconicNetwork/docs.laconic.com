---
title: "Use Codegen"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 2
---

### Setup

In the `packages/codegen` directory, there is an `example.config.yaml` file that looks like:

```yaml
contracts:
    # Contract name:
  - name: SimpleContract
    # Contract file path or an url:
    path: ./SimpleContract.sol
    # Contract kind (should match name of dataSource in {subgraphPath}/subgraph.yaml if subgraphPath provided)
    #kind: Example1

# Output folder path (logs output using `stdout` if not provided).
outputFolder: ../simple-contract-watcher

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

and a solidity contract that looks like:

```solidity
pragma solidity ^0.8.0;

contract SimpleContract {
    uint256 public data;

    function setData(uint256 _data) public {
        data = _data;
    }

    function getData() public view returns (uint256) {
        return data;
    }
}
```


Generate a watcher from the example contract:

```bash
yarn codegen --config-file ./example.config.yaml
```

This will create a directory containing the generated code at the path provided in config file. Let's explore its contents in `watcher-ts/packages/simple-contract-watcher`.

### GraphQL Queries

The code generator create files that, with a little bit of configuration, are able to run out of the box. This framework is an alternative to [The Graph](https://thegraph.com).

```bash
ls
```
```bash
LICENSE  README.md  environments  package.json  src  tsconfig.json
```

The README contains all the necessary instructions for configuring and serving your watcher. It will outline how to setup the postgres database, and available CLI commands. It is helpful to familiarize yourself with this part before moving on to Stack Orchestrator.

### Using Stack Orchestrator

Our stack orchestration tool, `laconic-so` is what we now use to deploy watchers. It makes it easier to package your app from back to front and deliver it to your users in a more decentralized way than is typically done. In the "Creating a stack" guide, Azimuth will provide a useful example of packaging your watcher into an SO stack.

### RPC and GQL endpoints

These come from running `ipld-eth-server` which requires loading the relevant data into `ipld-eth-db`. This data is generated via our statediffing technology (currently in transition from a fork of go-ethereum to a plugin in the plugeth framework.
 Please contact us if you've gotten this far.
