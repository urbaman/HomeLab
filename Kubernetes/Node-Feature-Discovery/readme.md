# Node feature discovery installation

## Install node feature discovery

Get the repo

```bash
helm repo add nfd https://kubernetes-sigs.github.io/node-feature-discovery/charts
helm repo update
```

Eventually modify the values enabling the topology updater

```bash
helm show values nfd/node-feature-discovery > nodefeaturediscovery.yaml
vi nodefeaturediscovery.yaml
```

Install

```bash
helm upgrade -i nfd nfd/node-feature-discovery --namespace node-feature-discovery --create-namespace [-f nodefeaturediscovery.yaml]
```

## Install GPU feature discovery

Get the repo and install (nfd is a prerequisite)

```bash
helm repo add nvgfd https://nvidia.github.io/gpu-feature-discovery
helm repo update
helm upgrade -i nvgfd nvgfd/gpu-feature-discovery --namespace gpu-feature-discovery --create-namespace
```

## Check the discovery

```bash
kubectl get no -o json | jq .items[].metadata.labels
```
