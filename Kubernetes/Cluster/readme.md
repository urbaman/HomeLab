# Creating the Cluster

## Basic bootstrapping

### Kubeadm/Microk8s (doable with terraform)

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [(optional) External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [(optional, see kubevip) High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)/microk8s

### Talos Linux

Deploy, with or wothout Omni

## Pre-production

Manually install basic tools, without service/pod monitors, through manifests, eventually templating helm charts (doable with ansible):

- ArgoCD
- Storage (NFS, Ceph,...)
- Cert-manager
- Secrets vault
- Ingress (Traefik)
- Metallb (if needed)
- Databases
- External Secrets Operator
- Sealed Secrets (optional)
- Git-manager (Gitea)
- Monitoring (Prometheus Stack)

## Production

Point ArgoCD to a repo with the proper ArgoApps to (re)deploy, adding service/pod monitors and all of the apps
