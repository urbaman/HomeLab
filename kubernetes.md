# Kubernetes

- 3x controllers
- 3x (minimum) workers
- glusterfs volume

## Kubernetes

### High availability

https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

### Ecternal Etcd

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/

## Glusterfs

https://itnext.io/kubernetes-storage-part-2-glusterfs-complete-tutorial-77542c12a602

# Keepalived

## All

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

## Master

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

## Backup

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
