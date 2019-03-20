# Flux on AKS - Quickstart

Before beginning, clone this repository.

This is a short guide to creating an [AKS cluster](https://azure.microsoft.com/en-au/services/kubernetes-service/) and installing [Flux](https://github.com/weaveworks/flux) on that cluster.

I have provided a [Dockerfile](docker/Dockerfile) that builds a container with all the required dependencies.

## Build the CLI

```sh
cd docker
docker build -t flux-cli --build-arg AZ_VER=2.0.49 --build-arg FLUXCTL_VER=1.11.0 --build-arg HELM_VERSION=v2.11.0 .
```

## Run the CLI

```sh
docker run -it flux-cli
```


# Create an AKS cluster

## Auth with Azure

```az login```

## Create Azure resource group

```az group create -n flux -l australiasoutheast```

## Create an Azure Service Principal

```sh
$ az ad sp create-for-rbac --skip-assignment -n myApp
{
  "appId": "XXXXX",
  "displayName": "myApp",
  "name": "http://myApp",
  "password": "YYYYYY",
  "tenant": "ZZZZZZZ"
}

```

## Create the actual cluster

```az aks create -n flux -g flux --generate-ssh-keys --service-principal XXXXX --client-secret YYYYY```

## Authenticate with the cluster

```az aks get-credentials -n flux -g flux```

# Flux the cluster

## Deploy Helm

Create a service account and a cluster role binding for Tiller:

```sh
kubectl -n kube-system create sa tiller

kubectl create clusterrolebinding tiller-cluster-rule \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller
```

## Deploy Tiller

```helm init --skip-refresh --upgrade --service-account tiller```

## Install Flux on k8s

Add the weaveworks repo for flux

```helm repo add weaveworks https://weaveworks.github.io/flux```

and apply the helm release CRD

```kubectl apply -f https://raw.githubusercontent.com/weaveworks/flux/master/deploy-helm/flux-helm-release-crd.yaml```


and configure flux to use your git repo

> Fork [flux-get-started](https://github.com/weaveworks/flux-get-started)
    on GitHub and replace the `weaveworks` with your GitHub username in
    [here](https://github.com/weaveworks/flux-get-started/blob/master/releases/ghost.yaml#L13)


```sh
helm upgrade -i flux \
--set helmOperator.create=true \
--set helmOperator.createCRD=false \
--set git.url=git@github.com:YOURUSERNAME/flux-get-started \
--namespace flux \
weaveworks/flux
```

## Watch the pods to check they're created

```watch kubectl -n flux get pods```

## Give flux write access to the repo

Open GitHub, navigate to your fork, go to **Setting** > **Deploy keys**, click on Add deploy key, give it a Title, check **Allow write access**, paste the Flux public key and click Add key.

You can get the public key by running:

```fluxctl identity --k8s-fwd-ns flux```

## Committing a small change

`flux-get-started` is a simple example in which three services
(mongodb, redis and ghost) are deployed. Here we will simply update the
version of mongodb to a newer version to see if Flux will pick this
up and update our cluster.

The easiest way is to update your fork of `flux-get-started` and
change the `image` argument.

Replace `YOURUSER` in `https://github.com/YOURUSER/flux-get-started/edit/master/releases/mongodb.yaml`
with your GitHub ID, open the URL in your browser, edit the file,
change the `tag:` line to the following:

```yaml
  values:
    image:
      repository: bitnami/mongodb
      tag: 4.0.6
```

## Check the deployment

Your deployment should look similar to this:

```sh
$ helm list --namespace demo
NAME    REVISION        UPDATED                         STATUS          CHART           APP VERSION     NAMESPACE
ghost   1               Wed Mar 20 02:03:50 2019        DEPLOYED        ghost-2.1.16    1.21.5          demo
mongodb 2               Wed Mar 20 02:05:59 2019        DEPLOYED        mongodb-4.9.0   4.0.3           demo
redis   1               Wed Mar 20 02:03:52 2019        DEPLOYED        redis-5.1.3     4.0.12          demo
```

## Troubleshoot

This quickstart is in part based on [this guide](https://github.com/weaveworks/flux/blob/master/site/helm-get-started.md)

