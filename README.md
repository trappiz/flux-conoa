# Kubernetes cluster
## Using minikube to make things as simple as possible in a demo envirnoment

minikube start --cpus 4 --memory 16384

## Enable metallb for loadbalancer locally
minikube addons enable metallb

## add lb ips
minikube addons configure metallb
192.168.39.110
192.168.39.120

## Flux install Install
curl -s https://fluxcd.io/install.sh | sudo bash

or (the paranoid way :D):

wget https://github.com/fluxcd/flux2/releases/download/v0.38.3/flux_0.38.3_linux_amd64.tar.gz
tar -xvf flux_0.38.3_linux_amd64.tar.gz
sudo mv flux /usr/local/bin/flux

# Run pre-checks flux
flux check --pre

xport GITHUB_TOKEN=<my-token>

flux bootstrap github \                                     
  --owner=trappiz \
  --repository=flux-cluster-demo \
  --branch=main \
  --path=./clusters/democluster \
  --personal

# Navigate out of the folder
cd ..


git clone git@github.com:trappiz/flux-cluster-demo.git
cd flux-cluster-demo

flux create source git podinfo \
  --url=https://github.com/trappiz/argocd-conoa \
  --branch=master \
  --interval=30s \
  --export > ./clusters/democluster/podinfo-source.yaml


# Commit!

git add -A && git commit -m "Add podinfo GitRepository"
git push

# Create kustomize app

flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize/podinfo" \
  --prune=true \
  --interval=1m \
  --export > ./clusters/democluster/podinfo-kustomization.yaml

git add -A && git commit -m "Add podinfo Kustomization"
git push


# Check status!

flux get kustomizations --watch

## Install helm app 
## In this case we will install nginx

This is the same as running:

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

flux create source helm ingress-nginx --url https://kubernetes.github.io/ingress-nginx
flux create helmrelease ingress-nginx --chart ingress-nginx \
  --source HelmRepository/ingress-nginx \
  --chart-version 4.1.4 \
  --create-target-namespace \
  --target-namespace=ingress-nginx

## Override helm values

spec:
  ..
  values:
    controller:
      service:
        type: ClusterIP



## Create flux ingress

## Podinfo url
https://podinfo.home.lab
