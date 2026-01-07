# Talos Linux

## Talos

See Omni (you'll have a GUI managing your clusters)

## Omni

- Install Omni (see docker installation), can use Authentic for SAML auth instead of Auth0

### Manual VM creation (or bare metal nodes)

- Download installation media
- (Optional) Create virtual machines (Proxmox?)
- Start the nodes with the installation media (do not install)
- Get the machines in Omni, set the patches (see the examples), create the cluster
- Get kubectl and setup the kubelogin plugin for OIDC connection, download the kubeconfig file from Omni, connect
- Install Cilium (or some other CNI, see Calico on the Talos documentation for example)

### Cilium

See [Cilium deployment notes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cilium)
