# Install roundcube webmail

Create a roundcube db in postgresql (or in Mysql) and change the db settings in the deployment and vars in the secret, then deploy the yaml.

```bash
echo -n '<db username>' | base64
echo -n '<db password>' | base64
vi roundcube.yaml
kubectl apply -f roundcube.yaml
```
