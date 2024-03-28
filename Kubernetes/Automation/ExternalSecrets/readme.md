# ExternalSecrets deployment

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm upgrade -i external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
```

Create the ClusterSecretStore and the Vault credentials secret.

Create ExternalSecrets or PushSecrets
