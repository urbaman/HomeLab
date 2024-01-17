# Kubernetes

## cgroups

Some systems will mount cgroup v1 and cgroup v2 by default, just in different locations. It can help to see where those are with:

```bash
grep ^cgroup /etc/mtab
```

Example output (on Ubuntu 20.04 LTS):

```bash
cgroup2 /sys/fs/cgroup/unified cgroup2 rw,nosuid,nodev,noexec,relatime 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,name=systemd 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/rdma cgroup rw,nosuid,nodev,noexec,relatime,rdma 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
```

However, this does not strictly tell you if your system supports cgroup v2. As the other answer mentioned, grep cgroup /proc/filesystems is great for that.

```bash
grep cgroup /proc/filesystems
```

If your system supports cgroupv2, you would see:

```bash
nodev   cgroup
nodev   cgroup2
```

On a system with only cgroupv1, you would only see:

```bash
nodev   cgroup
```

## Disable swap

Turn swap off:

```bash
sudo swapoff -a
```

And prevents it from turning on after reboots by commenting it in the /etc/fstab file:

```bash
sudo vi /etc/fstab
```

## Containerd

Load the necessary modules for Containerd:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

Setup the required kernel parameters:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### Using Docker Repository

Docker offers the containerd containerd.io package in the .deb format through the Docker repository. So, install the required packages for containerd installation.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

#### Set up Docker Repository

Then, add the Docker’s GPG key to your system.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

And then, add the Docker repository to the system by running the below command.

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

#### Install Containerd

After adding the Docker’s repository, update the repository index and install containerd

```bash
sudo apt update
sudo apt install -y containerd.io
```

By now, the containerd service should be up and running.

```bash
sudo systemctl status containerd
```

Output:

```bash
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-05-01 13:59:49 EDT; 18s ago
       Docs: https://containerd.io
    Process: 2182 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 2183 (containerd)
      Tasks: 8
     Memory: 18.8M
        CPU: 63ms
     CGroup: /system.slice/containerd.service
             └─2183 /usr/local/bin/containerd

May 01 13:59:49 ubuntu-2204 containerd[2183]: time="2022-05-01T13:59:49.941358574-04:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 01 13:59:49 ubuntu-2204 containerd[2183]: time="2022-05-01T13:59:49.941402747-04:00" level=info msg=serving... address=/run/containerd/containerd.sock
May 01 13:59:49 ubuntu-2204 systemd[1]: Started containerd container runtime.
```

#### Containerd configuration for Kubernetes

Containerd uses a configuration file config.toml for managing demons. For Kubernetes, you need to configure the Systemd group driver for runC.

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Set the cgroup driver for runc to systemd which is required for the kubelet. For more information on this config file see the containerd configuration docs here and also here.

At the end of this section in /etc/containerd/config.toml

```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
...
```

Around line 112, change the value for SystemCgroup from false to true.

```bash
SystemdCgroup = true
```

Next, use the below command to restart the containerd service.

```bash
sudo systemctl restart containerd
```

Do it all in one command

```bash
sudo mkdir -p /etc/containerd && sudo containerd config default | sudo tee /etc/containerd/config.toml && sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml && sudo systemctl restart containerd
```

## kubelet, kubeadm, kubectl

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Kubernetes public signing key:

```bash
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the Kubernetes apt repository:

```bash
sudo echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```bash
sudo apt-get update
sudo apt-get install -y kubelet=1.28.2-00 kubeadm=1.28.2-00 kubectl=1.28.2-00
sudo apt-mark hold kubelet kubeadm kubectl
```

## Make it a template

It's a perfect time to make it a template from which we can create both control panes and workers.

## Set up the etcd cluster

Follow [these instructions](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd) to set up the etcd cluster.

Copy the following files from any etcd node in the etcd cluster to the first control panel node:
Copy etcd certs to first control panel

```bash
export CONTROL_PLANE="ubuntu@10.0.50.51"
scp /etc/etcd/etcd-ca.crt "${CONTROL_PLANE}":/home/ubuntu/ca.crt
scp /etc/etcd/server.crt "${CONTROL_PLANE}":/home/ubuntu/apiserver-etcd-client.crt
scp /etc/etcd/server.key "${CONTROL_PLANE}":/home/ubuntu/apiserver-etcd-client.key
```

Replace the value of CONTROL_PLANE with the user@host of the first control-plane node.

On the first control panel, copy the files to the following locations:

```bash
sudo mkdir /etc/kubernetes/pki
sudo mkdir /etc/kubernetes/pki/etcd
sudo cp ca.crt /etc/kubernetes/pki/etcd/
sudo cp apiserver-etcd-client.crt /etc/kubernetes/pki/
sudo cp apiserver-etcd-client.key /etc/kubernetes/pki/
```

## Set up the first control plane node

Create a file called kubeadm-config.yaml with the following contents (modify IPs and hostnames as needed, then eliminate comments):

```bash
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "k8cp.urbaman.it:6443" # change this (see below)
networking:
  podSubnet: "10.10.0.0/16" # change this (see below)
  serviceSubnet: "10.1.0.0/16" # change this (see below)
etcd:
  external:
    endpoints:
      - https://k8cp.urbaman.it:2379 # change to the haproxy vip ip/host appropriately
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```

Note: The difference between stacked etcd and external etcd here is that the external etcd setup requires a configuration file with the etcd endpoints under the external object for etcd. In the case of the stacked etcd topology, this is managed automatically.

Remember to change the controlPlaneEndpoint, podSubnet (equal to following calico IP CIDR), etcd endpoints (equal to the endpoints of the external etcd)

Note: the podSubnet (CIDR of IPs assigned to the pods) must be a private network CIDR (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) of other local network CIDRs already in use. I use 10.0.x.x and 192.168.1.x, so I assign 192.168.0.0/16 to kubernetes internal network. Also keep in mind that kubernetes by default assigns a /24 set of IPs to each node, so there will in my case 24-12=12, 2^12=4.096 nodes in the cluster, each with a 65-110 pods max. Please note that there's quite a bit of IPs wasted, as top cluster nodes is 1000 and I do only 6. Being in my own nome network simplifies a little bit, but beware the network implications of these settings.

The following steps are similar to the stacked etcd setup:

```bash
sudo kubeadm init --config kubeadm-config.yaml --upload-certs
```

Write the output join commands that are returned to a text file for later use.

### Setup kubectl for a normal user

To start using your cluster, you need to run the following as a regular user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Apply the CNI plugin of your choice

Note: You must pick a network plugin that suits your use case and deploy it before you move on to next step. If you don't do this, you will not be able to launch your cluster properly. We will use Calico:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml -O
```

If you wish to customize the Calico install, customize the downloaded custom-resources.yaml manifest locally (for example, customizing the IP CIDR to match kubeadm podSubnet above).

```bash
kubectl create -f custom-resources.yaml
```

Then, intstall the calicoctl to manage Calico.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/calicoctl.yaml
alias calicoctl="kubectl exec -i -n kube-system calicoctl -- /calicoctl" 
```

This way, in order to use the calicoctl alias when reading manifests, redirect the file into stdin, for example:

```bash
calicoctl create -f - < my_manifest.yaml
```

#### Enable ebpf mode (not working)

Create a configmap with informations on the cluster service:

```bash
kubectl cluster-info
vi tigera-config-map.yaml
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: kubernetes-services-endpoint
  namespace: tigera-operator
data:
  KUBERNETES_SERVICE_HOST: '<API server host>'
  KUBERNETES_SERVICE_PORT: '<API server port>'
```

```bash
kubectl apply -f tigera-config-map.yaml
```

Then disable kubeproxy and enable ebpf

```bash
kubectl patch ds -n kube-system kube-proxy -p '{"spec":{"template":{"spec":{"nodeSelector":{"non-calico": "true"}}}}}'
kubectl patch installation.operator.tigera.io default --type merge -p '{"spec":{"calicoNetwork":{"linuxDataplane":"BPF", "hostPorts":null}}}'
calicoctl patch felixconfiguration default --patch='{"spec": {"bpfExternalServiceMode": "DSR"}}'
```

### Add second netwotk with multus and wereabouts

See [Second network with Multus and Whereabouts](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Multus)

## Steps for the rest of the control plane nodes

The steps are the same as for the stacked etcd setup:

Make sure the first control plane node is fully initialized.

Join each control plane node with the join command you saved to a text file. It's recommended to join the control plane nodes one at a time.
Don't forget that the decryption key from --certificate-key expires after two hours, by default.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Install workers

Worker nodes can be joined to the cluster with the command you stored previously as the output from the kubeadm init command:

```bash
sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

From one of the control planes:

```bash
sudo scp /etc/kubernetes/admin.conf ubuntu@k8w1:/home/ubuntu
sudo scp /etc/kubernetes/admin.conf ubuntu@k8w2:/home/ubuntu
sudo scp /etc/kubernetes/admin.conf ubuntu@k8w3:/home/ubuntu
```

On the nodes:

```bash
mkdir -p $HOME/.kube
sudo cp -i admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Control plane node isolation

By default, your cluster will not schedule Pods on the control plane nodes for security reasons. If you want to be able to schedule Pods on the control plane nodes, for example for a single machine Kubernetes cluster, run:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-
```

## Upgrading the cluster

On the first control plane:

```bash
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm=1.xx.y-* && \
sudo apt-mark hold kubeadm
kubeadm version
sudo kubeadm upgrade plan
```

```bash
sudo kubeadm upgrade apply v1.xx.y
kubectl drain <node> --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet=1.xx.y-* kubectl=1.xx.y-* && \
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon <node>
```

On all other nodes

```bash
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm=1.xx.y-* && \
sudo apt-mark hold kubeadm
sudo kubeadm upgrade node
kubectl drain <node> --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet=1.xx.y-* kubectl=1.xx.y-* && \
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon <node>
```

## Delete the cluster

On each node, one by one, keeping the first control plane

```bash
sudo kubeadm reset && sudo apt remove kubeadm kubectl kubelet containerd.io --purge -y --allow-change-held-packages && sudo apt autoremove --purge -y && sudo rm -rf /etc/cni/* && sudo rm -rf /opt/cni/* && sudo rm -rf /etc/kubernetes && sudo rm -rf /longhorn/storage/*
```

Remember to wipe the etcd cluster, if external. On an etcd node:

```bash
etcdctl --cacert /etc/etcd/etcd-ca.crt --cert /etc/etcd/server.crt --key /etc/etcd/server.key --endpoints https://<etcdIP1>:2379,https://<etcdIP1>:2379,https://<etcdIP1>:2379 del "" --from-key=true
```
