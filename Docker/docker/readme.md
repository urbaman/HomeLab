# Notes on containers

## Grafana

Remember to chown the data and config directories to your user, and run grafana with your user.

## Wazuh

Create a wazuh directory, get the git repo there, then create a directory for the indexer data and point that volume there in the compose file.
Deploy.
