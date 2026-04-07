# Docker Registry — Self-Hosted

Registry Docker privé, sécurisé avec TLS (Let's Encrypt via Caddy) et authentification htpasswd.

## Stack

- **[registry:3.1.0](https://hub.docker.com/_/registry)** — Registry Docker officiel
- **[Caddy](https://caddyserver.com/)** — Reverse proxy avec TLS automatique (Let's Encrypt)
- **htpasswd** — Authentification basique

## Prérequis

- Docker & Docker Compose installés sur le serveur
- Un hostname résolvable publiquement (ex: `vpsXXXXXX.vps.ovh.net`)
- Ports **80** et **443** ouverts sur le serveur

## Structure

```
registry/
├── docker-compose.yml
├── Caddyfile
├── auth/
│   └── htpasswd
└── data/
```

## Installation

### 1. Cloner / créer le projet

```bash
mkdir registry && cd registry
```

### 2. Créer le fichier htpasswd

```bash
sudo apt install apache2-utils   # si pas déjà installé

mkdir -p auth
htpasswd -Bc auth/htpasswd <TON_USER>
# Le mot de passe sera demandé interactivement
```

### 3. Configurer le Caddyfile

Édite le `Caddyfile` et remplace `vpsXXXXXX.vps.ovh.net` par ton hostname réel.

### 4. Ouvrir les ports (si ufw actif)

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### 5. Démarrer

```bash
docker compose up -d
```

Caddy obtient automatiquement un certificat Let's Encrypt au premier démarrage.

## Utilisation côté client

### Login

```bash
docker login vpsXXXXXX.vps.ovh.net
```

### Push

```bash
docker tag mon-image vpsXXXXXX.vps.ovh.net/mon-image:latest
docker push vpsXXXXXX.vps.ovh.net/mon-image:latest
```

### Pull

```bash
docker pull vpsXXXXXX.vps.ovh.net/mon-image:latest
```

## Gestion des utilisateurs

### Ajouter un utilisateur

```bash
htpasswd auth/htpasswd <NOUVEL_USER>
```

### Supprimer un utilisateur

```bash
htpasswd -D auth/htpasswd <USER>
```

> Aucun redémarrage nécessaire, le fichier est lu dynamiquement.

## Maintenance

### Voir les logs

```bash
docker compose logs -f
```

### Garbage collection (libérer l'espace disque)

À exécuter après suppression d'images pour récupérer l'espace disque :

```bash
docker compose exec registry registry garbage-collect /etc/distribution/config.yml
```

### Mettre à jour les images

```bash
docker compose pull
docker compose up -d
```

## Sécurité

- Le trafic est chiffré en TLS (certificat Let's Encrypt renouvelé automatiquement par Caddy)
- Toutes les requêtes sont authentifiées via htpasswd
- Le registry n'est pas exposé directement, uniquement via Caddy

## Dépannage

| Problème | Solution |
|---|---|
| `unauthorized: authentication required` | Vérifier les credentials avec `docker login` |
| `x509: certificate signed by unknown authority` | TLS non encore prêt, attendre quelques secondes |
| `error parsing HTTP 413` | Augmenter `max_size` dans le `Caddyfile` |
| Caddy ne démarre pas | Vérifier que les ports 80/443 sont libres et ouverts |