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

# 5) Commandes à exécuter :

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


<br/>

# 6) Annexe 1 - donner des droits sudo **permanents** à l’utilisateur `eleve`.

```bash
# 1) (si besoin) créer l'utilisateur et définir son mot de passe
sudo adduser eleve

# 2) ajouter l'utilisateur au groupe "sudo" (droits admin persistants)
sudo usermod -aG sudo eleve
# (équivalent) sudo adduser eleve sudo

# 3) vérifier
id eleve        # voit le groupe sudo ?
getent group sudo
sudo -v         # teste sudo (si vous êtes déjà dans une session avec sudo)
su - eleve      # se reconnecter en tant que eleve
groups          # doit afficher ... sudo
sudo -l         # liste des privilèges

# 4) (optionnel, à vos risques) sudo sans mot de passe pour eleve
# Ouvre un fichier sudoers dédié et sûr :
sudo visudo -f /etc/sudoers.d/eleve
# -> y coller la ligne suivante, puis enregistrer :
# eleve ALL=(ALL) NOPASSWD:ALL
```

### Notes rapides

* L’ajout au groupe **ne prend effet qu’à la reconnexion** de `eleve` (`su - eleve` ou nouvelle session SSH).
* Toujours modifier la configuration via `visudo` (vérifie la syntaxe).
* Pour revenir en arrière :

  ```bash
  sudo gpasswd -d eleve sudo
  sudo rm -f /etc/sudoers.d/eleve   # si vous aviez activé NOPASSWD
  ```

<br/>

# 7) Annexe 2  - Les commandes (selon la distro) pour donner **définitivement** les droits `sudo` à l’utilisateur `eleve`.

### Debian/Ubuntu & dérivés

1. (Facultatif) Créer l’utilisateur s’il n’existe pas

```bash
sudo adduser eleve
# ou en root : adduser eleve
```

2. Ajouter `eleve` au groupe `sudo` (droits admin persistants)

```bash
sudo usermod -aG sudo eleve
# alternative équivalente :
sudo adduser eleve sudo
```

3. Rafraîchir la session et vérifier

```bash
# se reconnecter ou :
su - eleve
groups
sudo -v        # teste l’accès sudo
sudo -l        # liste des privilèges
```

#### RHEL/CentOS/Rocky/Alma (et Fedora)

1. Ajouter `eleve` au groupe `wheel`

```bash
sudo usermod -aG wheel eleve
```

2. Vérifier que `wheel` a bien les droits sudo (dans /etc/sudoers)

```bash
sudo visudo
# ligne à avoir décommentée :
# %wheel ALL=(ALL) ALL
```

#### (Optionnel) Sudo **sans mot de passe** pour `eleve`

> À utiliser avec prudence (sécurité).

```bash
sudo visudo -f /etc/sudoers.d/eleve
# y mettre exactement :
eleve ALL=(ALL) NOPASSWD:ALL
```

#### Notes utiles

* **Reconnexion nécessaire** pour que l’ajout au groupe prenne effet (`su - eleve` ou nouvelle session).
* **Ne pas** éditer `/etc/sudoers` avec un éditeur classique : utilisez toujours `visudo` (vérification de syntaxe).
* Pour vérifier la présence de `sudo` (sur Debian/Ubuntu minimal) :

  ```bash
  sudo apt update && sudo apt install -y sudo
  ```


