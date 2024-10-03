# Microk8s cluster

## On all nodes (not to do)

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

## this part about the loadbalancer is not working yet (not to do)

Add the Loadbalancer DNS as DNS.100 and the Loadbalancer IP as IP.100


```bash
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/client.config
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/controller.config
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/kubelet.config
sed -i 's\https://127.0.0.1\https://k8cp.urbaman.it\g' /var/snap/microk8s/current/credentials/scheduler.config
```

```bash
sudo microk8s add-node
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

## Cluster with Loadbalance between the CPs with Kube-vip

1. Install microk8s on the 3 nodes that form the control plane cluster:

```bash
sudo snap install microk8s --classic --channel=1.30
sudo usermod -a -G microk8s $USER
mkdir -p ~/.kube
chmod 0700 ~/.kube
su $USER
```

2. Create new certificates on all nodes allowing for the VIP address:
   a. Edit `/var/snap/microk8s/current/certs/csr.conf.template` on all control planes
   b. Add IP.99 = a.b.c.d where a.b.c.d is the VIP address
   c. Add DNS.99 = mk8.domain.com for the loadbalancer FQDN
   c. Generate new certificates on all nodes with `sudo microk8s refresh-certs`
3. Form the cluster following the instructions listed here: https://microk8s.io/docs/high-availability
4. Prepare the basic cluster components microk8s enable rbac helm3 dns
5. Install kube-vip via helm-chart (as a deamon set)

```bash
microk8s helm repo add kube-vip https://kube-vip.github.io/helm-charts
microk8s helm repo update
microk8s helm install kube-vip kube-vip/kube-vip --namespace kube-system -f values.yaml
```

The file values.yaml can be found here https://github.com/kube-vip/helm-charts/blob/main/charts/kube-vip/values.yaml
and I simply modified so that the following was configured

```bash
config:
  address: "192.168.0.30"  # This is my VIP

env:
  vip_interface: "eth0" # This is the interface, same on all nodes where the VIP is announced
  vip_arp: "true"  # mandatory for L2 mode
  lb_enable: "true" # enable load balancing
  lb_port: "16443" # description here -   https://kube-vip.io/architecture/#control-plane-load-balancing , changed only because microk8s used 16443 instead of 6443
  vip_cidr: "32"
  cp_enable: "true" # enable control plane load balancing
  svc_enable: "false" # disable SVC load balancing
  vip_leaderelection: "true" # mandatory for L2 mode

nodeSelector:
  node.kubernetes.io/microk8s-controlplane: microk8s-controlplane
```

At this point we are done, i get a kubectl config from microk8s using microk8s config and simply make sure that the server IP is changed to the VIP instead of localhost used by default

Notice that the deamon set comes with a toileration and node-selector so that will only run on control plane nodes and not on workers ones.

I have tested also a join from a worker, everything seems to be working fine (i can join and i get proper HA on failures) , the only thing is that you want to run "microk8s add-node" from the node where the VIP is currently active (i used  ip addr show eth0 to find the server with the VIP being active on ) .. not sure if it's mandatory, but it worked well for me .
The only thing i had to take care of was to edit `/var/snap/microk8s/current/traefik/provider.yaml` to only have my VIP there, and make the change stable by setting `--refresh-interval 0` in `/var/snap/microk8s/current/args/apiserver-proxy`.
