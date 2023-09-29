# Microk8s cluster

See [https://microk8s.io/docs/high-availability#set-failure-domains-4](https://microk8s.io/docs/high-availability#set-failure-domains-4) to set the failure domanis (probably the pve nodes)

See [https://microk8s.io/docs/external-etcd](https://microk8s.io/docs/external-etcd) for external etcd

See [https://microk8s.io/docs/add-launch-config](https://microk8s.io/docs/add-launch-config) to set the loadbalancer IP in che SANs

See [https://discuss.kubernetes.io/t/adding-a-worker-node-only-with-microk8s/14801](https://discuss.kubernetes.io/t/adding-a-worker-node-only-with-microk8s/14801) to set a loadbalancer (?)

## On all nodes

```bash
sudo mkdir -p /var/snap/microk8s/current/certs/
```

From an etcd node:

```bash
export CONTROL_PLANE="ubuntu@10.0.50.51"
scp /etc/etcd/etcd-ca.crt "${CONTROL_PLANE}":/home/ubuntu/etcd-cluster-ca.crt
scp /etc/etcd/server.crt "${CONTROL_PLANE}":/home/ubuntu/etcd-cluster-client.crt
scp /etc/etcd/server.key "${CONTROL_PLANE}":/home/ubuntu/etcd-cluster-client.key
```

From the control plane

```bash
sudo cp etcd-cluster-ca.crt /var/snap/microk8s/current/certs/etcd-cluster-ca.crt
sudo cp etcd-cluster-client.crt /var/snap/microk8s/current/certs/etcd-cluster-client.crt
sudo cp etcd-cluster-client.key /var/snap/microk8s/current/certs/etcd-cluster-client.key
```

View the provided microk8s.yaml file to set the CIDRs, add the Load Balancer IP and address, and other settings.

```bash
sudo mkdir -p /var/snap/microk8s/common/
sudo cp microk8s.yaml /var/snap/microk8s/common/.microk8s.yaml
```

```bash
sudo snap install microk8s --classic --channel=1.28
sudo cp etcd-cluster-ca.crt /var/snap/microk8s/current/certs/etcd-cluster-ca.crt
sudo cp etcd-cluster-client.crt /var/snap/microk8s/current/certs/etcd-cluster-client.crt
sudo cp etcd-cluster-client.key /var/snap/microk8s/current/certs/etcd-cluster-client.key
sudo snap restart microk8s
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
microk8s status --wait-ready
vi /var/snap/microk8s/current/certs/csr.conf.template
```

## this part about the loadbalancer is not working yet

Add the Loadbalancer DNS as DNS.100 and the Loadbalancer IP as IP.100


```bash
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/client.config
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/controller.config
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/kubelet.config
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/scheduler.config
```

```bash
microk8s add node
```

Output:

```bash
From the node you wish to join to this cluster, run the following:
microk8s join 10.0.50.51:25000/e223f1b3d2040b82ee82aa6db88bfc5d/5bed05e3b3a8

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 10.0.50.51:25000/e223f1b3d2040b82ee82aa6db88bfc5d/5bed05e3b3a8 --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 10.0.50.51:25000/e223f1b3d2040b82ee82aa6db88bfc5d/5bed05e3b3a8
microk8s join 10.0.90.51:25000/e223f1b3d2040b82ee82aa6db88bfc5d/5bed05e3b3a8
```

Use the displayed command on the node, then repeat for all of the nodes (be aware of the command for the workers)

On a control plane:

```bash
sudo microk8s join 10.0.50.51:25000/e223f1b3d2040b82ee82aa6db88bfc5d/5bed05e3b3a8
```

On a worker:

```bash
sudo microk8s join 10.0.50.51:25000/e223f1b3d2040b82ee82aa6db88bfc5d/5bed05e3b3a8 --worker
```
