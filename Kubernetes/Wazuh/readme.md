# Wazuh Security Scanner

Follow their guide, but remember to change the following:

- Properly define the storageClass, copying from an existing sc for the selected provider
- Change all of the services to ClusterIP
- Change the indexer and the wazuh api URLs in the dashboard deployment to properly point to the relative services:
  - indexer.wazuh.svc.cluster.local:9200
  - wazuh.wazuh.svc.cluster.local
- Do your Traefik things to reverse proxy the dashboard, the indexer, the api and all of the other ports and services
- Deploy agents
