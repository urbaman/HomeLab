# Dbgate installation

## Installation

Create a namespace

```bash
kubectl create namespace cloudbeaver
```

Create a longhorn volume, pv and pvc

- name: cloudbeaver-pvc
- dimension: 5Gi
- namespace: cloudbeaver

Deploy the yaml file

```bash
kubectl apply -f cloudbeaver.yaml
```
