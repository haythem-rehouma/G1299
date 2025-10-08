# 1) Pré-requis

* Docker Engine + Docker Compose v2 (`docker compose version`)
* Un dossier de travail vide sur ta machine/VM (ex. `/opt/n8n`)
* (Prod) Un **nom de domaine** pointant vers l’IP de ta VM et les ports **80/443** ouverts

> VM Azure : ouvre les ports dans le **Network Security Group (NSG)** de ta NIC/VM (80/443 pour HTTPS avec proxy, ou 5678 si tu exposes n8n directement, ce qui est déconseillé en prod).



# 2) Variante A — Installation rapide (SQLite, sans domaine)

Idéal pour un essai local ou un POC.

## a) Arborescence

```
/opt/n8n
├─ .env
└─ docker-compose.yml
```

## b) `.env` (variables ici)

```bash
# Timezone
TZ=America/Toronto

# Port exposé sur l’hôte
N8N_PORT=5678

# Sécurité (génère une clé aléatoire)
# Linux/macOS : openssl rand -hex 32
# Windows PowerShell : -join ((65..90)+(97..122)+(48..57) | Get-Random -Count 64 | % {[char]$_})
N8N_ENCRYPTION_KEY=remplace_moi_par_64_hexa

# Optionnel : URL publique si tu appelles des webhooks depuis l’extérieur
# WEBHOOK_URL=https://mon-ip-publique-ou-domaine:5678/
```

## c) `docker-compose.yml`

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "${N8N_PORT}:5678"
    environment:
      - TZ=${TZ}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      # Décommente si tu utilises des webhooks depuis l’extérieur
      # - WEBHOOK_URL=${WEBHOOK_URL}
      # Pour restreindre l’UI (facultatif) :
      # - N8N_BASIC_AUTH_ACTIVE=true
      # - N8N_BASIC_AUTH_USER=admin
      # - N8N_BASIC_AUTH_PASSWORD=motdepassefort
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
```

## d) Lancer

```bash
cd /opt/n8n
docker compose up -d
docker compose logs -f n8n
```

Ouvre `http://<IP>:5678` puis crée le compte admin.



# 3) Variante B — Production (PostgreSQL + Traefik + HTTPS)

Recommandée pour un usage sérieux : base PostgreSQL, domaine, Let’s Encrypt automatique.

## a) `.env`

```bash
# Domaine public pointant vers ta VM
N8N_HOST=automations.example.com
N8N_PROTOCOL=https
TZ=America/Toronto

# Email pour Let's Encrypt
LETSENCRYPT_EMAIL=toi@example.com

# Clé d’encryption n8n
N8N_ENCRYPTION_KEY=remplace_moi_par_64_hexa

# Postgres
POSTGRES_USER=n8n
POSTGRES_PASSWORD=mot_de_passe_pg_solide
POSTGRES_DB=n8n

# (Facultatif) Taille des uploads (Traefik)
TRAEFIK_BODY_LIMIT=50m
```

## b) `docker-compose.yml`

```yaml
networks:
  web:
    external: false

volumes:
  n8n_data:
  pg_data:
  traefik_letsencrypt:

services:
  traefik:
    image: traefik:v3.1
    restart: unless-stopped
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.le.acme.tlschallenge=true"
      - "--certificatesresolvers.le.acme.email=${LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.le.acme.storage=/letsencrypt/acme.json"
      # Limite corps requête (facultatif)
      - "--serversTransport.maxIdleConnsPerHost=32"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "traefik_letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    networks:
      - web

  postgres:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      TZ: ${TZ}
    volumes:
      - pg_data:/var/lib/postgresql/data
    networks:
      - web

  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    depends_on:
      - postgres
      - traefik
    environment:
      TZ: ${TZ}
      N8N_HOST: ${N8N_HOST}
      N8N_PORT: 5678
      N8N_PROTOCOL: ${N8N_PROTOCOL}
      # Base de données Postgres
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
      DB_POSTGRESDB_USER: ${POSTGRES_USER}
      DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
      # Sécurité
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
      # URL publique (webhooks, OAuth)
      WEBHOOK_URL: https://${N8N_HOST}/
      # (Optionnel) Auth basique sur l'UI
      # N8N_BASIC_AUTH_ACTIVE: "true"
      # N8N_BASIC_AUTH_USER: admin
      # N8N_BASIC_AUTH_PASSWORD: motdepassefort
    volumes:
      - n8n_data:/home/node/.n8n
    labels:
      - "traefik.enable=true"
      # HTTP -> HTTPS
      - "traefik.http.routers.n8n-web.rule=Host(`${N8N_HOST}`)"
      - "traefik.http.routers.n8n-web.entrypoints=web"
      - "traefik.http.routers.n8n-web.middlewares=n8n-redirect"
      - "traefik.http.middlewares.n8n-redirect.redirectscheme.scheme=https"

      # HTTPS
      - "traefik.http.routers.n8n-secure.rule=Host(`${N8N_HOST}`)"
      - "traefik.http.routers.n8n-secure.entrypoints=websecure"
      - "traefik.http.routers.n8n-secure.tls.certresolver=le"
      - "traefik.http.services.n8n.loadbalancer.server.port=5678"

      # (Facultatif) limite corps requête
      - "traefik.http.routers.n8n-secure.middlewares=n8n-size"
      - "traefik.http.middlewares.n8n-size.buffering.maxrequestbodybytes=${TRAEFIK_BODY_LIMIT}"
    networks:
      - web
```

## c) Lancer

```bash
cd /opt/n8n
docker compose up -d
docker compose logs -f traefik
docker compose logs -f n8n
```

Attends l’obtention du certificat, puis ouvre `https://automations.example.com`.



# 4) Où mettre les variables ?

* **Toujours dans le fichier `.env`** à la racine du projet (même dossier que `docker-compose.yml`).
* Tu y mets **toutes** tes valeurs sensibles (mots de passe, clés, domaine, e-mail), puis tu les références dans le `docker-compose.yml` via `${VARIABLE}`.

> Astuce sécurité : garde le `.env` **hors** d’un repo public (ajoute-le à `.gitignore`).



# 5) Démarrage automatique au reboot (optionnel)

Crée un service systemd minimal pour relancer le stack au démarrage :

```bash
sudo tee /etc/systemd/system/n8n.service >/dev/null <<'EOF'
[Unit]
Description=n8n docker compose stack
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/opt/n8n
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
```



# 6) Notes utiles & bonnes pratiques

* **Versionner l’image** : remplace `n8nio/n8n:latest` par une version fixe (ex. `n8nio/n8n:1.76.1`) pour éviter les surprises.
* **Backups** : sauvegarde `n8n_data` et (en prod) `pg_data`.
* **Reverse proxy** : préfère **Traefik**/Nginx + HTTPS en prod, évite d’exposer 5678 sur Internet brut.
* **Mises à jour** : `docker compose pull && docker compose up -d`.
* **Azure** : pense au NSG (80/443) et, si tu utilises un IP public, configure un **A record** `automations.example.com → IP VM`.


