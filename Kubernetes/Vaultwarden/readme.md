# Installing Vaultwarden

## Basic Installation

Set the ADMIN_TOKEN:

```bash
openssl rand -base64 48
```

Set the proper variable:

```bash
ADMIN_TOKEN=some_random_token_as_per_above_explanation
```

Use the given `vaultwarden-deploy.yaml` file, then go through the admin page and settings with the above token.

Default variables (be carefull to make numbers and boolean to strings):

```bash
DOMAIN=https://vaultwarden.domain.tld
TZ="Europe/Rome"
SIGNUPS_ALLOWED=false
INVITATIONS_ALLOWED=false
SHOW_PASSWORD_HINT=false
SMTP_HOST=<smtp.domain.tld>
SMTP_FROM=<vaultwarden@domain.tld>
SMTP_PORT=465
SMTP_SECURITY=force_tls
SMTP_USERNAME=<username>
SMTP_PASSWORD=<password>
SMTP_ACCEPT_INVALID_CERTS=false
```

## Use MariaDB

Create db and user, set the following variables:

```bash
DATABASE_URL='mysql://<vaultwarden_user>:<vaultwarden_pw>@mysql/vaultwarden'
ENABLE_DB_WAL='false'
```

## Use PostgtreSQL

Create db and user, set the following variables:

```bash
DATABASE_URL=postgresql://[[user]:[password]@]host[:port][/database]
ENABLE_DB_WAL='false'
```

## [Enabling Mobile Client push notification](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-Mobile-Client-push-notification)

Go to the Bitwarden URL to get the keys, we choose EU servers.

```bash
environment:
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID= <ID>
      - PUSH_INSTALLATION_KEY= <KEY>
      - PUSH_RELAY_URI=https://api.bitwarden.eu # For EU only
      - PUSH_IDENTITY_URI=https://identity.bitwarden.eu # For EU only
```
