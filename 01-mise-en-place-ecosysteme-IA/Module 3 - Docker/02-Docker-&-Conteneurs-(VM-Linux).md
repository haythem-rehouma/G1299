# Évaluation sommative — Docker & Conteneurs (VM Linux)

**Durée :** 2 h 15  •  **Total :** 100 points  •  **Barème de réussite :** ≥ 60/100
**Modalité :** en laboratoire (sur VM Linux) + remise d’un dossier `.zip` sur Moodle / Teams
**Ressources autorisées :** aide‑mémoire personnel (papier) + `man`, `--help`. Internet **non** autorisé sauf documentation locale et `--help`.

<br/>

## Objectifs évalués

1. Installation et vérification de Docker sur VM Linux.
2. Maîtrise des commandes de base (ps, images, run, exec, stop, rm, rmi…).
3. Gestion multi‑conteneurs (Nginx, ports, lifecycle).
4. Construction d’images (Dockerfile) et déploiement d’une app Flask.
5. Orchestration locale avec Docker Compose (WordPress + DB).
6. Opérations d’images (tag, push, save, load).
7. Réseaux & volumes Docker (bridge, bind, named).
8. Notions d’orchestration (pourquoi Kubernetes en IA).
9. Mise en œuvre d’une app tierce : **n8n** (bonus : deepseek/llm en conteneur).



## Consignes générales

* Créez un dossier de travail **`/home/<user>/eval-docker-<NomPrenom>`**.
* Tout ce que vous créez doit persister dans ce dossier (codes, compose, exports).
* **Journalisez** vos étapes dans un fichier `LOG.md` (commandes exécutées + captures `docker ps`, etc.).
* À la fin, compressez **uniquement** ce dossier en `eval-docker-<NomPrenom>.zip`.

> **Anti-plagiat.** Deux variantes existent (**Version A** et **Version B**). Le surveillant vous attribue votre version. Ne mélangez pas les ports/variables.



## Partie 0 — Pré‑check VM (non notée, mais exigée)

* Montrez `cat /etc/os-release` et `ip a` dans `LOG.md` pour prouver l’environnement et l’IP.
* Montrez `docker --version` et `docker compose version` (ou `docker-compose --version`).

<br/>

## Partie 1 — Commandes de base (15 pts)

**Version A (ports 8080/8081/8082)**
**Version B (ports 8083/8084/8085)**

1. Lancez **3 conteneurs Nginx** détachés, nommés `c1`, `c2`, `c3` mappés sur les ports de votre version. (6 pts)
   *Ex. A :* `-p 8080:80`, `-p 8081:80`, `-p 8082:80`.
2. Vérifiez l’accessibilité avec `curl` (dans `LOG.md`, montrez la sortie de `curl localhost:<port>`). (3 pts)
3. Démontrez l’usage de : `docker ps`, `docker ps -a`, `docker stop`, `docker start`, `docker rm`. (3 pts)
4. Nettoyez tous les conteneurs **arrêtés** sans supprimer les images. (3 pts)

> **Point d’attention :** n’utilisez `rm -f` que si nécessaire ; privilégiez un arrêt propre.

<br/>

## Partie 2 — Gestion des images & nettoyage (10 pts)

1. Listez les images présentes (`docker images`). (2 pts)
2. Supprimez **uniquement** les images non utilisées (dangling) et montrez la commande employée. (4 pts)
3. Exportez l’image officielle `hello-world` :

   * `docker pull hello-world`
   * `docker save hello-world -o images/hello-world.tar` (créez le dossier `images/`). (4 pts)

<br/>

## Partie 3 — Dockerfile + Flask (20 pts)

Créez le dossier `flask-app/` avec **ces deux fichiers** :

**`Dockerfile`**

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py ./
EXPOSE 8080
CMD ["python", "app.py"]
```

**`requirements.txt`**

```
flask==3.0.3
```

**`app.py`**

```python
from flask import Flask
app = Flask(__name__)

@app.get("/")
def index():
    return "Hello from Dockerized Flask!"

@app.get("/health")
def health():
    return {"status": "ok"}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080, debug=False)
```

Exigences :

1. Construisez l’image `my-flask-image:<votre_version>` (ex. `v1`). (6 pts)
2. Lancez un conteneur `my-flask-container` mappé :

   * **Version A :** `-p 9000:8080`
   * **Version B :** `-p 9001:8080`
     Testez avec `curl` et capturez la sortie dans `LOG.md`. (6 pts)
3. Montrez l’usage de `docker exec -it` pour vérifier la présence de `app.py` dans `/app`. (4 pts)
4. Exposez proprement l’état `/health` et joignez la réponse JSON dans `LOG.md`. (4 pts)

<br/>

## Partie 4 — Docker Compose (WordPress + DB) (20 pts)

Dans `compose-wp/`, créez :

**`docker-compose.yml`**

```yaml
version: "3.8"
services:
  db:
    image: mysql:5.7
    container_name: wp-db
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - db_data:/var/lib/mysql
    restart: unless-stopped

  wordpress:
    image: wordpress:latest
    container_name: wp-app
    depends_on:
      - db
    ports:
      - "8000:80"  # ne changez pas ce port
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    restart: unless-stopped

volumes:
  db_data:
```

Tâches :

1. Démarrez en arrière‑plan (`docker compose up -d`). (4 pts)
2. Prouvez le fonctionnement (`docker ps`, capture d’écran ou `curl localhost:8000`). (6 pts)
3. Expliquez (dans `LOG.md`) la différence entre **`CTRL+C`** (arrêt sans suppression) et **`docker compose down`** (arrêt avec suppression). (4 pts)
4. Détruisez proprement l’environnement puis **relancez**‑le. (6 pts)

<br/>

## Partie 5 — Réseaux & Volumes (10 pts)

1. Créez un réseau bridge **nommé** `lab_net` et relancez un Nginx attaché à ce réseau (`--network lab_net`). (4 pts)
2. Montez un **volume nommé** `webdata` sur `/usr/share/nginx/html`. Placez un fichier `index.html` personnalisé et prouvez qu’il est servi. (6 pts)

<br/>

## Partie 6 — Registry & cycle de vie d’image (10 pts)

> Si vous ne disposez pas de compte Docker Hub en évaluation fermée, **simulez** l’étape push en montrant les commandes **avec** un `docker tag` correct et documentez la raison du skip dans `LOG.md`.

1. `docker tag my-flask-image:v1 <votre_namespace>/my-flask-image:v1` (3 pts)
2. **(Si autorisé)** `docker login` puis `docker push <votre_namespace>/my-flask-image:v1`. (3 pts)
3. Sauvegardez votre image en tar : `docker save -o images/my-flask-image-v1.tar <votre_namespace>/my-flask-image:v1` (ou `my-flask-image:v1`). (2 pts)
4. Chargez‑la sur une autre VM/ou simulez : `docker load -i images/my-flask-image-v1.tar` (preuve de commande). (2 pts)

<br/>

## Partie 7 — n8n (automatisation) (10 pts)

Dans `n8n-stack/` :

**`docker-compose.yml`**

```yaml
version: "3.8"
services:
  postgres:
    image: postgres:15-alpine
    container_name: n8n-postgres
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: n8npass
      POSTGRES_DB: n8n
    volumes:
      - pg_data:/var/lib/postgresql/data
    restart: unless-stopped

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    depends_on:
      - postgres
    ports:
      - "5678:5678"
    environment:
      N8N_BASIC_AUTH_ACTIVE: "true"
      N8N_BASIC_AUTH_USER: admin
      N8N_BASIC_AUTH_PASSWORD: admin123
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: n8n
      DB_POSTGRESDB_USER: n8n
      DB_POSTGRESDB_PASSWORD: n8npass
      GENERIC_TIMEZONE: America/Toronto
    volumes:
      - n8n_data:/home/node/.n8n
    restart: unless-stopped

volumes:
  pg_data:
  n8n_data:
```

Tâches :

1. Démarrez la stack et prouvez l’accès à `http://localhost:5678` (capture/curl). (6 pts)
2. Expliquez dans `LOG.md` pourquoi on **monte** `n8n_data` et `pg_data` (persistance). (4 pts)


<br/>

## Partie 8 — Question de cours (5 pts)

**« Pourquoi Kubernetes (k8s) est pertinent pour l’IA ? »**
Rédigez **un paragraphe** (8–12 lignes) évoquant : scaling de workers, planification GPU, tolérance aux pannes, rolling updates, gestion des secrets/config, opérateurs (MLFlow, Ray, etc.).

<br/>

## Bonus — Deepseek / modèle LLM en conteneur (+5 pts)

* Lancez une image LLM **CPU** (ex. `ollama/ollama` ou serveur REST léger) et prouvez un appel HTTP local.
* Décrivez les limites CPU/RAM sur VM et l’intérêt d’un GPU en production.

<br/>

## Rendus attendus

1. Dossier compressé **`eval-docker-<NomPrenom>.zip`** contenant :

   * `LOG.md` (journal clair, copies/sorties des commandes clés).
   * `flask-app/` (Dockerfile, app.py, requirements.txt).
   * `compose-wp/` (docker-compose.yml).
   * `n8n-stack/` (docker-compose.yml).
   * `images/` (`hello-world.tar`, `my-flask-image-v1.tar`, etc.).
   * Éventuels fichiers `index.html` pour volumes.

2. **Ne mettez pas** d’images système massives autres que les `.tar` demandés.

<br/>

## Barème détaillé (100 pts)

* Partie 1 – Commandes de base multi‑Nginx : **15**
* Partie 2 – Images & nettoyage : **10**
* Partie 3 – Dockerfile + Flask : **20**
* Partie 4 – Compose WordPress : **20**
* Partie 5 – Réseaux & Volumes : **10**
* Partie 6 – Registry & cycle d’image : **10**
* Partie 7 – n8n stack : **10**
* Partie 8 – Question de cours : **5**
* **Bonus** Deepseek/LLM : **+5** (max 105, arrondi à 100 si nécessaire)

> **Critères transverses (intégrés au barème) :** structure des dossiers, clarté de `LOG.md`, rigueur des commandes, hygiène (nettoyage), cohérence des ports selon la version, captures/sorties probantes.

<br/>

## Grille de correction (référence enseignant)

* **Preuve de service HTTP** = réponse 200 + body lisible (Nginx default page ou Flask string/JSON).
* **Nettoyage** = conteneurs arrêtés supprimés ; images **non** réclamées intactes ; dangling supprimées.
* **Compose** = `up -d` fonctionne ; `down` supprime les conteneurs ; relance ok.
* **Volumes** = persistance vérifiée (ex. redémarrage sans perdre `index.html` ou DB).
* **Réseau** = conteneur attaché au réseau custom listé par `docker network ls`.
* **Registry** = tag correct `<namespace>/<repo>:<tag>` ; commande push correcte (même si simulée).
* **n8n** = UI joignable ; volumes montés ; logs sans erreurs critiques.
* **Question k8s** = mentionne au moins 4 axes parmi scaling, GPU scheduling, HA, rolling, secrets, opérateurs ML, autoscaling.

<br/>

## Annexes — Aide mémoire (extraits autorisés)

* Lister : `docker ps`, `docker ps -a`, `docker images`, `docker network ls`, `docker volume ls`
* Lancer : `docker run -d --name c1 -p 8080:80 nginx`
* Exécuter : `docker exec -it c1 bash`
* Stop/Start : `docker stop c1 && docker start c1`
* Supprimer : `docker rm c1` (ou `docker rm -f c1`)
* Nettoyage images : `docker image prune -f` (dangling) ; `docker rmi <id>`
* Build : `docker build -t myimg:v1 .`
* Tag : `docker tag myimg:v1 ns/myimg:v1`
* Save/Load : `docker save ns/myimg:v1 -o myimg.tar` ; `docker load -i myimg.tar`
* Compose : `docker compose up -d` ; `docker compose down`


<br/>

### Versionnage des ports (récapitulatif)

* **Version A :** Nginx → 8080/8081/8082, Flask → 9000
* **Version B :** Nginx → 8083/8084/8085, Flask → 9001


> Le surveillant indiquera votre version au début de l’épreuve.


**Fin de l’évaluation.** Bonne réussite !

