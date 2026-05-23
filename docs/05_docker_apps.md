# Docker et Nextcloud

## Objectif

Installer Docker sur le Raspberry Pi 5 afin d'héberger Nextcloud dans des conteneurs tout en stockant les données persistantes dans la partition chiffrée LUKS.

Architecture retenue :

```text
Nextcloud
├── PostgreSQL
└── Redis
```

Accès :

```text
Tailscale only
```

Aucun port n'est ouvert sur Internet.

---

# Installation de Docker

Installation via le script officiel :

```bash
curl -fsSL https://get.docker.com | sh
```

---

# Ajouter l'utilisateur au groupe Docker

Vérifier l'utilisateur :

```bash
whoami
```

Dans ce projet :

```text
pi_admin
```

Ajouter l'utilisateur au groupe Docker :

```bash
sudo usermod -aG docker pi_admin
```

Déconnexion puis reconnexion SSH :

```bash
exit
```

Puis reconnecter :

```bash
ssh pi_wan
```

Vérifier :

```bash
groups
```

Le groupe :

```text
docker
```

doit apparaître.

---

# Vérification Docker

Afficher les conteneurs :

```bash
docker container ls
```

Afficher la version :

```bash
docker version
```

Afficher la version de Compose :

```bash
docker compose version
```

---

# Architecture retenue

Les fichiers de configuration Docker restent sur la partition système :

```text
/home/pi_admin/docker/
```

Les données persistantes sont stockées sur la partition LUKS :

```text
/srv/secure-data/
```

---

# Création des dossiers de données

Création de l'arborescence :

```bash
mkdir -p /srv/secure-data/docker-data/nextcloud
mkdir -p /srv/secure-data/docker-data/postgres
mkdir -p /srv/secure-data/docker-data/redis
```

Vérification :

```bash
tree /srv/secure-data/docker-data
```

Résultat :

```text
docker-data/
├── nextcloud
├── postgres
└── redis
```

---

# Création du dossier Docker

Créer le dossier contenant la stack :

```bash
mkdir -p ~/docker/nextcloud
```

Entrer dans le dossier :

```bash
cd ~/docker/nextcloud
```

Vérification :

```bash
pwd
```

Résultat :

```text
/home/pi_admin/docker/nextcloud
```

---

# Création du fichier compose.yaml

Créer le fichier :

```bash
nano compose.yaml
```

Contenu :

```yaml
services:
  postgres:
    image: postgres:17
    container_name: nextcloud-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - /srv/secure-data/docker-data/postgres:/var/lib/postgresql/data

  redis:
    image: redis:7
    container_name: nextcloud-redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - /srv/secure-data/docker-data/redis:/data

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped
    depends_on:
      - postgres
      - redis
    ports:
      - "8080:80"
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_HOST: postgres
      REDIS_HOST: redis
    volumes:
      - /srv/secure-data/docker-data/nextcloud:/var/www/html
```

---

# Création du fichier .env

Créer :

```bash
nano .env
```

Exemple :

```env
POSTGRES_DB=nextcloud
POSTGRES_USER=nextcloud
POSTGRES_PASSWORD=VOTRE_MOT_DE_PASSE
```

---

# Sécurisation du fichier .env

Restreindre les permissions :

```bash
chmod 600 .env
```

Vérification :

```bash
ls -l .env
```

Résultat attendu :

```text
-rw------- 1 pi_admin pi_admin
```

---

# Empêcher l'envoi du .env sur GitHub

Créer :

```bash
nano .gitignore
```

Contenu :

```text
.env
```

---

# Vérification de la configuration

Vérifier la syntaxe :

```bash
docker compose config
```

Docker charge :

- compose.yaml
- .env

et génère la configuration finale.

L'affichage du mot de passe dans cette commande est normal.

---

# Démarrage de la stack

Lancer :

```bash
docker compose up -d
```

Vérifier :

```bash
docker container ls
```

Résultat attendu :

```text
nextcloud
nextcloud-postgres
nextcloud-redis
```

État :

```text
Up
```

---

# Consultation des logs

Afficher les logs :

```bash
docker compose logs -f
```

Quitter :

```text
CTRL+C
```

Les logs ont confirmé :

- PostgreSQL opérationnel ;
- Redis opérationnel ;
- Nextcloud initialisé correctement.

---

# Accès à Nextcloud

Accès local :

```text
http://IP_LOCALE_DU_PI:8080
```

Exemple :

```text
http://192.168.1.50:8080
```

---

# Tailscale

L'accès via :

```text
http://IP_TAILSCALE:8080
```

---

# Architecture finale

```text
Raspberry Pi 5
├── Docker
├── Nextcloud
├── PostgreSQL
├── Redis
├── UFW
├── Tailscale
└── LUKS
```

Données :

```text
/srv/secure-data/docker-data/
├── nextcloud
├── postgres
└── redis
```

Configuration :

```text
/home/pi_admin/docker/nextcloud/
├── compose.yaml
├── .env
└── .gitignore
```
