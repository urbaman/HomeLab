# Firefly III installation with data importer

After having deployed postgresql with a dedicated db (eventually called firefly), create a pv and pvc, size 10Gi

Generate a random 32 character key to set as APP_KEY:

```bash
head /dev/urandom | LC_ALL=C tr -dc 'A-Za-z0-9' | head -c 32 && echo
```

Deploy the yaml file setting the APP_KEY value and the Postgresql db values.

```bash
kubectl apply -f fireflyIII.yaml
```

Login and create a Personal Access Token, and set the variables in fireflyIIIimporter.yaml, then deploy.

```bash
kubectl apply -f fireflyIIIimporter.yaml
```
