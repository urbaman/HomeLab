# Stirling PDF installation

## Preparation

Create a Longhorn Volume, and the required s-pdf.domain.com DNS entry both internally (we use pfSense) and externally (we use cloudflare)

## Deploy

Run the yaml file to deploy Stirling PDF, after having changed the config file.

```bash
kubectl apply -f s-pdf.yaml
```

The first username is `admin`, password `stirling`. You'll have to change them.
