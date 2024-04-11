# Cert Manager (to manage SSL certs with Traefik in HA mode)

## Install Cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
```

Or via helm

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade -i cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true --set extraArgs='{--dns01-recursive-nameservers-only,--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```

Set the installCRDs=true to install the CRDs (best method)

## Let's Encrypt through Cloudflare DNS Challenge

Create a secret with Cloudflare API Token (all domains)

API Token recommended settings:

- Permissions:
  - Zone - DNS - Edit
  - Zone - Zone - Read
- Zone Resources:
  - Include - All Zones

Set the token and the email in the `cert-manager-issuer-cfdns01.yaml` file, then apply.

### Certificate creation

Let's create our wildcard certificate using the `cert.yaml` template.
