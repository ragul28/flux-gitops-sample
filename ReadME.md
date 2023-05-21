# Flux gitops sample


## EKS - eksctl

###  Create EKS cluster
```sh
eksctl create cluster -f eks-cluster.yaml
```

### Get & Delete cluster
```sh
eksctl get cluster --region us-west-1
eksctl delete cluster test-cluster --region us-west-1
```

## Flux Bootstrap repo
* Github token is needed to create repo with full repo access. (classic PAT token https://fluxcd.io/flux/get-started/).

```sh
flux bootstrap github \
  --owner=ragul28 \
  --repository=flux-gitops-sample \
  --path=eks-cluster \
  --personal
```

## Setup flux monitoring using kube prom stack
```sh
flux create source git flux-monitoring \
  --interval=30m \
  --url=https://github.com/fluxcd/flux2 \
  --branch=main > eks-cluster/flux-monitoring-source.yaml


flux create kustomization kube-prometheus-stack \
  --interval=1h \
  --prune \
  --source=flux-monitoring \
  --path="./manifests/monitoring/kube-prometheus-stack" \
  --health-check-timeout=5m > eks-cluster/kube-prometheus-stack-kustomization.yaml


flux create kustomization monitoring-config \
  --depends-on=kube-prometheus-stack \
  --interval=1h \
  --prune=true \
  --source=flux-monitoring \
  --path="./manifests/monitoring/monitoring-config" \
  --health-check-timeout=1m
```