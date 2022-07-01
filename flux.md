# Flux Setup

## Bootstrapping
Follow the [fluxcd bootstap docs](https://fluxcd.io/docs/installation/#bootstrap) to install flux on your cluster

## Cluster setup
```
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod
```

## Source
```
flux create source git podinfo-image-automation \
    --url=https://github.com/rparmer/podinfo-image-automation.git \
    --branch=release \
    --username=oauth2 \
    --password=$GITHUB_TOKEN
```
> Authentication with write access is required in order to push back to the repo

## Kustomizations
### dev
```
flux create kustomization podinfo-image-automation \
    --namespace=dev \
    --source=GitRepository/podinfo-image-automation.flux-system \
    --path="./dev" \
    --prune=true \
    --interval=1m
```

### staging
```
flux create kustomization podinfo-image-automation \
    --namespace=staging \
    --source=GitRepository/podinfo-image-automation.flux-system \
    --path="./staging" \
    --prune=true \
    --interval=1m
```

### prod
```
flux create kustomization podinfo-image-automation \
    --namespace=prod \
    --source=GitRepository/podinfo-image-automation.flux-system \
    --path="./prod" \
    --prune=true \
    --interval=1m
```

## Clean up
```
flux delete kustomization podinfo-image-automation --namespace=dev --silent
flux delete kustomization podinfo-image-automation --namespace=staging --silent
flux delete kustomization podinfo-image-automation --namespace=prod --silent
flux delete source git podinfo-image-automation --silent
kubectl delete namespace dev
kubectl delete namespace staging
kubectl delete namespace prod
```