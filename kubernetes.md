# Kubernetes

## Cluster structure

- 3x controllers: k8cp1, k8cp2, k8cp3
- 3x workers: k8w1, k8w2, k8w3
- 3x etcd cluster: etcd1, etce2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3
- 3x glusterfs cluster: truenas1, truenas2, truenas3

## 2) Ecternal Etcd

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/

## 1) Glusterfs

https://itnext.io/kubernetes-storage-part-2-glusterfs-complete-tutorial-77542c12a602

## 2) Ecternal Etcd

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/

## 3) kubernetes

...

## 4) High availability

https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

### Keepalived

#### All

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

#### Master

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

#### Backup

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

### Haproxy

#### All

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
