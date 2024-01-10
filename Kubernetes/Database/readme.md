# Expose DBs with Traefik

The majority of DBs (postgres, mariadb, ...)to not interact via HTTP(s), but need TCP.

- Create an appropriate entrypoint in the Traefik deployment with the desired port
- Create a simple IngressRouteTCP on that entrypoint going to the needed DB
- Be aware: you'll need TLS with SNI to recognize the host and use the same port for different DBs. Alternative: an entrypoint for each DB and ('*') as HostSNI.
