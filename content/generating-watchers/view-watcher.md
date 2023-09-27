---
title: "View Watcher"
date: 2022-12-30T09:19:28-05:00
draft: true
weight: 3
---

In the previous section, we generated a watcher for your smart contract and explored the contents of its generated files. The watcher is a GraphQL app and explained below is how to run it. Recall from earlier, we used `laconic-so` to deploy the back end for this watcher.


### Setup

Run the following command to install required packages:

```bash
yarn
```

In the config file (`environments/local.toml`):


* Create the databases configured in `environments/local.toml`. 

Run:

```bash
yarn server
```

Now, go to localhost:3000 and explore your watcher!
