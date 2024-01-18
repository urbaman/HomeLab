# Nextcloud

## What we need

To properly install nextcloud, we will need the following:

- a DNS entry, reachable from the outside
- a MySQL database (we will use mariadb) with a ready database and user
- a Redis instance
- a clamav instance [(https://github.com/wiremind/wiremind-helm-charts/tree/main/charts/clamav)](https://github.com/wiremind/wiremind-helm-charts/tree/main/charts/clamav) and [https://docs.nextcloud.com/server/22/admin_manual/configuration_server/antivirus_configuration.html](https://docs.nextcloud.com/server/22/admin_manual/configuration_server/antivirus_configuration.html)

After having set up all of the prerequisites, we start our process.

## Secret

We start by getting the base64 version of our strings. Get the Redis password from the Redis deployment.

```bash
echo 'admin-password' | base64
echo 'dbname' | base64
echo 'dbusername' | base64
echo 'mysql-password' | base64
echo 'redis-password' | base64
echo 'smtp-password' | base64
```

## Nextcloud deployment

We now deploy nextcloud, after having setup the various env variables (DB and REDIS HOSTS, SMTP HOST, ...).

```bash
kubectl apply -f nextcloud-deploy.yaml
```
