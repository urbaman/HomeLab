# Semaphore

## External behind Traefik

```bash
kubectl apply -f ig-semaphore.yaml
```

## Deploy in the Cluster

Create a MySQL/MariaDB db and user (semaphore/semaphore)

Create a key (for the SEMAPHORE_ACCESS_KEY_ENCRYPTION environment variable)

```bash
head -c32 /dev/urandom | base64
```

Deploy

semaphore:
    restart: unless-stopped
    ports:
      - 3000:3000
    image: semaphoreui/semaphore:latest
    environment:
      SEMAPHORE_DB_USER: semaphore
      SEMAPHORE_DB_PASS: <semaphore>
      SEMAPHORE_DB_HOST: mysql
      SEMAPHORE_DB_PORT: 3306
      SEMAPHORE_DB_DIALECT: mysql
      SEMAPHORE_DB: semaphore
      SEMAPHORE_PLAYBOOK_PATH: /tmp/semaphore/
      SEMAPHORE_ADMIN_PASSWORD: <changeme>
      SEMAPHORE_ADMIN_NAME: admin
      SEMAPHORE_ADMIN_EMAIL: admin@localhost
      SEMAPHORE_ADMIN: admin
      SEMAPHORE_ACCESS_KEY_ENCRYPTION: <encryptionKey>
      SEMAPHORE_LDAP_ACTIVATED: 'no'
      TZ: Europe/Rome
      SEMAPHORE_EMAIL_ALERT: true
      SEMAPHORE_EMAIL_SENDER: sender@domain.com
      SEMAPHORE_EMAIL_HOST: host@domain.com
      SEMAPHORE_EMAIL_PORT: 465
      SEMAPHORE_EMAIL_USERNAME: username@domain.com
      SEMAPHORE_EMAIL_PASSWORD: <passoword>
      SEMAPHORE_EMAIL_SECURE: true
