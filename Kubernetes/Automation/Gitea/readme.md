# Install Gitea

Create a postgres db and user, then get the helm chart and customize the values

```bash
helm repo add gitea-charts https://dl.gitea.com/charts/
helm repo update
helm show values gitea-charts/gitea > gitea-values.yaml
```

Set the metrics, serviceMonitor to true, then set the configuration ([see here](https://docs.gitea.com/administration/config-cheat-sheet))


```yaml
  config:
    #  APP_NAME: "Gitea: Git with a cup of tea"
    #  RUN_MODE: dev
    server:
      SSH_PORT: 22 # rootful image
      SSH_LISTEN_PORT: 2222 # rootless image
      ROOT_URL: https://gitea.domain.com # set to traefik domain
    database:
      DB_TYPE: postgres
      HOST: postgresql-vectorchord-175-043-rw.postgresql.svc.cluster.local:5432
      NAME: gitea
      USER: gitea
      PASSWD: "<password>"
    cache:
      ADAPTER: redis
      HOST: redis://:<password>@valkey-primary.valkey.svc.cluster.local:6379/1?pool_size=100&idle_timeout=180s
    mailer:
      ENABLED: true
      SMTP_ADDR: mail.domain.com
      SMTP_PORT: 587
      USER: user@domain.com
      PASSWD: <password>
      FROM: gitea@domain.com
    session:
      PROVIDER: redis
      PROVIDER_CONFIG: redis://:<password>@valkey-primary.valkey.svc.cluster.local:6379/2
    queue:
      TYPE: redis
      CONN_STR: redis://:<password>@valkey-primary.valkey.svc.cluster.local:6379/3
```

Then install, and finally apply the ingressRoute

```bash
helm upgrade -i gitea gitea-charts/gitea -n gitea --create-namespace -f gitea-values.yaml
kubectl apply -f ig-gitea.yaml
```
