# Microk8s cluster

See [https://microk8s.io/docs/high-availability#set-failure-domains-4](https://microk8s.io/docs/high-availability#set-failure-domains-4) to set the failure domanis (probably the pve nodes)

See [https://microk8s.io/docs/external-etcd](https://microk8s.io/docs/external-etcd) for external etcd

See [https://microk8s.io/docs/add-launch-config](https://microk8s.io/docs/add-launch-config) to set the loadbalancer IP in che SANs

See [https://discuss.kubernetes.io/t/adding-a-worker-node-only-with-microk8s/14801](https://discuss.kubernetes.io/t/adding-a-worker-node-only-with-microk8s/14801) to set a loadbalancer (?)

## On all nodes

```bash
sudo mkdir -p /var/snap/microk8s/common/
sudo vi /var/snap/microk8s/common/.microk8s.yaml
```

View the provided microk8s.yaml file to set the CIDRs, add the Load Balancer IP and address, and other settings.

```bash
sudo snap install microk8s --classic --channel=1.28
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
microk8s status --wait-ready
```

## External etcd

From an external etcd node, copy the certificates on the control planes

```bash
export CONTROL_PLANE="ubuntu@10.0.50.51"
scp /etc/etcd/etcd-ca.crt "${CONTROL_PLANE}":/var/snap/microk8s/current/certs/etcd-ca.crt
scp /etc/etcd/server.crt "${CONTROL_PLANE}":/var/snap/microk8s/current/certs/client.crt
scp /etc/etcd/server.key "${CONTROL_PLANE}":/var/snap/microk8s/current/certs/etcd-client.key
```

On the first control plane, add the ectd and the external loadbalancer configs

```bash
sudo vi /var/snap/microk8s/current/args/kube-apiserver
```

```bash
--etcd-servers=https://etcd.urbaman.it:2379
--etcd-cafile=${SNAP_DATA}/certs/etcd-ca.crt
--etcd-certfile=${SNAP_DATA}/certs/client.crt
--etcd-keyfile=${SNAP_DATA}/certs/etcd-client.key
--bind-address=0.0.0.0
```

Restart microk8s and re-apply the calico conf

```bash
sudo snap restart microk8s
microk8s enable dns:10.0.50.2,10.0.50.10
microk8s kubectl apply -f /var/snap/microk8s/current/args/cni-network/cni.yaml
microk8s kubectl delete ippools default-ipv4-ippool
microk8s kubectl rollout restart daemonset/calico-node -n kube-system
```

Get the content of the `/var/snap/microk8s/current/credentials/client.config` file, change the IP to the Load Balancer IP and use it with kubectl
