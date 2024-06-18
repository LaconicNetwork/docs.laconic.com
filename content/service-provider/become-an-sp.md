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

1. Set unique hostnames 

```
hostnamectl set-hostname changeme
```

In the following example, we've named each machine like so:
```
lx-daemon               23.111.69.218
lx-cad-cluster-worker   23.111.78.182
lx-cad-cluster-control  23.111.78.179
```

See below for the full list of DNS records to be configured.

2. Next, update base packages:

```
apt update && apt upgrade -y
apt autoremove
```

3. Install additional packages:

```
apt install -y doas zsh tmux git jq acl curl wget netcat fping rsync htop iotop iftop tar less firewalld sshguard wireguard iproute2 iperf3 zfsutils-linux net-tools ca-certificates gnupg
```

4. Verify status of firewalld and enable sshguard:

```
systemctl enable --now firewalld
systemctl enable --now sshguard
```

5. Disable and remove snapd

```
systemctl disable snapd.service
systemctl disable snapd.socket
systemctl disable snapd.seeded
systemctl disable snapd.snap-repair.timer

apt purge -y snapd

rm -rf ~/snap /snap /var/snap /var/lib/snapd
```

## Daemon-only packages (do not install on the worker and control nodes)

1. Install nginx and certbot:

```
apt install -y nginx certbot python3-certbot-nginx
```

2. Install Docker:

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


## Ansible Playbook to setup a simple k8s cluster

1. On another machine (e.g., your laptop), install ansible via virtual env:

```
sudo apt install python3-pip python3.10-venv
python3.10 -m venv ~/.local/venv/ansible
source ~/.local/venv/ansible/bin/activate
pip install ansible
ansible --version
```

2. Clone repo and enter the directory:

```
git clone https://git.vdb.to/cerc-io/service-provider-template.git
cd service-provider-template/
```

3. Update to the template

- review [this commit](https://git.vdb.to/cerc-io/service-provider-template/commit/32e1ad0bd73f0754c0978c96eaee526fa841ddb4) and modify the domain, IP, and hostnames, etc., to match your setup.

4. Install required roles:

```
ansible-galaxy install -f -p roles -r roles/requirements.yml
```

5. Generate token for the cluster (this assumes ansible vault has been setup)

```
./roles/k8s/files/token-vault.sh ./group_vars/lx_cad/k8s-vault.yml
```

6. Configure firewalld and nginx for hosts

```
ansible-playbook -i hosts site.yml --tags=firewalld,nginx
```

7. Install Stack Orchestrator for hosts

```
ansible-playbook -i hosts site.yml --tags=so --limit=so
```

8. Deploy k8s

```
ansible-playbook -i hosts site.yml --tags=k8s --limit=lx_cad
```

9. Install k8s helper tools

- on Linux systems:
```
sudo ~/lx-cad-deploy/roles/k8s/files/get-kube-tools.sh
```

- on a Mac:
```
brew install kubie kubectl yq helm
```

10. Verify cluster creation

```
kubie ctx lx-cad
kubectl get nodes -o wide
```

## Configure DNS

Like this:

|  Type  |            Hostname                |            Value                   |
|--------|------------------------------------|------------------------------------|
| A      | lx-daemon.audubon.app              | 23.111.69.218                      |
| A      | lx-cad-cluster-worker.audubon.app  | 23.111.78.182                      |
| A      | lx-cad-cluster-control.audubon.app | 23.111.78.179                      |
| NS     | audubon.app                        | ns1.digitalocean.com.              |
| NS     | audubon.app                        | ns2.digitalocean.com.              |
| NS     | audubon.app                        | ns3.digitalocean.com.              |
| CNAME  | www.audubon.app                    | audubon.app.                       |
| CNAME  | laconicd.audubon.app               | lx-daemon.audubon.app.             |
| CNAME  | lx-backend.audubon.app             | lx-daemon.audubon.app.             |
| CNAME  | lx-console.audubon.app             | lx-daemon.audubon.app.             |
| CNAME  | lx-cad.audubon.app                 | lx-cad-cluster-worker.audubon.app. |
| CNAME  | *.lx-cad.audubon.app               | lx-cad-cluster-worker.audubon.app. |
| CNAME  | pwa.audubon.app                    | lx-cad-cluster-worker.audubon.app. |
| CNAME  | *.pwa.audubon.app                  | lx-cad-cluster-worker.audubon.app. |

In DigitalOcean, it looks like:

![sp-dns](/images/dns-sp-docs.png)


## Kubernetes
TODO

- `kubie ctx default`
- `kubectl.yaml`, used by SO --> how did it get to `/home/so/.kube/config-mito-lx-cad.yaml` 
- basic commands

```
kubectl get secrets --all-namespaces
kubectl get clusterissuer
kubectl get certificates
```


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

Now, publishing records to the Laconic Registry will trigger deployments. See below for more details.

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

- https://lx-console.audubon.app displays registry records (webapp deployments)
- https://container-registry.pwa.audubon.app hosts docker images used by webapp deployments
- https://webapp-deployer-api.pwa.audubon.app listens for ApplicationDeploymentRequest and runs `laconic-so deploy-webapp-from-registry` behind the scenes
- https://webapp-deployer-ui.pwa.audubon.app displays status and logs for webapps deployed via the Laconic Registry
- https://my-test-app.pwa.audubon.app as an example webapp deployment (but not deployed via the registry)

Let's take a look at how to configure CI/CD workflow to deploy webapps via from the registry.

## Publishing webapps

1. Create a `.gitea/workflows/publish.yml` that looks like:

```
name: Publish ApplicationRecord to Registry
on:
  release:
    types: [published]

env:
  CERC_REGISTRY_USER_KEY: ${{ secrets.CICD_LACONIC_USER_KEY }}
  CERC_REGISTRY_BOND_ID: ${{ secrets.CICD_LACONIC_BOND_ID }}

jobs:
  cns_publish:
    runs-on: ubuntu-latest
    steps:
      - name: "Clone project repository"
        uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: "Enable Yarn"
        run: corepack enable
      - name: "Install registry CLI"
        run: |
          npm config set @cerc-io:registry https://git.vdb.to/api/packages/cerc-io/npm/
          npm install -g @cerc-io/laconic-registry-cli          
      - name: "Install jq"
        run: apt -y update && apt -y install jq
      - name: "Publish Application Record"
        run: scripts/publish-app-record.sh
      - name: "Request Deployment"
        run: scripts/request-app-deployment.sh
```
and set the `envs` using the `userKey` and `bondId` that were previously created.

2. Add two scripts that each look like:
```
#!/bin/bash

# scripts/publish-app-record.sh
set -e

RECORD_FILE=tmp.rf.$$
CONFIG_FILE=`mktemp`

CERC_APP_TYPE=${CERC_APP_TYPE:-"webapp/next"}
CERC_REPO_REF=${CERC_REPO_REF:-${GITHUB_SHA:-`git log -1 --format="%H"`}}
CERC_IS_LATEST_RELEASE=${CERC_IS_LATEST_RELEASE:-"true"}

rcd_name=$(jq -r '.name' package.json | sed 's/null//')
rcd_desc=$(jq -r '.description' package.json | sed 's/null//')
rcd_repository=$(jq -r '.repository' package.json | sed 's/null//')
rcd_homepage=$(jq -r '.homepage' package.json | sed 's/null//')
rcd_license=$(jq -r '.license' package.json | sed 's/null//')
rcd_author=$(jq -r '.author' package.json | sed 's/null//')
rcd_app_version=$(jq -r '.version' package.json | sed 's/null//')

cat <<EOF > "$CONFIG_FILE"
services:
  cns:
    restEndpoint: '${CERC_REGISTRY_REST_ENDPOINT:-http://console.laconic.com:1317}'
    gqlEndpoint: '${CERC_REGISTRY_GQL_ENDPOINT:-http://console.laconic.com:9473/api}'
    chainId: ${CERC_REGISTRY_CHAIN_ID:-laconic_9000-1}
    gas: 950000
    fees: 200000aphoton
EOF

next_ver=$(laconic -c $CONFIG_FILE cns record list --type ApplicationRecord --all --name "$rcd_name" 2>/dev/null | jq -r -s ".[] | sort_by(.createTime) | reverse | [ .[] | select(.bondId == \"$CERC_REGISTRY_BOND_ID\") ] | .[0].attributes.version" | awk -F. -v OFS=. '{$NF += 1 ; print}')

if [ -z "$next_ver" ] || [ "1" == "$next_ver" ]; then
  next_ver=0.0.1
fi

cat <<EOF | sed '/.*: ""$/d' > "$RECORD_FILE"
record:
  type: ApplicationRecord
  version: ${next_ver}
  name: "$rcd_name"
  description: "$rcd_desc"
  homepage: "$rcd_homepage"
  license: "$rcd_license"
  author: "$rcd_author"
  repository:
    - "$rcd_repository"
  repository_ref: "$CERC_REPO_REF"
  app_version: "$rcd_app_version"
  app_type: "$CERC_APP_TYPE"
EOF


cat $RECORD_FILE
RECORD_ID=$(laconic -c $CONFIG_FILE cns record publish --filename $RECORD_FILE --user-key "${CERC_REGISTRY_USER_KEY}" --bond-id ${CERC_REGISTRY_BOND_ID} | jq -r '.id')
echo $RECORD_ID

if [ -z "$CERC_REGISTRY_APP_CRN" ]; then
  authority=$(echo "$rcd_name" | cut -d'/' -f1 | sed 's/@//')
  app=$(echo "$rcd_name" | cut -d'/' -f2-)
  CERC_REGISTRY_APP_CRN="crn://$authority/applications/$app"
fi

laconic -c $CONFIG_FILE cns name set --user-key "${CERC_REGISTRY_USER_KEY}" --bond-id ${CERC_REGISTRY_BOND_ID} "$CERC_REGISTRY_APP_CRN@${rcd_app_version}" "$RECORD_ID"
laconic -c $CONFIG_FILE cns name set --user-key "${CERC_REGISTRY_USER_KEY}" --bond-id ${CERC_REGISTRY_BOND_ID} "$CERC_REGISTRY_APP_CRN@${CERC_REPO_REF}" "$RECORD_ID"
if [ "true" == "$CERC_IS_LATEST_RELEASE" ]; then
  laconic -c $CONFIG_FILE cns name set --user-key "${CERC_REGISTRY_USER_KEY}" --bond-id ${CERC_REGISTRY_BOND_ID} "$CERC_REGISTRY_APP_CRN" "$RECORD_ID"
fi

rm -f $RECORD_FILE $CONFIG_FILE
```
and
```
#!/bin/bash

set -e

RECORD_FILE=tmp.rf.$$
CONFIG_FILE=`mktemp`

rcd_name=$(jq -r '.name' package.json | sed 's/null//' | sed 's/^@//')
rcd_app_version=$(jq -r '.version' package.json | sed 's/null//')

cat <<EOF > "$CONFIG_FILE"
services:
  cns:
    restEndpoint: '${CERC_REGISTRY_REST_ENDPOINT:-http://console.laconic.com:1317}'
    gqlEndpoint: '${CERC_REGISTRY_GQL_ENDPOINT:-http://console.laconic.com:9473/api}'
    chainId: ${CERC_REGISTRY_CHAIN_ID:-laconic_9000-1}
    gas: 950000
    fees: 200000aphoton
EOF

if [ -z "$CERC_REGISTRY_APP_CRN" ]; then
  authority=$(echo "$rcd_name" | cut -d'/' -f1 | sed 's/@//')
  app=$(echo "$rcd_name" | cut -d'/' -f2-)
  CERC_REGISTRY_APP_CRN="crn://$authority/applications/$app"
fi

APP_RECORD=$(laconic -c $CONFIG_FILE cns name resolve "$CERC_REGISTRY_APP_CRN" | jq '.[0]')
if [ -z "$APP_RECORD" ] || [ "null" == "$APP_RECORD" ]; then
  echo "No record found for $CERC_REGISTRY_APP_CRN."
  exit 1
fi

cat <<EOF | sed '/.*: ""$/d' > "$RECORD_FILE"
record:
  type: ApplicationDeploymentRequest
  version: 1.0.0
  name: "$rcd_name@$rcd_app_version"
  application: "$CERC_REGISTRY_APP_CRN@$rcd_app_version"
  dns: "$CERC_REGISTRY_DEPLOYMENT_SHORT_HOSTNAME"
  deployment: "$CERC_REGISTRY_DEPLOYMENT_CRN"
  config:
    env:
      CERC_WEBAPP_DEBUG: "$rcd_app_version"
  meta:
    note: "Added by CI @ `date`"
    repository: "`git remote get-url origin`"
    repository_ref: "${GITHUB_SHA:-`git log -1 --format="%H"`}"
EOF

cat $RECORD_FILE
RECORD_ID=$(laconic -c $CONFIG_FILE cns record publish --filename $RECORD_FILE --user-key "${CERC_REGISTRY_USER_KEY}" --bond-id ${CERC_REGISTRY_BOND_ID} | jq -r '.id')
echo $RECORD_ID

rm -f $RECORD_FILE $CONFIG_FILE
```

Now, anytime a release is created, a new set oof records will be published to the Laconic Registry, and eventually picked up by the `deployer`, which will target the k8s cluster that we setup.

## Notes and debugging

- pvc
- 413 request too large
- using `container-registry.pwa.audubon.app/laconic-registry` or `container-registry.pwa.audubon.app` seems to both work, TODO, investigate


### DNS Secret example

```
apiVersion: v1
data:
  access-token: XXX
kind: Secret
metadata:
  name: someprovider-dns
  namespace: cert-manager
```
