# Nextcloud

## What we need

To properly install nextcloud, we will need the following:

- a DNS entry, reachable from the outside
- a MySQL database ([we will use mariadb](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mariadb)) or a [PostgreSQL database](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql) with a ready database and user (nextcloud, nextcloud)
- [a Redis instance](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)
- [a ClamAV instance](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ClamAV)
- [Eventually, Collabora Online already up and running](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Collabora)

After having set up all of the prerequisites, we start our process.

## Secret

We start by getting the base64 version of our strings. Get the Redis password from the Redis deployment.

```bash
echo -n 'admin-password' | base64
echo -n 'dbname' | base64
echo -n 'dbusername' | base64
echo -n 'mysql-password' | base64
echo -n 'redis-password' | base64
echo -n 'smtp-password' | base64
```

## Nextcloud deployment

We now deploy nextcloud, after having setup the various env variables (DB and REDIS HOSTS, SMTP HOST, ...).

```bash
kubectl apply -f nextcloud-deploy.yaml
```

Log in to the nfs server and set www-data:www-data the owner of the created data volume (no need for the config one). Probably not needed when installing.

Login and setup

- ClamAV: [https://docs.nextcloud.com/server/22/admin_manual/configuration_server/antivirus_configuration.html](https://docs.nextcloud.com/server/22/admin_manual/configuration_server/antivirus_configuration.html).
- Collabora Online.
- Mail: add the following in the config/config.php file in the pvc (should be accessible if used NFS or CephFS)

```bash
'mail_smtpstreamoptions' =>
  array (
    'ssl' =>
    array (
      'allow_self_signed' => true,
      'verify_peer' => false,
      'verify_peer_name' => false,
    ),
  ),
```

## Lauch occ cli commands from inside the pod

```bash
kubectl exec -ti -n nextcloud nextcloud-6d556d7f46-2qf7w -- /bin/bash
su -s /bin/bash www-data
```

launch the commands
