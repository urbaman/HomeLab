# Kubernetes

## Cluster structure

- 3x controllers: k8cp1, k8cp2, k8cp3
- 3x workers: k8w1, k8w2, k8w3
- 3x etcd cluster: etcd1, etce2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3
- 3x glusterfs cluster: truenas1, truenas2, truenas3

## 1) Glusterfs

https://itnext.io/kubernetes-storage-part-2-glusterfs-complete-tutorial-77542c12a602

## 2) External Etcd

### Downloading etcd Binaries

On each Linux host, run the below commands to download and install the latest version of the binaries:

```bash
ETCD_VER=v3.5.4
DOWNLOAD_URL=https://storage.googleapis.com/etcd

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test
mkdir -p /tmp/etcd-download-test
curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
chmod +x /tmp/etcd-download-test/etcd
chmod +x /tmp/etcd-download-test/etcdctl 

#Verify the downloads
/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version

#Move them to the bin folder
sudo mv /tmp/etcd-download-test/etcd /usr/local/bin
sudo mv /tmp/etcd-download-test/etcdctl /usr/local/bin
sudo mv /tmp/etcd-download-test/etcdutl /usr/local/bin
```

### Generating and Distributing the Certificates

We will use cfssl tool from Cloudflare to generate the certificates and keys.

From a linux workstation:

```bash
apt install golang-cfssl 
```

Make a directory called certs and run the below commands to generate the CA certificate and server certificate and key combinations for each host.

```bash
mkdir certs
cd certs
```

First, let’s create the CA certificate that’s going to be used by all the etcd servers and the clients.

```bash
echo '{"CN":"CA","key":{"algo":"rsa","size":2048}}' | cfssl gencert -initca - | cfssljson -bare ca -
echo '{"signing":{"default":{"expiry":"43800h","usages":["signing","key encipherment","server auth","client auth"]}}}' > ca-config.json
```

This results in three files – ca-key.pem, ca.pem, and ca.csr

Next, we will generate the certificate and key for the first node.

```bash
export NAME=node-1
export ADDRESS=10.0.0.60,$NAME
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

Repeat the steps for the next two nodes.

```bash
export NAME=node-2
export ADDRESS=10.0.0.61,$NAME
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

```bash
export NAME=node-3
export ADDRESS=10.0.0.62,$NAME
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

At this point, we have the certificates and keys generated for the CA and all the three nodes.

It’s time to distribute these certificates to each node of the cluster.

Run the below command to copy the certificates to the respective nodes by replacing the username and the IP address.

```bash
HOST=10.0.0.60
USER=ubuntu

scp ca.pem $USER@$HOST:etcd-ca.crt
scp node-1.pem $USER@$HOST:server.crt
scp node-1-key.pem $USER@$HOST:server.key
```

SSH into each node and run the below commands to move the certificates into an appropriate directory.

```bash
HOST=10.0.0.60
USER=ubuntu

ssh $USER@$HOST
sudo mkdir -p /etc/etcd
sudo mv * /etc/etcd
sudo chmod 600 /etc/etcd/server.key
```

We are done with the generation and distribution of certificates on each node. In the next step, we will create the configuration file and the Systemd unit file per each node.

### Configuring and Starting the etcd Cluster

On node 1, create a file called etcd.conf in /etc/etcd directory with the below contents:

```bash
ETCD_NAME=node-1
ETCD_LISTEN_PEER_URLS="https://10.0.0.60:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.60:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER="node-1=https://10.0.0.60:2380,node-2=https://10.0.0.61:2380,node-3=https://10.0.0.62:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.60:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.60:2379"
ETCD_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_CERT_FILE="/etc/etcd/server.crt"
ETCD_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CLIENT_CERT_AUTH=true
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CERT_FILE="/etc/etcd/server.crt"
ETCD_DATA_DIR="/var/lib/etcd"
```

For node 2, use the below content.

```bash
ETCD_NAME=node-2
ETCD_LISTEN_PEER_URLS="https://10.0.0.61:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.61:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER="node-1=https://10.0.0.60:2380,node-2=https://10.0.0.61:2380,node-3=https://10.0.0.62:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.61:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.61:2379"
ETCD_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_CERT_FILE="/etc/etcd/server.crt"
ETCD_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CLIENT_CERT_AUTH=true
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CERT_FILE="/etc/etcd/server.crt"
ETCD_DATA_DIR="/var/lib/etcd"
```

Finally, create the configuration file for the last node.

```bash
ETCD_NAME=node-3
ETCD_LISTEN_PEER_URLS="https://10.0.0.62:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.62:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER="node-1=https://10.0.0.60:2380,node-2=https://10.0.0.61:2380,node-3=https://10.0.0.62:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.62:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.62:2379"
ETCD_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_CERT_FILE="/etc/etcd/server.crt"
ETCD_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CLIENT_CERT_AUTH=true
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CERT_FILE="/etc/etcd/server.crt"
ETCD_DATA_DIR="/var/lib/etcd"
```

With the configuration in place, it’s time for us to create the systemd unit file on each node.

Create the file, etcd.service at /lib/systemd/system with the below content:

```bash
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
Type=notify
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

Since the configuration per node is moved into the dedicated file (/etc/etcd/etcd.conf), the unit file remains the same for all the nodes.

Now we mount glusterfs for external data persiostence.

```bash
sudo apt install glusterfs-client
sudo mkdir -p /var/lib/etcd
sudo vi /etc/fstab
```

Add

```bash
glusterserver1:/volname/etcd/etcd1 /var/lib/etcd glusterfs defaults,_netdev,backup-volfile-servers=glusterserver2:glusterserver3 0 0
```

We are now ready to start the service. Run the below command on each node to start the etcd cluster.

```bash
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

Make sure that the etcd service is up and running with no errors.

```bash
sudo systemctl status etcd
```

### Testing and Validating the Cluster

SSH into one of the nodes to connect to the cluster through the etcdctl CLI.

```bash
etcdctl --endpoints https://10.0.0.60:2379 --cert /etc/etcd/server.crt --cacert /etc/etcd/etcd-ca.crt --key /etc/etcd/server.key put foo bar
```

We inserted a key into the etcd database. Let’s see if we can retrieve it.

```bash
etcdctl --endpoints https://10.0.0.60:2379 --cert /etc/etcd/server.crt --cacert /etc/etcd/etcd-ca.crt --key /etc/etcd/server.key get foo
```

Next, let’s use the API endpoint to check the health of the cluster.

```bash
curl --cacert /etc/etcd/etcd-ca.crt --cert /etc/etcd/server.crt --key /etc/etcd/server.key https://10.0.0.60:2379/health
```

Finally, let’s ensure that all the nodes are participating in the cluster.

```bash
etcdctl --endpoints https://10.0.0.60:2379 --cert /etc/etcd/server.crt --cacert /etc/etcd/etcd-ca.crt --key /etc/etcd/server.key member list
```
Congratulations! You now have a secure, distributed, highly available etcd cluster that’s ready for a production-grade K3s cluster environment.

## 3) kubernetes

### cgroups

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

### Disable swap

Turn swap off:

```bash
sudo swapoff -a
```

And prevents it from turning on after reboots by commenting it in the /etc/fstab file:

```bash
sudo vi /etc/fstab
```

### Containerd

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

#### Using Docker Repository

Docker offers the containerd containerd.io package in the .deb format through the Docker repository. So, install the required packages for containerd installation.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

##### Set up Docker Repository

Then, add the Docker’s GPG key to your system.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

And then, add the Docker repository to the system by running the below command.

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

##### Install Containerd

After adding the Docker’s repository, update the repository index.

```bash
sudo apt update
```

Then, install containerd using the apt command.

```bash
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

##### Containerd configuration for Kubernetes

Containerd uses a configuration file config.toml for managing demons. For Kubernetes, you need to configure the Systemd group driver for runC.

```bash
cat <<EOF | sudo tee -a /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
EOF
```

Then, enable CRI plugins by commenting out disabled_plugins = ["cri"] in the config.toml file.

```bash
sudo sed -i 's/^disabled_plugins \=/\#disabled_plugins \=/g' /etc/containerd/config.tomlCOPY
```

Next, use the below command to restart the containerd service.

```bash
sudo systemctl restart containerd
```

### kubelet, kubeadm, kubectl

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Google Cloud public signing key:

```bash
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Add the Kubernetes apt repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

### High availability

https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

Install both keepalived and haproxy

```bash
apt install keepalived haproxy
```

#### Keepalived

##### All

```bash
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
sudo sysctl -p
```

/etc/keepalived/check_apiserver.sh

```bash
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl --silent --max-time 2 --insecure https://localhost:6443/ -o /dev/null || errorExit "Error GET https://localhost:6443/"
if ip addr | grep -q 10.0.50.64; then
    curl --silent --max-time 2 --insecure https://10.0.50.64:6443/ -o /dev/null || errorExit "Error GET https://10.0.50.64:6443/"
fi
```

##### Master

```bash
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens18
    virtual_router_id 51
    priority 101
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        10.0.50.64
    }
    track_script {
        check_apiserver
    }
}
```

##### Backup

```bash
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens18
    virtual_router_id 51
    priority 100
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        10.0.50.64
    }
    track_script {
        check_apiserver
    }
}
```

##### All

```bash
sudo service keepalived restart
```

#### Haproxy

##### All

```bash
# /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          20s
    timeout server          20s
    timeout http-keep-alive 10s
    timeout check           10s

#---------------------------------------------------------------------
# apiserver frontend which proxys to the control plane nodes
#---------------------------------------------------------------------
frontend apiserver
    bind *:6443
    mode tcp
    option tcplog
    default_backend apiserver

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server k8cp1 k8cp1.urbaman.it:6443 check
        server k8cp2 k8cp2.urbaman.it:6443 check
        server k8cp3 k8cp3.urbaman.it:6443 check
```

Restart haproxy

```bash
sudo service haproxy restart
```

And check connection from another machine

```bash
nc -v 10.0.50.64 6443
```
