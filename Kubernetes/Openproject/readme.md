# Install Openproject

Create a postgresql db and user.

```bash
helm repo add openproject https://charts.openproject.org
helm repo update
helm shaw values openproject/openproject > openproject-values.yaml
vi openproject-values.yaml
```

Fill the values with your custom settings (SMTP, postgresql, memcached, ...)

```bash
helm upgrade -i openproject -n openproject --create-namespace openproject/openproject -f openproject-values.yaml
kubectl apply -f ig-openproject
```
