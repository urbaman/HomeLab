# Install Cyberchef

```bash
helm repo add cyberchef https://tamcore.github.io/cyberchef
helm repo update
helm upgrade -i -n cyberchef --create-namespace cyberchef cyberchef/cyberchef
kubectl apply -f ig-cyberchef.yaml
```
