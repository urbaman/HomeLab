# ExternalSecrets deployment

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install external-secrets \
   external-secrets/external-secrets \
    -n external-secrets \
    --create-namespace
```

Create che ClusterSecretStore and the Vault credentials secret.

Create ExternalSecrets or PushSecrets
