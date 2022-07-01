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
flux create source git podinfo \
    --url=https://github.com/rparmer/podinfo.git \
    --branch=release
```

## Kustomizations
### dev
```
flux create kustomization podinfo \
    --namespace=dev \
    --source=GitRepository/podinfo.flux-system \
    --path="./dev" \
    --prune=true \
    --interval=1m
```

### staging
```
flux create kustomization podinfo \
    --namespace=staging \
    --source=GitRepository/podinfo.flux-system \
    --path="./staging" \
    --prune=true \
    --interval=1m
```

### prod
```
flux create kustomization podinfo \
    --namespace=prod \
    --source=GitRepository/podinfo.flux-system \
    --path="./prod" \
    --prune=true \
    --interval=1m
```

## Clean up
```
flux delete kustomization podinfo --namespace=dev --silent
flux delete kustomization podinfo --namespace=staging --silent
flux delete kustomization podinfo --namespace=prod --silent
flux delete source git podinfo --silent
kubectl delete namespace dev
kubectl delete namespace staging
kubectl delete namespace prod
```