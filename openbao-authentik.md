# OpenBao + Authentik OIDC — Configurazione definitiva

## Architettura

```
Utente → OpenBao UI/CLI → redirect Authentik → login
       ← id_token con claim groups: ["bao-admins"] ←
OpenBao legge groups_claim → matcha alias "bao-admins"
       → gruppo interno bao-admins → policy "admin"
       → token OpenBao emesso con capabilities admin
```

| Componente | Ruolo |
|---|---|
| Authentik Property Mapping | Inietta i nomi dei gruppi nel JWT |
| OpenBao `auth/oidc` | Gestisce il flusso OIDC interattivo |
| OpenBao `identity/group` | Gruppi interni con policy associate |
| OpenBao `identity/group-alias` | Collega i gruppi Authentik a quelli interni |
| Ruolo `default` | Unico ruolo OIDC, senza bound_claims |

---

## 1. Authentik

### 1.1 Property Mapping per i gruppi

**Customization → Property Mappings → Create → Scope Mapping**

```python
# Name: groups
# Scope name: groups
return {
    "groups": [group.name for group in request.user.ak_groups.all()]
}
```

> Questo mapping inietta i **nomi** dei gruppi nel JWT. Senza di esso, Authentik invia gli UUID dei gruppi, che non corrispondono ai bound_claims di OpenBao.

### 1.2 Provider OAuth2/OpenID

**Applications → Providers → Create → OAuth2/OpenID Provider**

| Campo | Valore |
|---|---|
| Name | `openbao` |
| Client type | `Confidential` |
| Redirect URI | `https://openbao.example.com/v1/auth/oidc/oidc/callback` |
| Redirect URI | `https://openbao.example.com/ui/vault/auth/oidc/oidc/callback` |
| Redirect URI | `http://localhost:8250/oidc/callback` *(solo se usi CLI locale)* |
| Signing Key | `authentik Self-signed Certificate` |

In **Advanced Protocol Settings** aggiungere `groups` agli **Additional Scopes**.

Annotare **Client ID** e **Client Secret** generati.

### 1.3 Gruppi Authentik

Creare i gruppi che verranno usati come binding in OpenBao:

- `bao-admins` — utenti con accesso amministrativo
- `bao-readers` — utenti con accesso in sola lettura

Assegnare gli utenti ai gruppi appropriati.

### 1.4 Application

**Applications → Applications → Create**

| Campo | Valore |
|---|---|
| Name | `OpenBao` |
| Provider | `openbao` |
| Slug | `openbao` |

Aggiungere il binding ai gruppi autorizzati ad accedere all'applicazione.

### 1.5 Verifica del token

Per verificare che le claims siano corrette, usare il **Preview** del provider in Authentik. Il JWT deve contenere:

```json
{
  "groups": [
    "bao-admins",
    "bao-readers"
  ]
}
```

Se il campo `groups` contiene UUID invece di nomi, il Property Mapping non è applicato correttamente.

---

## 2. OpenBao — config.json

```json
{
  "ui": true,
  "cluster_addr": "https://127.0.0.1:8201",
  "api_addr": "https://127.0.0.1:8200",
  "log_level": "info",
  "storage": {
    "raft": {
      "path": "/openbao/file",
      "node_id": "raft_node_1"
    }
  },
  "listener": [
    {
      "tcp": {
        "address": "0.0.0.0:8200",
        "tls_disable": "true"
      }
    }
  ]
}
```

> `tls_disable: true` è accettabile se OpenBao è dietro un reverse proxy (es. Traefik) che gestisce il TLS.

---

## 3. OpenBao — Configurazione OIDC

### 3.1 Abilitare il metodo OIDC

```bash
bao auth enable oidc
```

### 3.2 Configurare il provider

```bash
bao write auth/oidc/config \
  oidc_discovery_url="https://authentik.example.com/application/o/openbao/" \
  oidc_client_id="<CLIENT_ID>" \
  oidc_client_secret="<CLIENT_SECRET>" \
  default_role="default"
```

> Il trailing slash nel discovery URL è obbligatorio con Authentik.

### 3.3 Ruolo OIDC unico

Il ruolo non usa `bound_claims` — l'assegnazione della policy avviene tramite i Group Aliases (vedi sezione 5).

```bash
bao write auth/oidc/role/default - <<EOF
{
  "bound_audiences": ["<CLIENT_ID>"],
  "allowed_redirect_uris": [
    "https://openbao.example.com/v1/auth/oidc/oidc/callback",
    "https://openbao.example.com/ui/vault/auth/oidc/oidc/callback",
    "http://localhost:8250/oidc/callback"
  ],
  "user_claim": "sub",
  "groups_claim": "groups",
  "oidc_scopes": ["openid","profile","email","groups"],
  "token_ttl": "8h"
}
EOF
```

---

## 4. OpenBao — Policy

```bash
# Policy admin — accesso totale
bao policy write admin - <<EOF
path "*" {
  capabilities = ["create","read","update","delete","list","sudo"]
}
EOF
```

```bash
# Policy reader — sola lettura su secret/
bao policy write reader - <<EOF
path "secret/data/*" {
  capabilities = ["read","list"]
}
path "secret/metadata/*" {
  capabilities = ["list"]
}
EOF
```

---

## 5. OpenBao — Identity Groups e Group Aliases

Questo è il meccanismo che assegna automaticamente la policy corretta in base al gruppo Authentik dell'utente.

### 5.1 Creare i gruppi interni

```bash
# Gruppo admin — annotare il canonical_id restituito
bao write identity/group \
  name="bao-admins" \
  type="external" \
  policies="admin"

# Gruppo reader — annotare il canonical_id restituito
bao write identity/group \
  name="bao-readers" \
  type="external" \
  policies="reader"
```

### 5.2 Recuperare l'accessor del metodo OIDC

```bash
bao auth list -detailed | grep oidc
# Copiare il valore della colonna Accessor → auth_oidc_xxxxxxxx
```

### 5.3 Creare gli alias

```bash
# Alias bao-admins
bao write identity/group-alias \
  name="bao-admins" \
  mount_accessor="<ACCESSOR>" \
  canonical_id="<CANONICAL_ID_ADMIN>"

# Alias bao-readers
bao write identity/group-alias \
  name="bao-readers" \
  mount_accessor="<ACCESSOR>" \
  canonical_id="<CANONICAL_ID_READER>"
```

---

## 6. Login

### Da UI (consigliato se OpenBao è su server remoto)

Navigare su `https://openbao.example.com/ui` e selezionare:

- **Method**: OIDC
- **Role**: lasciare vuoto (usa il `default_role`)

### Da CLI locale

Il client `bao` deve essere installato ed eseguito sul proprio PC, non sul server remoto. Il flusso OIDC da CLI richiede che il processo ascolti su `localhost:8250` per ricevere il callback.

```bash
export BAO_ADDR=https://openbao.example.com
bao login -method=oidc
```

> Se OpenBao è su un server remoto e si usa SSH o un terminale remoto, il flusso CLI non funzionerà perché il callback OIDC torna su `localhost:8250` del proprio PC, non del server. In questo caso usare la UI o installare il client `bao` localmente.

---

## 7. Debug

### Abilitare il logging OIDC verboso (temporaneo)

```bash
bao write auth/oidc/config \
  oidc_discovery_url="..." \
  oidc_client_id="..." \
  oidc_client_secret="..." \
  default_role="default" \
  verbose_oidc_logging=true
```

> Disabilitare dopo il debug — logga dati sensibili.

### Abilitare l'audit log

In `config.json`:

```json
{
  "audit": {
    "file": {
      "type": "file",
      "path": "file",
      "options": {
        "file_path": "/tmp/audit.log"
      }
    }
  }
}
```

```bash
# Seguire l'audit log in tempo reale (Docker)
docker exec -it openbao tail -f /tmp/audit.log
```

### Verificare le claims Authentik

```bash
# Userinfo endpoint — mostra le claims effettivamente emesse
curl -s -H "Authorization: Bearer <ACCESS_TOKEN>" \
  https://authentik.example.com/application/o/userinfo/
```

In alternativa usare il **Preview** del provider in Authentik UI.

### Verificare la configurazione OpenBao

```bash
bao read auth/oidc/config
bao read auth/oidc/role/default
bao list identity/group/name
bao read identity/group/name/bao-admins
```
