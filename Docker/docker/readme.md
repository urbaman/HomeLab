# Notes on containers

## Grafana

Remember to chown the data and config directories to your user, and run grafana with your user.

## Wazuh

Create the needed configs and data directories, give them 775 permissions and set the running user as owner.
Create the following directories in the config directory:

```bash
wazuh_api_configuration
wazuh_etc
wazuh_logs
wazuh_queue
wazuh_var_multigroups
wazuh_integrations
wazuh_active_response
wazuh_agentless
wazuh_wodles
filebeat_etc
filebeat_var
wazuh-dashboard-config
wazuh-dashboard-custom
```

And the following one in the data directory:

```bash
wazuh-indexer-data
```

Get the relevant version git repo [following the documentation](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html), then copy all of the single-node (or multi-node) config directory content to the wazuh config directory

Move the two compose files where relevant, point the different volumes to the relevant directories, for example:

```bash
volumes:
  wazuh_api_configuration:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: $CONFIGDIR/wazuh/wazuh_api_configuration
```

Make other relevant changes to the compose file, like adding networks or the like.
Follow the documentation for certs creation, and deploy. See that all directory permissions are are ok.

Finally, follow the [documentation to change the passwords](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html#change-pwd-existing-usr).

## Crowdsec

Just start the container, then add the plugin and the middleware to traefik.

Add a whitelist for ips and cidrs, add a mail notification.
