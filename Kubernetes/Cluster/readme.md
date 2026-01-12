# Creating the Cluster

## Basic bootstrapping

### Kubeadm/Microk8s (doable with terraform)

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [(optional) External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [(optional, see kubevip) High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

or:

4. [Microk8s](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Microk8s)

### Talos Linux

[Deploy, with or without Omni](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)/microk8s

## Pre-production

Prerequisites, already running:

- Git Server (Gitea)
- Secrets Vault (OpenBao, for External Secrets operator)

Manually install basic tools, without service/pod monitors, through manifests, eventually templating helm charts (doable with kustomize/ansible):

- ArgoCD
- External Secrets Operator (with OpenBao)
- Sealed Secrets (with Git)

## Production

Point ESO to the Secrets Vault or SS to the Git Server and get the secrets, then point ArgoCD to a repo with the proper ArgoApps to (re)deploy

- Metrics Server
- Storage (NFS, Ceph,...)
- Monitoring (Prometheus Stack, and service/pod monitors where not already deployed: CNI, ArgoCD, ESO/SS, Storage, ...)
- Cert-manager
- Ingress (Traefik)
- Metallb (if needed)
- Databases
- ...
