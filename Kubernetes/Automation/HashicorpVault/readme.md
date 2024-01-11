# Hashicorp Vault deployment

## Standalone

See the `hashicorp-vault-values.yaml` and `ig-hashicorp-vault.yaml` files to customize, then install.

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade -i --namespace vault --create-namespace vault hashicorp/vault --values hashicorp-vault-values.yaml
kubectl apply -f ig-hashicorp-vault.yaml
```

To initialise the vault first open a terminal on the container using:

```bash
kubectl exec -it --stdin=true --tty=true vault-0 /bin/ash
```

Now we need to turn off TLS verification as we are accessing vault on localhost, do this by setting up VAULT_SKIP_VERIFY environment variable:

```bash
export VAULT_SKIP_VERIFY=1
vault operator init
```

Make sure you save the unseal keys and root token somewhere safe and then unseal this vault by running `vault operator unseal` 3 times and providing 3 of the unseal keys.

## High Availability (not working yet)

See the `hashicorp-vault-values-ha.yaml` and `ig-hashicorp-vault-ha.yaml` files to customize, then install.

```bash
kubectl create namespace vault
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade -i --namespace vault --create-namespace vault hashicorp/vault --values hashicorp-vault-values-ha.yaml
kubectl apply -f ig-hashicorp-vault-ha.yaml
```

To initialise one of the vaults first open a terminal on the container using:

```bash
kubectl exec -it --stdin=true --tty=true -n vault vault-0 /bin/ash
```

Now we need to turn off TLS verification as we are accessing vault on localhost, do this by setting up VAULT_SKIP_VERIFY environment variable:

```bash
export VAULT_SKIP_VERIFY=1
vault operator init
```

Make sure you save the unseal keys and root token somewhere safe and then unseal this vault by running `vault operator unseal` 3 times and providing 3 of the unseal keys. Then ssh onto the other two vault containers using the `kubectl exec` command given above but changing the pod name to `vault-1` and `vault-2`, and repeat.
