# Karakeep installation

Generate two random strings and encode them in base64 with

```bash
openssl rand -base64 36 | base64
```

Put them in the `NEXTAUTH_SECRET` and `MEILI_MASTER_KEY` env values in the secret, then apply the deployment

```bash
kubectl apply -f karakeep.yaml
```
