# 1) Fichiers

#### `docker-compose.yml`

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    container_name: n8n-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
      TZ: ${TZ}
    volumes:
      - ${DATA_FOLDER}/pg_data:/var/lib/postgresql/data
    networks:
      - n8n_net

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    # Fait tourner le conteneur avec l'UID/GID 1000 (compatible avec les fichiers montés)
    user: "1000:1000"
    depends_on:
      - postgres
    ports:
      - "${N8N_PORT}:5678"
    environment:
      # Sécurité de base
      N8N_BASIC_AUTH_ACTIVE: "true"
      N8N_BASIC_AUTH_USER: ${N8N_BASIC_AUTH_USER}
      N8N_BASIC_AUTH_PASSWORD: ${N8N_BASIC_AUTH_PASSWORD}

      # Chiffrement (tokens, credentials…)
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}

      # Base de données Postgres
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: ${DB_NAME}
      DB_POSTGRESDB_USER: ${DB_USER}
      DB_POSTGRESDB_PASSWORD: ${DB_PASSWORD}
      DB_POSTGRESDB_SCHEMA: public

      # Hôte/URL (utile si reverse proxy plus tard)
      N8N_HOST: ${N8N_HOST}
      N8N_PROTOCOL: ${N8N_PROTOCOL}
      N8N_PORT: 5678
      GENERIC_TIMEZONE: ${TZ}

      # Dossier de travail interne d'n8n
      N8N_USER_FOLDER: /home/node/.n8n
    volumes:
      - ${DATA_FOLDER}/n8n_data:/home/node/.n8n
    networks:
      - n8n_net

networks:
  n8n_net:
    driver: bridge
```

#### `.env` (exemple à adapter)

```dotenv
# Dossier ABSOLU sur l'hôte pour stocker les données
DATA_FOLDER=/home/eleve/Desktop/n8n-demo1

# Port d'accès depuis l'hôte
N8N_PORT=5678

# Fuso horaire
TZ=America/Toronto

# Auth de base pour l'UI
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=ChangeThisStrongPwd

# Clé de chiffrement (32+ caractères aléatoires)
# Génère par ex. avec:  openssl rand -base64 32
N8N_ENCRYPTION_KEY=REMPPLACEZ_PAR_UNE_CLE_LONGUE_ALEATOIRE

# DB Postgres
DB_USER=n8n
DB_PASSWORD=ChangeThisDbPwd
DB_NAME=n8n

# Réglages d’hôte (utile si reverse proxy)
N8N_HOST=localhost
N8N_PROTOCOL=http
```


<br/>
<br/>

# 2) Préparation des dossiers et permissions

```bash
cd /home/eleve/Desktop/n8n-demo1

# Crée les dossiers de persistance
mkdir -p n8n_data pg_data

# Donne les droits au même UID/GID que le user du conteneur (1000:1000)
sudo chown -R 1000:1000 n8n_data pg_data
```

<br/>
<br/>


# 3) Démarrage

```bash
docker-compose pull
docker-compose up -d
docker logs -f n8n   # Ctrl+C pour quitter les logs
```

Accès: [http://localhost:5678](http://localhost:5678)


<br/>
<br/>

# 4) Notes utiles

* **Erreur EACCES**: si tu la vois encore, vérifie que `n8n_data` appartient bien à `1000:1000` et que ton `DATA_FOLDER` dans `.env` est **absolu**.
* **Mises à jour**: `docker compose pull && docker compose up -d`.
* **Arrêt**: `docker compose down` (les données persistent dans `n8n_data` et `pg_data`).

<br/>
<br/>

# Annexe



```bash
su
apt install docker-compose
docker --version
docker-compose --version
cd Desktop
mkdir n8n-demo1
cd n8n-demo1/
mkdir -p n8n_data pg_data
sudo chown -R 1000:1000 n8n_data pg_data
nano docker-compose.yml
nano .env
docker-compose pull
docker-compose up -d
docker ps
docker logs n8n
docker logs n8n-postgres
docker-compose down
```
