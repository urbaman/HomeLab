# Composecraft deployment

## Prerequisites

You need a MongoDB up and running

## Deployment

Get the root MongoDB password and define the mongodb-uri (mongodb://root:password@mongodb.mongodb.svc.cluster.local), then generate a random 64 length secret key, and put them in the secret value

```bash
export LC_CTYPE=C; cat /dev/urandom | tr -dc 'a-zA-Z0-9' | head -c 64; echo
echo -n '<secret-key-64>' | base64
echo -n '<mongodb-uri>' | base64
echo -n 'true' | base64
echo -n 'http://localhot:3000' | base64
```

```bash
kubectl apply -f composecraft.yaml
```
