# Install reflector

```bash
helm repo add emberstack https://emberstack.github.io/helm-charts
helm repo update
helm upgrade -i reflector emberstack/reflector --create-namespace --n reflector
```
