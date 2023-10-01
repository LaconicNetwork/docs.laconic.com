---
title: "Packing your WebApp"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 4
---

We previously saw in the stack demo with the Laconic console, that it was built using npm packages from our self-hosted package registry. We then saw how to setup your own git server and package registry. Together, these give you and your users ownership of the code delivery pipeline.

Another goal of ours - intended to facilitate interoperable applications - it clear separation of concerns. For example, most applications today are built and deployed in the same place. This confuses the hosted front end with withn back end API.

In this example, we go through how this separation of concerns exist for building and deploying the Laconic console.

TODO
