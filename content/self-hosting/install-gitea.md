---
title: "Install Gitea"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1 
---

In the Gitea x Laconicd stack demo, we used the local Gitea instance to publish NPM packages, which were consumed by the build for a basic `laconicd` fixturenet. That demo intentionally omitted details about the Gitea self-hosting solution that we will cover below.

### Build and Install Gitea
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

Note: the above commands can take several minutes depending on the specs of your machine.

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

// END HERE but dive into gitea more, and explain everything that is going on, the hosting repo, etc.

