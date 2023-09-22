---
title: "Gitea with Laconicd"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 2
---

This is a quick demo of our Gitea self-hosting solution. The locally deployed Gitea instance is the npm registry from which the `laconicd` stack is built.

Check out the "Self Hosting" section for a deep dive. The next demo "Laconic Registry" uses our (external) Gitea npm registry to build the stack, and showcases the registry and console front-end.

### Setup and Deploy Gitea

```
laconic-so --stack build-support build-containers
laconic-so --stack package-registry setup-repositories
laconic-so --stack package-registry build-containers 
laconic-so --stack package-registry deploy up
```

```
[+] Running 3/3
 ⠿ Network laconic-aecc4a21d3a502b14522db97d427e850_gitea       Created                                                                                    0.0s
 ⠿ Container laconic-aecc4a21d3a502b14522db97d427e850-db-1      Started                                                                                    1.2s
 ⠿ Container laconic-aecc4a21d3a502b14522db97d427e850-server-1  Started                                                                                    1.9s
New user 'gitea_admin' has been successfully created!
This is your gitea access token: <your-token>. Keep it safe and secure, it can not be fetched again from gitea.
To use with laconic-so set this environment variable: export CERC_NPM_AUTH_TOKEN=<your-token>
Created the organization cerc-io
Gitea was configured to use host name: gitea.local, ensure that this resolves to localhost, e.g. with sudo vi /etc/hosts
Success, gitea is properly initialized
```

**Note:** the above commands can take several minutes depending on the specs of your machine.

### Configure the hostname gitea.local

How to do this depends on your operating system  but usually involves editing a `hosts` file. For example, on Linux add this line to the file `/etc/hosts` (needs sudo):

```
127.0.0.1       gitea.local
```

Test with:

```
ping gitea.local
```
```
PING gitea.local (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.147 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.033 ms
```

Although not necessary in order to build and publish packages, you can now access the Gitea web interface at: [http://gitea.local:3000](http://gitea.local:3000) using these credentials: `gitea_admin/admin1234` (Note: please properly secure Gitea if public internet access is allowed).

Now npm packages can be built:

### Build npm Packages

Next, clone the required repositories:

```
laconic-so --stack fixturenet-laconicd setup-repositories
```

Ensure that `CERC_NPM_AUTH_TOKEN` is set with the token printed above when the package-registry stack was deployed (the actual token value will be different than shown in this example):

```
export CERC_NPM_AUTH_TOKEN=<your-token>
```

```
laconic-so --stack fixturenet-laconicd build-npms
```

Navigate to the Gitea console and switch to the `cerc-io` user then find the `Packages` tab to confirm that these two npm packages have been published.

![npm-package](../images/test.png)

### Build fixturenet containers

```
laconic-so --stack fixturenet-laconicd build-containers
```

Check the logs:

```
laconic-so --stack fixturenet-laconicd deploy logs
```

### Test with the registry CLI

```
laconic-so --stack fixturenet-laconicd deploy exec cli "laconic cns status"
```

Try additional CLI commands, documented [here](https://github.com/cerc-io/laconic-registry-cli#operations). Note that in order to publish records, you'll need to `docker cp` the `watcher.yml` file.
