# Hashicorp Vault deployment

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade -i --namespace vault --create-namespace vault hashicorp/vault --values hashicorp-vault-values.yaml
```
