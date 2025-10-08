# 1 - Installation de n8n

# **Compatibilité**

* n8n fonctionne très bien dans une **VM Ubuntu 22.04**, que ce soit sous **VirtualBox**, **VMware**, **Hyper-V**, **Proxmox**, etc.
* Il n’a besoin que de :

  * **2 Go de RAM minimum** (4 Go recommandés si tu fais tourner plusieurs workflows)
  * **1 vCPU**
  * **1 Go d’espace disque libre** (plus si tu stockes des workflows volumineux)


<br/>
<br/>

# 2 - **Procédure minimale**

Sur ta VM Ubuntu fraîchement installée :

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nodejs npm
sudo npm install -g n8n
```

Puis lance n8n :

```bash
n8n start
```

Par défaut, l’interface est disponible sur
👉 **[http://localhost:5678](http://localhost:5678)**
(si tu veux y accéder depuis ta machine hôte, configure le port 5678 en mode NAT ou Bridge selon ton hyperviseur).


<br/>
<br/>

# 3 - **Conseils**

* Si c’est juste pour tester, tu peux te connecter directement avec le navigateur intégré (dans la VM).
* Si tu veux garder n8n actif même après fermeture du terminal :

  ```bash
  nohup n8n start &
  ```
* Pour un usage plus propre : tu peux ensuite créer un service `systemd` pour qu’il démarre automatiquement au boot.








# Annexe 1

## **Option 1 — Docker (recommended)**

This is the cleanest and most maintainable setup.

```bash
sudo apt update && sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
```

Then create a simple `docker-compose.yml`:

```yaml
version: "3.8"
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=yourpassword
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - GENERIC_TIMEZONE=America/Toronto
    volumes:
      - ./n8n_data:/home/node/.n8n
```

Then run:

```bash
docker-compose up -d
```

Access n8n at [http://localhost:5678](http://localhost:5678)



## **Option 2 — Native installation (Node.js)**

If you prefer running it directly on your system:

```bash
sudo apt update
sudo apt install -y curl git build-essential
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g n8n
```

Run it:

```bash
n8n start
```

Or for background execution:

```bash
n8n start --tunnel &
```

You can also configure a **systemd service** to run automatically on boot.



### **Works perfectly on Ubuntu 22.04**

* Node.js ≥ 18 is supported
* Both SQLite (default) and Postgres are fine
* It’s stable for production or local testing
* You can integrate it easily with PM2, Nginx reverse proxy, SSL (Let’s Encrypt), etc.




