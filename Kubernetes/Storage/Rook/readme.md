# Rook with Proxmox Ceph

## Prepare the environment

1. Give access to both Ceph networks (Public and Cluster) to the K8s nodes
2. From one of the nodes, run the following command

```bash
wget https://raw.githubusercontent.com/rook/rook/release-1.12/deploy/examples/create-external-cluster-resources.py -O create-external-cluster-resources.py
python3 create-external-cluster-resources.py --rbd-data-pool-name Ceph-NVMe2TB --namespace rook-ceph-external --format bash
```

Copy the output

```bash
export NAMESPACE=rook-ceph-external
export ROOK_EXTERNAL_FSID=<ID>
export ROOK_EXTERNAL_USERNAME=client.healthchecker
export ROOK_EXTERNAL_CEPH_MON_DATA=pvenode1=<IP>:6789
export ROOK_EXTERNAL_USER_SECRET=<secret>
export CSI_RBD_NODE_SECRET=<secret>
export CSI_RBD_NODE_SECRET_NAME=csi-rbd-node
export CSI_RBD_PROVISIONER_SECRET=<secret>
export CSI_RBD_PROVISIONER_SECRET_NAME=csi-rbd-provisioner
export CEPHFS_POOL_NAME=<cephfs_pool_data_name>
export CEPHFS_METADATA_POOL_NAME=<cephfs_pool_metadata_name>
export CEPHFS_FS_NAME=C<ceph_pool_name>
export CSI_CEPHFS_NODE_SECRET=<secret>
export CSI_CEPHFS_PROVISIONER_SECRET=<secret>
export CSI_CEPHFS_NODE_SECRET_NAME=csi-cephfs-node
export CSI_CEPHFS_PROVISIONER_SECRET_NAME=csi-cephfs-provisioner
export MONITORING_ENDPOINT=<IP>
export MONITORING_ENDPOINT_PORT=9283
export RBD_POOL_NAME=<secret>
export RGW_POOL_PREFIX=default
```

## Deployment

1. Paste the previous output on the control plane
2. Run the import script (beware of the version in the URL)

```bash
wget https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/import-external-cluster.sh -O import-external-cluster.sh
. import-external-cluster.sh
```

3. Deploy the manifests (check the documentation for changes). **Note**: beware of the version in the URL

```bash
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/common.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/operator.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/common-external.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/cluster-external.yaml
```

4. Create the storage classes, changing names and pools to your Proxmox Ceph pools

```bash
kubectl apply -f rook-sc-nvme1tb.yaml
kubectl apply -f rook-sc-nvme2tb.yaml
kubectl apply -f rook-sc-sas900gb.yaml
```

5. Check the storage classes, changing the pvc storage classes in the following yaml files

```bash
kubectl apply -f rook-mysql.yaml
kubectl apply -f rook-wordpress.yaml
```
