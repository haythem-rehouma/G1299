
# Installez n8n

# 1. Installer Node.js (version LTS recommandée)

Exemple avec la version 20 LTS :

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs build-essential
```

Vérifie :

```bash
node -v
npm -v
```



# 2. Installer n8n globalement

```bash
sudo npm install -g n8n
```



# 3. Lancer n8n en local

```bash
n8n start
```

Tu auras accès à l’interface ici :
👉 [http://localhost:5678](http://localhost:5678)



# 4. (Optionnel) Lancer en arrière-plan avec `pm2`

Pour éviter de garder ton terminal ouvert, tu peux installer **pm2** :

```bash
sudo npm install -g pm2
pm2 start n8n
pm2 save
pm2 startup
```



# 5. (Optionnel) Lancer en tant que service systemd

Crée le fichier `/etc/systemd/system/n8n.service` :

```ini
[Unit]
Description=n8n workflow automation
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/n8n
Restart=on-failure
Environment=GENERIC_TIMEZONE=Europe/Paris
Environment=N8N_PORT=5678
Environment=N8N_BASIC_AUTH_ACTIVE=true
Environment=N8N_BASIC_AUTH_USER=admin
Environment=N8N_BASIC_AUTH_PASSWORD=admin123

[Install]
WantedBy=multi-user.target
```

Active le service :

```bash
sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
```

<br/>

# Arréter le service 

- Pour **désactiver/arrêter** n8n avec les commandes systemd :
- Si tu as installé **n8n comme service systemd** (fichier `/etc/systemd/system/n8n.service`), tu peux le **désactiver/arrêter** facilement avec les commandes systemd :



# 1. Arrêter immédiatement le service (mais il reste activé au redémarrage)

```bash
sudo systemctl stop n8n
```



# 2. Désactiver le service (il ne redémarrera pas automatiquement au prochain boot)

```bash
sudo systemctl disable n8n
```



# 3. Supprimer complètement le service (optionnel, si tu veux désinstaller)

```bash
sudo systemctl stop n8n
sudo systemctl disable n8n
sudo rm /etc/systemd/system/n8n.service
sudo systemctl daemon-reload
```



# 4. Vérifier l’état du service

```bash
systemctl status n8n
```



👉 Résumé :

* `stop` = arrêt temporaire.
* `disable` = désactive au redémarrage.
* `rm + daemon-reload` = suppression définitive du service.

