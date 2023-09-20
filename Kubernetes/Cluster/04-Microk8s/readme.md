# Microk8s cluster

## On all nodes

```bash
sudo snap install microk8s --classic --channel=1.28
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
microk8s status --wait-ready
```
## Not working properly
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
--etcd-certfile=${SNAP_DATA}/certs/etcd-client.crt
--etcd-keyfile=${SNAP_DATA}/certs/etcd-client.key
--service-cluster-ip-range=10.10.0.0/16
--advertise-address=k8cp.urbaman.it
--bind-address=0.0.0.0
```

Change the Pod subnet

```bash
sudo vi /var/snap/microk8s/current/args/cni-network/cni.yaml
```

```bash
            - name: CALICO_IPV4POOL_CIDR
              value: "172.16.0.0/12"
```

```bash
sudo vi /var/snap/microk8s/current/args/kube-proxy
```

```bash
--cluster-cidr=172.16.0.0/12
```

Restart microk8s and re-apply the calico conf

```bash
sudo snap restart microk8s
microk8s kubectl apply -f /var/snap/microk8s/current/args/cni-network/cni.yaml
microk8s kubectl delete ippools default-ipv4-pool
microk8s kubectl rollout restart daemonset/calico-node
```
