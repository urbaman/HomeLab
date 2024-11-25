# Install Shlink

Get a Geolite license key, setup a redis and a postgresql server, and a postgresql dedicated user and db; setup the values in the secret (base64) and in the configmap.

```bash
echo -n '<DB_PASSWORD>' | base64
echo -n '<GEOLITE_LICENSE_KEY>' | base64
echo -n '<REDIS_SERVERS>' | base64
vi shlink.yaml
kubectl apply -f shlink.yaml
```

Generate an API key, put it in the shlink-web-client secret in base64, then deploy shlink-web-client.

```bash
kubectl exec -ti -n shlink shlink-5846b5d888-jn6zw -- shlink api-key:generate
echo -n '<SHLINK_SERVER_API_KEY>' | base64
vi shlink-web-client.yaml
kubectl apply -f shlink-web-client.yaml
```
