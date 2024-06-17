---
title: "Become a Service Provider"
date: 2022-12-30T09:19:28-05:00
draft: false
weight: 1
---

## Select and boot servers

- daemon (4G RAM, 25G Disk)
- control (16G RAM, 300G Disk)
- worker (16G RAM, 300G Disk)

## Access control

This is personal preference. At a minimum, create a new user on each machine and disable root access.

## Initial Ubuntu base setup

**On all machines:**

Set unique hostnames 

```
hostnamectl set-hostname changeme
```

In this example, we've named each machine like so:
```
lx-daemon               23.111.69.218
lx-cad-cluster-worker   23.111.78.182
lx-cad-cluster-control  23.111.78.179
```

And they are mapped to each IP via A records.

Next, update base packages:

```
apt update && apt upgrade -y
apt autoremove
```

Install additional packages:

```
apt install -y doas zsh tmux git jq acl curl wget netcat fping rsync htop iotop iftop tar less firewalld sshguard wireguard iproute2 iperf3 zfsutils-linux net-tools ca-certificates gnupg
```

Verify status of firewalld and enable sshguard:

```
systemctl enable --now firewalld
systemctl enable --now sshguard
```

Disable and Remove snapd

```
systemctl disable snapd.service
systemctl disable snapd.socket
systemctl disable snapd.seeded
systemctl disable snapd.snap-repair.timer

apt purge -y snapd

rm -rf ~/snap /snap /var/snap /var/lib/snapd
```

## Daemon-only packages (do not install on the worker and control nodes)

Install nginx and certbot:

```
apt install -y nginx certbot python3-certbot-nginx
```

Install Docker:

```
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update -y && apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## Get a domain

In this example, we are using audubon.app and its [nameservers point to Digital Ocean](https://docs.digitalocean.com/products/networking/dns/getting-started/dns-registrars/). You'll need to do the same.


## Ansible

1. On another machine (e.g., your laptop), install ansible and its utilities:
- https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

2. Create a gpg key, configure the vault, and add your key to it.

3. Clone the template repo and modify the domain, IP address, and names as shows in [this commit](https://git.vdb.to/cerc-io/service-provider-template/commit/32e1ad0bd73f0754c0978c96eaee526fa841ddb4)

## Configure DNS

Like this:

![sp-dns](/images/dns-sp-docs.png)


## Kubernetes
- `kubie ctx default`
- `kubectl.yaml`, used by SO --> how did it get to `/home/so/.kube/config-mito-lx-cad.yaml` 
- basic commands
- annotations


## Certificates

## Stack Orchestrator
- installed on the daemon machine

### Deploy container registry

This will be the first test that everything is configured correctly.

```
laconic-so --stack container-registry deploy init --output container-registry.spec
laconic-so --stack container-registry deploy create --deployment-dir container-registry --spec-file container-registry.spec
```

The above commands created a new directory; `container-registry`. It looks like:

```
$ ls
compose/  config.env  data/  deployment.yml  pods/  spec.yml  stack.yml
```
and we need to make a few modifications:

the file `container-registry/compose/docker-compose-container-registry.yml` should look like:

```
services:
  registry:
    image: docker.io/library/registry:2.8
    restart: always
    environment:
      REGISTRY_LOG_LEVEL: ${REGISTRY_LOG_LEVEL}
    volumes:
     - config:/config:ro
     - registry-data:/var/lib/registry
    ports:
     - '5000'
volumes:
  config:
  registry-data:
```

the `container-registry/spec.yaml` should look like:

```
stack: container-registry
deploy-to: k8s
kube-config: /home/so/.kube/config-mito-lx-cad.yaml
network:
  ports:
    registry:
     - '5000'
  http-proxy:
    - host-name: container-registry.pwa.audubon.app
      routes:
        - path: '/'
          proxy-to: registry:5000
volumes:
  registry-data:
configmaps:
  config: ./configmaps/config
```

copy in the kubectl file:
```
cp /home/so/.kube/config-mito-lx-cad.yaml
container-registry/kubeconfig.yml
```

### Htpasswd

delete the `container-registry/data` directory
and create a new `container-registry/configmaps/config/` directory which will contain the htpasswd file

create the `htpasswd` file:

```
htpasswd -b -c container-registry/configmaps/config/htpasswd so-reg-user pXDwO5zLU7M88x3aA
```

the resulting file should look like:

```
so-reg-user:$2y$05$Eds.WkuUgn6XFUL8/NKSt.JTX.gCuXRGQFyJaRit9HhrUTsVrhH.W
```

Next, configure the file `container-registry/config.env` like this:

```
REGISTRY_AUTH=htpasswd
REGISTRY_AUTH_HTPASSWD_REALM="Audubon Registry"
REGISTRY_AUTH_HTPASSWD_PATH="/config/htpasswd"
REGISTRY_HTTP_SECRET='$2y$05$Eds.WkuUgn6XFUL8/NKSt.JTX.gCuXRGQFyJaRit9HhrUTsVrhH.W'
```

using these credentials, create a `container-registry/my_password.json` that looks like:

```
{
  "auths": {
    "container-registry.pwa.audubon.app": {
      "username": "so-reg-user",
      "password": "$2y$05$Eds.WkuUgn6XFUL8/NKSt.JTX.gCuXRGQFyJaRit9HhrUTsVrhH.W",
      "auth": "c28tcmVnLXVzZXI6cFhEd081ekxVN004OHgzYUE="
    }
  }
}
```
where the `auth:` field is the output of:
```
echo -n "so-reg-user:pXDwO5zLU7M88x3aA" | base64 -w0
```

Finally, add the container registry credentials as a secret available to the cluster:

```
kubectl create secret generic laconic-registry --from-file=.dockerconfigjson=container-registry/my_password.json --type=kubernetes.io/dockerconfigjson
```

And deploy it:

```
laconic-so deployment --dir container-registry start
```

Check the logs:
```
TODO
```

With a successful container registry deployed, it is now possible to deploy webapps to the cluster

#### Deploy a test app

```
git clone git@git.vdb.to:cerc-io/test-progressive-web-app.git ~/cerc/test-progressive-web-app
laconic-so build-webapp --source-repo ~/cerc/test-progressive-web-app
```
explain
```
laconic-so deploy-webapp create --kube-config /home/so/.kube/config-mito-lx-cad.yaml --image-registry container-registry.pwa.audubon.app --deployment-dir webapp-k8s-deployment --image cerc/test-progressive-web-app:local --url https://my-test-app.pwa.audubon.app --env-file ~/cerc/test-progressive-web-app/.env
```
explain
```
laconic-so deployment --dir webapp-k8s-deployment push-images
laconic-so deployment --dir webapp-k8s-deployment start
```
If everything worked, after a couple minutes, you should see a pod for this webapp and the webapp running at https://my-test-app.pwa.audubon.app

### Deploy the laconicd registry and console

Follow the instructions in [this document](https://git.vdb.to/cerc-io/stack-orchestrator/src/branch/main/docs/laconicd-with-console.md)

After publishing sample records, you'll have a `bondId`. Also retreive your `userKey` (private key) which will be required later.

#### Nginx

TODO: sync with Shane
test console.myurl.com

### Deploy deployer back end


```
laconic-so --stack webapp-deployer deploy init --output webapp-deployer.spec
laconic-so --stack webapp-deployer deploy create --deployment-dir webapp-deployer --spec-file webapp-deployer.spec
```
Modify the contents of `webapp-deployer`

`config.env`:

```
DEPLOYMENT_DNS_SUFFIX="pwa.audubon.app"
DEPLOYMENT_RECORD_NAMESPACE="mito"
IMAGE_REGISTRY="container-registry.pwa.audubon.app"
IMAGE_REGISTRY_USER="so-reg-user"
IMAGE_REGISTRY_CREDS="pXDwO5zLU7M88x3aA"
CLEAN_DEPLOYMENTS=false
CLEAN_LOGS=false
CLEAN_CONTAINERS=false
SYSTEM_PRUNE=false
WEBAPP_IMAGE_PRUNE=true
CHECK_INTERVAL=5
FQDN_POLICY="allow"
```

In `webapp-deployer/data/config/` there needs to be two files:
  1. `kube.yml` --> copied from `/home/so/.kube/config-mito-lx-cad.yaml`
  2. `laconic.yml` --> with the details for talking to laconicd

The latter looks like:

```
services:
  cns:
    restEndpoint: 'https://lx-daemon.audubon.app:1317'
    gqlEndpoint: 'https://lx-daemon.audubon.app/api'
    userKey: e64ae9d07b21c62081b3d6d48e78bf44275ffe0575f788ea7b36f71ea559724b
    bondId: ad9c977f4a641c2cf26ce37dcc9d9eb95325e9f317aee6c9f33388cdd8f2abb8
    chainId: laconic_9000-1
    gas: 9950000
    fees: 500000aphoton
  registry:
    restEndpoint: 'https://lx-daemon.audubon.app:1317'
    gqlEndpoint: 'https://lx-daemon.audubon.app/api'
    userKey: e64ae9d07b21c62081b3d6d48e78bf44275ffe0575f788ea7b36f71ea559724b
    bondId: ad9c977f4a641c2cf26ce37dcc9d9eb95325e9f317aee6c9f33388cdd8f2abb8
    chainId: laconic_9000-1
    gas: 9950000
    fees: 500000aphoton
```
Deduplication of the `cns` and `registry` fields will happen with `laconic2d` but are required for now)

Start up the deployer
```
laconic-so --stack webapp-deployer deployment start
```

Now, publishing records to the laconicd registry should trigger deployments; for example, using these two scripts:
https://git.vdb.to/zramsay/birbit/src/branch/main/.github/workflows/publish.yaml#L41-L45

### Deploy deployer UI

To view the status and logs of deployments, we can deploy the UI

```
git clone git@git.vdb.to:cerc-io/webapp-deployment-status-ui.git ~/cerc/webcerc/webapp-deployment-status-ui
laconic-so build-webapp --source-repo ~/cerc/webapp-deployment-status-ui
```
explain
```
laconic-so deploy-webapp create --kube-config /home/so/.kube/config-mito-lx-cad.yaml --image-registry container-registry.pwa.audubon.app --deployment-dir webapp-ui --image cerc/webapp-deployment-status-ui:local --url https://webapp-deployer-ui.pwa.audubon.app --env-file ~/cerc/webcerc/webapp-deployment-status-ui/.env
```
explain
```
laconic-so deployment --dir webapp-ui push-images
laconic-so deployment --dir webapp-ui start
```

Now view https://webapp-deployer-ui.pwa.audubon.app for the status and logs of each deployment

## Result

We now have:

- https://lx-console.audubon.app which displays registry records (webapp deployments)
- https://container-registry.pwa.audubon.app which is used by the cluster for hosting docker images used by deployments
- https://webapp-deployer-api.pwa.audubon.app listens for new ApplicationDeploymentRequest and, under the hood, runs `laconic-so deploy-webapp-from-registry` as requested
- https://webapp-deployer-ui.pwa.audubon.app displays webapp deployment status and logs
- https://my-test-app.pwa.audubon.app as an example webapp deployment. 

## Notes and debugging

- pvc
- 413 request too large
- using `container-registry.pwa.audubon.app/laconic-registry` or `container-registry.pwa.audubon.app` seems to both work, TODO, investigate
