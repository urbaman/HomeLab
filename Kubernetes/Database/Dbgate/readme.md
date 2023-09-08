# Dbgate installation

## Installation

Create a namespace

```bash
kubectl create namespace dbgate
```

Create a longhorn volume, pv and pvc

- name: dbgate-pvc
- dimension: 5Gi
- namespace: dbgate

Deploy the yaml file

```bash
kubectl apply -f dbgate.yaml
```
