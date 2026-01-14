# Creating the Cluster

## Prerequisites

- Git Server (Gitea)
- Secrets Vault (OpenBao, for External Secrets operator)

**Design notes:**

- We will use hostpath for database clusters, setting three nodes for scheduling the clusters. You need to install a proper database-ready shared storage to replace this (rookCeph?) but in a homeLab environment having both database cluster replicas and storage replicas would be too much of a bottleneck. Rook Ceph is there just for an example. You can also use NFS, but be aware of the settings to make it database-ready, as some of them do not support NFS or need specific settings.
- ...

## Repository design

We will need a git repository with the following structure

```
project # kubeadm, microk8s, talos, ...
└── cluster
    ├── bootstrap # bootstrapping the cluster, you can select the needed directory (kubeadm, microk8s, talos) and move the file up one level in /bootstrap directory
    |   ├── kubeadm
    |   |   ├── kubeadm-config.yaml
    |   |   ├── cilium-argocd-chart.yaml
    |   |   └── cilium-values.yaml
    |   ├── microk8s
    |   |   └── microk8s.yaml
    |   └── talos
    |       ├── control-panel.yaml
    |       ├── hostname.yaml
    |       ├── worker.yaml
    |       ├── cilium-argocd-chart.yaml
    |       └── cilium-values.yaml
    ├── pre-production # pre-production setup
    |   ├── metricsServer
    |   |   ├── manifest-kubelet-serving-cert-approver-argocd.yaml #copy the kubelet-serving-cert-approver manifest, standalone or high-availability
    |   |   └── manifest-metrics-server-argocd.yaml #copy the metrics server manifest, standalone or high-availability
    |   ├── externalSecretsOperator
    |   |   ├── eso-argocd-chart.yaml
    |   |   ├── eso-values.yaml
    |   |   └── manifest-eso-argocd-external-secret-store.yaml
    |   ├── sealedSecrets
    |   ├── rookCeph
    |   |   ├── rook-ceph-argocd-chart.yaml
    |   |   ├── rook-ceph-values.yaml
    |   |   ├── rook-ceph-cluster-argocd-chart.yaml
    |   |   └── rook-ceph-cluster-values.yaml
    |   ├── NFS
    |   |   ├── nfs-subdir-external-provisioner-argocd-chart.yaml
    |   |   ├── nfs-subdir-external-provisioner-values.yaml
    |   |   └── manifest-nfs-subdir-external-provisioner-argocd-sc-vss.yaml
    |   ├── valkey
    |   └── argoCD
    └── production # production setup
```

## Basic bootstrapping

***Beware:*** use helm template instead of helm install or upgrade -i when installing things like Cilium or other Pre-Production stuff if you're going to use ArgoCD for GitOps, as it makes use of helm template, and you'll make the tools manageable through ArgoCD

```bash
helm template cilium cilium/cilium -n kube-system -f cilium-values.yaml > cilium.yaml
kubectl apply -f cilium.yaml
```

### Kubeadm/Microk8s (doable with terraform)

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [(optional) External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [(optional, see kubevip) High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

or:

4. [Microk8s](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Microk8s)

### Talos Linux

[Deploy, with or without Omni](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/05-Talos)

## Pre-production

Manually install basic tools, without service/pod monitors, through manifests, eventually templating helm charts (doable with ansible) if using ArgoCD for GitOps.

- External Secrets Operator (with OpenBao)
- Sealed Secrets (with Git)
- Storage (for Valkey)
- Valkey (for ArgoCD)
- ArgoCD

## Production

Point ESO to the Secrets Vault or SS to the Git Server and get the secrets, then point ArgoCD to a repo with the proper ArgoApps to (re)deploy

- Metrics Server
- Storage (NFS, Ceph,... if not already deployed for Valkey)
- Monitoring (Prometheus Stack, and service/pod monitors where not already deployed: CNI, ArgoCD, ESO/SS, Storage, ...)
- Cert-manager
- Ingress (Traefik)
- Metallb (if needed)
- Databases (other than Valkey)
- ...
