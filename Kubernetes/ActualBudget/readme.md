# Actual Busget installation

Create a volume, pv, pvc in longhorn

- name: actual-budget (-pv, -pvc)
- namespace: actual-budget
- size: 5Gi

Deploy the yaml file

```bash
kubectl apply -f actual-budget.yaml
```
