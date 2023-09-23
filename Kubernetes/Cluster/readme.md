# Creating the Cluster

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

See [https://microk8s.io/docs/high-availability#set-failure-domains-4](https://microk8s.io/docs/high-availability#set-failure-domains-4) to set the failure domanis (probably the pve nodes)

See [https://microk8s.io/docs/external-etcd](https://microk8s.io/docs/external-etcd) for external etcd

See [https://microk8s.io/docs/add-launch-config](https://microk8s.io/docs/add-launch-config) to set the loadbalancer IP in che SANs

See [https://discuss.kubernetes.io/t/adding-a-worker-node-only-with-microk8s/14801](https://discuss.kubernetes.io/t/adding-a-worker-node-only-with-microk8s/14801) to 