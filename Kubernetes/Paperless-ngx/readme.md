# Install Paperless-ngx

Create a paperless db in postgresql (or mysql/mariadb, changing the environment variables) with user paperless, define the env variables in the secret, then apply the deployment

```bash
kubectl apply -f paperless-ngx.yaml
```
