# Renovate

Renovate can be used to check image updates in Docker Compose, Kubernetes and or ArgoCD (or other) manifests.

## Basic config

The basic config file `config.js` is quite simple, setting the repository to be checked for manifests (the example is for Gitea)

Requirements:

- The gitAuthor should correspond to a valid user with a working Personal Access Token being collaborator of the repository

- The Github PAT is needed to get the changelogs from github if the project have them

For Kubernetes and or ArgoCD you need to add `managerFilePatterns` for the managers pointing to the desired files to scrape (you can set more fine-grained regexes if needed, maximizing performances)

For Docker Compose the default is the following, add and change it if you need

```json
{
  "managerFilePatterns": [
    "/(^|/)(?:docker-)?compose[^/]*\\.ya?ml$/"
  ]
}
```

```json
module.exports = {
  "gitAuthor": "Amministratore Gitea <dockeradmin@urbaman.it>",
  "endpoint": "https://gitea.urbaman.it",
  "platform": "gitea",
  "token": "<token>",
  "hostRules": [
    {
      "token": "<gityhub-pat>",
      "matchHost": "github.com"
    }
  ],
  "repositories": [
    'admin/prova-renovate'
  ],
  "prConcurrentLimit": 0,
  "prHourlyLimit": 0,
  "enabledManagers": ["kubernetes", "argocd", "docker-compose"],
  "pinDigests": true,
  "argocd": {
    "managerFilePatterns": ["/argocd/.+\\.yaml$/"]
  },
  "kubernetes": {
    "managerFilePatterns": ["/kubernetes/.+\\.yaml$/"]
  }
}
```

The file has to be added to the container in the `/usr/src/app/config.js` path

On the root of the repo, add the following `renovate.json5` file

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json"
}
```

## Running the container

When done configuring, you can run the container

### Docker

```bash
docker run --rm -it -v $PATH_TO_CONFIG_JS/config.js:/usr/src/app/config.js -e LOG_LEVEL=debug renovate/renovate
```

You can also add it to a cronjob to schedule it

### Kubernetes

You can define a secret, a configmap and add them to the container in the cronjob

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: renovate-env
type: Opaque
stringData:
  RENOVATE_GITHUB_COM_TOKEN: 'any-personal-user-token-for-github-com-for-fetching-changelogs'
  RENOVATE_ENDPOINT: 'https://gitea.domain.com'
  RENOVATE_GIT_AUTHOR: 'Renovate Bot <bot@renovateapp.com>'
  RENOVATE_PLATFORM: 'gitea'
  RENOVATE_TOKEN: '<git platform Personal Acccess Token>'
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: renovate-config
data:
  config.json: |-
    module.exports = {
      "repositories": [
        'user/repo'
      ],
      "enabledManagers": ["kubernetes", "argocd", "docker-compose"],
      "pinDigests": true,
      "argocd": {
        "managerFilePatterns": ["/argocd/.+\\.yaml$/"]
      },
      "kubernetes": {
        "managerFilePatterns": ["/kubernetes/.+\\.yaml$/"]
      }
    }

---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: renovate-bot
spec:
  schedule: '@hourly'
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - image: renovate/renovate:42.71.0 # check the latest version or use latest tag
              name: renovate-bot # For illustration purposes, please use secrets.
              envFrom:
                - secretRef:
                    name: renovate-env
              env:
                - name: LOG_LEVEL
                  value: debug
                - name: RENOVATE_CONFIG_FILE
                  value: '/opt/renovate/config.json'
              volumeMounts:
                - name: config-volume
                  mountPath: /opt/renovate/
          restartPolicy: Never
          volumes:
            - name: config-volume
              configMap:
                name: renovate-config
```
