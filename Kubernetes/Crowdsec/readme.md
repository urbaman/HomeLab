# Install Crowdsec for Traefik

## Install Crowdsec

Go to [Crowdsec](https://www.crowdsec.net/), register and add an instance, copy the Enroll Key.

```bash
helm repo add crowdsec https://crowdsecurity.github.io/helm-charts
helm repo update
helm show values crowdsec/crowdsec > crowdsec-values.yaml
```

Change the Enroll Key to the one created above in `crowdsec-values.yaml` file, set IPs to whitelist in `config.parsers.s02-enrich` and comment the `DISABLE_PARSERS` env variable for the agent to whitelist internal IPs

```bash
helm upgrade -i crowdsec crowdsec/crowdsec -f crowdsec-values.yaml -n crowdsec --create-namespace
```

## Add the bouncer

Create a bouncer API key

```bash
kubectl -n crowdsec exec -it crowdsec-lapi-6d686984d4-gbw6p -- sh
cscli bouncers add traefik-bouncer
```

```bash
helm show values crowdsec/crowdsec-traefik-bouncer > crowdsec-traefik-bouncer-values.yaml
vi crowdsec-traefik-bouncer-values.yaml
```

Set the api key and the crowdsec service dns (good for the default) in the `crowdsec-traefik-bouncer-values.yaml` file

```bash
helm upgrade -i -n traefik traefik-bouncer crowdsec/crowdsec-traefik-bouncer -f crowdsec-traefik-bouncer-values.yaml
```

**Note**: You'll probably need to clone the repo and install helm from local version

Then, follow the instructions to add the bouncer globally or service specific

## Managing decisions

Enter the LAPI container, list and delete the decisions

```bash
kubectl -n crowdsec exec -it crowdsec-lapi-fc44857bf-8ln6j -- sh
cscli decisions list
cscli decisions delete --ip <IP>
```
