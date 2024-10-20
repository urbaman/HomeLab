# Cloudbeaver installation

## Installation

Create and substitute the Environment Variables in the secret:

```bash
echo -n '<Server Name>' | base64
echo -n '<Admin Username>' | base64
echo -n '<Admin Password>' | base64
echo -n '<Server URL: https://cloudbeaver.domain.com>' | base64
```

Deploy the proposed yaml after checking the domains and other specs

```bash
kubectl apply -f cloudbeaver.yaml
```
