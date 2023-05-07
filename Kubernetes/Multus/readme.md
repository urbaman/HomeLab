# Multus and Whereabouts for second network in pods on second interface on nodes

## Install multus

Multus plugin allows to add a second network interface to the pods, and can be associated to a different physical interface on the nodes (you need same interface on all the nodes)

```
kubectl create -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-thick.yml
```

## Install whereabouts for different IPs in differnet nodes

Multus can't manage different IPs for pods on different nodes, leading to IP incompatibilities. Whereabouts is the solution.

```
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/whereabouts/master/doc/crds/daemonset-install.yaml
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/whereabouts/master/doc/crds/whereabouts.cni.cncf.io_ippools.yaml
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/whereabouts/master/doc/crds/whereabouts.cni.cncf.io_overlappingrangeipreservations.yaml
```

We are ready to create the Network Attached Definition for the second network (the first and default is the one for the default CNI installed - Calico for us). We chose a 10.90.x.x network, associated to eth1 interface on the nodes, with ipam l2 plugin.

```
cat <<EOF | kubectl create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
   name: ipvlan-conf
spec:
   config: '{
      "cniVersion": "0.3.0",
      "type": "ipvlan",
      "master": "eth1",
      "mode": "l2",
      "ipam": {
         "type": "whereabouts",
         "range": "10.90.0.0/24",
         "range_start": "10.90.0.101",
         "range_end": "10.90.0.250",
         "gateway": "10.90.0.1"
      }
 }'
EOF
```

## Check

To check the installation, we install two test pods on different nodes, and see if they correctly get the IPs on the new interface and if they talk to each other.

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: samplepod
  annotations:
    k8s.v1.cni.cncf.io/networks: ipvlan-conf
spec:
  containers:
  - name: samplepod
    command: ["/bin/bash", "-c", "sleep 2000000000000"]
    image: dougbtv/centos-network
  nodeName: k8w1
EOF
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: samplepod2
  annotations:
    k8s.v1.cni.cncf.io/networks: ipvlan-conf
spec:
  containers:
  - name: samplepod
    command: ["/bin/bash", "-c", "sleep 2000000000000"]
    image: dougbtv/centos-network
  nodeName: k8w2
EOF
```

```
kubectl exec -it samplepod -- ip a
```

Output:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0@if55: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8950 qdisc noqueue state UP
    link/ether e2:bb:18:78:c4:40 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.20.221.124/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::e0bb:18ff:fe78:c440/64 scope link
       valid_lft forever preferred_lft forever
4: net1@if26: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc noqueue state UNKNOWN
    link/ether b2:cf:54:9b:6a:b8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.90.0.101/24 brd 10.90.0.255 scope global net1
       valid_lft forever preferred_lft forever
    inet6 fe80::b2cf:5400:19b:6ab8/64 scope link
       valid_lft forever preferred_lft forever
```

```
kubectl exec -it samplepod2 -- ip a
```

Output:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0@if73: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8950 qdisc noqueue state UP
    link/ether d2:2d:99:0b:df:4e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.29.148.10/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::d02d:99ff:fe0b:df4e/64 scope link
       valid_lft forever preferred_lft forever
4: net1@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc noqueue state UNKNOWN
    link/ether 12:45:7a:48:ab:1c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.90.0.102/24 brd 10.90.0.255 scope global net1
       valid_lft forever preferred_lft forever
    inet6 fe80::1245:7a00:148:ab1c/64 scope link
       valid_lft forever preferred_lft forever
```

```
kubectl exec -it samplepod -- ping 10.90.0.102
```

Output:

```
PING 10.90.0.102 (10.90.0.102) 56(84) bytes of data.
64 bytes from 10.90.0.102: icmp_seq=1 ttl=64 time=0.684 ms
64 bytes from 10.90.0.102: icmp_seq=2 ttl=64 time=0.661 ms
```

```
kubectl exec -it samplepod2 -- ping 10.90.0.101
```

Output:

```
PING 10.90.0.101 (10.90.0.101) 56(84) bytes of data.
64 bytes from 10.90.0.101: icmp_seq=1 ttl=64 time=0.614 ms
64 bytes from 10.90.0.101: icmp_seq=2 ttl=64 time=0.653 ms
```
